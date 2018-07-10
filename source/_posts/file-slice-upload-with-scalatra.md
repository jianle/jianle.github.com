---
title: 文件切片上传
date: 2018-07-10 11:03:38
tags:
  - js
  - scalatra
categories:
  - Java
lede: "对于大文件的上传，如果使用长连接，后端服务压力会很大。所以多数都会采用切片上传/断点续传的方式来实现前端小文件上传，后端合并成大文件"
thumbnail: /img/scalatra.png
---

一般而言，企业服务里面的 nginx 全局设置的 timeout 时长都不会超过 15s，假设要上传一个 100M 的文件到服务端，平均上传速度 1M/s，那也得 100s 远远超过了 15s 的时长。
所以必须采用切片上传/断点续传的方式来实现前端小文件上传，后端合并成大文件。


### 前端切片   

```js
//slice upload
var file = document.getElementById('file').files[0];
var fileSize = file.size;
var chunkSize = 2 * 1024 * 1024; // buffer bytes
var offset = 0;
var filename = file.name;

while (offset < fileSize) {
  var size = chunkSize + offset;
  if (size > fileSize) {
    size = fileSize;
  }
  //切片
  var loaded = file.slice(offset, size, file.type);
  offset = size;
  var xmlRequest = new XMLHttpRequest();
  xmlRequest.open('POST', config.contextPath + '/upload/api/file-slice', false);
  var data = new FormData();
  data.append('filename', filename);
  data.append('type', file.type);
  data.append('file', loaded);
  xmlRequest.send(data);
}
```

切片上传中 `file.slice(offset, size, file.type)` 的 file.type 是保留切片之后的文件类型，可了解[MIME 类型](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)

### 后端接收

后端接收因每个服务框架不同而采用不同的处理，下面例子是 scalatra 框架的代码片段:

```scala
post("/api/file-slice") {
  contentType = formats("json")

  val fileName = params.getOrElse[String]("filename", "")
  val out = new FileOutputStream(outputFileTmp(fileName), true)
  fileParams.get("file") match {
    case Some(file) => IOUtils.copy(file.getInputStream, out)
  }

  out.close()
  Map(
    "status"    -> "ok"
  )
}
```

我这边没有用 md5 的验证，而直接使用文件名来续写，这个按照每个人的不同需求做不同的处理，标准做法都应该给 md5 的验证，仅供参考。
