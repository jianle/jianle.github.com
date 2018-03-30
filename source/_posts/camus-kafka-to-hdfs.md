---
title: Camus 落地 kafka Topic
date: 2016-04-17 11:21:58
tags:
 - kafka
 - 日志
categories: BigData
thumbnail: /img/kafka.png
lede: "Apache Kafka is publish-subscribe messaging rethought as a distributed commit log"
---
应用场景，fluentd 实时采集日志写入 kafka 解析后数据写入另一 topic. 为了实时获取分析数据，将 topic 落地 hdfs 然后 load 到 hive 表供分析使用  

* clone [代码](https://github.com/linkedin/camus)  

```shell
$ git clone https://github.com/linkedin/camus.git
$ cd camus
```
<!-- more -->
* 新增 StringMessageDecoder 类(接收数据不规范，自定义接收处理逻辑)  

```java
package com.linkedin.camus.etl.kafka.coders;

import com.linkedin.camus.coders.CamusWrapper;
import com.linkedin.camus.coders.Message;
import com.linkedin.camus.coders.MessageDecoder;

import org.apache.log4j.Logger;
import java.io.UnsupportedEncodingException;
import java.util.Properties;

/**
 * MessageDecoder class that will convert the payload into a String object,
 * System.currentTimeMillis() will be used to set CamusWrapper's timestamp
 * property
 *
 * This MessageDecoder returns a CamusWrapper that works with Strings payloads,
 */
public class StringMessageDecoder extends MessageDecoder<Message, String> {
    private static final Logger log = Logger.getLogger(StringMessageDecoder.class);

    @Override
    public void init(Properties props, String topicName) {
        this.props = props;
        this.topicName = topicName;
    }

    @Override
    public CamusWrapper<String> decode(Message message) {
        // TODO Auto-generated method stub
        long timestamp = 0;
        String payloadString;

        try {
            payloadString = new String(message.getPayload(), "UTF-8");
          } catch (UnsupportedEncodingException e) {
            log.error("Unable to load UTF-8 encoding, falling back to system default", e);
            payloadString = new String(message.getPayload());
          }

        timestamp = System.currentTimeMillis();

        return new CamusWrapper<String>(payloadString, timestamp);
    }
}
```

* 修改 camus下pom.xml hadoop版本  

* 配置camus.properties

```bash
camus.job.name=soj_dtl_transfer

fs.default.name=hdfs://nameservice1
etl.destination.path=/tmp/saboloh/topics
etl.execution.base.path=/tmp/saboloh/exec
etl.execution.history.path=/tmp/saboloh/camus/exec/history

camus.message.decoder.class=com.linkedin.camus.etl.kafka.coders.JsonStringMessageDecoder
etl.record.writer.provider.class=com.linkedin.camus.etl.kafka.common.StringRecordWriterProvider

camus.message.timestamp.field=log_time
camus.message.timestamp.format=yyyy-MM-dd HH:mm:ss.SSS
etl.partitioner.class=com.linkedin.camus.etl.kafka.partitioner.HourlyPartitioner

kafka.max.historical.days=1
kafka.blacklist.topics=
kafka.whitelist.topics=soj_topic_rt
kafka.brokers=localhost:9092,localhost:9093
kafka.client.name=soj_dtl_transfer_client


etl.hourly=hourly

log4j.configuration=true
mapred.output.compress=true
etl.output.codec=deflate
etl.deflate.level=6

etl.default.timezone=Asia/Shanghai
etl.output.file.time.partition.mins=60
etl.keep.count.files=false
```


* 编译打包

```bash
$ mvn clean [compile] package [-DskipTests]
```

* 运行

```bash
$ cd camus-example
$ cp target/camus-example-0.1.0-SNAPSHOT-shaded.jar .
$ hadoop jar camus-example-0.1.0-SNAPSHOT-shaded.jar com.linkedin.camus.etl.kafka.CamusJob -P ../camus.properties
```

* 错误异常处理

```shell
Exception in thread "main" java.io.IOException: Mkdirs failed to create /var/folders/hb/s74xd1353n92ht1h8pr8mngh0000gn/T/hadoop-unjar7038503362115517488/META-INF/license
    at org.apache.hadoop.util.RunJar.ensureDirectory(RunJar.java:128)
    at org.apache.hadoop.util.RunJar.unJar(RunJar.java:104)
    at org.apache.hadoop.util.RunJar.unJar(RunJar.java:81)
    at org.apache.hadoop.util.RunJar.run(RunJar.java:209)
    at org.apache.hadoop.util.RunJar.main(RunJar.java:136)

上述错误信息由于Mac 上的特殊原因导致的貌似。
http://stackoverflow.com/questions/10522835/hadoop-java-io-ioexception-mkdirs-failed-to-create-some-path
修改jar
$ zip -d camus-example-0.1.0-SNAPSHOT-shaded.jar META-INF/LICENSE  
$ zip -d camus-example-0.1.0-SNAPSHOT-shaded.jar LICENSE
上述命令删除jar包种指定信息
$ jar -tvf camus-example-0.1.0-SNAPSHOT-shaded.jar |grep META-INF/LICENSE
```
