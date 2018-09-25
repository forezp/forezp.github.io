---
layout: post
title:  "Openresty最佳案例 | 第9篇：Openresty实现的网关权限控制"
categories: Openresty 
tags:  Openresty Nginx
---

* content
{:toc}


采用openresty 开发出的api网关有很多，比如比较流行的kong、orange等。这些API 网关通过提供插件的形式，提供了非常多的功能。这些组件化的功能往往能够满足大部分的需求，如果要想达到特定场景的需求，可能需要二次开发，比如RBAC权限系统。本小节通过整合前面的知识点，来构建一个RBAC权限认证系统。

<!--more-->

## 技术栈
本小节采用了以下的技术栈：

- Openresty(lua+nginx)
- mysql
- redis
- cjson


## 验证流程

- 用户请求经过nginx，nginx的openresty的模块通过拦截请求来进行权限判断
- openresty的access_by_lua_file模块，进行了一系列的判断
   - 用户的请求是否为白名单uri，如果为白名单uri，则直接通过验证，进入下一个验证环节content_by_lua_file,这个环节直接打印一句话：“恭喜，请求通过。”
   - 如果用户请求不为白名单url，则需要取出请求header中的token,如果请求的header不存在token,则直接返回结果401，无权限访问。
   - 如果用户请求的uri的请求头包含token ，则取出token，解密token取出用户id
   - 根据取出的userid去查询数据库获取该用户的权限，如果权限包含了该请求的uri，请求可以通过，否则，请求不通过。
 
- 请求如果通过access_by_lua_file模块，则进入到content_by_lua_file模块,该模块直接返回一个字符串给用户请求，在实际的开发中，可能为路由到具体的应用程序的服务器。

验证流程图如下所示：

![WX20171106-114542@2x.png](http://upload-images.jianshu.io/upload_images/2279594-182b922ba5df4321.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

vim /usr/example/example.conf ，加上以下的配置：

```
 location / {
    default_type "text/html";
    access_by_lua_file /usr/example/lua/api_access.lua;
    content_by_lua_file /usr/example/lua/api_content.lua;
  }

```

以上的配置表示，要不符合已有location路径的所有请求，将走这个location为/  的路径。符合这个location的请求将进入 access_by_lua_file和 content_by_lua_file的模块判断。

vim /usr/example/lua/access_by_lua_file ，加上以下代码：

```
local tokentool = require "tokentool"
local mysqltool = require "mysqltool"

 function is_include(value, tab)
   for k,v in ipairs(tab) do
      if v == value then
           return true
       end
    end
    return false
 end

local white_uri={"/user/login","/user/validate"}
  
--local user_id = ngx.req.get_uri_args()["userId"]
--获取header的token值
local headers = ngx.req.get_headers() 
local token=headers["token"]
local url=ngx.var.uri
if ( not token) or (token==null) or (token ==ngx.null) then
  if is_include(url,white_uri)then
     
  else
    return ngx.exit(401)
  end  
else 
  ngx.log(ngx.ERR,"token:"..token)
  local user_id=tokentool.get_user_id(token)
  if (not user_id) or( user_id ==null) or ( user_id == ngx.null) then
      return ngx.exit(401)   
  end 
  
  ngx.log(ngx.ERR,"user_id"..user_id)
  local permissions={}
  permissions =tokentool.get_permissions(user_id)
  if(not permissions)or(permissions==null)or( permissions ==ngx.null) then
      permissions= mysqltool.select_user_permission(user_id)
      if permissions and permissions ~= ngx.null then
         tokentool.set_permissions(user_id,permissions)
      end
  end  
  if(not permissions)or(permissions==null)or( permissions ==ngx.null) then
     return ngx.exit(401)
  end 
  local is_contain_permission = is_include(url,permissions) 

  if is_contain_permission == true  then
     -- ngx.say("congratuation! you have pass the api gateway")
  else
      return ngx.exit(401) 
  end   
end

```

在上述代码中：

- is_include(value, tab)，该方法判断某个字符串在不在这个table中。
- white_uri={"/user/login","/user/validate"} 是一个白名单的列表。
- local headers = ngx.req.get_headers()从请求的uri的请求头获取token
- is_include(url,white_uri)判断该url是否为白名单url
- local user_id=tokentool.get_user_id(token)根据token获取该token对应的用户的user_id，在常见情况下，是根据token解析出user_id，但在不同的语言加密和加密token存在盐值不一样的情况，比较麻烦，所以我偷了个懒，直接存了redis，用户登录成功后存一下。
- permissions =tokentool.get_permissions(user_id)根据user_id
从redis获取该用户的权限。
- permissions= mysqltool.select_user_permission(user_id)如果redis没有存该用户的权限，则从数据库读。
- tokentool.set_permissions(user_id,permissions)，将从数据库中读取的权限点存在reddis中。
- local is_contain_permission = is_include(url,permissions)，判断该url 在不在该用户对应的权限列表中。


如果所有的判断通过，则该用户请求的具有权限访问，则进入content_by_lua_file模块，直接在这个模块给请求返回“congratulations! you have passed the api gateway”。

vim  /usr/example/lua/api_content.lua ,添加以下内容：

```
ngx.say("congratulations!"," you have passed ","the api gateway")  
----200状态码退出  
return ngx.exit(200)  

```

## 验证演示

打开浏览器访问http://116.196.177.123/user/login，浏览器显示：

> congratulations! you have passed the api gateway
> 

/user/login这个url 在白名单的范围内，所以它是可以通过权限验证的。


打开浏览器访问http://116.196.177.123/user/sss，显示以下内容：

> 401 Authorization Required
> 
> openresty/1.11.2.4


在redis中添加一对key-value，key为token_forezp，value为1，即token_forezp对应的用户的id为1.


```
/usr/servers/redis-3.2.6

src/redis-cli

set token_forezp 1

```

初始化以下的sql脚本，即给用户id为1的用户关联角色，角色并关联权限：

```
INSERT INTO `permission` VALUES ('1', '/user/orgs');
INSERT INTO `role` VALUES ('1', 'user');
INSERT INTO `role_permission` VALUES ('1', '1', '1');
INSERT INTO `user` VALUES ('1', 'forezp');
INSERT INTO `user_role` VALUES ('1', '1', '1');

```

用postman请求，在请求头中加入token，值为token_forezp，请求结果如下：

![WX20171104-182432@2x.png](http://upload-images.jianshu.io/upload_images/2279594-79defe538fe153a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


