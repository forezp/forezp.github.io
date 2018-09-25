---
layout: post
title:  "Openresrt最佳案例 | 第1篇：Nginx介绍"
categories: Openresty 
tags:  Openresty Nginx
---

* content
{:toc}


Nginx是一个高性能的Web 服务器，同时是一个高效的反向代理服务器，它还是一个IMAP/POP3/SMTP
代理服务器。

由于Nginx采用的是事件驱动的架构，能够处理并发百万级别的tcp连接，高度的模块化设计和自由的BSD许可，使得Nginx有着非常丰富的第三方模块。比如Openresty、API网关Kong。

> BSD开源协议是一个给予使用者很大自由的协议。基本上使用者可以"为所欲为",可以自由的使用，修改源代码，也可以将修改后的代码作为开源或者专有软件再发布。

<!--more-->

## Nginx的优点

- 高并发响应性能非常好，官方Nginx处理静态文件并发5w/s
- 反向代理性能非常强。（可用于负载均衡）
- 内存和cpu占用率低。（为Apache的1/5-1/10）
- 对后端服务有健康检查功能。
- 支持PHP cgi方式和fastcgi方式。
- 配置代码简洁且容易上手。


## Nginx的安装

Centos系统安装，请参考这里http://www.linuxidc.com/Linux/2016-09/134907.htm。先复制粘贴下它的文章。


###  1.gcc 安装

安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：

> yum install gcc-c++

### 2.PCRE pcre-devel 安装

PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：

> yum install -y pcre pcre-devel

### 3.zlib 安装

zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。

> yum install -y zlib zlib-devel

### 4.OpenSSL 安装
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。

> yum install -y openssl openssl-devel

### 5.官网下载

1.直接下载.tar.gz安装包，地址：https://nginx.org/en/download.html

2.使用wget命令下载（推荐）。

> wget -c https://nginx.org/download/nginx-1.10.1.tar.gz

### 6.解压

依然是直接命令：

> tar -zxvf nginx-1.10.1.tar.gz
cd nginx-1.10.1

### 7.配置

其实在 nginx-1.10.1 版本中你就不需要去配置相关东西，默认就可以了。当然，如果你要自己配置目录也是可以的。
使用默认配置

> ./configure

### 8.编译安装

> make
> make install

查找安装路径：

> whereis nginx


## Nginx的模块组成


Nginx的模块从结构上分为核心模块、基础模块和第三方模块：

- 核心模块：HTTP模块、EVENT模块和MAIL模块
- 基础模块：HTTP Access模块、HTTP FastCGI模块、HTTP Proxy模块和HTTP Rewrite模块，
- 第三方模块：HTTP Upstream Request Hash模块、Notice模块和HTTP Access Key模块。

Nginx的高并发得益于其采用了epoll模型，与传统的服务器程序架构不同，epoll是linux内核2.6以后才出现的。Nginx采用epoll模型，异步非阻塞，而Apache采用的是select模型。

- Select特点：select 选择句柄的时候，是遍历所有句柄，也就是说句柄有事件响应时，select需要遍历所有句柄才能获取到哪些句柄有事件通知，因此效率是非常低。
- epoll的特点：epoll对于句柄事件的选择不是遍历的，是事件响应的，就是句柄上事件来就马上选择出来，不需要遍历整个句柄链表，因此效率非常高。



## Nginx常用命令

nginx 环境变量配置：

> export PATH=$PATH:/usr/servers/nginx/sbin

- 查看nginx进程 
ps -ef|grep nginx


- 启动nginx
nginx
启动结果显示nginx的主线程和工作线程，工作线程的数量跟nginx.conf中的配置参数worker_processes有关。

- 平滑启动nginx 
kill -HUP `cat /var/run/nginx.pid` 
或者
nginx -s reload

- 强制停止nginx 
pkill -9 nginx

- 检查对nginx.conf文件的修改是否正确 
nginx -t 

- 停止nginx的命令
nginx -s stop或者pkill nginx

- 查看nginx的版本信息
nginx -v

- 查看完整的nginx的配置信息 
nginx -V


## Nginx的配置

通常情况下，Nginx的配置在Ngix的安装目录下的/conf/config.default 文件里,基本配置如下：


```
worker_process # 表示工作进程的数量，一般设置为cpu的核数

worker_connections # 表示每个工作进程的最大连接数

server{} # 块定义了虚拟主机
    listen # 监听端口
    server_name # 监听域名
    location {} # 是用来为匹配的 URI 进行配置，URI 即语法中的“/uri/”
    location /{} # 匹配任何查询，因为所有请求都以 / 开头
        root # 指定对应uri的资源查找路径，这里html为相对路径，完整路径为
        # /opt/nginx-1.7.7/html/
        index # 指定首页index文件的名称，可以配置多个，以空格分开。如有多
        # 个，按配置顺序查找。

```

