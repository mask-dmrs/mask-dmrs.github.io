---
title: SpringCloud-config client不能连接server
permalink: springcloud-config-client-is-unable-to-conect-to-server
tags:
  - Java
  - Spring Boot
  - Spring Cloud
categories: Java相关
date: 2017-10-30 21:01:24
---
[错误详情](https://github.com/spring-cloud/spring-cloud-config/issues/830)

### config client
### java代码
```java
new SpringApplicationBuilder(Application.class).properties("spring.cloud.config.enabled:true").web(true).run(args);
```
### yml配置
```yml
spring:
  application:
    name: masque
  cloud:
    config:
      uri: http://localhost:8888
      profile: dev
      label: final
      enabled: true
```
### 依赖
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
</dependencies>
```
### 出现的问题
官方示例中不需要配置 `spring.cloud.config.enabled`默认是`true`现在的情况是必须配置这两个`enabled`,否则`client`不会去找`server`  项目正常启动,没有**任何异常**

### 解决办法
在对照了所有的java代码,翻阅了网上的一些博客发现,`client`多了一个依赖`spring-cloud-config-server` 去掉这个就正常了

