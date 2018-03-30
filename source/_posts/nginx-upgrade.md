---
title: Nginx 升级
date: 2016-04-16 14:53:49
categories: Linux
tags: nginx
thumbnail: /img/nginx.png
lede: "nginx1.3版本之后支持了websocket ，简单的配个代理就能让你的websocket工作正常"
---

* [覆盖式升级](http://leftshift.io/upgrading-nginx-to-the-latest-version-on-ubuntu-servers)

* 更换目录重新安装  

升级目的：
nginx1.3版本之后支持了websocket ，简单的配个代理就能让你的websocket工作正常
本次针对已有nginx非覆盖式升级

<!-- more -->
* 下载[nginx](http://nginx.org/en/download.html) 
```bash 
$ weget http://nginx.org/download/nginx-1.8.0.tar.gz  
```

* 解压编译 
```bash
$ tar -zxvf nginx-1.8.0.tar.gz
$ cd nginx-1.8.0
$ ./configure --prefix=/usr/local/nginx-1.8.0

Mac OX 下也许你能顺利通过，但是如果Ubuntu下可能会报PCRE依赖包没有 so 你可以如下操作：
 
$ apt-get install libpcre3 libpcre3-dev
 
成功后可执行安装了
$ make && make install

这样在你的/usr/local/nginx-1.8.0下就成功安装了一个nginx 和原有的不冲突
 
$ cd /usr/local/nginx-1.8.0
$ ls -lrt
drwxr-xr-x  2 root     root 4096 Aug 17 10:32 sbin
drwxr-xr-x  2 root     root 4096 Aug 17 10:32 html
drwxr-xr-x  2 root     root 4096 Aug 17 10:35 logs
drwx------  2 www-data root 4096 Aug 17 10:35 uwsgi_temp
drwx------  2 www-data root 4096 Aug 17 10:35 scgi_temp
drwx------  2 www-data root 4096 Aug 17 10:35 fastcgi_temp
drwxr-xr-x  5 root     root 4096 Aug 17 19:49 conf
drwx------ 12 www-data root 4096 Aug 17 22:37 proxy_temp
drwx------  2 www-data root 4096 Aug 17 22:46 client_body_temp
 
ldd nginx 可以看到并没有自己的.so 依赖文件  

$ ldd /usr/local/nginx-1.8.0/sbin/nginx  (ubuntu)
$ otool -L ~/dwetl/nginx1.8.0/sbin/nginx (Mac OX)
```

* 备份拷贝原有配置文件  

* 关闭&重启服务
```bash
$ vim opt.sh 
echo 'stop nginx ....'
service nginx stop
 
echo 'cp config file.'
cp /etc/nginx/nginx.conf /usr/local/nginx-1.8.0/conf/
cp -r /etc/nginx/conf.d/ /usr/local/nginx-1.8.0/conf/
 
echo 'upgrade  nginx.'
mv /usr/sbin/nginx /usr/sbin/nginx1.1.19    # 保留之前的nginx 万一出问题了咋办呢，留条后路
ln -s /usr/local/nginx-1.8.0/sbin/nginx /usr/sbin/nginx
 
echo 'restart  nginx.'
service nginx restart
```


