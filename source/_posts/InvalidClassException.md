---
title: InvalidClassException
permalink: invalidclasexception
tags:
  - Java
  - Spring Boot
  - Exception
categories:
  - Java相关
date: 2017-10-31 21:53:44
---
### InvalidClassException
#### 背景
项目使用的`SpringBoot`架构,使用的是默认的`Redis`存储登陆信息
#### 过程
添加了一个有关于`UsUser`的接口,需要把当前登录信息返回给前端。
#### 结果
访问时`InvalidClassException`异常
```java
org.springframework.data.redis.serializer.SerializationException: Cannot deserialize; nested exception is org.springframework.core.serializer.support.SerializationFailedException: Failed to deserialize payload. Is the byte array a result of corresponding serialization for DefaultDeserializer?; nested exception is java.io.InvalidClassException: com.usc.core.model.UsUser; local class incompatible: stream classdesc serialVersionUID = -4391106051528831723, local class serialVersionUID = -3047559744731102659
	at org.springframework.data.redis.serializer.JdkSerializationRedisSerializer.deserialize(JdkSerializationRedisSerializer.java:82)
```
#### 思考&总结
看到这个的第一反应是有人修改了`serialVersionUID`,然后发现这个类并没有这个字段.然后联想到这个信息是从`redis`获取的然后清理了`redis`中的登陆信息,异常不再出现.最后注意到这个
`org.springframework.data.redis.serializer.JdkSerializationRedisSerializer.deserialize`,已证明联想正确,出现问题到解决5分钟不到.
#### 相关知识
`serialVersionUID`适用于`Java`的序列化机制。简单来说，`Java`的序列化机制是通过判断类的`serialVersionUID`来验证版本一致性的。在进行反序列化时，JVM会把传来的字节流中的`serialVersionUID`与本地相应实体类的`serialVersionUID`进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常，即是`InvalidCastException。`