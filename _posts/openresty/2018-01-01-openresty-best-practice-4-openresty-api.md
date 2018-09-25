---
layout: post
title:  "Openresty最佳案例 | 第4篇：OpenResty常见的api"
categories: Openresty 
tags:  Openresty Nginx
---

* content
{:toc}


这篇文章主要讲解OpenResty常见的api。

<!--more-->

vim  /usr/example/example.conf

```
 location /lua_var {
        default_type 'text/plain';
        content_by_lua_block {
         ngx.say(ngx.var.arg_a)
        }
   }

```



重新加载nginx配置文件： nginx -s reload

在浏览器上访问http://116.196.177.123/lua_var?a=323,浏览器显示：
 > 323
 
 在上述代码中，涉及到了2个api, 一是ngx.say（直接返回请求结果）；二是ngx.var，它是获取请求的参数，比如本例子上的？a=323,获取之后，直接输出为请求结果。
 
 
## 获取请求类型

vim /usr/example/example.conf

```
  location /lua_request{
       default_type 'text/html';
       lua_code_cache off;
       content_by_lua_file  /usr/example/lua/lua_request.lua;
   }


```

vim /usr/example/lua/lua_request.lua ，添加一下代码：

```

local arg = ngx.req.get_uri_args()
for k,v in pairs(arg) do
   ngx.say("[GET ] key:", k, " v:", v)
end

ngx.req.read_body() -- 解析 body 参数之前一定要先读取 body
local arg = ngx.req.get_post_args()
for k,v in pairs(arg) do
   ngx.say("[POST] key:", k, " v:", v)
end

```
在上述例子中有以下的api:

- ngx.req.get_uri_args 获取在uri上的get类型参数，返回的是一个table类型的数据结构。
- ngx.req.read_body 读取body，这在解析body之前，一定要先读取body。
- ngx.req.get_post_args 获取form表单类型的参数，返回结果是一个table类型的数据。


使用curl模拟请求：
> curl 'http://116.196.177.123/lua_request?a=323&b=ss' -d 'c=12w&d=2se3'

返回的结果：

```
[GET ] key:b v:ss
[GET ] key:a v:323
[POST] key:d v:2se3
[POST] key:c v:12w

```

## 获取请求头

vim /usr/example/lua/lua_request.lua ，在原有的代码基础上，再添加一下代码:

```

local headers = ngx.req.get_headers()
ngx.say("headers begin", "<br/>")
ngx.say("Host : ", headers["Host"], "<br/>")
ngx.say("user-agent : ", headers["user-agent"], "<br/>")
ngx.say("user-agent : ", headers.user_agent, "<br/>")
for k,v in pairs(headers) do
    if type(v) == "table" then
        ngx.say(k, " : ", table.concat(v, ","), "<br/>")
    else
        ngx.say(k, " : ", v, "<br/>")
    end
end

```

重新加载nginx -s reload

使用curl模拟请求：
> curl 'http://116.196.177.123/lua_request?a=323&b=ss' -d 'c=12w&d=2se3'

```
[GET ] key:b v:ss
[GET ] key:a v:323
[POST] key:d v:2se3
[POST] key:c v:12w
headers begin<br/>
Host : 116.196.77.157<br/>
user-agent : curl/7.53.0<br/>
user-agent : curl/7.53.0<br/>
host : 116.196.77.157<br/>
content-type : application/x-www-form-urlencoded<br/>
accept : */*<br/>
content-length : 12<br/>
user-agent : curl/7.53.0<br/>

```

## 获取http的其他方法

vim /usr/example/lua/lua_request.lua ，在原有的代码基础上，再添加一下代码:

```
ngx.say("ngx.req.http_version : ", ngx.req.http_version(), "<br/>")  
--请求方法  
ngx.say("ngx.req.get_method : ", ngx.req.get_method(), "<br/>")  
--原始的请求头内容  
ngx.say("ngx.req.raw_header : ",  ngx.req.raw_header(), "<br/>")  
--请求的body内容体  
ngx.say("ngx.req.get_body_data() : ", ngx.req.get_body_data(), "<br/>")  
ngx.say("<br/>") 

```

重新加载nginx -s reload

使用curl模拟请求：
> curl 'http://116.196.177.123/lua_request?a=323&b=ss' -d 'c=12w&d=2se3'

```
//....
ngx.req.http_version : 1.1<br/>
ngx.req.get_method : POST<br/>
ngx.req.raw_header : POST /lua_request?a=323&b=ss HTTP/1.1
Host: 116.196.77.157
User-Agent: curl/7.53.0
Accept: */*
Content-Length: 12

```

## 输出响应

> vim /usr/example/example.conf，添加一个location,代码如下：


```
 location /lua_response{
        default_type 'text/html';
        lua_code_cache off;
        content_by_lua_file /usr/example/lua/lua_response.lua ;
  }

```

> vim /usr/example/lua/lua_response.lua 添加一下代码：

```
ngx.header.a="1"
ngx.header.b={"a","b"}
ngx.say("hello","</br>")
ngx.print("sss")
return ngx.exit(200)



```

上述代码中有以下api:

- ngx.header 向响应头输出内容
- ngx.say 输出响应体
- ngx.print输出响应体
- ngx.exit 指定http状态码退出


