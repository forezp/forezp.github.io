---
layout: post
title:  "5分钟入门入redis"
categories: j2ee
tags:  j2ee redis
---

* content
{:toc}


### 1.redis概述
 redis是一个开源的，先进的 key-value 存储可用于构建高性能的存储解决方案。它支持数据结构有字符串，哈希，列表，集合，带有范围查询的排序集，位图，超文本和具有半径查询的地理空间索引。 NoSQL，Not Only [SQL]，泛指非关系型的数据库。所以redis是一种nosql。*敲黑板画重点：redis是一种nosql.*
 
 <!--more-->
 
redis的优点：

 * 异常快速
 * 支持丰富的数据类型 
 * 操作都是原子的

### 2.下载安装

linux 系统下安装：

````
$ wget http://download.redis.io/releases/redis-3.2.6.tar.gz
$ tar xzf redis-3.2.6.tar.gz
$ cd redis-3.2.6
$ make

```

启动服务器：

 ```
    $ src/redis-server
 ```
 
 启动客户端
 
 ```
 $ src/redis-cli
 ```
  
mac下安装:

 ```
brew install redis
```
启动：

```
redis-server
redis-cli 

```
 windows下安装:

由于官方并没有提供windows 版本，不过微软为了能够应用redis 到 windows服务器，由微软维护了windows版的redis，下载地址：[点击进入](https://github.com/MSOpenTech/redis/releases).建议下载msi 版本，直接安装即可。

![sss](http://upload-images.jianshu.io/upload_images/2279594-ddad8a98ff77bace.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 
启动成功：

```

[35142] 01 May 14:36:28.939 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
[35142] 01 May 14:36:28.940 * Max number of open files set to 10032
                _._
              _.-``__ ''-._
        _.-``    `.  `_.  ''-._           Redis 2.6.12 (00000000/0) 64 bit
    .-`` .-```.  ```\/    _.,_ ''-._
  (    '      ,       .-`  | `,    )     Running in stand alone mode
  |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
  |    `-._   `._    /     _.-'    |     PID: 35142
    `-._    `-._  `-./  _.-'    _.-'
  |`-._`-._    `-.__.-'    _.-'_.-'|
  |    `-._`-._        _.-'_.-'    |           http://redis.io
    `-._    `-._`-.__.-'_.-'    _.-'
  |`-._`-._    `-.__.-'    _.-'_.-'|
  |    `-._`-._        _.-'_.-'    |
    `-._    `-._`-.__.-'_.-'    _.-'
        `-._    `-.__.-'    _.-'
            `-._        _.-'
                `-.__.-'

[35142] 01 May 14:36:28.941 # Server started, Redis version 2.6.12
[35142] 01 May 14:36:28.941 * The server is now ready to accept connections on port 6379

```
 
### 3.redis 支持的数据类型
 
#### 3.1字符串
启动客户端 ,存储字符串到redis.

```
redis> SET name forezp
OK
```
 
 取字符串:
 
```
 redis> get name 
"forezp"
 ```

#### 3.2Hashes - 哈希值

```

redis > HMSET king username forezp password xxdxx age 22
redis > HGETALL king
1) "username"
2) "forezp "
3) "password "
4) "xxdxx "
5) "age "
6) "22"
```

#### 3.3 Lists - 列表

```
redis> lpush pricess jack
(integer) 1
redis 127.0.0.1:6379> lpush pricess jolin
(integer) 2
redis 127.0.0.1:6379> lpush pricess mayun
(integer) 3
redis 127.0.0.1:6379> lrange pricess 0 10
1) "jack"
2) "jolin"
3) "mayun"

```
#### 3.4 Redis有序集合
Redis有序集合类似Redis集合存储在设定值唯一性。不同的是，一个有序集合的每个成员带有分数，用于以便采取有序set命令，从最小的到最大的分数有关。

```
redis > ZADD kindom 1 redis
(integer) 1
redis> ZADD kindom 2 mongodb
(integer) 1
redis > ZADD kindom 3 mysql
(integer) 1
redis > ZADD kindom 3 mysql
(integer) 0
redis > ZADD kindom 4 mysql
(integer) 0
redis > ZRANGE kindom 0 10 WITHSCORES
1) "redis"
2) "1"
3) "mongodb"
4) "2"
5) "mysql"
6) "4"

```

#### 3.5 Redis发布订阅

开启客户端作为接受者
```

redis> SUBSCRIBE myking messages...
 (press Ctrl-C to quit
)1) "subscribe"
2) "myking "
3) (integer) 1

```

开启另一个客户端作为发送者：

```
redis > PUBLISH myking "Redis is a great caching technique"
(integer) 1

```

这样接受者就可以收到:

 ```
"Redis is a great caching technique"
```
#### 3.6 其他的一些操作

1.获取所以的key

 ```
redis> KEYS *

```

2,判断key是否存在

```
EXISTS key
```

3.删除key 

```
DEL key [key …]
```

4.获取数据类型

```

TYPE key
```

5.向尾部添加

 ```
APPEND key value

```

6.获取字符串长度

```
strlen key
```

当然这里只是介绍简单的一些操作，复杂的参考官方文档。

###  4. 在java应用中使用redis---jedis

前提是redis 已经安装，并且已经开启服务。

 jedis 下载地址 [https://github.com/xetorthio/jedis](https://github.com/xetorthio/jedis)

>Jedis is a blazingly small and sane [Redis](http://github.com/antirez/redis) java client.
Jedis was conceived to be EASY to use.

>翻译： jedis是一个非常小的java客户端，被认为是容易使用。

*怎么使用？*



```

    public static void main(String[] args){

        Jedis jedis = new Jedis("localhost");
        System.out.println("Connection to server sucessfully");
        //check whether server is running or not
        System.out.println("Server is running: "+jedis.ping());
        jedis.lpush("forezp-list", "Redis");
        jedis.lpush("forezp-list", "Mongodb");
        jedis.lpush("forezp-list", "Mysql");
        // Get the stored data and print it
        List<String> list = jedis.lrange("forezp-list", 0 ,5);
        for(int i=0; i<list.size(); i++) {
            System.out.println("Stored string in redis:: "+list.get(i));
        }

    }

```

运行：

> Connection to server sucessfully
Server is running: PONG
Stored string in redis:: Mysql
Stored string in redis:: Mongodb
Stored string in redis:: Redis
Stored string in redis:: Mysql
Stored string in redis:: Mongodb
Stored string in redis:: Redis

 redis 入门介绍就到这里了。另外，*敲黑板，划重点：* 遇到问题首先不要去百度搜，要去官网搜。聪明的你，是不是自己安装下 ，实践下。

### 优秀文章推荐：
* [史上最简单的 SpringCloud 教程 | 终章](http://blog.csdn.net/forezp/article/details/70148833)
* [史上最简单的 SpringCloud 教程 | 第一篇: 服务的注册与发现（Eureka）](http://blog.csdn.net/forezp/article/details/69696915)
* [史上最简单的SpringCloud教程 | 第七篇: 高可用的分布式配置中心(Spring Cloud Config)](http://blog.csdn.net/forezp/article/details/70037513)