---
title: Fluentd数据采集写入Kafka
date: 2016-04-16 18:48:56
tags: 
 - kafka
 - 日志
categories: BigData
thumbnail: /img/fluentd-kafka.png
lede: "A fluentd plugin to both consume and produce data for Apache Kafka"
---

### 日志实时采集

* 安装[fluent](http://www.fluentd.org/)

```bash 
$ sudo apt-get install td-agent
```
<!-- more -->
* 修改ruby 的gem源 

```bash
安装好Ruby
$ gem source  # 查看源

删除默认源：
$ gem source -r <url>
添加新指向：
$ gem source -a https://ruby.taobao.org/
```
* 安装[fluent-plugin-kafka](https://github.com/htgc/fluent-plugin-kafka/) (github上有详细介绍) 

* fluentd --setup /path/xxx  

* 配置/path/xxx下的fluent.conf

```bash
<source>
  @type forward
  @id forward_input
</source>
 
<source>
  @type http
  @id http_input
  port 8888
</source>
 
<source>
  @type    tail  #### tail方式采集日志
  #format   none
  format   /^(?<all>.*)$/
  path     /tmp/app_log.txt
  pos_file /data/logs/td-agent/httpd-access.log.pos
  tag      app_log.tag
</source>
 
<match app_log.**>
  @type               kafka
  #brokers            localhost:9002,localhost:9003
  zookeeper           localhost:2181
  default_topic       app_log_raw  
  #刷新间隔
  flush_interval      30
  buffer_type         file
  ack_timeout_ms      2000
  output_data_type    attr:all
  compression_codec   none
</match>
 
<source>
  @type monitor_agent
  @id monitor_agent_input
  port 24220
</source>
 
<source>
  @type debug_agent
  @id debug_agent_input
  bind 127.0.0.1
  port 24230
</source>
 
<match debug.**>
  @type stdout
  @id stdout_output
</match>
 
<match system.**>
  @type forward
  @id forward_output
 
  <server>
    host 192.168.0.11
  </server>
  <secondary>
    <server>
      host 192.168.0.12
    </server>
  </secondary>
</match>
```

* 确保[kafka](http://kafka.apache.org/documentation.html#quickstart)已经启动完成

* 启动fluentd

```
$ fluentd -c fluent.conf
```

* 查看kafka中数据是否进入 

```
$ bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic app_log_raw
```


