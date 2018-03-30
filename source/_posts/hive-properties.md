---
title: Hive常用配置参数
date: 2016-04-16 18:09:09
tags:
 - hive
 - 参数配置
categories: BigData
thumbnail: /img/hive-hadoop.png
---

[HIVE配置详情](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties)

熟悉HIVE配置，了解HIVE调优及一些配置所带来的效果

<!-- more -->
* Map相关参数

```sql
set mapred.max.split.size = 256000000 ; //最大分割
set mapred.min.split.size=1 ; //最小分割
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat; //Map端输入、合并文件之后按照block的大小分割（默认）
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat; //Map端输入，不合并 一个文件起一个Map
set hive.mapjoin.maxsize=1000000; //Map Join 所处理的最大的行数。超过此行数，Map Join进程会异常退出  
set hive.merge.mapfiles = true ;//是否开启合并 Map 端输出小文件
set hive.merge.mapredfiles = false ;//是否开启合并 Map/Reduce 端输出小文件
```

* Reduce参数配置

```sql
set mapred.reduce.tasks=-1 ; //设置Reduce的个数，-1为默认，表示由hive自动分配管理
```

* 并行计算参数

```sql
set hive.exec.parallel=true;  //无关的计算可以并行调起JOB  
set hive.exec.parallel.thread.number=8 ; //并发提交时的并发线程的个数
```

* group by

* 自动分区

```sql
set hive.exec.dynamic.partition = false ; // 是否打开动态分区。
set hive.exec.dynamic.partition.mode = strict ;//打开动态分区后，动态分区的模式，有 strict 和 nonstrict 两个值可选，strict 要求至少包含一个静态分区列，nonstrict 则无此要求。
set hive.exec.max.dynamic.partitions = 1000; // 所允许的最大的动态分区的个数。
set hive.exec.max.dynamic.partitions.pernode = 100; //单个 reduce 结点所允许的最大的动态分区的个数。
set hive.exec.default.partition.name = '__HIVE_DEFAULT_PARTITION__';//默认的动态分区的名称，当动态分区列为''或者null时，使用此名称。''
```

* 笛卡尔积join

```sql
set hive.mapred.mode = strict ;//阻止笛卡尔积    
set hive.mapred.mode = nostrict ;//允许笛卡尔积
```

* Map-Side Join

```sql
set hive.auto.convert.join = true ; //将小表刷入内存中  
set hive.mapjoin.smalltable.filesize = 2500000 ;//刷入内存表的大小(字节)  
Map Join 的计算步骤分两步，将小表的数据变成hashtable广播到所有的map 端，将大表的数据进行合理的切分，然后在map 阶段的时候用大表的数据一行一行的去探测(probe) 小表的hashtable. 如果join key 相等，就写入HDFS.
map join 之所以叫做map join 是因为它所有的工作都在map 端进行计算.

hive 在map join 上做了几个优化：
hive 0.6 的时候默认认为写在select 后面的是大表，前面的是小表， 或者使用 /*+mapjoin(map_table) */ 提示进行设定.
hive 0.7 的时候这个计算是自动化的，它首先会自动判断哪个是小表，哪个是大表，这个参数由（hive.auto.convert.join=true）来控制. 然后控制小表的大小由（hive.smalltable.filesize=25000000L）参数控制（默认是25M），当小表超过这个大小，hive 会默认转化成common join.
你可以查看HIVE-1642.首先小表的Map 阶段它会将自己转化成MapReduce Local Task ，然后从HDFS 取小表的所有数据，将自己转化成Hashtable file 并压缩打包放入DistributedCache 里面.
目前hive 的map join 有几个限制，一个是它打算用BloomFilter 来实现hashtable , BloomFilter 大概比hashtable 省8-10倍的内存, 但是BloomFilter 的大小比较难控制.
现在DistributedCache 里面hashtable默认的复制是3份，对于一个有1000个map 的大表来说，这个数字太小，大多数map 操作都等着DistributedCache 复制.

```

* Skew Join

```sql
真实数据中数据倾斜是一定的, hadoop 中默认是使用
hive.exec.reducers.bytes.per.reducer = 1000000000

也就是每个节点的reduce 默认是处理1G大小的数据，如果你的join 操作也产生了数据倾斜，那么你可以在hive 中设定
set hive.optimize.skewjoin = true;
set hive.skewjoin.key = skew_key_threshold （default = 100000）

hive 在运行的时候没有办法判断哪个key 会产生多大的倾斜，所以使用这个参数控制倾斜的阈值，如果超过这个值，新的值会发送给那些还没有达到的reduce, 一般可以设置成你（处理的总记录数/reduce个数）的2-4倍都可以接受.  
倾斜是经常会存在的，一般select 的层数超过2层，翻译成执行计划多于3个以上的mapreduce job 都很容易产生倾斜，建议每次运行比较复杂的sql 之前都可以设一下这个参数. 如果你不知道设置多少，可以就按官方默认的1个reduce 只处理1G 的算法，那么  skew_key_threshold  = 1G/平均行长. 或者默认直接设成250000000 (差不多算平均行长4个字节)

```

* Map/Reduce Memory

```sql
set mapreduce.reduce.memory.mb=8192;
set mapreduce.map.memory.mb=8192;
```

* Hive cli调试日志

```bash
$ hive --hiveconf hive.root.logger=DEBUG,console
hive> set hiveconf:hive.root.logger=DEBUG,console;
```

* Hive cli打印字段名

```
hive> set hive.cli.print.header=true;
hive> set hive.resultset.use.unique.column.names=false; //该参数表示结果列是否带表名
```

* Support subdirecotries

```
hive.mapred.supports.subdirectories true
mapreduce.input.fileinputformat.input.dir.recursive true
```
