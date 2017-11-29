---
title: APP扫描二维码登陆流程
tags:
  - 二维码
  - 登陆
  - APP
  - 流程
categories: Java相关
permalink: ap-scan-two-dimensional-code-landing-proces
date: 2017-11-14 20:27:57
---
![image](https://dmrsme.toolbox.xin/20171114/APP%E6%89%AB%E6%8F%8F%E4%BA%8C%E7%BB%B4%E7%A0%81%E7%99%BB%E9%99%86.png)
#### 流程
1. 客户端页面请求二维码(带有客户端标识和操作标识),这个二维码实际上来自安全中心的后台生成.(二维码状态:`待扫描`),此时信息需要放在`redis`里面保存起来
2. 客户端页面获取到二维码之后,轮询客户端后台,查看这个标识的状态.
3. 安全中心APP(已登录)扫描客户端前台生成的二维码,然后将识别出来的信息获取后访问安全中心后台,安全中心后台将当前APP用户的唯一标示和当前的二维码信息绑定,回调客户端后台.(二维码状态:`已扫描`)
4. 安全中心APP上操作是否确认,然后改变当前二维码标示(二维码状态:`已同意`或`已拒绝`)
5. 客户端前台轮询到`已扫描`可以提示用户确认,轮询到`已同意`可直接系统自动登录(通过绑定的用户唯一标识获取到用户加密的密码,走正常的登录流程,).成功之后注意`清空密码信息`.


#### 二维码生成工具
```xml
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>core</artifactId>
    <version>3.0.0</version>
</dependency>
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>javase</artifactId>
    <version>3.0.0</version>
</dependency>
```

#### 图片由[ProcessOn](https://www.processon.com/)生成

