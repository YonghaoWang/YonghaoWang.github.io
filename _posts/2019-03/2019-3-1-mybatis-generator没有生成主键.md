---
layout:     post
title:      mybatis-generator自动创建实体类的时候没有update语句
subtitle:   Cannot obtain primary key information from the database, generated objects may be incomplete
date:       2019-03-01
author:     Reed
header-img: images/2019-3/IMG_20190201_152609 1.jpg
catalog: true
tags:
    - Mybatis
---
# 前言

在使用mybatis-generator自动创建实体类的时候，创建成功了，但是输出了这样的信息:
``` ant
> Task :mbGenerator
[ant:mbgenerator] Cannot obtain primary key information from the database, generated objects may be incomplete
```
并且生成的.xml文件中不包含update和selectByPrimaryKey语句等，通过上边的输出也能知道原因了，正是它没有检测到主键，才会发生这样的情况。
# 解决
这时可以看一下mysql的版本和驱动版本，我连接的mysql是5.7版本的，然而`mybatis-generator.xml`配置文件的一部分是这样的:
``` xml
    <classPathEntry location="G:\mysql-connector-java-8.0.13.jar"/>
    <context id="zhuimengedu" targetRuntime="MyBatis3Simple" defaultModelType="flat">
        <!--自动实现Serializable接口-->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>

        <!-- 去除自动生成的注释 -->
        <commentGenerator>
            <property name="suppressAllComments" value="true" />
        </commentGenerator>

        <!--数据库链接地址账号密码-->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=utf-8&amp;serverTimezone=UTC&amp;useSSL=false&amp;tinyInt1isBit=false"
                        userId="root"
                        password="root">
        </jdbcConnection>
        <!-- ···省略 -->
    </context>
```
由于配置文件里的jar包是mysql较新的版本，而这时的驱动也更换了，所以出现了识别不到主键的情况，这时我们可以选择更换驱动（最快速）或者更新主机的MySQL版本。

#### 修改配置文件
我将jar包依赖和驱动都更换成了旧版本，可以参考下我的配置:
``` xml
    <classPathEntry location="G:\mysql-connector-java-5.1.47.jar"/>
        <!-- ···省略 -->
        <!--数据库链接地址账号密码-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=utf-8&amp;serverTimezone=UTC&amp;useSSL=false&amp;tinyInt1isBit=false"
                        userId="root"
                        password="root">
        </jdbcConnection>
        <!-- ···省略 -->
    </context>

```