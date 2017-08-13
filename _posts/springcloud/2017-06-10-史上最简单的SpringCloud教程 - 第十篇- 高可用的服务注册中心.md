---
layout: post
title:  "SpringCloud教程第10篇：高可用的服务注册中心"
categories: SpringCloud
tags:  SpringCloud eureka
---

* content
{:toc}

文章 [史上最简单的 SpringCloud 教程 | 第一篇: 服务的注册与发现（Eureka）](http://blog.csdn.net/forezp/article/details/69696915) 介绍了服务注册与发现，其中服务注册中心Eureka Server，是一个实例，当成千上万个服务向它注册的时候，它的负载是非常高的，这在生产环境上是不太合适的，这篇文章主要介绍怎么将Eureka Server集群化。

<!--more-->

### 一、准备工作

> Eureka can be made even more resilient and available by running multiple instances and asking them to register with each other. In fact, this is the default behaviour, so all you need to do to make it work is add a valid serviceUrl to a peer, e.g.
> 
>  摘自官网

Eureka通过运行多个实例，使其更具有高可用性。事实上，这是它默认的熟性，你需要做的就是给对等的实例一个合法的关联serviceurl。

这篇文章我们基于[第一篇文章的工程](https://github.com/forezp/SpringCloudLearning/tree/master/chapter1)，来做修改。

### 二、改造工作

在eureka-server工程中resources文件夹下，创建配置文件application-peer1.yml:

```
server:
  port: 8761

spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1
  client:
    serviceUrl:
      defaultZone: http://peer2:8769/eureka/

```

并且创建另外一个配置文件application-peer2.yml：

```

server:
  port: 8769

spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka/

```

这时eureka-server就已经改造完毕。

> ou could use this configuration to test the peer awareness on a single host (there’s not much value in doing that in production) by manipulating /etc/hosts to resolve the host names.

按照官方文档的指示，需要改变etc/hosts，linux系统通过vim /etc/hosts ,加上：

```
127.0.0.1 peer1
127.0.0.1 peer2
```
 windows电脑，在c:/windows/systems/drivers/etc/hosts 修改。
 
这时需要改造下service-hi:

```
eureka:
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka/
server:
  port: 8762
spring:
  application:
    name: service-hi

```

### 三、启动工程

启动eureka-server：
>java -jar eureka-server-0.0.1-SNAPSHOT.jar - -spring.profiles.active=peer1
>
java -jar eureka-server-0.0.1-SNAPSHOT.jar - -spring.profiles.active=peer2

>


启动service-hi:

>java -jar service-hi-0.0.1-SNAPSHOT.jar
>

访问：localhost:8761,如图：


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2279594-659c68e405bd70bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

你会发现注册了service-hi，并且有个peer2节点，同理访问localhost:8769你会发现有个peer1节点。

client只向8761注册，但是你打开8769，你也会发现，8769也有 client的注册信息。

个人感受：这是通过看官方文档的写的demo ，但是需要手动改host是不是不符合Spring Cloud 的高上大？


> ### Prefer IP Address
In some cases, it is preferable for Eureka to advertise the IP Adresses of services rather than the hostname. Set eureka.instance.preferIpAddress to true and when the application registers with eureka, it will use its IP Address rather than its hostname.
> 
> 摘自官网
> 

eureka.instance.preferIpAddress=true是通过设置ip让eureka让其他服务注册它。也许能通过去改变去通过改变host的方式。

此时的架构图：

![有点丑e.png](http://upload-images.jianshu.io/upload_images/2279594-a052854a3084fdd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
 
Eureka-eserver peer1 8761,Eureka-eserver peer2 8769相互感应，当有服务注册时，两个Eureka-eserver是对等的，它们都存有相同的信息，这就是通过服务器的冗余来增加可靠性，当有一台服务器宕机了，服务并不会终止，因为另一台服务存有相同的数据。

本文源码下载：
[https://github.com/forezp/SpringCloudLearning/tree/master/chapter10](https://github.com/forezp/SpringCloudLearning/tree/master/chapter10)

### 四、参考文献

[high_availability_zones](http://projects.spring.io/spring-cloud/spring-cloud.html#_high_availability_zones_and_regions)



### 相关文章推荐：
* [史上最简单的 SpringCloud 教程 | 终章](http://blog.csdn.net/forezp/article/details/70148833)
* [史上最简单的 SpringCloud 教程 | 第一篇: 服务的注册与发现（Eureka）](http://blog.csdn.net/forezp/article/details/69696915)
* [史上最简单的SpringCloud教程 | 第七篇: 高可用的分布式配置中心(Spring Cloud Config)](http://blog.csdn.net/forezp/article/details/70037513)



