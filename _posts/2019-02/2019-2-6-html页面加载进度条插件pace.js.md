---
layout:     post
title:      html页面加载进度条插件pace.js介绍
subtitle:   让数据加载不再空旷
date:       2019-02-06
author:     Reed
header-img: images/2019-2/IMG_20190130_130021.jpg
catalog: true
tags:
    - Unarchived
---
# 前言
> 网页在加载的过程中，由于网络延迟和数据量的关系，页面加载的时间因人而异，数据量我们可以通过分页来解决，网络延迟便可以增加页面加载进度提示，让用户知道页面页面正在加载而不是卡死了:(.

这里介绍一个轻量级的进度条插件`pace.js`，它自动监控ajax请求，事件循环延迟，文档就绪状态以及页面上的元素以显示进度。

# 使用
`pace.js`的官网在[这里](https://github.hubspot.com/pace/docs/welcome/).

首先在JavaScript引用它的核心js文件。
``` html
<script src="https://raw.githubusercontent.com/HubSpot/pace/v1.0.0/pace.min.js"></script>
```
然后还需要选择一个样式文件！不要忘了这一点，否则页面不会有任何的效果~
![](/images/2019-2/206115642.png)
在[官网](https://github.hubspot.com/pace/docs/welcome/)提供了一些简洁实用的样式，我们点击 COPY TO CLIPBOARD即可复制，然后在本地的一个css样式文件里保存即可。
下一步就是引用这个样式文件
``` html
    <link href="../static/assets/css/pace.style.css"  rel="stylesheet">
```

自此,便可以自动检测页面的请求信息并显示进度条了。

### 额外的配置
我的网站用到了异步加载页面和数据，因此做了一些额外的配置。（需要在引入pace.js文件前写入）
``` html
<script>
  window.paceOptions = {
    ajax: true,
    eventLag: false,
    elements: false
  }
</script>
```