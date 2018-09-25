---
layout: post
title:  "我说如何根据豆瓣api来理解Restful API设计的"
categories: 微服务
tags:  架构
---

* content
{:toc}

### 1.什么是REST
REST全称是Representational State Transfer,表述状态转移的意思。它是在Roy Fielding博士论文首次提出。REST本身没有创造新的技术、组件或服务，它的理念就是在现有的技术之上，更好的使用现有的 web规范。用REST规范的web服务器，能够更好的展现资源，客户端能够更好的使用资源。每个资源都由URI/ID标识。REST本身跟http无关，但是目前http是与它相关的唯一实例。REST有着优雅、简洁的特性，本文是根据豆瓣api来谈谈自己对restful的一些理解。


<!--more-->

### 2.URI规范

* URI(Uniform Resource Identifiers) 统一资源标示符
* URL(Uniform Resource Locator) 统一资源定位符
 
URI 的格式：
```

URI的格式定义如下：  
URI = scheme "://" authority "/" path [ "?" query ] [ "#" fragment ]  


```
* uri代表的是一种资源，要做到优雅、简洁。
* 最好在api地址标明版本

比如 

```
https://api.douban.com/v2
```

* 关于分隔符“／”，比如：

```
"/"分隔符一般用来对资源层级的划分,比如：

https://api.douban.com/v2/book/1220562

表述了豆瓣api，version2下的图书仓库下的编号为1220562的图书。

```
* URI尽量使用“-”代替下划线“_“。
* URI统一使用小写字母
* URI不包含文件扩展名
* 使用？用来过滤资源，比如？limit=10 :指定返回10条记录。
* 不使用无意义的字符串、数字，要做到简洁。


### 3.正确使用method

* get -只用做资源的读取。
* post-通过用作创建一个新的资源。
* delete-通过用作资源的删除。
* put -通过用作更新资源或者创建资源
* head-只获取某个资源的头部信息。

比如 [豆瓣图书api](https://developers.douban.com/wiki/?title=book_v2)：
 
  name |  method  | api
  --------- | --------- | --------
 获取图书信息 | get | /v2/book/:id
 用户收藏某本图书 | post |/v2/book/:id/collection
 用户修改对某本图书的收藏  | put |/v2/book/:id/collection
 用户删除对某个图书的收藏 | delete | /v2/book/:id/collection
 

 
 * 另外，在一些不符合curd的情况下，使用　post。
 * 把动作转换成资源
 
 比如，上述接口中，用户收藏某本书对外暴露的接口是"/v2/book/:id/collection",收藏动作通过post方法来展现，而不直接写着api中，collection "收藏"，名次，动作直接转换成了资源。
 
###4.选择合适的状态码

http请求需要返回状态码，约定俗成的状态码能够帮助开发团队提高沟通效率。

* 2xx: 请求正常处理并返回
* 3xx: 重定向
* 4xx: 客户端请求有错误
* 5xx: 服务端请求有错误

比如豆瓣api返回的状态码说明：
 
状态码|  含义  | 说明
  --------- | --------- | --------
  200 | ok | 请求成功
  201 | created | 创建成功
  202 | accepted | 更新成功
  400 | bad request | 请求不存在
  401 | unauthorized | 未授权
  403 | forbidden | 禁止访问
  404 | not found | 资源不存在
  500 | internal server error | 内部错误
 
### 5.使用通用的错误码
 
  通用错误码，具体产品由具体产品api给出。比如豆瓣api:
  
 错误码 |  错误信息  | 含义
  --------- | --------- | --------
  999 | unknow_v2_error | 未知错误
  1000 | need_permission | 需要权限
  1001 | uri_not_found | 资源不存在 
  ...  | .... |...
  
  太多了，只列出几条，具体见豆瓣 api。
  
### 6. 安全

这部分内容不属于这篇文章，但是稍微说明下：

* 使用https 
* 使用jwt验证
* 使用参数签名，防止参数被篡改。
* 使用权限验证，shiro ,或者自己建数据库（用户、角色、权限）  


### 7.api文档

接口文档的编写至关重要，最好是写一个在线接口文档。接口文档能够方便团队查阅，减少不必要的沟通。如果对外公开api，api文档的质量直接反应了一个公司的技术水平，甚至一个公司的文化气质。


### 8.参考资料

本文参考了以下的资料:

[豆瓣api](https://developers.douban.com/wiki/?title=book_v2)

[理解restful架构](http://mccxj.github.io/blog/20130530_introduce-to-rest.html)

[restful introduction](https://www.tutorialspoint.com/restful/restful_introduction.htm)

[跟着github学习restful api设计](http://cizixs.com/2016/12/12/restful-api-design-guide)

[REST接口设计规范](http://wangwei.info/about-rest-api/)

[restful api 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)