---
layout:     post
title:      spring_boot中引用自定义字段
subtitle:    在使用spring boot进行开发的时候会使用到自定义的一些字段
date:       2018-11-14
author:     Yonghao Wang
header-img: images/2018-11/DSC_0972.JPG
catalog: true
tags:
    - Spring boot
---
# 前言

 在spring boot开发的时候会使用到自定义的一些字段，这个时候我们就可以使用spring boot提供的` @Value `注解来达到这一目的。

# 正文
首先在` application.yml `文件里设置好我们需要用到的字段
```
jwt:
  authKey: theKey 
```
当然，如果配置文件是` application.properties `的话，应该像这样
```
jwt.authKey = theKey 
```
把需要的字段配置好了之后呢，我们就在代码里来使用它。
在需要使用到它的类里面使用` @Value("${your_propertie_name}") `注解增加到变量声明前即可使用。
```
public class AuthUtils {

    @Value("${jwt.authKey}")
    private String authKey;


    public void findUser() {
        String Key = authKey; //直接使用名字即可调用
        ···
    }
}
```

但是呢，有时候我们的方法是` static `修饰的，这个时候按照上图的方法将会发现无法正确赋值了，
我们可以按照下边的方法来操作。  

首先记得在类名前使用` @Component ` 注解，然后使用` setAuthKey `方法来将我们需要的字段的值给传递进来。
```
@Component
public class AuthUtils {

    private static String authKey;

    @Value("${jwt.authKey}")
    public void setAuthKey(String authKey) {
        AuthUtils.authKey = authKey;

    }   
     public void findUser() {
        String Key = AuthUtils.authKey; 
        ···
    }
}
```

# 结语
根据实际情况使用上述的方法，我们就能顺利的调用到application文件里自定义的字段了。