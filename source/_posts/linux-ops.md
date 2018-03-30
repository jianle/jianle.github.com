---
title: Linux运维
date: 2016-04-16 18:21:16
tags: 
  - shell
  - jdk
  - wget
  - find
  - rsync
categories: 
  - Linux
thumbnail: /img/linux.png
lede: "记录工作中使用Linux所用到的一些"
featured: false
---

记录Linux相关ISSUE


## 邮件发送失败问题排查

* jstack 查看堆栈信息(RUNNABLE)

```
$ jstack 85869 | less 
$ printf "%d\n" 0x1450a  -- 对应进程ID(83210) 
```
<!-- more -->

* 查看TCP相关信息

```
$ lsof -p 85869 | grep TCP 
$ ss -atnp | grep :25  (查看邮件相关网络请求) 
$ ss -atnp | grep :25 | grep  223.252.214
ESTAB      0      69120          10.126.97.98:39905       223.252.214.64:25     users:(("java",85869,195))
ESTAB      0      288000         10.126.97.98:30519       223.252.214.64:25     users:(("java",85869,185))
ESTAB      0      43835          10.126.97.98:58921       223.252.214.65:25     users:(("java",85869,59))
ESTAB      0      201600         10.126.97.98:50990       223.252.214.65:25     users:(("java",85869,183))
ESTAB      0      276480         10.126.97.98:51453       223.252.214.65:25     users:(("java",85869,186))
ESTAB      0      243360         10.126.97.98:31148       223.252.214.64:25     users:(("java",85869,181))

$ lsof -p 85869 | grep TCP | grep smtp 
java    85869 root   59u  IPv4           96189338        0t0      TCP tjtx-97-98.58os.org:58921->mail0401.icoremail.net:smtp (ESTABLISHED)
java    85869 root  181u  IPv4           96062679        0t0      TCP tjtx-97-98.58os.org:31148->mail0402.icoremail.net:smtp (ESTABLISHED)
java    85869 root  183u  IPv4           96036545        0t0      TCP tjtx-97-98.58os.org:50990->mail0401.icoremail.net:smtp (ESTABLISHED)
java    85869 root  185u  IPv4           96047561        0t0      TCP tjtx-97-98.58os.org:30519->mail0402.icoremail.net:smtp (ESTABLISHED)
java    85869 root  186u  IPv4           96047309        0t0      TCP tjtx-97-98.58os.org:51453->mail0401.icoremail.net:smtp (ESTABLISHED)
java    85869 root  195u  IPv4           96240773        0t0      TCP tjtx-97-98.58os.org:39905->mail0402.icoremail.net:smtp (ESTABLISHED)

$ tcpdump -A -nn host 223.252.214.65 and port 25
$ tcpdump -i any -A -nn host 223.252.214.65 and dst|src port 25
```

* 查看子进程详情信息

```
$ lsof -p 83210  
$ strace -p 83210  
sendto(195, "7JOIG5jMqlSTGQo1\r\ncdnx0tHxAWoauG"..., 6570, 0, NULL, 0) = 6570
sendto(195, "jFpOF0zfxyF3I8Zd/v/TtH+vJhng1yiP"..., 8190, 0, NULL, 0) = 8190
read(196, "\223\32\0Hm\36\227\230\35\0R\233\307\225\377\310\1\2446\217K\314/\0\351)\36\227\350\346!H"..., 8192) = 8192
sendto(195, "95EWryyG0blXjukGJwjLFb38OFTO0R5r"..., 8190, 0, NULL, 0) = 8190
read(196, "\10\342\271\360?a{\201\7p\37\272\330\36\240kj\177\225\303\325\364}U\335\177\236\22\240k\272\375"..., 8192) = 8192
sendto(195, "HxyjfNV3Yz2RBmgq7JZxCMUJ+rDu/C6t"..., 8190, 0, NULL, 0  

$ lsof -P -p 85869 | grep TCP | grep 195
java    85869 root  195u  IPv4           96240773        0t0      TCP tjtx-97-98.58os.org:39905->mail0402.icoremail.net:smtp (ESTABLISHED)  

$ lsof -p 85869 | grep 196
java    85869 root  196r   REG               8,10   14009262 18481291 /data1/dw_monitor/mini/excel/太平洋数据.xlsx

$ tcpdump -i any port 39905   
```

