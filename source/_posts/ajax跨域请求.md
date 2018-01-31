---
title: Spring boot ajax跨域请求，页面和java服务端的写法
date: 2017-05-05 14:50:23
tags: springBoot
---


```java
public CorsConfiguration buildConfig() { 
    CorsConfiguration corsConfiguration = new CorsConfiguration(); 
    corsConfiguration.addAllowedOrigin("*"); 
    corsConfiguration.addAllowedHeader("*"); 
    corsConfiguration.addAllowedMethod("*"); 
    return corsConfiguration; 
}

// 跨域过滤器 
@Bean 
public CorsFilter corsFilter() { 
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource(); 
    source.registerCorsConfiguration("/**", buildConfig()); // 4 
    return new CorsFilter(source);
} 
```