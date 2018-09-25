---
layout: post
title:  "Openresty最佳案例 | 第5篇：http和C_json模块"
categories: Openresty 
tags:  Openresty Nginx
---

* content
{:toc}

Openresty没有提供默认的Http客户端，需要下载第三方的http客户端。

<!--more-->

下载lua-resty-http到lualib目录下，使用以下的命令下载：

```
cd /usr/example/lualib/resty/  
wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http_headers.lua  

wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http.lua  

```

lua-resty-http模块的地址为https://github.com/pintsized/lua-resty-http

安装成功后，通过require("resty.http")引入 lua_http模块，它有以下的api方法：

- syntax: httpc = http.new() 创建一个 http对象
- syntax: res, err = httpc:request_uri(uri, params)根据参数获取内容，包括：
  - status 状态码
  - headers 响应头
  - body 响应体


vim /usr/example/lua/test_http.lua，写以下代码：

```
local http = require("resty.http")  

local httpc = http.new()  
  
local resp, err = httpc:request_uri("http://s.taobao.com", {  
    method = "GET",  
    path = "/search?q=hello",  
    headers = {  
        ["User-Agent"] = "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.111 Safari/537.36"  
    }  
})  
  
if not resp then  
    ngx.say("request error :", err)  
    return  
end  
  
 
ngx.status = resp.status  
  
  
for k, v in pairs(resp.headers) do  
    if k ~= "Transfer-Encoding" and k ~= "Connection" then  
        ngx.header[k] = v  
    end  
end  
  
ngx.say(resp.body)  
  
httpc:close()  


```

vim /usr/example/example.conf 加上以下的配置：

```
 location /lua_http {
   default_type 'text/html';
   lua_code_cache on;
   content_by_lua_file /usr/example/lua/test_http.lua;
 }

```
在Nginx的配置文件nginx.conf的http部分，加上以下dns解析：
 
vim /usr/servers/nginx/conf/nginx.conf
 
```
resolver 8.8.8.8;  

```

浏览器访问：http://116.196.177.123/lua_http,浏览器会显示淘宝的搜索页。

## lua_cjson模块

Json是一种常见的数据交换格式，常用于http通信协议和其他数据传输领域。在openresty默认内嵌了lua_cjson模块，用来序列化数据。

lua_cjson模块的地址：https://www.kyne.com.au/~mark/software/lua-cjson-manual.html

它常用的API如下：

- local cjson = require "cjson" 获取一个cjson对象
- local str = cjson.encode(obj) obj转换成string
- local obj = cjson.decode(str) 将string转obj

vim /usr/example/lua/test_cjson.lua，添加以下内容：

```
local cjson = require("cjson")  
  

local obj = {  
    id = 1,  
    name = "zhangsan",  
    age = nil,  
    is_male = false,  
    hobby = {"film", "music", "read"}  
}  
  
local str = cjson.encode(obj)  
ngx.say(str, "<br/>")  
  
 
str = '{"hobby":["film","music","read"],"is_male":false,"name":"zhangsan","id":1,"age":null}'  
local obj = cjson.decode(str)  
  
ngx.say(obj.age, "<br/>")  
ngx.say(obj.age == nil, "<br/>")  
ngx.say(obj.age == cjson.null, "<br/>")  
ngx.say(obj.hobby[1], "<br/>")  


```
vim /usr/example/example.conf添加以下内容：

```
 location ~ /lua_cjson {  
   default_type 'text/html';  
   lua_code_cache on;  
   content_by_lua_file /usr/example/lua/test_cjson.lua;  
 }   

```


在浏览器上访问http://116.196.177.123/lua_cjson，浏览器显示以下内容：

```
{"hobby":["film","music","read"],"is_male":false,"name":"zhangsan","id":1}
null
false
true
film


```