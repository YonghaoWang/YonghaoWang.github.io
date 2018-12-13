---
layout:     post
title:      caddy反向代理后页面跳转到内网localhost地址
# subtitle:   
date:       2018-12-04
author:     Reed
header-img: images/2018-12/photo_2018-12-11_12-43-07.jpg
catalog: true
tags:
    - Network
---

# 前言

今天发现前两个天启用了caddy反向代理的项目出了点问题，网站上每次跳转登录页面的时候，url就变成了`127.0.0.1://xxx`，正好是我项目的实际地址，因为之前未启用反向代理的时候没遇到这个情况，所以原因在于反向代理设置有误。

# 解决

查找了一下相关的信息，在官方的`http.proxy`里找到这样一段介绍：
> · transparent
> 
> 大多数后端应用程序期望原始请求中的主机信息。是下边的简写：
> 
> header_upstream Host {host}
> 
> header_upstream X-Real-IP {remote}
> 
> header_upstream X-Forwarded-For {remote}
> 
> header_upstream X-Forwarded-Proto {scheme}

这个`Host`里则是用户在浏览器输入的域名，因此需要将这个信息传递给我们的项目。

我在配置里直接加上了`transparent`，就解决了问题。
```
:80 {
  timeouts none
  proxy /ppp/ 127.0.0.1:8081 {
    transparent
  }
  proxy / 127.0.0.1:8080 {
    transparent
  }
}
```