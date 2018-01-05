---
title: 关于RedisTemplate能注入到ValueOperations对象的疑问解惑
tags:
  - redis
  - Resource
  - Autowired
  - Spring
categories: QA
permalink: why-ValueOperations-can-inject-RedisTemplate
date: 2018-01-04 20:02:20
---
## 疑问 
在刚开始接触`redis`的时候发现项目有这样一段看起来有问题的代码
```java
@Resource(name = "redisTemplate")
public void setValOps(ValueOperations<Object, Object> valOps) {
    CacheUtils.valOps = valOps;
}
```
对，这个是注入代码，我们所知道的注入必须类型匹配，于是我翻看了`RedisTemplate`的源码
` extends RedisAccessor implements RedisOperations<K, V>, BeanClassLoaderAware`
`RedisAccessor implements InitializingBean`
`BeanClassLoaderAware extends Aware`
而让我感到惊奇的是这些类和接口和 `RedisOperations` 没有任何的关联?
启动和使用过程都没有任何问题，这个是怎么回事呢？
## 发现
通过查询资料发现`ValueOperationsEditor`这个类,注释`PropertyEditor allowing for easy injection of {@link ValueOperations} from {@link RedisOperations}.` 很明显就是这个类讲两个没有任何关联的类能够注入绑定。 
### 代码如下:
```java
class ValueOperationsEditor extends PropertyEditorSupport {
  public void setValue(Object value) {
	if (value instanceof RedisOperations) {
	  super.setValue(((RedisOperations) value).opsForValue());
	} else {
		throw new java.lang.IllegalArgumentException("Editor supports only conversion of type " + RedisOperations.class);
	}
  }
}
```
`RedisTemplate`实现了`RedisOperations`接口,这段代码意思就是调用`RedisTemplate`的`opsForValue()`方法来给`ValueOperations`注入,事实上这个方法返回类型就是`ValueOperations`.即回答了我们的疑问.

## 答疑
`PropertyEditorSupport`这个核心的类提供了基类支持,就像注释写的一样
 - 这是一个帮助建立属性编辑器的支持类。它既可以用作基类，也可以用作委托。 </br>
 
## 扩展

### Resource
来看看`Resource`注解,搜索下有哪些地方使用了这个注解
  ![image](https://static.zuul.top/dmrs-me/20180104/1.png)

看到了我们熟悉的spring包
![image](https://static.zuul.top/dmrs-me/20180104/2.png)

很明显的是表示注解字段或`setter`方法注入信息的类，支持`@Resource`注释。
### Autowired
相关核心类`AutowiredAnnotationBeanPostProcessor`
![image](https://static.zuul.top/dmrs-me/20180104/3.png)

从代码的注释到方法命名字段命名都值得学习。
### 小窍门
`idea`的`Find Usages`功能(右键)非常强大
![image](https://static.zuul.top/dmrs-me/20180104/4.png)

不仅有注解还有`字段`、`类`、`注释`、`引用`。

  