location 常用配置如下：


模式 | 含义
---|---
location = /uri | = 表示精确匹配，只有完全匹配上才能生效
location ^~ /uri |  ^~ 开头对URL路径进行前缀匹配，并且在正则之前。
location ~ pattern |  开头表示区分大小写的正则匹配
location ~* pattern | 开头表示不区分大小写的正则匹配
location /uri  | 不带任何修饰符，也表示前缀匹配，但是在正则匹配之后
location / |  通用匹配，任何未匹配到其它location的请求都会匹配到，相当于switch中的default


Nginx的常用配置非常多，以下内容摘自于布尔教育课件，仅供参考：


```

#定义Nginx运行的用户和用户组
user  www www;
#启动进程,通常设置成和cpu的数量相等
worker_processes  8;
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;
#为每个进程分配cpu，上例中将8个进程分配到8个cpu，当然可以写多个，或者将一个进程分配到多个cpu。
worker_rlimit_nofile 102400;
#这个指令是指当一个nginx进程打开的最多文件描述符数目，理论值应该是最多打
#开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀
#，所以最好与ulimit -n的值保持一致。

#全局错误日志及PID文件
error_log  /usr/local/nginx/logs/error.log; 
#错误日志定义等级，[ debug | info | notice | warn | error | crit ]
pid        /usr/local/nginx/nginx.pid;

#一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（系统的值ulimit -n）与nginx进程数相除，但是nginx分配请求并不均匀.
#所以建议与ulimit -n的值保持一致。
worker_rlimit_nofile 65535;

#工作模式及连接数上限
events {
    use   epoll;             	#epoll是多路复用IO(I/O Multiplexing)中的一种方式,但是仅用于linux2.6以上内核,可以大大提高nginx的性能
    worker_connections  102400;	#单个后台worker process进程的最大并发链接数 （最大连接数=连接数*进程数）
    multi_accept on; #尽可能多的接受请求
}
#设定http服务器，利用它的反向代理功能提供负载均衡支持
http {
    #设定mime类型,类型由mime.type文件定义
    include       mime.types;
    default_type  application/octet-stream;
    #设定日志格式
    access_log    /usr/local/nginx/log/nginx/access.log;
	 sendfile      on;
    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用必须设为 on
	#如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
	#autoindex  on;  #开启目录列表访问，合适下载服务器，默认关闭。
	tcp_nopush on; #防止网络阻塞
	keepalive_timeout 60;
	#keepalive超时时间，客户端到服务器端的连接持续有效时间,当出现对服务器的后,继请求时,keepalive-timeout功能可避免建立或重新建立连接。
    tcp_nodelay   on; #提高数据的实时响应性
   #开启gzip压缩
   gzip on;
	gzip_min_length  1k;
	gzip_buffers     4 16k;
	gzip_http_version 1.1;
	gzip_comp_level 2; #压缩级别大小，最大为9，值越小，压缩后比例越小，CPU处理更快。
	#值越大，消耗CPU比较高。
	gzip_types       text/plain application/x-javascript text/css application/xml;
	gzip_vary on;
	client_max_body_size 10m;      #允许客户端请求的最大单文件字节数
    client_body_buffer_size 128k;  #缓冲区代理缓冲用户端请求的最大字节数，
    proxy_connect_timeout 90;      #nginx跟后端服务器连接超时时间(代理连接超时)
    proxy_send_timeout 90;         #后端服务器数据回传时间(代理发送超时)
    proxy_read_timeout 90;         #连接成功后，后端服务器响应时间(代理接收超时)
    proxy_buffer_size 4k;          #设置代理服务器（nginx）保存用户头信息的缓冲区大小
    proxy_buffers 4 32k;           #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
    proxy_busy_buffers_size 64k;   #高负荷下缓冲大小（proxy_buffers*2）
	
    #设定请求缓冲
    large_client_header_buffers  4 4k;
	client_header_buffer_size 4k;
	#客户端请求头部的缓冲区大小，这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k
	#不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得。
	open_file_cache max=102400 inactive=20s;
	#这个将为打开文件指定缓存，默认是没有启用的，max指定缓存数量，建议和打开文件数一致，inactive是指经过多长时间文件没被请求后删除缓存。
	open_file_cache_valid 30s;
	#这个是指多长时间检查一次缓存的有效信息。
	open_file_cache_min_uses 1;
	#open_file_cache指令中的inactive参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个文件在inactive
    #包含其它配置文件，如自定义的虚拟主机
    include vhosts.conf;
}

```

配置详解2如下：