使用curl模拟请求， curl 'http://116.196.177.123/lua_response' ，获取的响应体如下：

```
hello
sss

```

## 日志输出

在配置文件vim /usr/example/example.conf 加上以下代码：

```
 location /lua_log{
       default_type 'text/html';
       lua_code_cache off;
       content_by_lua_file  /usr/example/lua/lua_log.lua;
  }


```

> vim /usr/example/lua/lua_log.lua ，加上以下代码：

```

local log="i'm log"
local num =10
ngx.log(ngx.ERR, "log",log)
ngx.log(ngx.INFO,"num:" ,num)

```

重新加载配置文件nginx -s reload 

> curl 'http://116.196.177.123/lua_log'

打开nginx 的logs目录下的error.log 文件：

> tail -fn 1000  /usr/servers/nginx/logs/error.log 

可以看到在日志文件中已经输出了日志，这种日志主要用于记录和测试。

日志级别：

- ngx.STDERR -- 标准输出
- ngx.EMERG -- 紧急报错
- ngx.ALERT -- 报警
- ngx.CRIT -- 严重，系统故障，触发运维告警系统
- ngx.ERR -- 错误，业务不可恢复性错误
- ngx.WARN -- 告警，业务中可忽略错误
- ngx.NOTICE -- 提醒，业务比较重要信息
- ngx.INFO -- 信息，业务琐碎日志信息，包含不同情况判断等
- ngx.DEBUG -- 调试

## 内部调用


vim /usr/example/example.conf 添加以下代码：

```
location /lua_sum{
      # 只允许内部调用
      internal;
      # 这里做了一个求和运算只是一个例子，可以在这里完成一些数据库、
      # 缓存服务器的操作，达到基础模块和业务逻辑分离目的
      content_by_lua_block {
         local args = ngx.req.get_uri_args()
         ngx.say(tonumber(args.a) + tonumber(args.b))
      }
  }

```

internal 关键字，表示只允许内部调用。使用curl模拟请求，请求命令如下：

> $ curl 'http://116.196.177.123/lua_sum?a=1&b=2'

由于该loction是一个内部调用的，外部不能返回，最终返回的结果为404，如下：

```
<html>
<head><title>404 Not Found</title></head>
<body bgcolor="white">
<center><h1>404 Not Found</h1></center>
<hr><center>openresty/1.11.2.4</center>
</body>
</html>


```


vim /usr/example/example.conf 添加以下代码：

```
location = /lua_sum_test {
   content_by_lua_block {
      local res = ngx.location.capture("/lua_sum", {args={a=3, b=8}})
      ngx.say("status:", res.status, " response:", res.body)
   }  
}

```

上述的代码通过ngx.location.capture去调用内部的location，并获得返回结果，最终将结果输出，采用curl模拟请求：

> $ curl 'http://116.196.177.123/lua_sum_test'


返回结果如下：

```
status:200 response:11

```

## 重定向

vim /usr


```
location /lua_redirect{
    default_type 'text/html';
    content_by_lua_file  /usr/example/lua/lua_redirect.lua;
}  
```



```
ngx.redirect("http://www.fangzhipeng.com", 302)

```

http://116.196.177.123/lua_redirect

## 共享内存

vim /usr/servers/nginx/cong/nginx.conf

在http模块加上以下：

```
lua_shared_dict shared_data 1m;
```

```
 location /lua_shared_dict{
     default_type 'text/html';
     content_by_lua_file /usr/example/lua/lua_shared_dict.lua;
  }


```


```
local shared_data = ngx.shared.shared_data
local i = shared_data:get("i")
if not i then
  i = 1
  shared_data:set("i",i)
end
i = shared_data:incr("i",1)
ngx.say("i:",i)

```

多次访问 http://116.196.177.123/lua_shared_dict，浏览器打印：

```
i:1
i:2
i:3
i:4
i:5

```


## OpenResty执行阶段的概念

以下内容来自于《openresty 最佳实践》

![微信截图_20170929150427.png](http://upload-images.jianshu.io/upload_images/2279594-b9ceb17709093dc8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图所示，openresty的执行阶段分为


这样我们就可以根据我们的需要，在不同的阶段直接完成大部分典型处理了。

- set_by_lua* : 流程分支处理判断变量初始化
- rewrite_by_lua* : 转发、重定向、缓存等功能(例如特定请求代理到外网) 
- access_by_lua* : IP 准入、接口权限等情况集中处理(例如配合 iptable 完成简单防火墙) 
- content_by_lua* : 内容生成
- header_filter_by_lua* : 响应头部过滤处理(例如添加头部信息)
- body_filter_by_lua* : 响应体过滤处理(例如完成应答内容统一成大写)
 
 

执行阶段概念：

- log_by_lua* : 会话完成后本地异步完成日志记录(日志可以记录在本地，还可以同步到其 他机器)
实际上我们只使用其中一个阶段 
- content_by_lua* ，也可以完成所有的处理。但这样做，会让 我们的代码比较臃肿，越到后期越发难以维护。把我们的逻辑放在不同阶段，分工明确，代 码独立，后期发力可以有很多有意思的玩法。