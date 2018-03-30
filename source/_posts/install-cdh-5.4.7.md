---
title: 安装CDH指定版本5.4.7
date: 2016-04-19 16:34:42
tags: 
 - hadoop
 - chd
 - cloudera
categories: BigData
thumbnail: /img/cloudera-logo.png
lede: "离线安装CDH5.4.7"
featured: false
---

离线安装CDH5.4.7

* [官方文档](http://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_install_path_a.html#cmig_topic_6_5_unique_2)

* 删除已有[cloudera manager](http://www.cloudera.com/documentation/enterprise/5-5-x/topics/cm_ig_uninstall_cm.html)

```bash
$ /usr/share/cmf/uninstall-cloudera-manager.sh

$ apt-get purge 'cloudera-manager-*'

$ umount cm_processes
$ rm -Rf /usr/share/cmf /var/lib/cloudera* /var/cache/yum/cloudera* /var/log/cloudera* /var/run/cloudera*

$ rm -Rf /var/lib/flume-ng /var/lib/hadoop* /var/lib/hue /var/lib/navigator /var/lib/oozie /var/lib/solr /var/lib/sqoop* /var/lib/zookeeper
```

<!-- more -->

* 查看系统版本 

```bash
$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 12.04.4 LTS
Release:    12.04
Codename:   precise
```


* 下载对应包并且配置本地源(避免使用cloudera-manager-installer.bin直接安装总是获取最新版CDH) 

```bash
下载好tarball

$ wget http://archive.cloudera.com/cm5/repo-as-tarball/5.4.7/cm5.4.7-ubuntu12-04.tar.gz
$ tar -zxvf cm5.4.7-ubuntu12-04.tar.gz
$ cp -r cm /var/spool/apt-mirror/mirror/archive.cloudera.com/

# 使用nginx配置
$ apt-get install nginx 
$ vim /etc/nginx/nginx.conf
# 访问本地目录列表
server {
    listen       80;
    server_name  dev-001;

    location / {
        root   /var/spool/apt-mirror/mirror/archive.cloudera.com/;
        index  index.html index.htm;
        autoindex on;
        autoindex_exact_size off;
        autoindex_localtime on;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}  
```

* 浏览确认是否能访问 

![](/img/filelist.png)

* 配置apt source

```
$ vim /etc/apt/source.list.d/my-private-cloudera-repo.list
# Packages for Clouderas Distribution for Hadoop, Version 4, on Ubuntu 12.04 x86_64
deb [arch=amd64] http://dev-001/cm precise-cm5 contrib
deb-src http://dev-001/cm precise-cm5.4.7 contrib
```

* 下载对应[parcel](http://archive.cloudera.com/cdh5/parcels/)

```
找到对应自己系统的parcels
$ wget http://archive.cloudera.com/cdh5/parcels/5.4.7/CDH-5.4.7-1.cdh5.4.7.p0.3-precise.parcel
$ wget wget http://archive.cloudera.com/cdh5/parcels/5.4.7/CDH-5.4.7-1.cdh5.4.7.p0.3-precise.parcel.sha1
$ wget http://archive.cloudera.com/cdh5/parcels/5.4.7/manifest.json

sha1需修改成sha
$ mv CDH-5.4.7-1.cdh5.4.7.p0.3-precise.parcel.sha1 CDH-5.4.7-1.cdh5.4.7.p0.3-precise.parcel.sha
```

* 下载安装[Cloudera Manager](https://archive.cloudera.com/cm5/installer/)

``` 
$ wget https://archive.cloudera.com/cm5/installer/5.4.7/cloudera-manager-installer.bin
```

* 安装

```
$ chmod +x cloudera-manager-installer.bin
$ ./cloudera-manager-installer.bin --skip_repo_package=1
```

