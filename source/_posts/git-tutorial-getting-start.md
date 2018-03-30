---
title: Git学习指南（一）— 入门
date: 2016-06-15 13:37:22
tags: Git
categories: Git
thumbnail: /img/git/git-logo.png
lede: "git入门教程，如何搭建git环境，初始化项目，提交远程仓库等等"
featured: false
---

在Windows下安装和配置Git
----------------------

* 前往[Git官网](http://git-scm.com/)，下载安装包并安装（建议安装至D:\Git目录），从开始菜单打开Git Bash：
<!-- more -->
![Git Bash](/img/git/git-bash.png)

* 增加中文支持。向/etc/git-completion.bash文件追加以下内容：

```
alias ls='ls --show-control-chars --color=auto'
alias gl='git log --graph --pretty=format:'\''%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset'\'' --abbrev-commit --date=relative'
```

并向/etc/gitconfig文件追加：

```
[gui]
encoding = utf-8
```

重新启动Git Bash程序。

* 配置Git用户名和邮箱，会在提交信息中显示。

```bash
$ git config --global user.name "张三"
$ git config --global user.email "sanzhang@xx.com"
```

* 生成SSH公私钥对，使用默认目录，建议不要设置密码：

```bash
$ ssh-keygen -t rsa
$ chmod 700 ~/.ssh
$ chmod 400 ~/.ssh/id_rsa
```

![ssh-keygen](/img/git/sshkey.png)

复制~/.ssh/id_rsa.pub公钥文件的内容（可以用资源管理器到C:\Documents and Settings\用户名\.ssh\中查看），使用域账号登录[GitHub](https://github.com/)，进入[Settings](https://github.com/settings/keys)，新增公钥并粘贴刚才复制的内容。

![](/img/git/configkey.png)

在Linux(ubuntu)下安装和配置Git
----------------------------

* 使用`sudo apt-get install git`安装git
* 配置环境变量 `vim .bashrc`

```
alias gl='git log --graph --pretty=format:'\''%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset'\'' --abbrev-commit --date=relative'
```
* 其余配置同上

### 安装TortoiseGit（可选）

* 前往[官网](https://code.google.com/p/tortoisegit/wiki/Download)下载安装包并安装（建议安装至D:\TortoiseGit目录）；
* 右键进入TortoiseGit - Settings界面，点击Network，设置SSH client为D:\Git\bin\ssh.exe
* 使用邮件菜单的Clone就能将远程仓库下载到本地了，并能使用Pull来更新：

### 安装EGit

**注：最新版的Eclipse已经包含了EGit插件，下面内容仅针对老版本Helios**

* 打开Eclipse，点击Help - Install New Software
* 在地址栏中输入 http://download.eclipse.org/egit/updates-2.1 ，回车
* 勾选Eclipse Git Team Provider - Eclipse EGit一项即可，安装后会重启Eclipse
* 导入仓库中的项目，右击选择Team - Share Project，使用Git
* 然后就能直接在Eclipse中进行各种Git操作了

新建Git仓库
----------

### 新建本地仓库

```bash
$ cd /path/to/your-new-repo
$ git init
$ git add -A
$ git commit -m 'init'
```

### 在github上新建远程仓库

[github](https://github.com/)中每个用户都可以在自己名下建立任意多个仓库。登录网站后，点击右上角的“+”号，选择“New repository”，输入仓库名称即可。

### 推送至远程仓库

#### 1. 对于新建的仓库

```bash
$ git remote add origin git@github.com:yourname/your-new-repo.git
$ git push origin master
$ git branch --set-upstream master origin/master
```

这三条语句的作用分别是：

1. 在本地仓库的配置文件中新增一个远程仓库地址，名称为origin。
2. 将本地仓库的master分支推送到线上。
3. 将本地仓库的master分支和远程仓库的master分支关联起来，这样就能直接运行git pull来拉取远程的改动。

#### 2. 对于从其他仓库克隆过来的项目

```bash
$ git remote remove origin
$ git remote add origin ... 同上
```

当然，你也可以使用新的远程仓库名称，这样可以让你的本地仓库和多个远程仓库相关联：

```bash
$ git remote add newname git@github.com:yourname/your-new-repo.git
$ git push newname master
$ git branch --set-upstream master newname/master
```

#### 3. 直接克隆新建的远程仓库

```bash
$ git clone git@github.com:yourname/your-new-repo.git
$ cd your-new-repo
```

如果你没有先在本地建立仓库，可以直接克隆新建好的远程仓库，这时origin地址、master分支的关联都会自动配置。这种情况用的比较少。
