---
title: Hive使用遇到的一些问题
date: 2016-04-28 15:28:33
tags:
 - Hive
 - ORC
 - hive
 - issue
categories:
 - BigData
thumbnail: /img/orc-logo.png
lede: "hive 使用orc所遇到的问题，以及其他查询报错场景分析"
featured: true
---

## 记录使用`ORC`所遇到的一些问题

### 查询报错

由于hiveserver2中默认设置了“矢量化执行器” enable为true， 这导致使用orc存储的表使用上了这个参数，走了“矢量化执行器”导致错误，其中遇到的错误有：

<!-- more -->
* 数组越界  

```sql  
select a.serv_dt, a.broker_id, a.log_time, b.log_time
       ,unix_timestamp(b.log_time)-unix_timestamp(a.log_time) as time_diff
from   (select *
        from   dw_db_temp.GL_20160422_noqiang_page_broker_action_rank
        where  page_id='1-150000'
       ) a
join   dw_db_temp.GL_20160422_noqiang_page_broker_action_rank b
  on   a.con=b.con and b.rk=a.rk+1;
```
* 空指向  

```sql  
drop table if exists dw_db_temp.gh_esf_tmp1;
create table dw_db_temp.gh_esf_tmp1
as
select cal_dt
       ,city_id
       ,region_name
       ,subregion_id, subregion_name
       ,count(distinct comm_id) as pm_comm
       ,count(distinct case when is_comm_repeat=1 then comm_id end) as pm_comm_repeat
from   dw_db_temp.gh_esf_pm_broker_last
where  subregion_name in ('长阳','良乡','窦店','梨园','通州北苑','武夷花园','中央别墅区','通州北苑','华南','泉水','泡崖','机场新区','西山水库','马坡')
group by cal_dt,city_id,region_name,subregion_id,subregion_name
;
```
解决办法，将`矢量化执行器`关闭即可：  

```sql  
set hive.vectorized.execution.enabled = false;
set hive.vectorized.execution.reduce.enabled = false;
```

### 查询无结果

hive表存成`ORC`格式，自关联出现无结果现象并且无任何错误，异常SQL如下：

```sql
SELECT a.*
       ,b.call_7
FROM   (SELECT comm_id
               ,call_30
               ,call_success_30
               ,call_nonunique_30
        FROM   dw_db_temp.carrie_mr_call_jt_160428_3
        WHERE  type=3
       ) a
LEFT OUTER JOIN
       (SELECT comm_id
               ,call_30
               ,call_success_30
               ,call_nonunique_30
        FROM   dw_db_temp.carrie_mr_call_jt_160428_3
        WHERE  type=2
       ) b
   ON  a.comm_id=b.comm_id
LIMIT 100
;
```
其导致原因是因为再hiveserver2上默认开启了参数`hive.optimize.index.filter`为true，解决办法：
* 将表格式存成textfile
* 关闭hive.optimize.index.filter

```sql
set hive.optimize.index.filter=false;
```


### [写表报错](https://issues.cloudera.org/browse/DISTRO-756)  

当默认使用orc写表时，where子句中出现 where col is null 将报  java.lang.IllegalArgumentException: Bad primitive category VOID 异常

```sql  
drop table if exists dw_db_temp.carrie_call_comm_160428_01;
create table dw_db_temp.carrie_call_comm_160428_01
as
select a.*,b.pcall_id
from   (select * from dw_db.dw_phone_call_logs
        where created_dt>=date_sub(${dealDate},7)
       ) a
left outer join
       (select date_index,phonenum,loupan_id,pcall_id
        from   dw_db.dw_nh_adm_transfer_log
        where  date_index>=date_sub(${dealDate},7)
          and  flag=2
          and  type>1
       )b
   on  a.created_dt=b.date_index and a.id=b.pcall_id
where  b.pcall_id is null
;
```

解决办法： 将hive表建成textfile即可   

```sql  
create table table_name stored as textfile
as
select ......
```


## MapJoin 字段类型不一致

主要是 string 和 int 的列做 join

日志里面看 job 长期 hang 在以下位置

查看你 map 的时候，map 的状态为 FAILED， 查看 map 的日志，最后有以下类似的信息：

```
2015-12-11 04:26:06,119 INFO [main] org.apache.hadoop.hive.ql.exec.MapJoinOperator: Load back 1 hashtable file from tmp file uri:file:/hadoop3/yarn/nm/usercache/hadoop/appcache/application_1449142557781_73597/container_e11_1449142557781_73597_01_000002/Stage-9.tar.gz/MapJoin-mapfile11--.hashtable
```

*解决方法* 保证用于 join 的列一致，cast 或者直接修改表的结构

如果还不能解决，先手动禁用掉 MapJoin

```
set hive.auto.convert.join=false;
```


## hive 使用的新的序列化方法后偶尔出现的问题

这种问题一般是代码里面设置了 `set hive.exec.parallel=true;` 运行的时候，偶尔会报以下错误，重跑有可能成功：

```java
 aliasToWork (org.apache.hadoop.hive.ql.plan.MapWork)
     at org.apache.hive.com.esotericsoftware.kryo.util.DefaultClassResolver.readClass(DefaultClassResolver.java:119)
     at org.apache.hive.com.esotericsoftware.kryo.Kryo.readClass(Kryo.java:656)
     at org.apache.hive.com.esotericsoftware.kryo.serializers.ObjectField.read(ObjectField.java:99)
     at org.apache.hive.com.esotericsoftware.kryo.serializers.FieldSerializer.read(FieldSerializer.java:507)
     at org.apache.hive.com.esotericsoftware.kryo.Kryo.readClassAndObject(Kryo.java:776)
```

如果有这种情况，解决的方法是先更换序列化的方法，即加入：
```sql
set hive.plan.serialization.format=javaXML;
```

## WHERE 条件int限制值超过Integer.MAX_VALUE值导致出错

```sql
select *
from   dual
where  id=2183402559
```
id为int类型，where里面id='2183402559' 限制值超过 Integer.MAX_VALUE=2147483647 导致下面报错

```java
org.apache.hive.service.cli.HiveSQLException: Error while compiling statement: FAILED: SemanticException [Error 10041]: No partition predicate found for Alias “a” Table > “dw_ganji_house_source_officetrade”
```
