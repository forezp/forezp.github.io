---
layout: post
title:  "openresty最佳案例案例-汇总"
categories: Openresty 
tags:  Openresty Nginx
---

* content
{:toc}


权限控制在任何的系统中都为一个基本模块，没有权限，就不要谈系统。权限控制的重要性不言而喻。在我所做的Spring Cloud微服务系统，采用的权限控制框架为Spring Cloud Oauth2和Spring Boot Securtiy，这两个框架组合构成了一个强大的权限安全模块。搭建好，设置好，其实是非常简单的。Spring Boot Securtiy是对方法层面的控制，所以要在方法上加注解。随着业务的扩张，注解这种方式无疑给开发人员带来了非常大的工作量，由于开发人员的不规范，甚至连注解都不愿意写了。我在思考能不能废弃掉Spring Cloud Oauth2和Spring Boot Securtiy，废弃掉注解，让开发人员从注解中解放出来。

带着这样的思考，我首先想到的是kong api gateway，kong提供了非常多的插件化的模块，能够满足大部分的业务需求，但不能满足RBAC（基于角色的权限控制）。后来就想自己去实现。实现的过程就是不断学习和探索的过程，我从0基础的lua和openresty，花了三天的时间就实现了本系列文章所写的全部功能。另外花了两个周末把这系列文章整理出来。

之所以说是整理文章，因为很多东西并非我原创，甚至直接拿了很多资料、博客的代码，直接复制粘贴。那我为什么要粘贴，为什么不自己写？一是个人时间精力有限，二是已经有了轮子，为什么要自己造一个？三是，为了保证系列文章的完整性。感谢openresty的作者章亦春、openresty 最佳实践的作者[WenMing](https://github.com/moonbingbing)、[张开涛](http://jinnianshilongnian.iteye.com/)等众多开源者和知名博主。

<!--more-->


## 目录

-  [Openresty最佳案例 | 第1篇：Nginx介绍](http://blog.csdn.net/forezp/article/details/78616591)
-  [Openresty最佳案例 | 第2篇：Lua入门](http://blog.csdn.net/forezp/article/details/78616622)
-  [Openresty最佳案例 | 第3篇：Openresty安装](http://blog.csdn.net/forezp/article/details/78616645)
-  [Openresty最佳案例 | 第4篇：OpenResty常见的api](http://blog.csdn.net/forezp/article/details/78616660)
-  [Openresty最佳案例 | 第5篇：http和c_json模块](http://blog.csdn.net/forezp/article/details/78616672)
-  [Openresty最佳案例 | 第6篇：OpenResty连接Mysql](http://blog.csdn.net/forezp/article/details/78616698)
-  [Openresty最佳案例 | 第7篇：模块开发、OpenResty连接Redis](http://blog.csdn.net/forezp/article/details/78616714)
-  [Openresty最佳案例 | 第8篇：RBAC介绍、sql和redis模块工具类](http://blog.csdn.net/forezp/article/details/78616738)
-  [Openresty最佳案例 | 第9篇：Openresty实现的网关权限控制](http://blog.csdn.net/forezp/article/details/78616779)


## 参考资料

所以在此说明下，我参考了甚至copy了以下的博客或者资料：

openresty官方网站：http://openresty.org/en/ 

openresty 最佳实践： http://wiki.jikexueyuan.com/project/openresty/

跟我学Nginx+Lua开发：http://www.iteye.com/blogs/subjects/nginx-lua

lua入门教程：http://www.runoob.com/lua/lua-tutorial.html

## 系列文章的源码下载

本系列的文章源码下载地址：
https://github.com/forezp/openresty-best-samples