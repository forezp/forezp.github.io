---
layout: post
title:  "Openresty最佳案例 | 第6篇：OpenResty连接Mysql"
categories: Openresty 
tags:  Openresty Nginx
---

* content
{:toc}

Centos系统下安装mysql，先下载mysql-community-release-el7-5.noarch.rpm，然后通过yum安装，安装过程一直确定【Y】即可。

<!--more-->

```
cd /usr/downloads/

wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm

rpm -ivh mysql-community-release-el7-5.noarch.rpm

yum install mysql-community-server


```

安装成功后，重启mysql，并进入mysql数据库，给root用户设置一个密码，密码为“123”。

```
service mysqld restart

mysql -u root -p

set password for root@localhost = password('123'); 

```

## openresty连接mysql

lua-resty-mysql模块的官方文档地址： https://github.com/openresty/lua-resty-mysql

> lua-resty-mysql - Lua MySQL client driver for ngx_lua based on the cosocket API 
> 


lua-resty-mysql模块是基于cosocket API 为ngx_lua提供的一个Lua MySQL客户端。它保证了100%非阻塞。



vim /usr/example/lua/test_mysql.lua，添加以下的代码：

```
local function close_db(db)  
    if not db then  
        return  
    end  
    db:close()  
end  
  
local mysql = require("resty.mysql")  
 
local db, err = mysql:new()  
if not db then  
    ngx.say("new mysql error : ", err)  
    return  
end  

db:set_timeout(1000)  
  
local props = {  
    host = "127.0.0.1",  
    port = 3306,  
    database = "mysql",  
    user = "root",  
    password = "123"  
}  
  
local res, err, errno, sqlstate = db:connect(props)  
  
if not res then  
   ngx.say("connect to mysql error : ", err, " , errno : ", errno, " , sqlstate : ", sqlstate)  
   return close_db(db)  
end  
 
local drop_table_sql = "drop table if exists test"  
res, err, errno, sqlstate = db:query(drop_table_sql)  
if not res then  
   ngx.say("drop table error : ", err, " , errno : ", errno, " , sqlstate : ", sqlstate)  
   return close_db(db)  
end  
  

local create_table_sql = "create table test(id int primary key auto_increment, ch varchar(100))"  
res, err, errno, sqlstate = db:query(create_table_sql)  
if not res then  
   ngx.say("create table error : ", err, " , errno : ", errno, " , sqlstate : ", sqlstate)  
   return close_db(db)  
end  
  

local insert_sql = "insert into test (ch) values('hello')"  
res, err, errno, sqlstate = db:query(insert_sql)  
if not res then  
   ngx.say("insert error : ", err, " , errno : ", errno, " , sqlstate : ", sqlstate)  
   return close_db(db)  
end  
  
res, err, errno, sqlstate = db:query(insert_sql)  
  
ngx.say("insert rows : ", res.affected_rows, " , id : ", res.insert_id, "<br/>")  
  
 
local update_sql = "update test set ch = 'hello2' where id =" .. res.insert_id  
res, err, errno, sqlstate = db:query(update_sql)  
if not res then  
   ngx.say("update error : ", err, " , errno : ", errno, " , sqlstate : ", sqlstate)  
   return close_db(db)  
end  
  
ngx.say("update rows : ", res.affected_rows, "<br/>")  
  
local select_sql = "select id, ch from test"  
res, err, errno, sqlstate = db:query(select_sql)  
if not res then  
   ngx.say("select error : ", err, " , errno : ", errno, " , sqlstate : ", sqlstate)  
   return close_db(db)  
end  
  
  
for i, row in ipairs(res) do  
   for name, value in pairs(row) do  
     ngx.say("select row ", i, " : ", name, " = ", value, "<br/>")  
   end  
end  
  
ngx.say("<br/>")  
  
local ch_param = ngx.req.get_uri_args()["ch"] or ''  
 
local query_sql = "select id, ch from test where ch = " .. ngx.quote_sql_str(ch_param)  
res, err, errno, sqlstate = db:query(query_sql)  
if not res then  
   ngx.say("select error : ", err, " , errno : ", errno, " , sqlstate : ", sqlstate)  
   return close_db(db)  
end  
  
for i, row in ipairs(res) do  
   for name, value in pairs(row) do  
     ngx.say("select row ", i, " : ", name, " = ", value, "<br/>")  
   end  
end  
  

local delete_sql = "delete from test"  
res, err, errno, sqlstate = db:query(delete_sql)  
if not res then  
   ngx.say("delete error : ", err, " , errno : ", errno, " , sqlstate : ", sqlstate)  
   return close_db(db)  
end  
  
ngx.say("delete rows : ", res.affected_rows, "<br/>")  
  
  
close_db(db)  


```

在上面的代码中，展示了基本的创表、插入数据、修改数据、查询数据、删除数据的一些功能。

其中用到的lua-resty-mysql的一些API方法：

- syntax: db, err = mysql:new() 创建一个mysql数据库连接对象
- syntax: ok, err = db:connect(options) 尝试远程连接mysql
  -  host mysql的主机名
  -  port 端口
  -  database 数据库名
  -  user 用户名
  -  password 密码
  -  charset 编码
- syntax: db:set_timeout(time) 设置数据库连接超时时间
- syntax: ok, err = db:set_keepalive(max_idle_timeout, pool_size) 设置连接池
- syntax: ok, err = db:close() 关闭数据库
- syntax: bytes, err = db:send_query(query) 发送查询


lua-resty-mysql的一些关键的API方法，见https://github.com/openresty/lua-resty-mysql#table-of-contents

vim /usr/example/example.conf 在配置文件配置：

```

location /lua_mysql {
   default_type 'text/html';
   lua_code_cache on;
   content_by_lua_file /usr/example/lua/test_mysql.lua;
 }

```

浏览器访问http://116.196.177.123/lua_mysql，浏览器显示以下的内容：

```
insert rows : 1 , id : 2
update rows : 1
select row 1 : ch = hello
select row 1 : id = 1
select row 2 : ch = hello2
select row 2 : id = 2

delete rows : 2

```