---
layout: post
title:  "kong api gateway 初体验"
categories: nginx kong
tags:  nginx kong

---

* content
{:toc}


kong api gateway 初体验（first sight?）。

Kong是一个可扩展的开源API层（也称为API网关或API中间件）。 Kong运行在任何RESTful API的前面，并通过插件扩展，它们提供超出核心平台的额外功能和服务。
Kong最初是在Mashape建立的，用于为其API Marketplace提供超过15,000个API和Microservices，并为超过20万的开发者每月生成数十亿个请求。 今天，Kong被用于小型和大型组织的关键任务部署

![](https://getkong.org/assets/images/docs/kong-architecture.jpg)

<!--more-->

## 使用的软件

- Unbuntu 虚拟机（有自己的服务器更好）
- PostgreSQL
- kong 
- kong-dashboard
- docker
- spring boot

## 安装 PostgreSQL

kong 需要使用到数据库，目前支持PostgreSQL和Cassandran ,我选择大象数据库，安装过程省略，可以参考这篇文章。
http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html

安装完后建一个kong的用户、密码为kong、建一个kong 数据库:

```
CREATE USER kong; CREATE DATABASE kong OWNER kong;
```

## 安装kong 

下载kong的源文件，下载地址：https://getkong.org/install/ubuntu/

下载完成之后会有这样一个文件kong-community-edition-0.11.0.*.deb，cd到这个文件的目录：


```
$ sudo apt-get update
$ sudo apt-get install openssl libpcre3 procps perl
$ sudo dpkg -i kong-community-edition-0.11.0.*.deb

```


## 配置kong

配置文档在这里：

https://getkong.org/docs/0.9.x/configuration/

复制配置文件：

```
$ cp /etc/kong/kong.conf.default /etc/kong/kong.conf
```

配置文件：

```
/etc/kong/kong.conf
/etc/kong.conf

```

打开配置文件，里面可以修改很多配置，修改数据库连接，用户名、密码

```
pg_host = 127.0.0.1             # The PostgreSQL host to connect to.
pg_port = 5432                  # The port to connect to.
pg_user = kong                  # The username to authenticate if required.
pg_password = kong              # The password to authenticate if required.
pg_database = kong

```


执行以下整合命令：

```
$ kong migrations up [-c /path/to/kong.conf]

```

启动kong :

```
kong start -c /etc/kong/kong.conf --vv
```
打开浏览器访问：localhost:8001，浏览器显示了一大串关于kong的json字符串，则启动成功。
kong管理端口为8001, 监控端口为8000。

管理端口用rest api对api进行操作，文档地址:https://getkong.org/docs/0.8.x/admin-api

## 安装 kong-dashboard

kong管理端的第三方网页，地址：https://github.com/PGBI/kong-dashboard

支持npm启动，但是没有成功过，直接选择了docker启动。
要求先安装docker,docker启动镜像

```
# Start Kong Dashboard  8080端口启动
docker run -d -p 8080:8080 pgbi/kong-dashboard:v2

# Start Kong Dashboard on a custom port  指定一个端口启动
docker run -d -p [port]:8080 pgbi/kong-dashboard:v2

# Start Kong Dashboard with basic auth  8080端口启动，带一个用户基本认证
docker run -d -p 8080:8080 pgbi/kong-dashboard:v2 -a user=password

```

## 演示实例

在电脑上开启一个spring boot 工程有一个api接口为http://10.10.20.187:8762/hi


其实kong管理api有一系列的接口，直接用crul 就可以完成管理，但是有第三个kong-dashboard，我就用了kong-dashboard的管理界面进行操作。

在上一小节启动docker之后，打开网页http://192.168.86.128:8080（我unbuntu虚拟机的host为192.168.86.128）,填写kong的管理urlhttp://192.168.86.128:8001，就可以进入了。

![](http://fangzhipeng.oss-cn-hangzhou.aliyuncs.com/blog/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170830111958.png?x-oss-process=style/caijai)

在kong管理界面创建一个api接口：

填写相关的参数即可，创建完成后如下：

![](http://fangzhipeng.oss-cn-hangzhou.aliyuncs.com/blog/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170830112038.png?x-oss-process=style/caijai)

在浏览器上访问：http://192.168.86.128:8000/hi

> hi forezp,i am from port:8762


添加api限流插件,一个ip一分钟10次。

![](http://fangzhipeng.oss-cn-hangzhou.aliyuncs.com/blog/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170830112602.png?x-oss-process=style/caijai)

访问超过10次后，会拒绝访问。

添加file-log的插件,文件存放目录为/temp/file.log  ：

![](http://fangzhipeng.oss-cn-hangzhou.aliyuncs.com/blog/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170830114620.png?x-oss-process=style/caijai)

再次访问：http://192.168.86.128:8000/hi

可以在打开/temp/file.log看见里面的日志信息。

kong 支持了20中插件，插件地址：https://getkong.org/plugins/

## 参考资料

https://getkong.org/about/

http://www.cnblogs.com/SummerinShire/category/861287.html

http://www.jianshu.com/p/f9a2210f6722

https://yq.aliyun.com/articles/63180

http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html