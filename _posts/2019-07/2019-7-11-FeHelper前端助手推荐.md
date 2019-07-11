---
layout:     post
title:      FeHelper前端助手推荐
date:       2019-06-13
author:     Reed
header-img: images/2019-6/photo_2019-06-13_22-09-46.jpg
catalog: true
tags:
    - Unarchived
---
# 前言
最近在postman里面写前端的json请求的时候，遇到了一些问题。

json是严格使用`""`和英文的半角逗号的，然而postman里的json一旦请求参数较多之后，就不是那么方便一眼看出哪里没有写正确了。
例如下面的请求参数，眼尖的同学或许一眼就能看出来有的地方符号有问题，总之我当初大眼瞪小眼瞅了半天才找到。
``` json
{
	"position": "后端开发",
	"name": "唐伯虎",
	"gender": "男"，
	"birthday": "1997-11-11",
	"education": "本科,
	"currentCompany": "暂无",
	"phone": "13312341234",
	"email": "qq@qq.com",
	"source": "其他"，
	"channel": "官网",
	"university": "其它大学",
	"graduatedDate": "2020-06-15"
}
```
在postman里是这个样子的：

![](/images/2019-7/1233701.png)
> 好吧，尴尬了。
> 现在发现postman截图中左侧有一个【×】的标志，这就是告诉我这行有错误,,ԾㅂԾ,,用例如这么久这才注意到~


- 解决这个问题之后我就在想有没有这么一款小工具可以更好的帮助我们呢，突然想到我的浏览器附加组件里似乎有这样一款工具。
# 强行推荐
总之用肉眼看和postman这样的提示并不是那么完善，这个时候有一款浏览器插件可以很好的帮助到我们了。

它就是——FeHelper，All In One的一个工具，包含多个独立小应用，比如：Json工具、代码美化工具、代码压缩、二维码工具、markdown工具、网页定制工具、便签笔记工具、信息加密与解密、随机密码生成、Crontab等等！

在chrome或者Firefox浏览器附加组件均能下载到这款应用，现在看一下它具有的功能（不只是json格式化！）。

<center>
    <img src="/images/2019-7/0711234323.png" width="20%"/>
</center>

- 在上边遇到json串格式问题的时候，可以用到这款工具的JSON串格式化的功能：粘贴进去即会清晰地显示某个地方的格式是不正确的，提醒我们进行改正。
![](/images/2019-7/234643.png)

- 其中诸如字符串编解码的功能，简直是超级方便有木有！

![](/images/2019-7/11234519.png)

- 还有时间戳工具，和时间有关的需求基本上都可以在这里解决了。

![](/images/2019-7/1235417.png)

# 链接直达
在这里提供一下浏览器的插件地址吧，方便直达。

- Firefox附加组件 [FeHelper前端助手](https://addons.mozilla.org/zh-CN/firefox/addon/web%E5%89%8D%E7%AB%AF%E5%8A%A9%E6%89%8B-fehelper/)

- chrome商店 [FeHelper前端助手](https://chrome.google.com/webstore/detail/web%E5%89%8D%E7%AB%AF%E5%8A%A9%E6%89%8Bfehelper/pkgccpejnmalmdinmhkkfafefagiiiad?hl=zh-CN)

感谢作者的辛勤开发，这里是作者博客地址。
- [作者博客](https://www.baidufe.com)