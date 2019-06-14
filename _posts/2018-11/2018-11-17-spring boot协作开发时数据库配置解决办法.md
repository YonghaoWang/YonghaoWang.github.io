---
layout:     post
title:      Spring Boot协作开发时数据库配置解决办法
# subtitle:   
date:       2018-11-17
author:     Reed
header-img: images/2018-11/698584157B008F2FAFA87CB64119ABF5.jpg
catalog: true
tags:
    - Spring Boot
---

# 前言
Spring Boot项目在大家共同开发的时候，如果每个人的数据库配置都不一样的话，提交代码更改时得万分小心才行，否则等着被锤吧O(∩_∩)O。

# 操作
其实Spring Boot查找配置文件的时候呢，有一个顺序，在前面的优先级更高。
```
file:./config/
file:./
classpath:/config/
classpath:/
```
一般我们的配置文件呢是放在`classpath:/`这个地方的，也就是优先级最低一个位置。因此只需要在这之前符合要求的任一位置放置配置文件，将你个人与团队不同的的配置写进去即可。

比如我在`classpath:/config/`目录下建立了一个`application.yml`文件，在里面写入了数据库的配置信息。
```
spring:
#   在这里设置自己的数据库连接信息
    datasource:
        password: root
        username: 123456
```
然后在`.gitignore`里面加入这个文件的相对路径，Git在提交的时候将会忽略掉它。
