---
title: Eclipse远程调试Java应用
date: 2016-04-21 15:08:13
tags: 
 - java
 - debug
categories: Java
thumbnail: /img/java-remote-debug.png
lede: "java remote debug"
featured: false
---

* 原文地址 [Debug Java applications remotely with Eclipse](http://www.ibm.com/developerworks/library/os-eclipse-javadebug/)

## 前提

如果你还没准备好， 下载Eclipse V3.4 (Ganymede)，在Ganymede中socket监听连接器已被添加到远程Java应用程序启动配置类型中。Eclipse新的socket监听连接器允许你启动Java调试模式，它监听特定的socket连接，可以使用命令行选项来启动程序连接调试模式开始程序调试。在Ganymede之前的发行版本中，只有提供了socket-attaching连接器，当前调试程序需作为调试主机被调试器连接。这对于移动设备由于内存电源不足是不切实际的。

要使用远程调试，Java虚拟机必须是5.0或更高的版本，如IBM® J9或者Sun Microsystems公司的Java SE开发工具包（JDK），在这篇文章中，我们重点关注远程调试，而不是Eclipse调试功能的每一个细节。请查阅相关[资料](http://www.ibm.com/developerworks/library/os-eclipse-javadebug/#resources) 有关于Eclipse调试及上述信息。

<!-- more -->
## JPDA介绍 

* 常用缩写
  * JDI— Java Debug Interface
  * JDT— Java Development Tools
  * JDWP— Java Debug Wire Protocol
  * JPDA— Java Platform Debugger Architecture
  * JVM— Java Virtual Machine
  * JVMDI— JVM Debug Interface
  * JVMTI— JVM Tool Interface
  * VM— Virtual Machine

Sun Microsystems的Java平台调试器架构（JPDA）技术是一个多层次的体系结构，允许你在任何情况下轻松调试Java应用程序。JPDA由两个接口组成（一个是JVM工具和JDI），另一个是协议（Java Debug Wire Protocol），这两个软件一起组成（前台和后台），它的设计是为了在任何环境下使用调试器。JPAD也不仅仅是桌面系统，同时也适用于嵌入式系统。
JVM工具接口（JVMTI）定义了一个虚拟机必须提供的调试（编者注：从java V5开始，JVMTI取代JVMDI，用于java V1.4），

















