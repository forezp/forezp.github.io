---
layout: post
title:  "Spring Boot教程第22篇：整合elk，搭建实时日志平台"
categories: SpringBoot
tags:  SpringBoot
---

* content
{:toc}

这篇文章主要介绍springboot整合elk.

<!--more-->

## elk 简介

* Elasticsearch是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。

* Logstash是一个完全开源的工具，他可以对你的日志进行收集、过滤，并将其存储供以后使用（如，搜索）。

* Kibana 也是一个开源和免费的工具，它Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志。

## elk下载安装

elk下载地址：[https://www.elastic.co/downloads/](https://www.elastic.co/downloads/)


建议在 linux上运行，elk在windows上支持得不好，另外需要jdk1.8 的支持，需要提前安装好jdk.


下载完之后： 安装，以logstash为栗子：

> cd /usr/local/
> 
> mkdir logstash
> 
> tar -zxvf logstash-5.3.2.tar.gz
> 
> mv logstash-5.3.2 /usr/local/logstash
> 


## 配置、启动 Elasticsearch

打开Elasticsearch的配置文件：

```
vim config/elasticsearch.yml
```
修改配置:

```
network.host=localhost
network.port=9200

```
它默认就是这个配置，没有特殊要求，在本地不需要修改。

启动Elasticsearch

```
./bin/elasticsearch

```

启动成功，访问localhost:9200,网页显示：

```
{
  "name" : "56IrTCM",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "e4ja7vS2TIKI1BsggEAa6Q",
  "version" : {
    "number" : "5.2.2",
    "build_hash" : "f9d9b74",
    "build_date" : "2017-02-24T17:26:45.835Z",
    "build_snapshot" : false,
    "lucene_version" : "6.4.1"
  },
  "tagline" : "You Know, for Search"
}
```

## 配置、启动 logstash

在 logstash的主目录下：

```
vim config/log4j_to_es.conf 

```

修改 log4j_to_es.conf 如下：

```
input {
  log4j {
    mode => "server"
    host => "localhost"
    port => 4560
  }
}
filter {
  #Only matched data are send to output.
}
output {
    elasticsearch {
    action => "index"          #The operation on ES
    hosts  => "localhost:9200"   #ElasticSearch host, can be array.
    index  => "applog"         #The index to write data to.
  }
}

```

修改完配置后启动：

```
./bin/logstash -f config/log4j_to_es.conf 

```
终端显示如下：


![image.png](http://upload-images.jianshu.io/upload_images/2279594-77bf8e6fba787399.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/720)



访问localhost:9600

```
{"host":"Pc-20130412.local","version":"5.3.2","http_address":"127.0.0.1:9600","id":"e6bb985c-c688-49a4-
a55b-4d362bb4136f","name":"Pc-20130412.local","build_date":
"2017-04-24T16:32:22Z","build_sha":"242159a5eea55fe213fe5c8
52d36455e24252c82","build_snapshot":false}
```

证明logstash启动成功。

## 配置、启动kibana

到kibana的安装目录：

```
./bin/kibana 
```

默认配置即可。

访问localhost:5601，网页显示：

![image.png](http://upload-images.jianshu.io/upload_images/2279594-6524e596d53ec119.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/720)

证明启动成功。

## 创建springboot工程

起步依赖如下：

```
<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
			<exclusions>
				<exclusion>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-log4j</artifactId>
			<version>1.3.8.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>


	</dependencies>

```


log4j的配置，／src/resources/log4j.properties如下：

```

log4j.rootLogger=INFO,console

# for package com.demo.elk, log would be sent to socket appender.
log4j.logger.com.forezp=DEBUG, socket

# appender socket
log4j.appender.socket=org.apache.log4j.net.SocketAppender
log4j.appender.socket.Port=4560
log4j.appender.socket.RemoteHost=localhost
log4j.appender.socket.layout=org.apache.log4j.PatternLayout
log4j.appender.socket.layout.ConversionPattern=%d [%-5p] [%l] %m%n
log4j.appender.socket.ReconnectionDelay=10000

# appender console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.target=System.out
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d [%-5p] [%l] %m%n

```

打印log测试：

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootElkApplicationTests {

	@Test
	public void contextLoads() {
	}

	private Logger logger = Logger.getLogger(getClass());

	@Test
	public void test() throws Exception {

		for(int i=0;i<100;i++) {
			logger.info("输出info  ");
			logger.debug("输出debug+skkkw嗡嗡嗡kw");
			logger.error("输出error  嗡嗡嗡我");
		}
	}


}
```

## 在kibana 实时监控日志

打开localhost:5601:

Management=>index pattrns=>add new:

![image.png](http://upload-images.jianshu.io/upload_images/2279594-14a8c8e08d971647.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


点击discovery:


![image.png](http://upload-images.jianshu.io/upload_images/2279594-1f43ebd237e8543e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 参考资料

[https://my.oschina.net/itblog/blog/547250](https://my.oschina.net/itblog/blog/547250)

## 源码下载
[https://github.com/forezp/SpringBootLearning](https://github.com/forezp/SpringBootLearning)

### 优秀文章推荐：

* 更多springboot 教程：[springBoot非官方教程 | 文章汇总](http://blog.csdn.net/forezp/article/details/70341818)
* 更多springcoud 教程：[史上最简单的 SpringCloud 教程 |  文章汇总](http://blog.csdn.net/forezp/article/details/70148833)
