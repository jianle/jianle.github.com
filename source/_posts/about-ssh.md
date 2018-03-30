---
title: SSH 使用技巧
date: 2016-04-16 09:32:09
tags: ssh
categories: Linux
thumbnail: /img/linux-ssh.jpg
---

在实际开发过程中，ssh 是必不可少的，在本文中我们将详细描述关于 ssh 的使用，如： 转发、代理、配置等等

<!-- more -->

### ssh|sshd 服务   

* 服务管理（启动/关闭/重启）

```bash   
$ sudo service ssh|sshd start/status/stop
or
$ /etc/init.d/ssh|sshd start
```

### 配置ssh  

* 生成key
* 配置.ssh/config

```bash
$ ssh-keygen -t rsa                             //生成key
$ ssh-add                                       //秘钥加入ssh-agent
$ ssh-add -l                                    //查看
$ cat ~/.ssh/id_rsa.pub >> authorized_keys
$ vim ~/.ssh/config
Host mid
     User saboloh                               //配置默认用户
```

### ssh 端口转发

* 端口转发/代理   

```bash
$ /usr/bin/ssh -f -N -Llocalhost:10000:10.126.215.131:10000 root@10.126.97.98
$ ssh -fNg -L 3309:10.126.81.130:3306 hadoop@10.126.81.130
$ /usr/bin/ssh -f -N -D 0.0.0.0:8081 localhost

本地查看端口：
$ telnet localhost 10000

如果服务器报错：
open failed: administratively prohibited: open failed

这是由于代理机器的ssh服务部支持转发 换过一台机子
```

### 使用ssh配置socks代理

* 创建本地代理  

```bash
$ ssh -f -N -D 0.0.0.0:8081 localhost
```

* 使用node.js实现FindProxyForURL  
  * app.js
  * index.html

```js  
//app.js
var express = require('express');
var path = require('path');
var app = express();


app.use(express.static(path.join(__dirname, 'public')));

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});

//index.html
function FindProxyForURL(url, host) {
  if (host.indexOf('10.') == 0 ||
    host.indexOf('tjtx-') == 0 ||
    host.indexOf('db20-') == 0 ||
    host.indexOf('ods10-') == 0
    ) {
    return "SOCKS 10.249.14.41:8081;";
  }
    return "SOCKS 10.249.14.41:8081;";
}
```
* 用supervisor守护node.js 进程   

```bash  
[program:app]
directory = /home/saboloh/dwetl/socks
command = /home/saboloh/developer/node-v5.1.0-linux-x64/bin/node app.js
user = root
stdout_logfile = /home/saboloh/dwetl/socks/log
redirect_stderr = true
```

* 配置url

```bash
http://localhost:3000
```
