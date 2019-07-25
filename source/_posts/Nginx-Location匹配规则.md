---
title: Nginx - Location匹配规则 和 负载均衡
tags:
  - Nginx
---

Location匹配规则 和 负载均衡

#### Location规则


- 语法规则： location [=|~|~*|^~] /uri/ {… }


```sh
location / {
         proxy_set_header Referer "";
         proxy_pass http://manage/;
       }
```

<!-- more -->

- 首先匹配 =，其次匹配^~,其次是按文件中顺序的正则匹配，最后是交给通用匹配。当有匹配成功时候，停止匹配，按当前匹配规则处理请求。


| 符号 | 含义 |
|----|----|
|=|开头表示精确匹配|
|^~|^~开头表示uri以某个常规字符串开头，理解为匹配 url路径即可。nginx不对url做编码，因此请求为/static/20%/aa，可以被规则^~ /static/ /aa匹配到（注意是空格）|
|~|~开头表示区分大小写的正则匹配|
|~*|~* 开头表示不区分大小写的正则匹配|
|!~和!~*|!~和!~*分别为区分大小写不匹配及不区分大小写不匹配的正则|
|/|用户所使用的代理（一般为浏览器）|

#### Nginx 负载均衡

```sh
# http模块
upstream backend {
      server localhost:3456 ;
      server localhost:4567 ;
      server localhost:5678 ;
      server localhost:6789 ;
    }

# server模块
location /api {
         proxy_set_header Referer "";
         proxy_pass http://backend/api;
         client_max_body_size    30m;
       }
```

负载均衡规则：

- 轮询(默认)
- weight

```sh
upstream nginx {
server 172.17.0.4:8081 weight=2;
server 172.17.0.5:8081 weight=1;
}
```

- ip_hash 同一客户端的请求总是发往同一个后端服务器，可以解决session的问题。

```sh
upstream nginx {
ip_hash;
server 172.17.0.4:8081;
server 172.17.0.5:8081;
}
```

#### location和rewrite进阶

- nginx运行阶段：  rewrite 阶段、access 阶段以及 content 阶段
- 不按代码顺序执行，是按阶段执行，顺序如下：

例 先执行命中的所有rewrite层指令（下面的set），再执行access，再执行content（下面的echo）
：

```sh
location  = / {
        set $a 32;
        echo $a;

        set $a 64;
        echo $a;
}
```

输出两个64








