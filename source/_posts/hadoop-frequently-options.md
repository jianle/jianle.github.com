---
layout: post
title: Hadoop常用操作
date: 2016-08-19 11:38:42
tags: 
  - Hive
  - hadoop
  - command
  - hive
categories: BigData
thumbnail: /img/hadoop.png
lede: "工作中常用的hadoop操作，logs、httpfs、rmapi's"
featured: true
---

## Hadoop Commands

工作中常用的hadoop操作

* [Yarn logs](http://hortonworks.com/blog/simplifying-user-logs-management-and-access-in-yarn/)
<!-- more -->
```bash 
$ yarn logs -applicationId <application ID> -appOwner <Application Owner> -containerId <Container ID> -nodeAddress <Node Address>   
```

* Yarn application  

* [HttpFs](https://hadoop.apache.org/docs/r2.4.1/hadoop-hdfs-httpfs/index.html)  
```bash 
$ http://localhost:14000/webhdfs/v1/user/history/done/2016/09/03/001509/job_1466675370769_1509892_conf.xml?op=open&user.name=hadoop 
```

* [ResourceManager REST API's](https://hadoop.apache.org/docs/r2.4.1/hadoop-yarn/hadoop-yarn-site/ResourceManagerRest.html)




