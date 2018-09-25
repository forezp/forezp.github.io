---
layout: post
title:  " Openresty最佳案例 | 第8篇：RBAC介绍、sql和redis模块工具类"
categories: Openresty 
tags:  Openresty Nginx
---

* content
{:toc}


RBAC（Role-Based Access Control，基于角色的访问控制），用户基于角色的访问权限控制。简单地说，一个用户拥有若干角色，每一个角色拥有若干权限。这样，就构造成“用户-角色-权限”的授权模型。在这种模型中，用户与角色之间，角色与权限之间，一般都是多对多的关系。如图所示：

<!--more-->

![WX20171105-214559@2x.png](http://upload-images.jianshu.io/upload_images/2279594-d6c2a08e6c32cf53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## sql_tool

在本案例中，采用的就是这种权限设计的方式。具体的sql语句脚本如下：

```
CREATE TABLE `user` (
`id`  int(11) NOT NULL AUTO_INCREMENT ,
`name`  varchar(255) CHARACTER SET latin1 COLLATE latin1_swedish_ci NULL DEFAULT NULL ,
PRIMARY KEY (`id`)
)
ENGINE=InnoDB
DEFAULT CHARACTER SET=latin1 COLLATE=latin1_swedish_ci
AUTO_INCREMENT=2
ROW_FORMAT=COMPACT
;


CREATE TABLE role(
`id`  int(11) NOT NULL AUTO_INCREMENT ,
`name`  varchar(255) CHARACTER SET latin5 NULL DEFAULT NULL ,
PRIMARY KEY (`id`)
)
ENGINE=InnoDB
DEFAULT CHARACTER SET=latin1 COLLATE=latin1_swedish_ci
AUTO_INCREMENT=2
ROW_FORMAT=COMPACT
;

CREATE TABLE permission(
`id`  int(11) NOT NULL AUTO_INCREMENT ,
`permission`  varchar(255) CHARACTER SET latin1 COLLATE latin1_swedish_ci NULL DEFAULT NULL ,
PRIMARY KEY (`id`)
)
ENGINE=InnoDB
DEFAULT CHARACTER SET=latin1 COLLATE=latin1_swedish_ci
AUTO_INCREMENT=3
ROW_FORMAT=COMPACT
;

CREATE TABLE user_role(
`id`  int(11) NOT NULL AUTO_INCREMENT ,
`user_id`  int(11) NULL DEFAULT NULL ,
`role_id`  int(11) NULL DEFAULT NULL ,
PRIMARY KEY (`id`)
)
ENGINE=InnoDB
DEFAULT CHARACTER SET=latin1 COLLATE=latin1_swedish_ci
AUTO_INCREMENT=2
ROW_FORMAT=COMPACT
;


CREATE TABLE role_permission(
`id`  int(11) NOT NULL AUTO_INCREMENT ,
`role_id`  int(11) NULL DEFAULT NULL ,
`permission_id`  int(11) NULL DEFAULT NULL ,
PRIMARY KEY (`id`)
)
ENGINE=InnoDB
DEFAULT CHARACTER SET=latin1 COLLATE=latin1_swedish_ci
AUTO_INCREMENT=3
ROW_FORMAT=COMPACT
;


```
初始化以下的sql脚本，即给用户id为1的用户关联角色，角色并关联权限：

```
INSERT INTO `permission` VALUES ('1', '/user/orgs');
INSERT INTO `role` VALUES ('1', 'user');
INSERT INTO `role_permission` VALUES ('1', '1', '1');
INSERT INTO `user` VALUES ('1', 'forezp');
INSERT INTO `user_role` VALUES ('1', '1', '1');

```
在本案例中，需要根据user表中的Id获取该Id对应的权限。首先根据userId获取该用户对应的角色，再根据根据该角色获取相应的权限，往往一个用户具有多个角色，而角色又有多个权限。比如查询userId为1 的用户的权限的sql语句如下：

```

SELECT  a.id,a.permission from permission a ,role_permission b,role c,user_role d,user e WHERE a.id=b.permission_id and c.id=b.role_id and d.role_id=c.id and d.user_id=e.id and e.id=1"

```

在Openresty中怎么连接数据库，怎么查询sql语句，在之前的文章已将讲述过了。根据用户id获取用户的权限的功能是一个使用率极高的功能，所以考虑将这个功能模块化。
 
vim /usr/example/lualib/sql_tool.lua ,编辑加入以下的代码：


```
local mysql = require("resty.mysql")  
 
local function close_db(db)  
    if not db then  
        return  
    end  
    db:close()  
end  

local function select_user_permission(user_id)

   local db, err = mysql:new()
   if not db then  
      ngx.say("new mysql error : ", err)  
      return  
   end 
   db:set_timeout(1000)  
  
   local props = {  
      host = "127.0.0.1",  
      port = 3306,  
      database = "test",  
      user = "root",  
      password = "123"  
   }

  local res, err, errno, sqlstate = db:connect(props)  
  
  if not res then  
     ngx.say("connect to mysql error : ", err, " , errno : ", errno, " , sqlstate : ", sqlstate)  
     close_db(db)
  end
  
  local select_sql = "SELECT  a.id,a.permission from permission a ,role_permission b,role c,user_role d,user e WHERE a.id=b.permission_id and c.id=b.role_id and d.role_id=c.id and d.user_id=e.id and e.id="..user_id
  res, err, errno, sqlstate = db:query(select_sql)  
  if not res then  
     ngx.say("select error : ", err, " , errno : ", errno, " , sqlstate : ", sqlstate)  
     return close_db(db)  
  end  

   local permissions={}
   for i, row in ipairs(res) do  
     for name, value in pairs(row) do
	if name == "permission" then
          table.insert(permissions, 1, value)
        end  
 
     end  
   end  
 return permissions 
end

local _M = {  
    select_user_permission= select_user_permission
}  
  
return _M  
  

```

在上面的代码中，有一个select_user_permission(user_id)方法，该方法根据用户名获取该用户的权限。查出来存在一个table 类型的 local permissions={}中。

vim /usr/example/example.conf  加上以下的代码：

```
location ~ /sql_tool{
  default_type 'text/html';
  content_by_lua_file /usr/example/lua/test_sql_tool.lua;
 }

```

在浏览器上访问http://116.196.177.123/sql_tool，浏览器显示如下的内容：

> /user/orgs
> 


## tokentool

在之前的文章讲述了如何使用Openresty连接redis，并操作redis。 这小节将讲述如何使用openresty连接redis，并写几个方法，用于存储用户的token等，并将这些信息模块化，主要有以下几个方法：

- close_redis(red)  通过连接池的方式释放一个连接
- connect() 连接redis
- has_token(token) redis中存在token 与否
- get_user_id(token) 根据token获取用户id
- set_permissions(user_id,permissions) 根据userid设置权限
- get_permissions(user_id)根据userid获取权限

vim /usr/example/lualib/tokentool.lua 编辑一下内容：


```
module("tokentool", package.seeall)
local redis = require "resty.redis"
local str = require "resty.string"
local cjson = require("cjson")  


local redis_host = "127.0.0.1"
local redis_port = 6379

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

local function connect()
    local red = redis:new()
    red:set_timeout(1000)
    local ok, err = red:connect(redis_host, redis_port)
    if not ok then
        return false
    end
    --local res, err = red:auth("xiaoantimes")
    --if not res then
     -- ngx.say("failed to authenticate: ", err)
     -- return false
    --end
    --ok, err = red:select(1)
    --if not ok then
      --  return false
    --end
    return red
end

function has_token(token)
    local red = connect()
    if red == false then
        return false
    end

    local res, err = red:get(token)
    if not res then
        return false
    end
    close_redis(red)  
    return true
end

function set_permissions(user_id,permissions)
  if (permissions==null) or( permissions==ngx.null) then
     return false
  end 
  local str = cjson.encode(permissions)  
  ngx.log(ngx.ERR,"set redis p:"..str)
  local red=connect()
  if red== false then
     return false
  end
  local ok, err = red:set(user_id,str)
  if not ok then
     return false
  end
  return true 
end

function get_permissions(user_id)
  local red=connect()
  if red== false then
     return false
  end
  local res, err = red:get(user_id)
  if (not res) or (res == ngx.null) then
     return
  end 
  ngx.log(ngx.ERR,"get redis p:"..res);
  local permissions=cjson.decode(res)  
  return permissions
end

function get_user_id(token)
    local red = connect()
    local resp, err = red:get(token)  
    if not resp then  
      ngx.say("get msg error : ", err)  
      return close_redis(red)  
    end  
    close_redis(red)  
    return resp
end

```

vim /usr/example/lua/test_token_tool.lua，加上以下的内容：

```
local tokentool= require "tokentool"
local ret = tokentool.has_token("msg")
ngx.log(ngx.ERR,ret)
if ret == true then
   ngx.say("ok")
else
   ngx.say("oops,error")
end
 
```

在/usr/example/example.conf加上以下的内容：

```
 location ~ /token_tool{
     default_type 'text/html';
     lua_code_cache on;
     content_by_lua_file /usr/example/lua/test_token_tool.lua;

 }
 
```

打开浏览器访问http://116.196.177.123/token_tool，浏览器显示：

>ok
>


