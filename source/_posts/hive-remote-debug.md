---
title: Hive/Java Remote Debug
date: 2018-09-13 11:31:08
tags:
 - hive
 - debug
categories: BigData
lede: "通过远程 DEBUG 排查 HIVE 脚本问题"
thumbnail: /img/hive_debug.png
---

背景： 在使用自定义 [Hive StorageHandlers](https://cwiki.apache.org/confluence/display/Hive/StorageHandlers) 时，被爆数据 bug. 出现重复记录，一开始怀疑是原始数据，单最后证明就是这个 `StorageHandlers` 出的问题，但是想不到是啥原因。于是采用了 remote debug 的方式来排查整个自定义类的执行过程。


### 先了解 Hive DEBUG 的使用  

在 hive-cli 中，默认是支持 `--debug` 的启动配置的。详情如下：

```bash  
hive --debug --help
Allows to debug Hive by connecting to it via JDI API

Usage: hive --debug[:comma-separated parameters list]

Parameters:

recursive=&lt;y|n&gt;             Should child JVMs also be started in debug mode. Default: y
port=&lt;port_number&gt;          Port on which main JVM listens for debug connection. Default: 8000
mainSuspend=&lt;y|n&gt;           Should main JVM wait with execution for the debugger to connect. Default: y
childSuspend=&lt;y|n&gt;          Should child JVMs wait with execution for the debugger to connect. Default: n
swapSuspend                 Swaps suspend options between main and child JVMs
```

可以看到， 默认启动 debug 的端口为 `8000`, 可以自行修改，包括是否挂起等设定。我这边启动命令为：    

```bash  
hive --debug:port=5005,mainSuspend=y -f debug.hql
```

说明: 由于我们限制了开发环境和正式环境的端口访问，也就是本地无法连接线上 debug 的 5005 这个端口，无奈用了 ssh 打了一个洞 `ssh -f -N -Lthishost:5005:targethost:5005 localhost`  


### 本地 IDEA 启动 Remote Debug

以 Intellij IDEA 为例：   

Run -> Debug... -> Edit Configurations ...  
![](/img/hive_debug.png)  

如上图所示，配置好 IP 和端口，点击 Debug.  

最终定位问题，因为在自定义 `getSplit` 的时候，文件做了切割，切割之后又做了转换，导致文件读取了2次。因为文件是压缩文件，不允许切割的，所以覆写了 `isSplitable` 就可以了。

```java  
@Override
protected boolean isSplitable(FileSystem fs, Path file) {
    return false;
}
```

### 参考资料  

* [Hive StorageHandlers](https://cwiki.apache.org/confluence/display/Hive/StorageHandlers)
* [Java Remote Debugging](https://stackify.com/java-remote-debugging/)
* [FileInputFormat API](https://hadoop.apache.org/docs/r2.7.5/api/org/apache/hadoop/mapred/FileInputFormat.html)
