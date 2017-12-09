---
title: Async Method Invocation
permalink: async-method-invocation
categories: 设计模式
date: 2017-12-07 22:07:23
tags:
 - Java
 - 中级难度
 -  译文
photos:
 - https://static.zuul.top/java-design-patterns-doc-cn/04_async-method-invocation.png
---

https://github.com/mask-dmrs/java-design-patterns/tree/my-master/04_async-method-invocation

## 意图
异步方法调用是在等待任务结果时调用线程未被阻塞的模式。 
该模式提供多个独立任务的并行处理，并通过回调检索结果，或等待一切完成。 

## 适用性
使用异步方法调用模式时

* 您有多个可以并行运行的独立任务
* 您需要改善一组连续任务的性能
* 您的处理能力或长时间运行的任务数量有限，主任务不应该等待所有的任务完成

## 示例

* [FutureTask](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/FutureTask.html), [CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html) and [ExecutorService](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html) (Java)
* [Task-based Asynchronous Pattern](https://msdn.microsoft.com/en-us/library/hh873175.aspx) (.NET)
