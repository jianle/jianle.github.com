---
title: hadoop 机房迁移相关问题
date: 2016-04-19 19:08:34
tags:
 - hadoop
categories: BigData
thumbnail: /img/hadoop.png
lede: "记录hadoop搬迁遇到的一些问题"
featured: false
---

更改机器的 ip 参考
------


* CDH 4 的时候需要修改数据库

`/etc/cloudera-scm-server/db.properties`这个文件记录 cm 用的 DB 的信息  

```bash
# psql  -h localhost -U scm -p 7432
scm=> \dt # show tables in pg
scm=> \list
scm=> select * from hosts; # show hostinfo in hosts
```

<!-- more -->

`/etc/cloudera-scm-agent/config.ini` 每个 agent 里面有记录 cm 的地址, 修改:   

```bash  
server_host=10.xx.xx.xx
```

CDH 5 的时候使用 uuid 来标识每台机器，看起来只需要修改一下 `/etc/cloudera-scm-agent/config.ini` 里面对应的 server_host 即可


* 相关参考：
  * [https://groups.google.com/a/cloudera.org/forum/#!mydiscussions/scm-users/gM-DtmxvkJ8](https://groups.google.com/a/cloudera.org/forum/#!mydiscussions/scm-users/gM-DtmxvkJ8)
  * [http://shulhi.com/change-ip-address-for-existing-nodes-in-cdh-5-3/](http://shulhi.com/change-ip-address-for-existing-nodes-in-cdh-5-3/)


除了上面写的，还需要保证:   
a. dns 解析正确，或者 `/etc/hosts` 里面配置正确，否则 `cloudera-scm-server` 会hang住，导致 7182 这个服务无法连接。`cloudera-scm-agent` 连不上   
b. 需要deploy client config  
c. namenode ha 启动会有问题，原因是之前 zk 里面保存的 hostname 发生改变，停掉 failover controller 去 initiale zookeeper 都不会失败  


需要手动去设置

```bash
HADOOP_USER_NAME=hdfs
hdfs --config /var/run/cloudera-scm-agent/process/150-failover-controller-initialize-znode zkfc -formatZK
```

提示我们 stop all hdfs service，输入 Y 确认格式掉 zookeeper 相关数据（注意，不是格式化 namenode）后面重启就好了。
