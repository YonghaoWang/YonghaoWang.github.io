---
layout:     post
title:      记录一次Redis被getshell
date:       2019-11-12
author:     Reed
# header-img: images/2019-6/photo_2019-06-13_22-09-46.jpg
catalog: true
tags:
    - Redis
---
# 前言
最近新买了个服务器（ubuntu18系统），打算作为一个项目的生产环境，刚刚装好各种环境、软件，当然作为缓存使用的Redis也在其中。

# 无法登录
为了方便和安全，服务器我一般是采用私钥登陆的，这台服务器也不例外。就在下午的时候，本想去瞅瞅花重金购买的服务器☺，老样子使用`ssh`命令连接，纳尼？居然让我输入密码！这个不应该啊，清楚的记得自己设置了密钥的，不可能让我直接输密码了。
# 排查问题
那只好先输入密码，登入进去看看到底是什么情况！

首先看看保存私钥的文件出了什么问题：
``` shell
vim authorized_keys
```
内容为
```
REDIS0008�	redis-ver4.0.9�
redis-bits�@�ctime���]�used-mem�
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDCMfLE4cEpVRQ2tBuuSCEQftjYUqJMBJ25XvlFYNKbMRfByJdZ8R362Hk2UFXacicQABc4hF0DFIrq8xxm3YwNrujW+ES1HnLW9QbCEF5BjZ8AXE+sJ3MaUSQdg3BKGEz+jCn5tpFodWbYwJ7rB5cWJctMRKcp2hfI5Uc66/i/Q72y1TDvoVDi/FkQHZw6B9Ulq+ZkN3H4NL29M6S0PZGfQeaj3FqqpB8LXWmBCbVtWaVqxNTlNTTuOw9OSrLIAAn/gpdzpDuuXXj4rZSVAHd9A0KbDeJt+lwr0D2l07E9xzqQbMTzFSpvGD4dZMakIfFArn994NNvd/yuuQ+M81sZ m@m-virtual-machine
#��Ғ��"
```
😨这一看就是服务器被攻击了，私钥后面的名字也不是我自己的，私钥被替换成别人的了。

然后第一行的内容是`redis`相关的，回忆了一下，中午的时候为了方便在本机调试某个功能，将redis服务的6379端口无密码地暴露在了公网，并未做任何限制，应该就是这个地方成了入口。

Google搜索，果然如此！
>因为redis有保存备份的功能，将备份文件目录改成任意地址，把备份文件名改成任意我们想要的，这时我们只要向redis插入一个键值对（包含我们想要写的内容），就可以通过redis向任意文件夹写文件，比如向web目录写webshell、向~/.ssh/目录写入sshkey，或者写crontab，反弹shell。

这里有提到`写sshkey`、`写crontab`，目前发现sshkey已经被写入了，那么crontab应该也不会幸免。

打开crontab查看，果然增加了几条记录，并且cron.d文件夹下多了一些奇怪的脚本。

这个时候知道情况已经很严重了，使用如下命令来看看在最近的三个小时到底有多少文件遭到了修改呢：
``` shell
find / -type f -mmin -180
```
一看吓一跳，屏幕不停输出的文件让我明白机器完全被攻破了~😨

# 解决
幸好我的服务器是新装的系统，里面未包含任何资料、文件、代码等，所以第一时间将机器直接重装操作系统了。

然后在安全组限制了端口和ip白名单，重新安装的各种软件也不再采用默认的弱口令或者无密码（🙃）
# 总结
造成这次事故的主要原因还是自己低估了公网环境的危险性，安全意识还不够，将常见的redis软件的默认`6379`端口给直接开放了（这岂不是敞开大门欢迎木马~）。还好这次攻击有较大的缺陷，直接把我原有的sshkey都给删除了，让我能够立马发现问题，未造成严重的后果。

今后在公网环境中的机器，一定要在安全组（云服务器）或者用`ufw`限制好端口与ip的访问权限，切忌`我家大门常打开`。

需要注意的有这几点
- 在安全组限制IP端口访问，设置白名单。
- 应用不要设置为弱口令或者无密码，如MySQL用`root`作为密码，或Redis直接无密码。
- 可将应用的端口修改为其他不常见的端口值，如ssh一般为`22`端口，公网环境下会有肉鸡不停的扫描`22`端口，因此可修改为如`2333`等端口。