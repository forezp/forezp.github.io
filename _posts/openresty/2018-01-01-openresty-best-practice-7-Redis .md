---
layout: post
title:  "Openresty最佳案例 | 第7篇： 模块开发、OpenResty连接Redis"
categories: Openresty 
tags:  Openresty Nginx
---

* content
{:toc}
 
在实际的开发过程中，不可能把所有的lua代码写在一个lua文件中，通常的做法将特定功能的放在一个lua文件中，即用lua模块开发。在lualib目录下，默认有以下的lua模块。

<!--more-->

```
lualib/
├── cjson.so
├── ngx
│   ├── balancer.lua
│   ├── ocsp.lua
│   ├── re.lua
│   ├── semaphore.lua
│   ├── ssl
│   │   └── session.lua
│   └── ssl.lua
├── rds
│   └── parser.so
├── redis
│   └── parser.so
└── resty
    ├── aes.lua
    ├── core
    │   ├── base64.lua
    │   ├── base.lua
    │   ├── ctx.lua
    │   ├── exit.lua
    │   ├── hash.lua
    │   ├── misc.lua
    │   ├── regex.lua
    │   ├── request.lua
    │   ├── response.lua
    │   ├── shdict.lua
    │   ├── time.lua
    │   ├── uri.lua
    │   ├── var.lua
    │   └── worker.lua
    ├── core.lua
    ├── dns
    │   └── resolver.lua
    ├── limit
    │   ├── conn.lua
    │   ├── req.lua
    │   └── traffic.lua
    ├── lock.lua
    ├── lrucache
    │   └── pureffi.lua
    ├── lrucache.lua
    ├── md5.lua
    ├── memcached.lua
    ├── mysql.lua
    ├── random.lua
    ├── redis.lua
    ├── sha1.lua
    ├── sha224.lua
    ├── sha256.lua
    ├── sha384.lua
    ├── sha512.lua
    ├── sha.lua
    ├── string.lua
    ├── upload.lua
    ├── upstream
    │   └── healthcheck.lua
    └── websocket
        ├── client.lua
        ├── protocol.lua
        └── server.lua


```

在使用这些模块之前，需要在nginx的配置文件nginx.conf中的http模块加上以下的配置：

```
 lua_package_path "/usr/example/lualib/?.lua;;";  #lua 模块  
 lua_package_cpath "/usr/example/lualib/?.so;;";  #c模块   

```

现在来简单的开发一个lua模块：

```
vim /usr/example/lualib/module1.lua 

```

在module1.lua文件加上以下的代码：

```
local count = 0  
local function hello()  
   count = count + 1  
   ngx.say("count : ", count)  
end  
  
local _M = {  
   hello = hello  
}  
  
return _M  


```

开发时将所有数据做成局部变量/局部函数；通过 _M导出要暴露的函数，实现模块化封装。

在／usr/example/lua目录下创建一个test_module_1.lua 文件，在该文件中引用上面的module1.lua文件。

```

vim /usr/example/lua/test_module_1.lua 

```

加上以下代码：

```
local module1 = require("module1")  
  
module1.hello() 
 
```
通过require("模块名")来加载模块，如果是多级目录，则需要通过require(“目录1.目录2.模块名”）加载。

在/user/example/example.conf中加上以下的配置：

```
location /lua_module_1 {  
    default_type 'text/html';  
    lua_code_cache on;  
    content_by_lua_file /usr/example/lua/test_module_1.lua;  
}
```

多次在浏览器上访问：http://116.196.177.123/lua_module_1，浏览器显示：

```
count : 1
count : 2
count : 3

...

```

## 安装redis 

linux下安装：
cd /usr/servers

```
$ wget http://download.redis.io/releases/redis-3.2.6.tar.gz
$ tar xzf redis-3.2.6.tar.gz
$ cd redis-3.2.6
$ make

```
启动redis:

```
nohup /usr/servers/redis-3.2.6/src/redis-server  /usr/servers/redis-3.2.6/redis.conf &  

```

查看是否启动：

```
ps -ef |grep redis
```

终端显示：

```
root     20985 14268  0 18:49 pts/0    00:00:00 /usr/servers/redis-3.2.6/src/redis-server 127.0.0.1:6379

```
可见redis已经启动。


## lua连接redis

lua_resty_redis模块地址：https://github.com/openresty/lua-resty-redis

> lua-resty-redis - Lua redis client driver for the ngx_lua based on the cosocket API
> 

lua_resty_redis 它是一个基于cosocket API的为ngx_lua模块提供Lua redis客户端的驱动。

创建一个test_redis_basic.lua文件

vim /usr/example/lua/test_redis_basic.lua

```
 local function close_redis(red)  
    if not red then  
        return  
    end  
  
    local pool_max_idle_time = 10000 --毫秒  
    local pool_size = 100 --连接池大小  
    local ok, err = red:set_keepalive(pool_max_idle_time, pool_size)  
    if not ok then  
        ngx.say("set keepalive error : ", err)  
    end  
end    
  
local redis = require("resty.redis")  
  

local red = redis:new()  
 
red:set_timeout(1000)  

local ip = "127.0.0.1"  
local port = 6379  
local ok, err = red:connect(ip, port)  
if not ok then  
    ngx.say("connect to redis error : ", err)  
    return close_redis(red)  
end  
 
ok, err = red:set("msg", "hello world")  
if not ok then  
    ngx.say("set msg error : ", err)  
    return close_redis(red)  
end  
  

local resp, err = red:get("msg")  
if not resp then  
    ngx.say("get msg error : ", err)  
    return close_redis(red)  
end  

if resp == ngx.null then  
    resp = ''  
end  
ngx.say("msg : ", resp)  
  
close_redis(red)  

 
```

上面的代码很简单，通过连接池连接Redis，连接上redis后，通过set一对键值对（msg,helloword）到redis中，然后get(msg)，并通过ngx.say()返回给浏览器。

vim /usr/example/example.conf,添加以下的配置代码：

```
location /lua_redis_basic {  
    default_type 'text/html';  
    lua_code_cache on;  
    content_by_lua_file /usr/example/lua/test_redis_basic.lua;  
 }  

```

浏览器访问：http://116.196.177.123/lua_redis_basic

浏览器显示：

> msg : hello world

lua_resty_redis支持所有的redis指令，本身Redis就支持lua语言操作。所以lua_resty_redis模块能够提高所有的redis操作的功能。


在很多时候，Redis是设置了口令的，连接时，如果需要验证口令，需要添加 local res, err = red:auth("foobared")，示例代码如下：

```
  local redis = require "resty.redis"
    local red = redis:new()

    red:set_timeout(1000) -- 1 sec

    local ok, err = red:connect("127.0.0.1", 6379)
    if not ok then
        ngx.say("failed to connect: ", err)
        return
    end

    local res, err = red:auth("foobared")
    if not res then
        ngx.say("failed to authenticate: ", err)
        return
    end

```

更多请关注的官方文档https://github.com/openresty/lua-resty-redis
和开涛的博客http://jinnianshilongnian.iteye.com/blog/2187328