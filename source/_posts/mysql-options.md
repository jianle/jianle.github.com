---
title: MySQL使用
date: 2016-08-26 12:14:36
tags:
  - mysql
categories:
  - BigData
thumbnail: /img/mysql.png
lede: "MySQL使用小技巧，mysqldump数据|安装|创建用户等等"
featured: false
---

### 基本操作

* MySQL 企业版安装后，root密码自动生成在: `/root/.mysql_secret`

* mysqldump 导出表结构到指定库    
<!-- more -->
```bash
$ mysqldump -hIP -uroot -ppassword --compact --skip-lock-tables --no-data database tableName > mysqldumpTmp.sql
$ mysql -hIP -uroot -ppassword -Ddatabase < mysqldumpTemp.sql
$ mysqldump -hIP -uroot -ppassword --compact --skip-lock-tables --no-data database tableName | mysql -uroot -ppassword databases
```

* MySQL 创建指定用户，并授权
```bash
$ CREATE USER 'dw_tools'@'localhost' IDENTIFIED BY 'dw_tools';
$ GRANT ALL ON *.* TO 'dw_tools'@'localhost';
$ FLUSH PRIVILEGES;
```

* 查看进程信息
```
$ SELECT * FROM information_schema.processlist WHERE state = 'locked';
```


### 切换MySQL 文件目录

* Stop MySQL using the following command:
```
sudo /etc/init.d/mysql stop
```

* Copy the existing data directory (default located in /var/lib/mysql) using the following command:
```
sudo cp -R -p /var/lib/mysql /newpath
```

* edit the MySQL configuration file with the following command:
```
sudo gedit /etc/mysql/my.cnf
```

* Look for the entry for datadir, and change the path (which should be /var/lib/mysql) to the new data directory

* In the terminal, enter the command:
```
sudo gedit /etc/apparmor.d/usr.sbin.mysqld
```

* Look for lines beginning with `/var/lib/mysql`. Change `/var/lib/mysql` in the lines with the new path.

* Save and close the file.

* Restart the AppArmor profiles with the command:
```
sudo /etc/init.d/apparmor reload
```

* Restart MySQL with the command:
```
sudo /etc/init.d/mysql restart
```

* Now login to MySQL and you can access the same databases you had before.


### 修改字符集

* 修改Column字符集   
```
CREATE TABLE `COLUMNS_V2` (
  `CD_ID` bigint(20) NOT NULL,
  `COMMENT` varchar(256) CHARACTER SET latin1 COLLATE latin1_bin DEFAULT NULL,
  `COLUMN_NAME` varchar(128) CHARACTER SET latin1 COLLATE latin1_bin NOT NULL,
  `TYPE_NAME` varchar(4000) DEFAULT NULL,
  `INTEGER_IDX` int(11) NOT NULL,
  PRIMARY KEY (`CD_ID`,`COLUMN_NAME`),
  KEY `COLUMNS_V2_N49` (`CD_ID`),
  CONSTRAINT `COLUMNS_V2_FK1` FOREIGN KEY (`CD_ID`) REFERENCES `CDS` (`CD_ID`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```
```
CREATE TABLE `PARTITION_KEYS` (
  `TBL_ID` bigint(20) NOT NULL,
  `PKEY_COMMENT` varchar(4000) CHARACTER SET latin1 COLLATE latin1_bin DEFAULT NULL,
  `PKEY_NAME` varchar(128) CHARACTER SET latin1 COLLATE latin1_bin NOT NULL,
  `PKEY_TYPE` varchar(767) CHARACTER SET latin1 COLLATE latin1_bin NOT NULL,
  `INTEGER_IDX` int(11) NOT NULL,
  PRIMARY KEY (`TBL_ID`,`PKEY_NAME`),
  KEY `PARTITION_KEYS_N49` (`TBL_ID`),
  CONSTRAINT `PARTITION_KEYS_FK1` FOREIGN KEY (`TBL_ID`) REFERENCES `TBLS` (`TBL_ID`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

```
ALTER TABLE COLUMNS_V2 MODIFY `COMMENT` varchar(256) CHARACTER SET utf8 collate utf8_bin;
ALTER TABLE PARTITION_KEYS MODIFY `PKEY_COMMENT` varchar(4000) CHARACTER SET utf8 collate utf8_bin;
```
