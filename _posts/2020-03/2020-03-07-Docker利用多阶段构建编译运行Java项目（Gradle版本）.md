---
layout:     post
title:      Docker利用多阶段构建编译运行Java项目（Gradle版本）
date:       2020-03-07
author:     Reed
# header-img: images/2019-6/photo_2019-06-13_22-09-46.jpg
catalog: true
tags:
    - Docker
    - Spring Boot
---
# 前言
在网上看到的大多数教程，都是在Spring Boot的build.gradle里通过配置Docker插件的方式来制作镜像，这时就需要本地有Docker环境支撑，或者是希望在服务器获得新的代码时，自动编译然后制作镜像然后做一系列操作；似乎我们需要一种新的方式来制作镜像了。

利用Docker的多阶段构建，将Gradle环境用来编译、打包为Jar包，然后在Java环境里运行获得的Jar包。在快速的从源代码获得可执行镜像的同时，并未包含编译环境，最大程度减小了镜像的体积。
# 配置Gradle镜像源
当然Gradle这边已经有提供各个版本的Docker镜像了，我这边使用的是基于`alpine`系统的镜像，体积相比其它系统是最小的。

> 由于国内的网络情况，需要使用到国内的镜像源方能快速访问maven仓库，这点基本上都有修改，其实插件需要另外配置镜像源，与Jar包仓库是不同的。

`build.gradle`配置仓库镜像源，这里采用阿里云。

``` groovy
repositories {
    maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
    mavenLocal()
}
```
`setting.gradle`配置插件镜像源，注意是`setting.gradle`文件！
``` groovy
pluginManagement {
    repositories {
        maven {
            url 'https://maven.aliyun.com/repository/gradle-plugin'
        }
        gradlePluginPortal()
    }
}
rootProject.name = 'statistics'
```
这样配置之后可保证在容器内可以高速下载依赖文件，当然配置自己的缓存仓库是更好的。

# 编写Dockerfile
如下为Dockerfile内容，在构建阶段我们用`FROM` `as`关键字标记，记住该阶段最后一个命令不能为`CMD`，否则将会被忽略，这里的`RUN`阶段我们编译生成了Jar文件。

在运行环境阶段，用`COPY --from=builder`命令复制之前阶段的文件，并对文件进行重命名，然后设置一下时区，最后运行Jar文件。

这样制作的镜像里并不包含构建环境，我们只使用了其构建产物。
``` Docker
FROM gradle:5.3-jdk-alpine as builder
WORKDIR /home/gradle/src
COPY --chown=gradle:gradle . /home/gradle/src
RUN gradle build -x test bootJar

FROM openjdk:8-jdk-alpine as prod
WORKDIR /app
COPY --from=builder /home/gradle/src/build/libs/*.jar /app/statistics.jar
RUN echo 'Asia/Shanghai' >/etc/timezone
EXPOSE 8080
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app/statistics.jar"]
```