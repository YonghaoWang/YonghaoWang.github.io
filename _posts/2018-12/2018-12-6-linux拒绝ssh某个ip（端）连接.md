---
layout:     post
title:      linux修改监听端口并拒绝ssh某个ip（端）连接
subtitle:   linux修改监听端口并设置ssh连接ip白名单（黑名单）
date:       2018-12-06
author:     Yonghao Wang
header-img: images/2018-12/photo_2018-12-06_20-28-35.jpg
catalog: true
tags:
    - Linux
    - Network
---

# 前言

高高兴兴连上自己买的云服务器，发现 诶？怎么会有这样的情况？！

![](/images/2018-12/06201145.png)

有公网ip的机器，你能方便快捷地连接上它，当然互联网的所有人也是同样的能够连接到你的服务器，这很明显就是有人扫描到了你的机器ssh端口，然后使用密码字典来暴力破解，如果密码设置得较为简单，那很可能就被植入了莫名其妙的东西！

# 解决

这个时候呢，我们可以设置一个ip白名单，只允许自己的ip连接上去，有的时候我们的ip不是固定的，所以这里可以扩大一下设置的范围，将白名单设置到一个ip端。

在Linux的`/etc/hosts.allow`文件里控制可以访问本机的IP地址，`/etc/hosts.deny`控制禁止访问本机的IP。如果两个文件的配置有冲突，以`/etc/hosts.deny`为准。

通过该项配置呢，我们可以允许或者拒绝某个ip或者ip段的客户访问linux的某项服务。我们利用这个配置来控制ip端对我们的服务器的`ssh`权限。

首先来编辑`/etc/hosts.allow`这个配置文件
```
vim /etc/hosts.allow
```
打开文件，然后在末尾加上这样的配置，其中的`sshd`表示要控制的服务名，中间是要允许的ip（段），然后`:allow`是允许访问，可以省略。
```
#
# hosts.allow   This file contains access rules which are used to
#               allow or deny connections to network services that
#               either use the tcp_wrappers library or that have been
#               started through a tcp_wrappers-enabled xinetd.
#
#               See 'man 5 hosts_options' and 'man 5 hosts_access'
#               for information on rule syntax.
#               See 'man tcpd' for information on tcp_wrappers
#
sshd:40.2.155.*:allow
```
保存好这个文件之后呢，我们来配置`/etc/hosts.deny`
```
vim /etc/hosts.deny
```
打开文件后，在末尾加入`sshd:all:deny`，如同上方的配置,`:deny`可以省略。
```
#
# hosts.deny    This file contains access rules which are used to
#               deny connections to network services that either use
#               the tcp_wrappers library or that have been
#               started through a tcp_wrappers-enabled xinetd.
#
#               The rules in this file can also be set up in
#               /etc/hosts.allow with a 'deny' option instead.
#
#               See 'man 5 hosts_options' and 'man 5 hosts_access'
#               for information on rule syntax.
#               See 'man tcpd' for information on tcp_wrappers
#
sshd:all
```

# 修改监听端口
为了避免掉一些扫描啊什么的，建议将默认的ssh监听`22`端口给修改一下。
```
vim /etc/ssh/sshd_config
```
打开ssh的配置文件，我们可以看到在前几行就有相关的配置`#Port 22`,这里就是端口的设置了。
```

#       $OpenBSD: sshd_config,v 1.100 2016/08/15 12:32:04 naddy Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/bin:/usr/bin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

# If you want to change the port on a SELinux system, you have to tell
# SELinux about this change.
# semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
#
#Port 22
```
将`#Port 22`行修改为你希望它监听的端口并保存即可，如`Port 23333` :).

# 保存更改

然后我们重启一下sshd服务使得以上的配置生效。
```
service sshd restart
```
这样就可以避免掉很多的安全隐患了（更好的办法是取消密码登陆，使用密钥登陆），自己一定要记住修改了端口，不然某天还以为机器失联了！🤣
