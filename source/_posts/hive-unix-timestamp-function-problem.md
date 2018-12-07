---
title: Hive unix_timestamp function problem
date: 2018-12-07 13:44:41
tags: hive
categories: BigData
thumbnail: /img/hive_unixtimestamp.png
lede: "有关 Hive 使用 unix_timestamp 函数返回结果 null 问题"
---


该函数使用的是 [`SimpleDateFormat`](https://docs.oracle.com/javase/tutorial/i18n/format/simpleDateFormat.html) 来格式化日期，但该类解析日期的时候，依赖  `Locale` 但是集群机器的 `Locale` 似乎并不全是一致的。

所以会出现这样的现象： 

```sql  
SELECT from_unixtime(unix_timestamp('Sun Jun 17 14:27:22 CST 2012',  'EEE MMM dd hh:mm:ss \'CST\' yyyy'))
```

该 sql 返回有时候为 null，有时候又是正常。
  

**解决办法**: 

```sql  
ADD JAR hdfs://nameservice/user/hadoop/udf/UDFUnixTimeStamp.jar;
CREATE TEMPORARY FUNCTION unixtimestamp AS 'com.hive.udf.UDFUnixTimeStamp';
SELECT from_unixtime(unixtimestamp('Sun Jun 17 14:27:22 CST 2012',  'EEE MMM dd hh:mm:ss \'CST\' yyyy'))
```

该自定义函数指定了 Locale 为 `en`，所以日期格式如果是中文的，是不可已使用的。

附加 UDF 代码： 

```java 
@UDFType(deterministic = false)
@Description(name = "unix_timestamp",
        value = "_FUNC_([date[, pattern]]) - Returns the UNIX timestamp",
        extended = "Converts the current or specified time to number of seconds "
                + "since 1970-01-01.")
public class UDFUnixTimeStamp extends GenericUDFToUnixTimeStamp {

    static {
        Locale locale = Locale.ENGLISH;
        Locale.setDefault(locale);
        System.setProperty("user.language", "en");
    }

    @Override
    protected void initializeInput(ObjectInspector[] arguments) throws UDFArgumentException {
        if (arguments.length > 0) {
            super.initializeInput(arguments);
        }
    }

    @Override
    protected String getName() {
        return "unix_timestamp";
    }

    @Override
    public Object evaluate(DeferredObject[] arguments) throws HiveException {
        if (arguments.length == 0) {
            retValue.set(System.currentTimeMillis() / 1000);
            return retValue;
        }
        return super.evaluate(arguments);
    }
}
```