```

	#这里为后端服务器wugk应用集群配置，根据后端实际情况修改即可，tdt_wugk为负载均衡名称，可以任意指定
	#但必须跟vhosts.conf虚拟主机的pass段一致，否则不能转发后端的请求。weight配置权重，在fail_timeout内检查max_fails次数，失败则剔除均衡。
	upstream tdt_wugk {
		server   127.0.0.1:8080 weight=1 max_fails=2 fail_timeout=30s;
		server   127.0.0.1:8081 weight=1 max_fails=2 fail_timeout=30s;
	}
   #虚拟主机配置
	server {
		#侦听80端口
        listen       80;
        #定义使用www.wuguangke.cn访问
        server_name  www.wuguangke.cn;
        #设定本虚拟主机的访问日志
        access_log  logs/access.log  main;
			root   /data/webapps/wugk;  #定义服务器的默认网站根目录位置
        index index.php index.html index.htm;   #定义首页索引文件的名称
        #默认请求
        location ~ /{
          root   /data/www/wugk;      #定义服务器的默认网站根目录位置
          index index.php index.html index.htm;   #定义首页索引文件的名称
          #以下是一些反向代理的配置.
		  proxy_next_upstream http_502 http_504 error timeout invalid_header;
		  #如果后端的服务器返回502、504、执行超时等错误，自动将请求转发到upstream负载均衡池中的另一台服务器，实现故障转移。
          proxy_redirect off;
          #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		   proxy_pass  http://tdt_wugk;     #请求转向后端定义的均衡模块
       }
	   
        # 定义错误提示页面
			error_page   500 502 503 504 /50x.html;  
            location = /50x.html {
            root   html;
        }
		#配置Nginx动静分离，定义的静态页面直接从Nginx发布目录读取。
		location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|txt|js|css)$
		{
			root /data/www/wugk;
			#expires定义用户浏览器缓存的时间为3天，如果静态页面不常更新，可以设置更长，这样可以节省带宽和缓解服务器的压力。
			expires      3d;
		}
        #PHP脚本请求全部转发到 FastCGI处理. 使用FastCGI默认配置.
        location ~ \.php$ {
            root /root;
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME /data/www/wugk$fastcgi_script_name;
            include fastcgi_params;
        }
        #设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status  on;
        }
     }
}

```

## Nginx 内置绑定变量



名称  | 说明 
---|---
$arg_name |请求中的name参数
$args |请求中的参数
$binary_remote_addr | 远程地址的二进制表示
$body_bytes_sent | 已发送的消息体字节数
$content_length HTTP | 请求信息里的"Content-Length"
$content_type | 请求信息里的"Content-Type"
$document_root | 针对当前请求的根路径设置值
$document_uri | 与$uri相同; 比如 /test2/test.php
$host | 请求信息中的"Host"，如果请求中没有Host行，则等于设置的服务器名
$hostname | 机器名使用 gethostname系统调用的值
$http_cookie | cookie 信息
$http_referer | 引用地址
$http_user_agent | 客户端代理信息
$http_via | 最后一个访问服务器的Ip地址。
$http_x_forwarded_for | 相当于网络访问路径
$is_args | 如果请求行带有参数，返回“?”，否则返回空字符串
$limit_rate | 对连接速率的限制
$nginx_version |当前运行的nginx版本号
$pid worker | 进程的PID
$query_string | 与$args相同
$realpath_root | 按root指令或alias指令算出的当前请求的绝对路径。其中的符号链接都会解析成真是文件路径，使用 Nginx 内置绑定变量
207$remote_addr | 客户端IP地址
$remote_port | 客户端端口号
$remote_user | 客户端用户名，认证用
$request | 用户请求
$request_body | 这个变量（0.7.58+） 包含请求的主要信息。在使用proxy_pass或fastcgi_pass指令的location中比较有意义
$request_body_file | 客户端请求主体信息的临时文件名
$request_completion | 如果请求成功，设为"OK"；如果请求未完成或者不是一系列请求中最后一部分则设为空
$request_filename | 当前请求的文件路径名，比如/opt/nginx/www/test.php
$request_method | 请求的方法，比如"GET"、"POST"等
$request_uri | 请求的URI，带参数
$scheme |所用的协议，比如http或者是https
$server_addr | 服务器地址，如果没有用listen指明服务器地址，使用这个变量将发起一次系统调用以取得地址(造成资源浪费)
$server_name | 请求到达的服务器名
$server_port | 请求到达的服务器端口号
$server_protocol | 请求的协议版本，"HTTP/1.0"或"HTTP/1.1"
$uri|  请求的URI，可能和最初的值有不同，比如经过重定向之类的

## 参考资料

http://www.linuxidc.com/Linux/2016-09/134907.htm

https://moonbingbing.gitbooks.io/openresty-best-practices/content/ngx/nginx.html

布尔教育课件