* 配置代理转发

```
root@cp:~# /usr/lib/autossh/autossh -M 0 -L 0.0.0.0:25:stmp.xxx.com:25 localhost
修改hosts

10.249.14.41 smtp.xxx.com  //如果配置后又要注释，看看是否有nscd服务吃住缓存
```

* nc 

```
$ nc -l 1234 < filename
$ nc ip port > result
```



## JDK配置/使用

* wget下载JDK

```
$ wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u77-b03/jdk-8u77-linux-x64.tar.gz
```

* JDK环境变量

```
$ sudo vim /etc/profile.d/jdk7.sh
#!/bin/bash
export JAVA_HOME=/usr/lib/jvm/java-7-oracle-cloudera
export JRE_HOME=$JAVA_HOME/jre
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib/rt.jar

//或者采用update-alternatives
$ update-alternatives --display|--config java   //可查看已有的信息（列表，优先级）
$ update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-7-oracle-cloudera/bin/java 316  // 316-优先级
$ update-alternatives --config java  // 回车通过序列号选择
```

* 查看JVM 情况

```
$ jstat -gcutil $pid 1s
看看 E 那一列是否一直 hang 在100， 或者hang在一个值不变
如果hang住，去看看 jvm 的进程
```

## Linux 常用技巧

* 文件查找

```
$ find ~/dwetl/ -name "*.sh" -exec grep "dw_hadoop_summary_touch_city_sd" -l {} \;
$ find ~/dwetl/ -name "*.mxm" -exec grep "dw_hadoop_summary_touch" -l {} \;
```

* 查看端口

```
$ lsof -n -P | grep 8018
$ netstat -an | grep 80  
$ netstat -nltp 
```

* sftp向服务器上传文件

```
$ sftp host@ip
> put file 
```

* 查找文件并删除

```
$ [sudo] find /path/ -name ".DS_Store" -depth -exec rm {} \;
$ [sudo] find . -name "*.DS_Store" -type f -delete
```

* 配置ssh

```
$ ssh-keygen -t rsa                             //生成key
$ ssh-add                                       //秘钥加入ssh-agent 
$ ssh-add -l                                    //查看
$ cat ~/.ssh/id_rsa.pub >> authorized_keys
$ vim ~/.ssh/config
Host mid
     User saboloh                               //配置默认用户
```

* 查看占用空间

```
$ ls / | grep -vP '(proc)' | awk '{print "du -s -h /" $1}' | bash  
```

* 查看源

```
$ ldd /usr/bin/nginx
$ ldconfig -p | grep odbc
$ vim /etc/ld.so.conf.d/usr.conf 

# 编译
$ export CFLAGS=-m32 LDFLAGS=-m32 CXXFLAGS=-m32
$ ./configure
$ vim libtool
$ make -j 4 
$ make install 
```

* 同步目录

```
$ rsync -a -f"+ */" -f"- *" /data/ root@10.126.92.72:/data/
```

* grep 使用 [零宽断言](http://deerchao.net/tutorials/regex/regex.htm#lookaround)
    * -E --extended-regexp
    * -o --only-matching
    * -P --perl-regexp

```
$ grep -oP '(?<=remote.ip=)(.*)' master.properties
$ ps -ef|grep '/home/hadoop/apache-tomcat-6.0.36'|grep -v grep|awk '{print $2}'|grep -Eo '^[0-9]+$'
```

* curl/dns

```
strace -ttf curl -w '{\n"time_namelookup": %{time_namelookup},\n"time_connect": %{time_connect},\n"time_appconnect": %{time_appconnect},\n"time_pretransfer": %{time_pretranfer},\n"time_redirect": %{time_redirect},\n"time_starttransfer": %{time_starttransfer},\n"time_total": %{time_total},\n"speed_download": %{speed_download},\n"speed_upload": %{speed_upload},\n"remote_ip": "%{remote_ip}",\n"remote_port": "%{remote_port}",\n"local_ip": "%{local_ip}",\n"local_port": "%{local_port}"\n}' -D /tmp/tmpOUQ_P_ -o /tmp/tmpAUjoDh -s -S 'http://search.tj.a.ajkdns.com/search/universal/v2?m=s&query=is_list=1%26info_type=1%26info_source=1%26size=20'
```


