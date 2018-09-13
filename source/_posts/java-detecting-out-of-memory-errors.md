---
title: Java 内存溢出排查
date: 2018-07-05 11:35:47
tags:
  - java
categories:
  - Java
lede: "Java OOM 毫无疑问是开发人员常见并且及其痛恨的问题，但是任何服务的开发都没法避免 OOM，该如何取定位排查及修复"
thumbnail: /img/mat_thumb.png
---

Java OOM 毫无疑问是开发人员常见并且及其痛恨的问题，但是任何服务的开发都没法避免 OOM。 因此，OOM 的排查及定位是每个 Java 工程师都必备的技能。

#### 所遇到的问题

在使用 scala 开发的一个 web 服务，在用户使用中，经常出现： `java.lang.OutOfMemoryError: Java heap space` 。而且还束手无策，每次都只能重启服务解决。


#### 准备

服务使用 jetty 发布的，先来看一下我这个服务的启动参数：

```java  
/opt/soft/jdk/jdk1.7.0_40/bin/java \
  -server -Xmx4G -XX:MaxPermSize=1024M -XX:PermSize=256M \
  -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:-CMSConcurrentMTEnabled -XX:CMSInitiatingOccupancyFraction=65 -XX:+CMSParallelRemarkEnabled \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/opt/soft/heapdump/ \
  -Dscala.concurrent.context.numThreads=500 \
  -Dscala.concurrent.context.maxThreads=500 \
  -Dfile.encoding=UTF-8 -jar start.jar >> log 2>&1 &
```

#### 排查

通过增加了参数 `-XX:+HeapDumpOnOutOfMemoryError` 和 `-XX:HeapDumpPath` 当在 OOM 的时候，服务会生成一个 `java_pid$pid.hprof` 二进制文件。   

下面就是使用工具分析这个 `.hprof` 文件来定位问题了。使用 [Memory Analyzer (MAT)](https://www.eclipse.org/mat/) 来分析该文件，效果如下：  
![](/img/overview_mat.png)

效果很吓人，什么鬼，什么东西，吃了 3.8G 的内存，我#%$#@#@#&^&^&#$....      
打开 Leak Suspects» Leaks» Problem Suspect 1 看到如下详情：    
![](/img/mat_find_problem.png)

一开始可能没那么快找到问题，但是这个图已经很明显说明了问题，是 `ArrayList` 的内容太大，沾满了内存。但是你可能还不清楚具体那块代码导致，这个时候你可以点击那个 `ArrayList` 在左侧栏看 `Attribute`。
然后一直鼠标右键 `into` 进去看里面的详情，最终是可以看内容的。   

#### 问题原因  

问题排查到最后，看到的是 `ArrayList` 里面存的全是 `ResponseBodyPart`， 然后就想到了项目使用到 [Dispatch](https://dispatchhttp.org/Dispatch.html) 请求下载结果文件，
于是乎去找到问题代码，错误代码如下：   

```scala
val outputReq = dispatch.url(url) / "task" / "output" / id
val outputFuture = Http(outputReq > { res =>
  val out = new FileOutputStream(outputFile(taskId), true)
  IOUtils.copy(res.getResponseBodyAsStream(), out)
  out.close
})
```  
看不出问题，感觉一切正常。翻源码会发现，`res.getResponseBodyAsStream()` 之前，已经将所有内容都存入一个 `ArrayList` 当中了。哎，没用对啊。   

#### 解决办法  

问题已经定位到，于是取了解了一下这个项目，该如何使用 stream 的方式来读取并写入文件流。然后发现，人家有一个 [read line by line](https://github.com/dispatch/reboot/blob/master/core/src/main/scala/stream/strings.scala) 的实现。但是切割上其实是有问题的，因为拿到一批 bytes 之后，直接转成了 string 并用分隔符分割，
奈何内容里面有中文，出现乱码了。

最终，参考项目本身的 `as.stream.Lines` 写了一个 `as.stream.Bytes` 来通过 bytes 边读边写，如下：  

```scala
val bos = new BufferedOutputStream(new FileOutputStream("/tmp/file.txt", true))
val outputFuture = Http(outputReq > as.stream.Bytes(bytes => {
  bos.write(bytes)
}))
```

#### 总结

主要描述了分析问题的思路和方向，问题都大同小异，OOM 总会有原因的，有原因肯定可以找到并解决。MAT 这个分析工具很实用，内容很详细。以前遇到 OOM 问题都是重启服务，治标不治本，还是要多分析问题并解决。
