---
layout:     post
title:      mybatis Parameter index out of range 错误
date:       2019-04-20
author:     Reed
header-img: images/2019-4/4-20_10-42-13.jpg
catalog: true
tags:
    - Mybatis
---
# 正文
这天用MyBatis的手写SQL语句查询的时候，涉及到了多表查询，查询语句不应该有问题的（毕竟也是很熟悉基本的SQL了），可是运行的时候确有这个错误：
``` java
ERROR 138920 --- [nio-8080-exec-1] Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is org.mybatis.spring.MyBatisSystemException: nested exception is org.apache.ibatis.type.TypeException: Could not set parameters for mapping: ParameterMapping{property='publishCode', mode=IN, javaType=class java.lang.Object, jdbcType=null, numericScale=null, resultMapId='null', jdbcTypeName='null', expression='null'}. Cause: org.apache.ibatis.type.TypeException: Error setting non null for parameter #2 with JdbcType null . Try setting a different JdbcType for this parameter or a different configuration property. Cause: org.apache.ibatis.type.TypeException: Error setting non null for parameter #2 with JdbcType null . Try setting a different JdbcType for this parameter or a different configuration property. Cause: java.sql.SQLException: Parameter index out of range (2 > number of parameters, which is 1).] with root cause

java.sql.SQLException: Parameter index out of range (2 > number of parameters, which is 1).
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:129)
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:97)
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:89)
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:63)
```
简化后的查询语句是这样的：
``` sql
select userCode
from record
where publishCode = #{publishCode}
group by userCode
```
参数数量错误？但是我在这个地方只用了一个参数，却告诉我需要两个参数...

在搜索引擎查询了一番，大家都说是Like语句取参数的问题，很明显和我的这个是不同情况，直到在一篇博文的留言里发现了问题！

其实我原来的SQL语句是这样的：
``` sql
    select userCode
    from record
    where publishCode = #{publishCode}
    group by userCode
#   SELECT a.amount, c.nickName 
#   FROM record a, users c
#   WHERE a.publishCode = #{publishCode} and c.userCode = a.userCode
```
有什么区别呢？也就多了注释而已，可是注释里面有`#{xxx}`，它会去传值的，我晕😵，删掉注释即可╯︿╰！