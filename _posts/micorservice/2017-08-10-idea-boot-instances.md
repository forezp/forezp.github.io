 ---
layout: post
title:  "如何在IDEA启动多个Spring Boot工程实例"
categories: SpringCloud
tags:  SpringBoot 
---

* content
{:toc}


在我讲解的案例中，经常一个工程启动多个实例，分别占用不同的端口，有很多读者百思不得其解，在博客上留言，给我发邮件，加我微信询问。所以有必要在博客上记录下，方便读者。

<!--more-->

### step 1
在IDEA上点击Application右边的下三角
,弹出选项后，点击Edit Configuration

![image.png](http://upload-images.jianshu.io/upload_images/2279594-1a2b0236f5162329.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###  step 2

打开配置后，将默认的Single instance only(单实例)的钩去掉。
![ 1](http://upload-images.jianshu.io/upload_images/2279594-79095555afd4e17b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### step 3
通过修改application文件的server.port的端口，启动。多个实例，需要多个端口，分别启动。

