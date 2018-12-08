---
title: Arthas to Detecting Online Server Problem
date: 2018-12-08 17:05:52
tags: java
categories: Java
thumbnail: /img/arthas.png
lede: "Arthas 是 Alibaba 开源的 Java 在线分析诊断工具，深受开发者喜爱"
---

[官方文档](https://github.com/alibaba/arthas/blob/master/README_CN.md) 如是说： 

Arthas 是 Alibaba 开源的 Java 诊断工具，深受开发者喜爱。

Arthas 可以帮助你解决：

* 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
* 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
* 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
* 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
* 是否有一个全局视角来查看系统的运行状况？
* 有什么办法可以监控到JVM的实时运行状态？
* Arthas支持 JDK 6+，采用命令行交互模式，同时提供丰富的 Tab 自动补全功能，进一步方便进行问题的定位和诊断。


**快速开始**

```bash  
$ wget https://alibaba.github.io/arthas/arthas-boot.jar
$ java -jar arthas-boot.jar [-h]
```


### [使用案例](https://alibaba.github.io/arthas/advanced-use.html)

排查 Session 过期问题，启动 arthas 并输入目标进程 ID

```java 
// 查看JVM已加载的类信息
$ sc -d javax.servlet.http.HttpSession
 class-info        com.bj58.sso.session.servlet.session.CacheHttpSession                       
 code-source       file:/data1/swan/swan-0.0.1-SNAPSHOT.jar!/lib/gsso-client-1.1.3.jar!/       
 name              com.bj58.sso.session.servlet.session.CacheHttpSession                       
 isInterface       false                                                                       
 isAnnotation      false                                                                       
 isEnum            false                                                                       
 isAnonymousClass  false                                                                       
 isArray           false                                                                       
 isLocalClass      false                                                                       
 isMemberClass     false                                                                       
 isPrimitive       false                                                                       
 isSynthetic       false                                                                       
 simple-name       CacheHttpSession                                                            
 modifier          public                                                                      
 annotation                                                                                    
 interfaces        javax.servlet.http.HttpSession                                              
 super-class       +-java.lang.Object                                                          
 class-loader      +-org.springframework.boot.loader.LaunchedURLClassLoader@8efb846            
                     +-sun.misc.Launcher$AppClassLoader@33909752                               
                       +-sun.misc.Launcher$ExtClassLoader@295a0382                             
 classLoaderHash   8efb846                                                                     

 class-info        javax.servlet.http.HttpSession                                              
 code-source       file:/data1/swan/swan-0.0.1-SNAPSHOT.jar!/lib/tomcat-embed-core-8.0.33.jar!/
 name              javax.servlet.http.HttpSession                                              
 isInterface       true                                                                        
 isAnnotation      false                                                                       
 isEnum            false                                                                       
 isAnonymousClass  false                                                                       
 isArray           false                                                                       
 isLocalClass      false                                                                       
 isMemberClass     false                                                                       
 isPrimitive       false                                                                       
 isSynthetic       false                                                                       
 simple-name       HttpSession                                                                 
 modifier          abstract,interface,public                                                   
 annotation                                                                                    
 interfaces                                                                                    
 super-class                                                                                   
 class-loader      +-org.springframework.boot.loader.LaunchedURLClassLoader@8efb846            
                     +-sun.misc.Launcher$AppClassLoader@33909752                               
                       +-sun.misc.Launcher$ExtClassLoader@295a0382                             
 classLoaderHash   8efb846                                                                     

Affect(row-cnt:2) cost in 204 ms.
```

确认是否加载的类与预期的一致，如果不一致，那么恭喜，已经找到问题了。如果一致，那么继续  


```java  
// 反编译该类代码
$ jad com.bj58.sso.session.servlet.session.CacheHttpSession

ClassLoader: 
+-org.springframework.boot.loader.LaunchedURLClassLoader@8efb846 
  +-sun.misc.Launcher$AppClassLoader@33909752   
    +-sun.misc.Launcher$ExtClassLoader@295a0382

Location: 
file:/data1/swan/swan-0.0.1-SNAPSHOT.jar!/lib/gsso-client-1.1.3.jar!/
...
```

如代码查查看不出问题，那么继续，观察方法执行数据 

```java 
$ watch com.bj58.sso.session.cache.support.WTableEngine get "{params,returnObj}" -x 3 -b -s
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 161 ms.
...
```

watch 函数的执行前后数据的变化，确认是否正常。在我排查问题的过程中，在这一步定位出数据参数问题，然后结合该类反编译源码，最终确认问题所在。  








