---
title: Nginx简介
tags:
  - Nginx
---

Nginx简介

#### Nginx作用

- 路由功能（与微服务对应）：域名/路径，进行路由选择后台服务器
- 负载功能（与高并发高可用对应）：对后台服务器集群进行负载
- 静态服务器（比tomcat性能高很多）：在mvvm模式中，充当文件读取职责

> 总结：实际使用中，这三项功用，会混合使用。比如先分离动静，再路由服务，再负载机器


#### Nginx安装

源码编译方式：

```sh

#一般系统中已经装了了make和g++，无须再装
yum -y install autoconf automake make
yum -y install gcc gcc-c++ 

#安装nginx依赖的库
yum -y install pcre pcre-devel    
yum -y install zlib zlib-devel
yum install -y openssl openssl-devel

# 下载 安装
wget  http://nginx.org/download/nginx-1.9.15.tar.gz
tar -zxvf nginx-1.9.0.tar.gz
cd nginx-1.9.0

# 后面的模块需要一起安装
./configure   --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module 

# 启动
nginx

# 停止 
nginx -s stop

#重启
nginx -s reload
```

<!-- more -->

#### nginx模型概念

Nginx会按需同时运行多个进程：

- 一个主进程(master)和几个工作进程(worker)，配置了缓存时还会有缓存加载器进程(cache loader)和缓存管理器进程(cache manager)等。 
- 所有进程均是仅含有一个线程，并主要通过“共享内存”的机制实现进程间通信。 
- 主进程以root用户身份运行，而worker、cache loader和cache manager均应以非特权用户身份（user配置项）运行。
 

主进程主要完成如下工作：

1. 读取并验正配置信息；
2. 创建、绑定及关闭套接字；
3. 启动、终止及维护worker进程的个数；
4. 无须中止服务而重新配置工作特性；
5. 重新打开日志文件；

worker进程主要完成的任务包括：

1. 接收、传入并处理来自客户端的连接；
2. 提供反向代理及过滤功能；
3. nginx任何能完成的其它任务；

#### nginx配置文件

例:

```sh
#user www-data;
worker_processes  4;

error_log  /var/log/nginx/error.log;
pid        /var/run/nginx.pid;

events {
	use epoll;
    worker_connections 2048;
    # multi_accept on;
}

http {
#    include       /etc/nginx/mime.types;

    types_hash_max_size 2048;
    server_names_hash_bucket_size 64;

    log_format custom '$remote_addr - $remote_user [$time_local] '
                      '"$request" $status $request_time $body_bytes_sent '
                      '"$http_referer" "$http_user_agent" "$request_body"';

    access_log  /var/log/nginx/access.log custom;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  120;
    tcp_nodelay        on;

    proxy_connect_timeout 120;
    proxy_send_timeout 120;
    proxy_read_timeout 120;
    send_timeout       120;

    gzip  on;
    gzip_disable "MSIE [1-6]\.(?!.*SV1)";

    upstream backend {
      server localhost:3456 ;
      server localhost:4567 ;
      server localhost:5678 ;
      server localhost:6789 ;
    }

    upstream manage {
      server 127.0.0.1:8080 ;
    }

    upstream web {
      server localhost:1603 ;
    }

    server {
       listen 80;
       location / {
         proxy_set_header Referer "";
         proxy_pass http://manage/;
       }
       location /api {
         proxy_set_header Referer "";
         proxy_pass http://backend/api;
         client_max_body_size    30m;
       }
       location /trig {
         proxy_set_header Referer "";
         proxy_pass http://backend/trig;
       }
    }

    server {
       listen 1604;
       location / {
         proxy_set_header Referer "";
         proxy_pass http://web/;
       }
    }
}
```

全局属性配置：

- user :主模块命令， 指定Nginx的worker进程运行用户以及用户组，默认由nobody账号运行。       
- worker_processes: 指定Nginx要开启的进程数。
- error log:用来定义全局错设日志文件的路径和日志名称。日志输出级别有debug，info，notice，warn，error，crit可供选择，其中debug输出日志最为详细，面crit（严重）输出日志最少。默认是error

- pid: 用来指定进程id的存储文件位置。
- event：设定nginx的工作模式及连接数上限，其中参数use用来指定nginx的工作模式（这里是epoll，epoll是多路复用IO(I/O Multiplexing)中的一种方式）,nginx支持的工作模式有select ,poll,kqueue,epoll,rtsig,/dev/poll。其中select和poll都是标准的工作模式，kqueue和epoll是高效的工作模式，对于linux系统，epoll是首选。worker_connection是设置nginx每个进程最大的连接数，默认是1024，所以nginx最大的连接数max_client=worker_processes * worker_connections。进程最大连接数受到系统最大打开文件数的限制，需要设置ulimit。

http服务器相关属性的设置：

- http 

server段虚拟主机的配置:

- server 

#### Nginx日志 - log_format

```sh
log_format custom '$remote_addr - $remote_user [$time_local] '
                      '"$request" $status $request_time $body_bytes_sent '
                      '"$http_referer" "$http_user_agent" "$request_body"';

access_log  /var/log/nginx/access.log custom;
```

access_log: 配置日志位置 和 输出格式

格式设置:

| 配置 | 作用 |
| ------ | ------ |
| $remote_addr | 客户端的ip地址(代理服务器，显示代理服务ip) |
| $remote_user | 用于记录远程客户端的用户名称（一般为“-”） |
| $time_local | 用于记录访问时间和时区 |
| $request | 用于记录请求的url以及请求方法 |
| $status | 响应状态码，例如：200成功、404页面找不到等。 |
| $body_bytes_sent | 给客户端发送的文件主体内容字节数 |
| $http_user_agent | 用户所使用的代理（一般为浏览器） |
| $http_x_forwarded_for | 可以记录客户端IP，通过代理服务器来记录客户端的ip地址 |
| $http_referer | 可以记录用户是从哪个链接访问过来的 |


#### 日志配置和及切割

```sh
/etc/init.d/rsyslog start  #系统日志，如不开启，看不到定时任务日志
/etc/rc.d/init.d/crond start	#定时任务开启
```
编写sh：

```sh
#!/bin/bash
#设置日志文件存放目录
LOG_HOME="/usr/local/nginx/logs/"
#备分文件名称
LOG_PATH_BAK="$(date -d yesterday +%Y%m%d%H%M)"
#重命名日志文件
mv ${LOG_HOME}/access.log ${LOG_HOME}/access.${LOG_PATH_BAK}.log
mv ${LOG_HOME}/error.log ${LOG_HOME}/error.${LOG_PATH_BAK}.log
#向nginx主进程发信号重新打开日志
kill -USR1 `cat ${LOG_HOME}/nginx.pid`
```
配置cron：

```sh
*/1 * * * * /usr/local/nginx/sbin/logcut.sh
```

> 可写脚本将日志存入mongo








