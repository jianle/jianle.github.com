---
title: 初试 Protocol Buffers
date: 2016-06-29 22:35:36
tags: java
categories:
  - Java
thumbnail: /img/protocol-buffers.png
lede: "Protocol buffers are a flexible, efficient, automated mechanism for serializing structured data – think XML, but smaller, faster, and simpler"
featured: true
---

[Protocol Buffers](https://developers.google.com/protocol-buffers/docs/javatutorial)
--------------
现有个[snappy](http://google.github.io/snappy/)压缩文件，解压后文件存储格式为：

```
[4 byte size][protobuf][4 byte size][protobuf]...
```
4个字节（高位存储）存储的是后面`protobuf`序列化的长度，需对文件进行解析

<!-- more -->

* install protoc

```bash
$ wget https://github.com/google/protobuf/releases/download/v2.6.1/protobuf-2.6.1.tar.gz
$ tar -zxvf protobuf-2.6.1.tar.gz
$ cd protobuf-2.6.1
$ ./configure
$ make & make install
```

`.protoc`文件内容如下：

```java
syntax = "proto2";

package tutorial;

option java_package = "com.example.tutorial";
option java_outer_classname = "CommonDumpOuter";

message CommonDump {
  optional uint32 tableId = 1;
  optional uint32 colSpace = 2;
  optional bytes rowKey = 3;
  optional bytes colKey = 4;
  optional bytes value = 5;
  optional int64 score = 6;
  optional uint32 ttl = 7;
}
```
* 通过protoc编译成`.java`文件

```
$ protoc --java_out=. commonDump.proto
```

* 通过java解析文件


a. 首先要知道如何将高位的4字节长度转换为int

```java
public static int bytesToInt(byte[] src) {
    int value;
    value = (int) ((src[3] & 0xFF)
            | ((src[2] & 0xFF) << 8)
            | ((src[1] & 0xFF) << 16)
            | ((src[0] & 0xFF) << 24));
    return value;
}
```

b. 然后使用SnappyInputStream读入file

```java
InputStream in = new FileInputStream("/tmp/path/dump_file.dat.sp");

in = new SnappyInputStream(in);

byte[] header = new byte[4]; //用于读入长度
CommonDump commonDump = null; // protpbuf
long length = 0; //记录数
while (in.read(header, 0, 4) != -1) {

    int bodylength = bytesToInt(header);
    byte[] body = new byte[bodylength];
    in.read(body, 0, bodylength);
    length++;

    commonDump = CommonDump.parseFrom(body);
    // 接下来可以打印数据看看
}
```

#### 总结：
1. 主要需要理解文件的存储格式，才能正确的将`protobuf`序列化文件反序列化。其次是4字节的高位存储，如何将高位存储的byte转换成int
2. 在文件读入的时候，使用snappy输入流
3. 解析是一部分，后续如何结合hive，定义表格式化读取文件展示才是大的挑战
