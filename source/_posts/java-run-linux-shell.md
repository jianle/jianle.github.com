---
title: Java 运行 Linux 命令
date: 2017-06-23 09:44:52
tags: 
  - java
  - linux
  - shell
categories:
  - Java
lede: "Java中如何运行linux命令，并输出stderr日志信息"
thumbnail: /img/java.jpg
---


博主尝试过很多方式运行shell命令，以前使用过 apache common 里面的 command 来运行。 
后面嫌麻烦，改用了 Java 自带的 Runtime.getRuntime().exec(command) 来运行命令，日志获取没问题，但是没法实现环境变量的设置。


于是使用 Java 自带的 ProcessBuilder 来实现。 实现方法如下：

```java 
ProcessBuilder pb = new ProcessBuilder("hadoop", "distcp",
        "-Dmapreduce.job.queuename=root.hadoop",
        "-pb", "-overwrite", source.toString(), target.toString());
logger.info(pb.command().toString());
Map<String, String> env = pb.environment(); 
//设置环境变量
env.put("HADOOP_PROXY_USER", "hadoop"); 
String USR_HOME = System.getProperty("user.home");
pb.directory(new File(USR_HOME));
try {
    Process p = pb.start();
    //等待进程执行完毕
    //p.waitFor();
    //如果进程运行结果不为0,表示进程是错误退出的
    //获得进程实例的错误输出
    new Thread(new DisCPLog(p, "err")).run();
    new Thread(new DisCPLog(p, "std")).run();

} catch (Exception ex) {
    System.out.println("Exeception ::: " + ex.getMessage());
}
```

获取日志代码：
```java 
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

/**
 * Created by jianle on 17-6-22.
 */
public class DisCPLog implements Runnable {

    Process p;
    String logType = "std";

    public DisCPLog(Process p, String logType) {
        this.p = p;
        if (logType != null) this.logType = logType;
    }

    @Override
    public void run() {
        BufferedReader Inreader = null;
        if ("err".equalsIgnoreCase(logType)) {
            Inreader = new BufferedReader(new InputStreamReader(p.getErrorStream()));
        } else {
            Inreader = new BufferedReader(new InputStreamReader(p.getInputStream()));
        }
        String line = "";
        try {
            while ((line=Inreader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

