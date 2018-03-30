---
title: Hive使用技巧
date: 2016-04-16 18:00:33
tags: 
- hive
- 分区
categories: BigData
thumbnail: /img/hive-hadoop.png
---

### Hive动态分区&Partition中使用函数

#### 自动分区

前段时间因为导表需求（从一张表中查找出数据，按日期分区overwrite 到指定分区表中），在Hive里面研究了一下自动分区

<!-- more -->
* 建好所需分区表 
* 设置分区参数 

```sql 
set hive.exec.dynamic.partition=true;(可通过这个语句查看：set hive.exec.dynamic.partition;) 
set hive.exec.dynamic.partition.mode=nonstrict; 
SET hive.exec.max.dynamic.partitions=100000;(如果自动分区数大于这个参数，将会报错)
SET hive.exec.max.dynamic.partitions.pernode=100000;
```
* 代码（特别注意：日期字段一定要放在末尾）

```sql  
INSERT OVERWRITE TABLE dw_stage.ds_info_msg partition(cal_dt)
select id        
       ,...
       ,pv
       ,0  uv
       ,cal_dt
from   dw_stage.tiny_info_msg
```

* 如果要在partition字段用函数处理也可通过通过这种方式实现

```sql  
set hive.exec.dynamic.partition=true;
set hive.exec.dynamic.partition.mode=nostrick;
INSERT OVERWRITE TABLE dw_stage.ds_info_msg partition(cal_dt)
SELECT id    
       ,...
       ,pv
       ,date_sub('2014-03-23',7) cal_dt
FROM   dw_stage.tiny_info_msg07;
```

### 常用命令

* Load data to table 

```sql
load data inpath '/user/hadoop/soj/output-final/hive-*' 
OVERWRITE INTO TABLE dw_load_table partition(log_dt='${outFileSuffix}');
```
* Alter table external

```sql
ALTER TABLE <table> SET TBLPROPERTIES('EXTERNAL'='TRUE')
```

* [Desc table/database](https://cwiki.apache.org/confluence/display/Hive/AdminManual+Configuration#AdminManualConfiguration-ConfigurationVariables)

```sql
desc formatted <table>
desc extended  <table>
desc database extended <dbname>
set hive.ddl.output.format=json; //可将输出格式化为json（支持0.9.0之后的版本）
```
* Locks

```sql
show locks;
show locks <db.tablename> extended;
```

* 查询常用设置  

```sql 
SET hive.mapred.mode = strict;                         // 严格模式 
SET hive.resultset.use.unique.column.names=false;      // 输出字段是否带表名
SET hive.cli.print.header = true;                      // 打印表头字段名
SET mapreduce.job.queuename=root.offline.xxx;          // 队列
```

* beeline

```
$ beeline -u jdbc:hive2://ip:10000 -n hadoop
```

* 正则匹配

```sql
https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html
http://www.unicode.org/reports/tr18/#Basic_Unicode_Support
去除汉字
select regexp_replace('adf汉字234', '\\p{script=Han}', '')
```

