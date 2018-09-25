---
layout: post
title:  "JWT简介"
categories: 工具
tags:  consule
---

* content
{:toc}


去官网下载：https://www.consul.io/downloads.html
解压：
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2279594-24b004549def507f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

<!--more-->

设置环境变量：
```
计算机  右键  属性 高级属性设置环境变量设置

在path下加上：E:\programfiles\consul；
```
cmd启动：
>consul agent -dev

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2279594-6379a5174dafb3f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
可以看到启动成功。

打开网址：http://localhost:8500  ，可以看到界面，相关服务发现的界面。

### 优秀文章推荐：
* [史上最简单的 SpringCloud 教程 | 终章](http://blog.csdn.net/forezp/article/details/70148833)
* [史上最简单的 SpringCloud 教程 | 第一篇: 服务的注册与发现（Eureka）](http://blog.csdn.net/forezp/article/details/69696915)
* [史上最简单的SpringCloud教程 | 第七篇: 高可用的分布式配置中心(Spring Cloud Config)](http://blog.csdn.net/forezp/article/details/70037513)