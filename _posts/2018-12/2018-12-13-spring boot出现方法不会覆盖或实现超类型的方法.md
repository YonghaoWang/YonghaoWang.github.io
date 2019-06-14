---
layout:     post
title:      Spring Boot出现方法不会覆盖或实现超类型的方法
date:       2018-12-13
author:     Reed
header-img: images/2018-12/photo_2018-12-11_12-43-07.jpg
catalog: true
tags:
    - Spring Boot
---

# 前言
一个有趣的错误，记录一下

# 内容
今天在微服务里新增一个Spring Boot项目，心想着反正是别的服务调用，前端也不需要直接访问它，那么我就可以不需要Tomcat了，还可以简化启动。Gradle依赖便是这样的：
``` gradle
dependencies {
    compile(
            'org.springframework.boot:spring-boot-starter-data-jpa',
//            'org.springframework.boot:spring-boot-starter-web',
            'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client',
            'mysql:mysql-connector-java',
            'org.projectlombok:lombok',
            'org.springframework.boot:spring-boot-starter-tomcat'
    )
    
    testImplementation('org.springframework.boot:spring-boot-starter-test')
}
```

看上去很好吧 :)

可是启动的时候发现报错了：
```
Error:(6, 8) java: 无法访问org.springframework.web.WebApplicationInitializer
  找不到org.springframework.web.WebApplicationInitializer的类文件
Error:(8, 5) java: 方法不会覆盖或实现超类型的方法
```
仔细一想，Spring Boot它内置了一个Web容器Tomcat，如果我们取消了它，这个War包就无处安放了。
所以需要另外配置一个Web容器，或者干脆就使用默认配置的吧，简单快捷:)

# 解决
我选择了使用内置的Tomcat，Gradle配置如下：
``` gradle
dependencies {
    compile(
            'org.springframework.boot:spring-boot-starter-data-jpa',
            'org.springframework.boot:spring-boot-starter-web',
            'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client',
            'mysql:mysql-connector-java',
            'org.projectlombok:lombok',
            'org.springframework.boot:spring-boot-starter-tomcat'
    )
    
    testImplementation('org.springframework.boot:spring-boot-starter-test')
}
```
