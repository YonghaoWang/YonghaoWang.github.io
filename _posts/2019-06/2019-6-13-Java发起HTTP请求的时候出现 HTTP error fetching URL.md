---
layout:     post
title:      Java发起HTTP请求的时候出现 HTTP error fetching URL. Status=400
date:       2019-06-13
author:     Reed
header-img: images/2019-6/photo_2019-06-13_22-09-46.jpg
catalog: true
tags:
    - Java
---
> 这么久都没更新文章了，最近又是考试又是比赛，尤其是比赛全搞糟了（这个得好好总结一下，抽时间写篇文章出来）。
> 又老是觉得自己平时做的事情乏善可陈，甚至有点怀疑自己了，至于花时间写到这里来嘛~
> 暑假要到来了，我也要开始暑期实习了，这亦是一个契机，调整好自己的状态。


# 出现问题
昨天晚上做一个将IP地址解析为地理位置的功能，我的选择是调用[ipstack](https://ipstack.com)的接口，注册免费账户即可拥有每月10,000次的请求次数，可以将请求的IP对应的地理位置存下来，这样就只需要有新的IP地址时才需要调用接口，所以这个数量对我来说也够用了。

问题来了，我请求的地址是`"http://api.ipstack.com/" + ip + "?access_key=xxxxxxxxx&format=1&language=zh"`，可是没有一次成功过，一直报出这样的错误：
``` java
HTTP error fetching URL. Status=400
```
# 尝试的方案
### 增加请求头
以前在做爬虫的时候遇到过这样的情况，服务端会对请求头做一次检验，过滤掉明显的非人为请求。但是我想这个api不就是给我们的程序调用的吗，不应该这样为难我们的程序吧😭，不多说，先试试就知道了。
我在请求头里增加了这些信息，仍然出现上述错误。
``` Java
    Map<String, String> header = new HashMap<>();
    header.put("Host", "http://api.ipstack.com");
    header.put("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:67.0) Gecko/20100101 Firefox/67.0");
    header.put("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8");
    header.put("Accept-Language", "en,zh-CN;q=0.8,zh;q=0.7,zh-TW;q=0.5,zh-HK;q=0.3,en-US;q=0.2");
```
### 代理？
考虑到我使用了某sr代理软件，我将代理关闭，重启开发工具，仍然出现这样的错误。
于是我考虑切换一个语言试试？ 于是用JavaScript代码测试了一下，同样的环境用JS可以成功而Java不能，这就可以排除我电脑主机的问题了。
### 检查服务器
这样会不会是对方服务器的问题呢，于是我先ping了一下测试机器连通性：
``` PowerShell
ping api.ipstack.com
正在 Ping apilayer.net [23.246.243.35] 具有 32 字节的数据:
来自 23.246.243.35 的回复: 字节=32 时间=291ms TTL=47
来自 23.246.243.35 的回复: 字节=32 时间=210ms TTL=47
来自 23.246.243.35 的回复: 字节=32 时间=234ms TTL=47
来自 23.246.243.35 的回复: 字节=32 时间=254ms TTL=47

23.246.243.35 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 210ms，最长 = 291ms，平均 = 247ms
```
嗯~~~虽然慢了些，但是联通还是没有问题的。
这个时候就很抓狂了，睡意袭来却还解决不了这个奇怪的bug，最后突然想了下，会不会是GFW的功劳呢(ง •_•)ง。
查看了一下这个域名可以解析出来的ip地址列表，嗯，还是有好几个呢，那我直接暴力的把域名换成ip地址去发起请求呢(●ˇ∀ˇ●)？
``` PowerShell
nslookup api.ipstack.com
非权威应答:
服务器:  UnKnown
Address:  200.200.200.200

DNS request timed out.
    timeout was 2 seconds.
名称:    apilayer.net
Addresses:  158.85.167.221
	  23.246.243.50
	  198.23.101.146
	  23.246.243.35
Aliases:  api.ipstack.com
```
终于成功发出请求并获取相应了😑，看来是这个域名在解析为ip地址的这个过程中出现了问题了！
![](/images/2019-6/0613224931.png)

# 突然又好了
昨晚弄好了之后我就去睡了，今天整理代码的时候，想想直接写死ip地址也不怎么可取，再试试用域名访问呢？

这个就奇怪了，今天无论怎么换着花样测都100%成功，没有出现任何问题~

> 遇到这个问题今天一直未能复现出来，先记录在这里。