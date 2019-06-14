---
layout:     post
title:      解决IDEA在application.yml文件没有提示的问题
# subtitle:   修改hibernate命名策略
date:       2018-12-01
author:     Reed
header-img: images/2018-12/DSC_1021111111111.jpg
catalog: true
tags:
    - Spring Boot
---

# 前言
在Spring Boot里，默认的配置文件是`application.properties`，而yml格式文件能更好的展现出配置结构以及简洁的表达。

令人奇怪的是，将`application.properties`文件更改为`application.yml`后，有时候会出现IDEA无法识别为配置文件的情况。同时文件的图标都变了。

![](/images/2018-12/2019.png)

# 解决
图标变了还好，但是没有提示这可不能忍啊！

在网上找了很多解决方法，也一一试过，最终发现了一个完美的解决办法。

File-->Settings...-->Plugins-->

![](/images/2018-12/1152315.png)

然后点击`Browse repositories...`

搜索`spring Assistant`插件，点击安装它，然后重启一下IDEA，我们熟悉的配置提示又出来了。

![](/images/2018-12/01152628.png)