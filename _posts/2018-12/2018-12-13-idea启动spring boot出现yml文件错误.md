---
layout:     post
title:      idea启动spring boot出现yml文件错误
subtitle:   修改hibernate命名策略
date:       2018-12-13
author:     Reed
header-img: images/2018-12/photo_2018-12-11_12-43-07.jpg
catalog: true
tags:
    - Spring Boot
---

# 解决

出现这个问题呢，是因为spring boot默认的`application.propertities`文件编码是GBK，我们在idea的右下角可以看到。

![](/images/2018-12/216113853.png)

所以我们需要将它修改为`UTF-8`即可，点击idea右下角的`GBK`，提供了多种选择，这里我们点击`UTF-8`。

如果文件里包含中文，会出现这样一个提示。

![](/images/2018-12/81216114119.png)

我们点击Convert，将中文字符转换为`UTF-8`编码就好了。

然后启动项目，成功运行。