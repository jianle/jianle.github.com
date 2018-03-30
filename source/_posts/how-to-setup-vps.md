---
title: 如何搭建VPS
date: 2016-04-18 08:07:33
tags: vps
categories: Linux
thumbnail: /img/google.png
lede: "搭建vps 轻松google"
featured: false
---

* 购买[vps](https://www.budgetvm.com/)  

本人在朋友推荐下购买了`budgetvm` 系统安装选择的是`ubuntu14.04`

购买完成之后初始化大概10分钟，之后便可自己通过ssh+pwd的方式登录vps（先确保能ping通，如果存在网络等任何问题，直接邮件development，解决速度还是可以的。密码在网上可以看到）

<!-- more -->
![budgetvm](/img/budgetvm.png)


* vps config

```bash
$ dpkg-reconfigure tzdata //时区
$ apt-get install locales // 字符编码
```

* prepare

```bash
$ apt-get install vim

$ grep -v ^PasswordAuthentication /etc/ssh/sshd_config | grep -v Port  > /tmp/sshd_config
$ echo "Port 50070" >> /tmp/sshd_config
$ echo "PasswordAuthentication no" >> /tmp/sshd_config
$ service ssh restart

$ apt-get install python-pip
$ pip install shadowsocks
$ apt-get install supervisor
```

* config server

```bash
$ vim /etc/supervisor/conf.d/ss.conf

[program:ss]
command=/usr/local/bin/ssserver -c /root/ss.conf -vv --user nobody
stdout_logfile=/var/log/ss.log
stderr_logfile=/var/log/ss.err
stdout_logfile_maxbytes=50MB
stderr_logfile_maxbytes=50MB
autorestart=true
autostart=true

$ vim /root/ss.conf

{
    "server_port":8080,
    "password":"{$yourpassword}",
    "method":"aes-256-cfb"
}
```

* config local

```bash
$ sudo apt-get install shadowscoks
$ sslocal  -c ss.conf -b 0.0.0.0 -d start --pid-file /tmp/ss.pid --log-file /tmp/ss.log

$ cat ss.conf
{
    "server":"${serverip}",
    "server_port":8080,
    "local_port":1080,
    "password":"${yourpassword}",
    "method":"aes-256-cfb"
}
```

备注： budgetvm主机服务在洛杉矶，所以国内登录过去会有点慢-_-|| 岛国的vps似乎很快，推荐一款[Vultr](https://www.vultr.com/)
