---
date: 2017-12-13
title: 如何配置 SSH 登陆时的欢迎信息
tags:
- 中文
- Linux
- Bash
- SSH
description: 在 SSH 登陆时，系统通常会在终端打印出一系列欢迎信息，比如当前的内核版本等。在本文中，我们将会描述如何自定义这些被称作 Message Of The Day 的系统信息。
---
# 如何配置 SSH 登陆时的欢迎信息

## 引言

在 SSH 登陆时，系统通常会在终端打印出一系列欢迎信息，比如当前的内核版本等。在本文中，我们将会描述如何自定义这些被称作 `MOTD：Message Of The Day` 的系统信息。

## 一、自定义系统的 MOTD 脚本
让我们先来回顾一下 SSH 登陆后系统会打印哪些信息。这里以 Ubuntu 16.04 为例，在默认设置下，系统会打印出如下信息：

```
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-96-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

Last login: Wed Dec  6 08:15:20 2017 from 5.6.7.8
```

这里主要列出了 Ubuntu 产品相关的一些链接，但我们并不希望每次登陆后总看到这些信息，若能把这些链接替换成当前系统的内存、硬盘、CPU 的相关信息，应该会更加实用。那么接下来，就让我们手动修改这些脚本：

```bash
#首先登陆到服务器
$ ssh haoxi911

#进入motd脚本的目录，查看所有脚本
$ cd /etc/update-motd.d/
$ ls
00-header         ...
```

可以看到，这个目录下面有很多脚本，而且每个脚本都有一个编号。Ubuntu 系统在用户登陆后，会按照这些编号，从小到大依次执行每个脚本，并把输出的信息打印到屏幕上。

首先，让我们将 10-help-text 脚本中的内容，也就是上面打印链接的那部分代码注释掉：

```bash
#编辑脚本并注释相关代码
$ sudo vi 10-help-text

#printf "\n"
#printf " * Documentation:  https://help.ubuntu.com\n"
#printf " * Management:     https://landscape.canonical.com\n"
#printf " * Support:        https://ubuntu.com/advantage\n"
```

接下来，我们将新建一个脚本文件 05-system-info ：

```bash
#新建文件并贴上如下代码
$ sudo vi 05-system-info
```

```bash
#! /usr/bin/env bash

# Basic Info
HOSTNAME=`uname -n`

# System Load
DISKSPACE1=`df -Ph | grep vda1 | awk '{print $2}' | tr -d '\n'`
DISKSPACE2=`df -Ph | grep vda1 | awk '{print $3}' | tr -d '\n'`
MEMORY1=`free -t -m | grep Mem | awk '{print $2"M";}'`
MEMORY2=`free -t -m | grep Mem | awk '{print $3"M";}'`
LOAD01=`cat /proc/loadavg | awk {'print $1'}`
LOAD05=`cat /proc/loadavg | awk {'print $2'}`
LOAD15=`cat /proc/loadavg | awk {'print $3'}`

echo "
 * Hostname   : $HOSTNAME
 * Disk Space : $DISKSPACE2 / $DISKSPACE1
 * CPU Usage  : $LOAD01, $LOAD05, $LOAD15 (1, 5, 15 min)
 * Memory     : $MEMORY2 / $MEMORY1
"
```

保存文件并退出 vim，这里需要注意，默认的新建文件并没有可执行权限，也就是说该文件可读可写，但是不能被运行。所以，我们需要对新添加的脚本设置可执行权限，如下：

```bash
$ sudo chmod +x 05-system-info

#重新登陆，观察新的欢迎信息
$ exit
$ ssh haoxi911
```

如果一切顺利，那么这时候的 MOTD 信息就会变成：

```
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-83-generic x86_64)

 * Hostname   : haoxi911.me
 * Disk Space : 2.7G / 25G
 * CPU Usage  : 0.00, 0.00, 0.00 (1, 5, 15 min)
 * Memory     : 203M / 992M

Last login: Tue Dec 12 09:28:14 2017 from 5.6.7.8
```

## 二、使用 Ubuntu 官方的 MOTD 脚本

Ubuntu 官方提供了一个名为 Landscape 的服务，这里我们不需要购买这个服务，只需要安装一个与其相关的软件包：

```bash
#安装landscape软件包
$ sudo apt install landscape-common

#进入motd脚本的目录，查看landscape生成的脚本
$ cd /etc/update-motd.d/
$ ls
50-landscape-sysinfo ...

#重新登陆，观察新的欢迎信息
$ exit
$ ssh haoxi911
```

可以看到，Landscape 自动生成了一个名为 50-landscape-sysinfo 的 MOTD 脚本。不出意外，系统的欢迎信息将会变成如下内容，是不是很实用？

```
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-83-generic x86_64)

  System information as of Tue Dec 12 09:50:02 UTC 2017

  System load:  0.01               Processes:           119
  Usage of /:   10.8% of 24.55GB   Users logged in:     0
  Memory usage: 36%                IP address for ens3: 1.2.3.4
  Swap usage:   0%

  Graph this data and manage this system at:
    https://landscape.canonical.com/

Last login: Tue Dec 12 09:48:27 2017 from 5.6.7.8
```

<ClientOnly>
  <Comment />
</ClientOnly>