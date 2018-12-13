---
layout:     post
title:      spring cloud zuul设置跨域
subtitle:   Spring cloud允许前端跨域访问
date:       2018-11-25
author:     Reed
header-img: images/2018-11/02A5A000BA2E4646CB35450A2E5B119E.jpg
catalog: true
tags:
    - Spring cloud
---

> 待更

# 前言
在微服务中使用spring cloud的一系列框架进行开发的时候，一般是前后端分离，然后采用网关作为连接前端与后端的桥梁。我在使用zuul做请求转发的时候遇到了跨域访问的问题，因此需要在网关配置一下zuul的规则。

# 内容
我们在网关启动类处添加这样的方法即可：
``` java
    public FilterRegistrationBean corsFilter() {

        final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        final CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true); // 允许cookies跨域
        config.addAllowedOrigin("*");// #允许向该服务器提交请求的URI，*表示全部允许，在SpringMVC中，如果设成*，会自动转成当前请求头中的Origin
        config.addAllowedHeader("*");// #允许访问的头信息,*表示全部
        config.setMaxAge(1800L);// 预检请求的缓存时间（秒），即在这个时间段里，对于相同的跨域请求不会再预检了
        config.addAllowedMethod("*");// 允许提交请求的方法，*表示全部允许
//       config.addAllowedMethod("HEAD");
//        config.addAllowedMethod("GET");// 允许Get的请求方法
//        config.addAllowedMethod("PUT");
//        config.addAllowedMethod("POST");
//        config.addAllowedMethod("DELETE");
//        config.addAllowedMethod("PATCH");
        source.registerCorsConfiguration("/**", config);
        FilterRegistrationBean bean = new FilterRegistrationBean(new org.springframework.web.filter.CorsFilter(source));
        bean.setOrder(0);
        return bean;
    }
```

