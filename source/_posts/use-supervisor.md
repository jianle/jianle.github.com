---
title: 使用supervisor守护进程
date: 2016-04-19 10:04:27
tags: supervisor
categories: Linux
thumbnail: /img/supervisord.png
lede: "Supervisor is a client/server system that allows its users to control a number of processes on UNIX-like operating systems"
featured: false
---

* Install 

```bash
$ sudo apt-get install supervisor
```

* 配置 

1. 使用默认路径下配置(/etc/supervisord.conf)
2. 指定目录`supervisord.conf`配置 

<!-- more -->
```bash
; supervisor config file

[unix_http_server]
file=/home/hadoop/supervisor/supervisor.sock   ; (the path to the socket file)
chmod=0700                       ; sockef file mode (default 0700)

[supervisord]
logfile=/home/hadoop/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/home/hadoop/supervisor/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
childlogdir=/home/hadoop/supervisor             ; ('AUTO' child log dir, default $TEMP)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///home/hadoop/supervisor/supervisor.sock ; use a unix:// URL  for a unix socket

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[include]
files = /home/hadoop/supervisor/conf.d/*.conf
```

启动：
```bash
$ cd /home/hadoop/supervisor/
$ supervisord -c supervisord.conf
```

* 守护进程`/home/hadoop/supervisor/conf.d/*.conf` 下创建`app.conf`

```
[program:app]
command=/home/hadoop/spark-1.5.2-bin-hadoop2.6/bin/spark-submit
  --executor-cores 1
  --num-executors 2
  --queue root.spark
  --master yarn-cluster
  --files /home/hadoop/spark-mining/log4j-spark.properties
  --class com.app.dw.MainClassCount
  --conf "spark.driver.extraJavaOptions=-Xms512M -Xmn256M -XX:+UseConcMarkSweepGC -Dlog4j.configuration=log4j-spark.properties -Ddm.logging.level=DEBUG"
  --conf "spark.executor.extraJavaOptions=-XX:+UseConcMarkSweepGC -Dlog4j.configuration=log4j-spark.properties -Ddm.logging.level=DEBUG"
  /home/hadoop/realtime/app.jar reset
autostart=true
autorestart=true
environment=JAVA_HOME="/usr/local/java/jdk1.8.0_60"
stdout_logfile=/home/hadoop/realtime/logs/app.log
redirect_stderr=true
user=hadoop
```

* 命令行操作

```bash
$ supervisorctl update|status|stop|start|restart   // 更新配置|查看状态|停止指定进程|启动进程|重启进程
```


