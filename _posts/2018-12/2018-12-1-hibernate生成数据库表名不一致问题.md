---
layout:     post
title:      Hibernate生成数据库表名不一致问题
subtitle:   修改Hibernate命名策略
date:       2018-12-01
author:     Reed
header-img: images/2018-12/photo_2018-12-11_12-43-04.jpg
catalog: true
tags:
    - Spring Boot
---

# 正文

在Spring Boot里使用Hibernate自动建表的时候，我们会遇到字段名与数据库的实际名称有所不同的情况，这是因为Hibernate有一个命名策略。

在application.yml文件里可以手动指定一下


1.PhysicalNamingStrategyStandardImpl：不做修改，直接映射 
```
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl

```
2.SpringPhysicalNamingStrategy:在进行映射时,首字母小写，大写字母变为下划线加小写
```
spring.jpa.hibernate.naming.physical-strategy=org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
```
