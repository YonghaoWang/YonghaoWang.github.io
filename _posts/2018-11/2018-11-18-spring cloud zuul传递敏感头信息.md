---
layout:     post
title:      spring cloud zuul传递敏感头信息
subtitle:   提供给下游服务请求头信息
date:       2018-11-18
author:     Yonghao Wang
header-img: images/2018-11/cold.jpg
catalog: true
tags:
    - Spring cloud
---

# 前言
今天我在微服务gateway后的服务需要用到token里的一些用户信息，发现zuul转发后的`HttpServletRequest`为空，不能拿到数据。
# 解决
按照通常的办法，先去Google找了一下，原来是zuul在转发请求的时候，默认把请求里的header给清空了，于是后边的服务便拿不到相关的信息。

我们可以在gateway的yml文件里配置一下，添加`sensitive-headers`，让它把header转发即可。
```
zuul:
  routes:
    user-service:
      path: /user-service/**
      serviceId: user-service
      sensitive-headers: #默认是转发Cookie,Set-Cookie,Authorization，如只需要其中一个，将名称填写进去即可。
```
这样操作是配置了具体的某一个服务，如果我们希望它在所有的服务转发时都带上敏感信息，那么可以把配置写成这样：
```
zuul:
  routes:
    user-service:
      path: /user-service/**
      serviceId: user-service
      sensitive-headers:
    pass-service:
      path: /pass-service/**
      serviceId: pass-service
  sensitive-headers:
```
# 结束
配置完成后，重启一下网关服务，这时就可以看到后边的服务里的请求已经有了相应的信息。
![](/images/2018-11/TIM20181118134918.png)