---
layout:     post
title:      Spring Boot Admin配置
date:       2019-03-11
author:     Reed
header-img: images/2019-3/IMG_20190201_152609 1.jpg
catalog: true
tags:
    - Spring Boot
---
# 前言
Spring Boot Admin是基于Spring Boot Actuator扩展的一款可视化应用监控框架，分为`server`和`client`端，需要被监控的项目属于客户端，这样当有多个不同的项目需要管理时，它们都去服务端注册自身，在Spring Boot Admin的管理界面可以统一查看。同时可以根据服务发生的变化进行消息通知，发邮件等等。

需要注意的是，这里采用的为`2.1.3`版本，与`1.x`旧版本的配置有较大的出入。
# 配置服务端
先来引入所需要的jar包

`build.gradle`
``` gradle
  implementation 'org.springframework.boot:spring-boot-starter-web'
  providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  // https://mvnrepository.com/artifact/de.codecentric/spring-boot-admin-server
  compile group: 'de.codecentric', name: 'spring-boot-admin-server', version: '2.1.3'
// https://mvnrepository.com/artifact/de.codecentric/spring-boot-admin-server-ui
  compile group: 'de.codecentric', name: 'spring-boot-admin-server-ui', version: '2.1.3'
  // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-mail
  compile group: 'org.springframework.boot', name: 'spring-boot-starter-mail', version: '2.1.3.RELEASE'
```
极少的配置便ok了，我这里设置了给自己发送邮件的一些信息。如果你要配置邮件的话，需要注意的是:`spring.mail.host.username字段需要与spring.boot.admin.notify.mail.from相同!`，并且都需要用`" "`双引号包裹着。

`application.yaml`
``` yaml
server:
  port: 8081
spring:
  mail:
    host: "smtp.163.com"
    username: "xxx@163.com"
    password: "xxx"
    properties:
      mail:
        default-encoding: UTF-8
        smtp:
          auth: true
          starttls:
            enable: true
            required: true
    protocol: "smtp"
  boot:
    admin:
      notify:
        mail:
          from: "xxx@163.com"
          to: "user@example.com"
```
然后在启动类加上`@EnableAdminServer`注解
``` java
@EnableAdminServer
@SpringBootApplication
public class SpringAdminApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringAdminApplication.class, args);
    }

}
```
现在启动项目，浏览器窗口打开`http://127.0.0.1:8081`即可看到这样的界面，正在检索应用，说明服务端启动成功了。

![](/images/2019-3/2203547.png)

# 配置客户端
配置`build.gradle`
``` gradle
dependencies {
    implementation('org.springframework.boot:spring-boot-starter-web')
    providedRuntime('org.springframework.boot:spring-boot-starter-tomcat')
    testImplementation('org.springframework.boot:spring-boot-starter-test')
    
// https://mvnrepository.com/artifact/de.codecentric/spring-boot-admin-starter-client
    compile group: 'de.codecentric', name: 'spring-boot-admin-starter-client', version: '2.1.3'
}
```
然后来到yaml文件进行配置，其中的`client.url`指的便是服务端的地址，`exposure.include`指的是对外暴露的信息有哪些，我这里表示全选，记得`*`需要用双引号`" "`包裹，否则启动项目的时候会因为无法读取配置而启动失败。
``` yaml
spring:
#    //管理页面
  boot:
    admin:
      client:
        url: http://127.0.0.1:8081

        #如果两个项目不在一个内网中（似乎且未使用内置tomcat容器时，暂未尝试），则需要配置下边的字段，使得服务端了解如何寻找到客户端。
        instance:
          service-base-url: http://example.com
server:
  port: 8080
#spring admin管理
management:
  endpoints:
    web:
      exposure:
        include: "*"
```
> 若客户端项目有安全机制，则需要放行`/actuator/**`，否则服务端无法反向获取项目的信息。

# 完成
启动客户端，这时切换到服务端的网页可以看到发生了这样的变化：

![](/images/2019-3/2205027.png)
已经检测到了项目的信息，然后点开查看详细且全面的信息（这里可以看到堆内存最大可以是`6.4GB`，tomcat默认为约为主机内存的`1/4`。（刚换了24G内存，嘿嘿😋））：

![](/images/2019-3/2205139.png)

当我关掉客户端后，我的邮箱收到了这样一条通知邮件（看起来很突兀~）告诉我服务已经关闭了，进行更多的配置后我们便可以及时的收到关于项目的事件报告：

![](/images/2019-3/2205505.png)


tips: 服务端不时会报错，该项目似乎还有一些小小的bug。

更多详细信息请参考 [官方文档](http://codecentric.github.io/spring-boot-admin/2.1.3/#getting-started)