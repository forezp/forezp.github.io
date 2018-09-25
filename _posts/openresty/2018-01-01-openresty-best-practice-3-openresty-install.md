---
layout: post
title:  " Openresty最佳案例 | 第3篇:Openresty的安装"
categories: Openresty 
tags:  Openresty Nginx
---

* content
{:toc}

我的服务器为一台全新的centos 7的服务器，所以从头安装openresty，并记录了安装过程中出现的问题，以及解决办法。 

<!--more-->

## 1.首先安装openresty

```
cd /usr
mkdir servers
mkdir downloads 

yum install libreadline-dev libncurses5-dev libpcre3-dev libssl-dev perl 
 
cd /usr/servers
 
wget https://openresty.org/download/openresty-1.11.2.4.tar.gz
tar -zxvf openresty-1.11.2.4.tar.gz
cd /usr/servers/bunble/LuaJIT-2.1-20170405

安装Lua
make clean && make && make install  
 
```

安装过程中出现以下的错误：

> gcc: Command not found


## 2.安装gcc

```
yum -y install gcc automake autoconf libtool make
```
## 3.重新make

```
make clean && make && make install

ln -sf luajit-2.1.0-alpha /usr/local/bin/luajit  

```

## 4.下载ngx_cache_purge模块，该模块用于清理nginx缓存

cd /usr/servers/ngx_openresty--1.11.2.4/bundle  
wget https://github.com/FRiCKLE/ngx_cache_purge/archive/2.3.tar.gz  
tar -xvf 2.3.tar.gz  


 
## 5.下载nginx_upstream_check_module模块，该模块用于ustream健康检查

cd /usr/servers/ngx_openresty-1.11.2.4/bundle  
wget https://github.com/yaoweibin/nginx_upstream_check_module/archive/v0.3.0.tar.gz  
tar -xvf v0.3.0.tar.gz   


## 6.重新安装opresty

```
cd /usr/servers/ngx_openresty-1.11.2.4

./configure --prefix=/usr/servers --with-http_realip_module  --with-pcre  --with-luajit --add-module=./bundle/ngx_cache_purge-2.3/ --add-module=./bundle/nginx_upstream_check_module-0.3.0/ -j2 

```

提示错误，安装pcre库

```
yum install -y pcre pcre-devel

```

<1> gcc 安装
安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：

```
yum install gcc-c++
```

<2> PCRE pcre-devel 安装

PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：

```
yum install -y pcre pcre-devel
```

<3> zlib 安装
zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。

```
yum install -y zlib zlib-devel
```

<4> OpenSSL 安装
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。

```
yum install -y openssl openssl-devel
```

<5>.重新安装OpenResty

```
cd /usr/servers/ngx_openresty-1.11.2.4

./configure --prefix=/usr/servers --with-http_realip_module  --with-pcre  --with-luajit --add-module=./bundle/ngx_cache_purge-2.3/ --add-module=./bundle/nginx_upstream_check_module-0.3.0/ -j2 

make && make install  
```

<6>.启动Nginx

```
/usr/servers/nginx/sbin/nginx
```

浏览器访问http://116.196.177.123：

```
Welcome to OpenResty!

If you see this page, the OpenResty web platform is successfully installed and working. Further configuration is required.

For online documentation and support please refer to openresty.org.

Thank you for flying OpenResty.
```
安装成功了。


## 6.配置nginx

```
vim /usr/servers/nginx/conf/nginx.conf  
```

错误提示没有安装vim

```
 yum -y install vim*
```

1、在http部分添加如下配置 

lua模块路径，多个之间”;”分隔，其中”;;”表示默认搜索路径，默认到/usr/servers/nginx下找 
 
lua_package_path "/usr/servers/lualib/?.lua;;";  #lua 模块  
lua_package_cpath "/usr/servers/lualib/?.so;;";  #c模块  

2、在nginx.conf中的http部分添加include lua.conf包含此文件片段 
Java代码  收藏代码
include lua.conf;

在/usr/server/nginx/conf下

vim lua.conf 

```
#lua.conf  
server {  
    listen       80;  
    server_name  _;  
    
    location /lua {  
    default_type 'text/html';  
        content_by_lua 'ngx.say("hello world")';  
    } 
}  


```

## 7.环境变量：

vim  /etc/profile

```
JAVA_HOME=/usr/local/jdk/jdk1.8.0_144
JRE_HOME=$JAVA_HOME/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib/dt.jar
export JAVA_HOME JRE_HOME PATH CLASSPATH
export PATH=$PATH:/usr/servers/nginx/sbin

```

source /etc/profile

测试：

nginx -t 

> nginx: the configuration file /usr/servers/nginx/conf/nginx.conf syntax is ok
> nginx: configuration file /usr/servers/nginx/conf/nginx.conf test is successful

nginx  -s reload  

浏览器访问http://116.196.177.123/lua 
，浏览器显示：

> hello world



## 8.将Lua项目化：

> mkdir /usr/example
>  cp -r /usr/servers/lualib/  /usr/example/
> mkdir /usr/example/lua

> cd /usr/example
> vim example.conf

```
server {  
    listen       80;  
    server_name  _;  
  
    location /lua {  
        default_type 'text/html';  
        lua_code_cache off;  
        content_by_lua_file /usr/example/lua/test.lua;  
    }  
} 

```

vim /usr/example/lua/test.lua  

```
ngx.say("hello world");  
```


cd /usr/servers/nginx/conf/

vim nginx.conf

http模块：

```
http {
    include       mime.types;
    default_type  application/octet-stream;
    lua_package_path "/usr/example/lualib/?.lua;;";  #lua 模块  
    lua_package_cpath "/usr/example/lualib/?.so;;";  #c模块   
    include /usr/example/example.conf;
 ....
 ....

}
```
nginx -t 

> nginx: [alert] lua_code_cache is off; this will hurt performance in /usr/example/example.conf:7
nginx: the configuration file /usr/servers/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/servers/nginx/conf/nginx.conf test is successful

nginx -s reload 

浏览器访问http://116.196.177.123/lua ，

>  hello world


导出history的所有命令：

```
在你的账户目录下    输入命令
ls -a   
找到 .bash_history
这个就是记录命令文件。
输入命令：
cat   .bash_history >> history.txt

```

## 参考资料


http://www.linuxidc.com/Linux/2016-09/134907.htm

http://jinnianshilongnian.iteye.com/blog/2186270

https://openresty.org/en/