---
layout: post
title:  "Zuul上传文件，中文文件名乱码解决办法"
categories: SpringCloud
tags:  SpringCloud zuul
---

* content
{:toc}

## 问题描述

在项目中又一个上传文件的oss服务，直接调用服务的上传文件成功，如果经过网关zuul服务，上传中文名字的文件，文件名会出现乱码，最终导致上传失败，如果上传英文名字的文件，没有任何问题。怀疑网关zuul对中文做编码处理。

 <!--more-->

## 解决问题的过程

这个问题出现之后，我个人的解决办法如下：

* 第一反应是看文档，文档地址：http://cloud.spring.io/spring-cloud-static/Dalston.SR1/#_uploading_files_through_zuul

* 粗略地看了下文档，以为没有给出解决方案（其实已经给出，只是没有理解好文档）。狂撸源码，依然没有找到解决办法。

* Google搜，搜到了这条Issue，https://github.com/spring-cloud/spring-cloud-netflix/issues/1385

这位大神给出的解决办法，使用zuul servlet去上传文件，而不是默认的spring mvc。使用 zuul servlet之需要在请求uri，前面加上"/zuul"即可。

![image.png](http://upload-images.jianshu.io/upload_images/2279594-e1eeda790fa3fe15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 解决方案

首先列出我的zuul服务的配置：

```
server.port: 5000
zuul:
   routes:
      oss-api:
      path: /oss/**
      serviceId: oss-service
```
oss服务上传文件的接口，代码如下：

```
@RestController
@RequestMapping("/file")
public class FileUploadController {
    @PostMapping("/upload")  
    public RespDTO handleFileUpload(@RequestParam("file") MultipartFile file) {
        //上传代码省略
        return RespDTO.onSuc(upLoadResult);
    }
```

那么，经过网关，调用上传文件的url地址如下：


>localhost:5000/oss/file/upload

这时如果出现中文文件名，上传文件的文件名会出现失败。按照上述大神的办法，直接在这个uri，前面加上"/zuul"，那么请求地址如下：

> localhost:5000/zuul/oss/file/upload

测试一下，果然通过，上传中文名的文件乱码问题解决。