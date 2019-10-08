---
title: Multi-version Testing with Cookies and Nginx
date: 2019-10-08 15:28:47
tags:
  - java
  - nginx
  - testing
categories: Java
lede: "使用 Nginx + Cookie 实现多版本测试发布"
thumbnail: /img/ab-testing.png
---

前言：

在一般而言，测试的时候，后台起一个服务，对应指定的某一个分支。但是在多分支并行开发不同模块的情况下，多个产品要同时测试和验收自己的模块开发。这样就要反复发布重启环境，并且无法做到多环境并行验收测试。本文描述如何使用 nginx 加上种 cookie 的方式来实现多版本测试。


## 准备工作

* 假设测试域名为： `saboloh.test`
* 克隆镜像仓库   

```bash
git clone --mirror git@github.com:jianle/spring-boot-example.git
```

## 编写发布脚本

定义脚本名： `build.sh`  

```bash  
#!/bin/bash

cd $(dirname $0)

REVISION="$1"
PORT="$2"

if [ -z "$REVISION" ]; then
  REVISION="master"
fi

REVISION_STR=$(echo ${REVISION} | sed "s/\//\-/g")

if [ -z "$PORT" ]; then
  PORT=$(grep -oP "(?<=${REVISION_STR}=)(.*)" version_port.txt)
fi

if [ -z "$PORT" ]; then
  LAST_PORT=$(tail -n1 version_port.txt | cut -d'=' -f2)
  PORT=$(($LAST_PORT + 4))
  echo "${REVISION_STR}=${PORT}" >> version_port.txt
  update_nginx.sh ${REVISION_STR} ${PORT}
fi

set -e

BASE_DIR=/root/testing
export JAVA_HOME=/opt/soft/jdk1.8
export PATH=${JAVA_HOME}/bin:$PATH

echo $PATH

cd $BASE_DIR/spring-boot-example.git
git remote update
git log -3

VERSION=`git rev-parse $REVISION`
VERSION=${VERSION:0:7}
DATE=$(date +%Y%m%d)
PREFIX="${DATE}.${VERSION}"

echo "Building $PREFIX.jar ${PORT}"

SOURCEDIR=$BASE_DIR/build/source/$PREFIX
RESOURCEDIR=$SOURCEDIR/src/main/resources/

echo "Extracting source to $SOURCEDIR"
rm -rf $SOURCEDIR
git archive --prefix=$PREFIX/ --format=tar $VERSION | tar xf - -C $BASE_DIR/build/source

echo "Building npm in $RESOURCEDIR"
cd $RESOURCEDIR

ln -s ${BASE_DIR}/node_modules $RESOURCEDIR
npm install --save-dev
PUBLIC_PATH=http://saboloh.test/ npm run build

echo "maven building jar in $SOURCEDIR"
cd $SOURCEDIR
mvn -Dmaven.test.skip=true clean package

echo "
----------- build result ----------
--- Version: $PREFIX $PORT
--- URL: http://saboloh.test/
-----------------------------------

"

$BASE_DIR/start-web.sh $PREFIX $PORT

exit
```

其中使用到的文件有：
* `version_port.txt`（版本对应端口号）

```bash
master=8188
testing=8192
```

* `update_nginx.sh`（更新 nginx 配置）

```bash
#!/bin/bash

VERSION_STR="$1"
PORT="$2"

MAP_FILE=/usr/local/nginx/conf.d/testing_version.map
STREAM_FILE=/usr/local/nginx/conf.d/testing_version.stream

CHECK_BH=$(grep ${VERSION_STR} ${MAP_FILE})
if [ -z "${CHECK_BH}" ]; then
  MNR=$(wc -l ${MAP_FILE} | cut -d' ' -f1)
  let MNR--
  sed -i '/}/i \ \ \ \ "'''${VERSION_STR}'''"\ \ \ \ "'''${VERSION_STR}'''";' ${MAP_FILE}
fi

CHECK_STM=$(grep ${VERSION_STR} ${STREAM_FILE})
if [ -z "${CHECK_STM}" ]; then
  echo "
upstream ${VERSION_STR} {
    server 127.0.0.1:${PORT};
}
" >> ${STREAM_FILE}
fi

nginx -t

if [ $? == 0 ]; then
  nginx -s reload
fi
```

* `start-web.sh`（启动服务）

```bash  
#!/bin/bash

version="$1"
port=$2
if [ -z $port ]
then
  port=8188
fi

JMX="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false  -Dcom.sun.management.jmxremote.port=3000"
GC="-XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:-CMSConcurrentMTEnabled -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled"
MEM="-Xms2G -Xmx2G"

cd $(dirname $0)

BASE_PATH=/root/testing/build/source/$version

jar=${BASE_PATH}/target/testing-0.0.1-SNAPSHOT.jar
CONF=/root/testing/conf/application.properties

export JAVA_HOME=/opt/soft/jdk1.8
export PATH=${JAVA_HOME}/bin:$PATH

# kill old process
OLD_PID=$(netstat -anltp|grep ${port}|awk '{print $7}'|cut -d'/' -f1|head -n1)
if [ ! -z "$OLD_PID" ]
then
  echo "------- Will kill old process ${OLD_PID} -------"
  echo ''
  kill -9 $OLD_PID
fi

nohup /opt/soft/jdk1.8/bin/java $MEM $GC -jar $jar --server.port=$port --spring.config.location=$CONF >> logs/${version}.${port}.log 2>&1 &
```

以上都是用于发布&启动的自动化脚本，其实质就是打包生成相关的 nginx 配置文件，然后重启 web 服务并重新加载 nginx 相关修改配置。


### 配置 Nginx 相关跳转

在 nginx 中，主要根据分支+版本情况，做不同跳转

```bash  
include  /usr/local/nginx/conf.d/testing_version.map;
include  /usr/local/nginx/conf.d/testing_version.stream;

server {
    listen 80;
    server_name saboloh.test; #绑定域名
    proxy_read_timeout 1000s;

    gzip on;
    gzip_vary on;
    gzip_min_length 10240;
    gzip_proxied expired no-cache no-store private auth;
    gzip_types text/plain text/css text/xml text/javascript application/x-javascript application/xml application/json;
    gzip_disable "MSIE [1-6]\.";

    location / {
        proxy_set_header Host $host;
        #if ($version_port != '8080') {
        #    break;
        #}
        proxy_pass http://$vstream; #使用 upstream
        proxy_set_header X-real-ip $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

nginx 配置过程中，一开始想直接在 `map` 文件中，定义版本对应的端口，然后根据版本，跳转端口，但是一直跳转失败（上述代码注释片段），具体我也不知道说明原因，后面改成使用 `upstream` 的方式都是正常的。如果有大神知道，还请留言指点一下。


### 总结

在上述方法中，为了方便，很多地方都使用了脚本来自动配置，如果想简单一些，直接启动号环境之后，修改 nginx 相关配置，即可。还有在使用 `upstream` 的过程中，为了安全生产，起始还可以在每一个 `upstream` 中，加一个默认的 server。设置为 `backup`，相关使用可以查看 nginx 相关文档。
