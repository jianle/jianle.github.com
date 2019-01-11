---
title: Hive常用函数
date: 2016-06-17 10:24:18
tags: hive
categories: BigData
thumbnail: /img/hive-functions.jpg
lede: "hive 常用udf，json、rank、date"
featured: false
---

* [get_json_object](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-get_json_object)
* rank  

```sql
SELECT a.*, rank() over(partition by key order by created desc) as rk
FROM   table_name a
```
<!-- more -->
* regexp_extract(string subject, string pattern, int index)

```sql
SELECT regexp_extract('https://cwiki.apache.org/confluence/display', 'org/([a-z]+)/', 1)
```

* regexp_replace(string INITIAL_STRING, string PATTERN, string REPLACEMENT)

```sql
SELECT regexp_replace('hive site:xxxx', '^[a-z]+', '')
```

* current_date
* date_add(string startdate, int days)
* date_sub(string startdate, int days)
* datediff(string enddate, string startdate)
* to_date(string timestamp)
* from_unixtime(string date[, string format]) 
* unix_timestamp(string date)
 
```sql
SELECT from_unixtime(unix_timestamp('2016-09-07 07:07:13.02'), 'HH');//02
```
* regexp_extract(string str, string regexp, int idx)

```sql
SELECT regexp_extract('2016-09-07 02:07:13.02', ' ([0-9]+):', 1); //02
```

* [reflect](https://cwiki.apache.org/confluence/display/Hive/ReflectUDF)  

```sql  
SELECT reflect("java.lang.String", "valueOf", 1),
       reflect("java.lang.String", "isEmpty"),
       reflect("java.lang.Math", "max", 2, 3),
       reflect("java.lang.Math", "min", 2, 3),
       reflect("java.lang.Math", "round", 2.5),
       reflect("java.lang.Math", "exp", 1.0),
       reflect("java.lang.Math", "floor", 1.9)
FROM src LIMIT 1;
 
1   true    3   2   3   2.7182818284590455  1.0
```

