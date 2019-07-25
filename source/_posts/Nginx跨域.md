---
title: Nginx - 跨域/防盗链/缓存/压缩/Https
tags:
  - Nginx
---

Nginx跨域/防盗链/缓存/压缩/Https

#### 跨域解决

流程：

- 当chrome发现ajax请求的网址，与当前主域名不一致（跨域）时，会在请求header中追加值页面主域名值，即：origin = http://static.enjoy.com
- nginx在接收到ajax请求时，会查看origin值，即请求我的网址是谁？此处使用正则来校验，即：只要是enjoy.com下的网址，都允许访问我，返回信息时，nginx追加header值：access-control-allow-origin = static.enjoy.com（回答浏览器，static域名网址可以访问我）
- chrome收到ajax返回值后，查看返回的header中access-control-allow-origin的值，发现其中的值是static.enjoy.com,正是当前的页面主域名。这是允许访问，于是执行ajax返回值内容。（ps：若此处access-control-allow-origin不存在，或者值不是static域名，chrome就拒绝执行返回值）

<!-- more -->

```sh
server{
	listen 80;
	server-name www.enjoy.com;

	if($http_origin ~ http://(.*).enjoy.com){
		set $allow_url $http_origin;
	}
	#是否允许请求带有验证信息
	add_header Access-Control-Allow-Credentials true;
	#允许跨域访问的域名,可以是一个域的列表，也可以是通配符*
	add_header Access-Control-Allow-Origin  $allow_url;
	#允许脚本访问的返回头
	add_header Access-Control-Allow-Headers 'x-requested-with,content-type,Cache-Control,Pragma,Date,x-timestamp';
	#允许使用的请求方法，以逗号隔开
	add_header Access-Control-Allow-Methods 'POST,GET,OPTIONS,PUT,DELETE';
	#允许自定义的头部，以逗号隔开,大小写不敏感
	add_header Access-Control-Expose-Headers 'WWW-Authenticate,Server-Authorization';
	#P3P支持跨域cookie操作
	add_header P3P 'policyref="/w3c/p3p.xml", CP="NOI DSP PSAa OUR BUS IND ONL UNI COM NAV INT LOC"';
	add_header test  1;

	 if ($request_method = 'OPTIONS') {
             return 204;
         }

}
```







#### 防盗链

目的：

1.	让资源只能在我的页面内显示
2.	不能单独来取或者下载

流程：

1.	chrome以url1首次请求web服务器，得到html页面。
2.	chrome再次发起url2资源请求，携带referers = url1。（注意，是url1，不是本次的url2）
3.	nginx校验referers值，决定是否允许访问。
4.	valid_referers：匹配域名白名单，如果不匹配，把内置变量$invalid_referers置为1，进入if块，返回404

```sh
location ^~ /mall {
		valid_referers *.enjoy.com;
    	if ($invalid_referer) {
    		return 404;
    	}
        root html/gzip;
     }
```

#### 缓存

```sh
location ^~ /qq.png {
	#	expires 2s;#缓存2秒
		expires 2m;#缓存2分钟
	#	expires 2h;#缓存2小时
	#	expires 2d;#缓存2天
		root html/gzip;
	}
```

#### 压缩

带宽资源很贵 /html/js/css压缩，/images不压缩			
过程：nginx压缩 ----》网络传输 ---》chrome解压（压缩和解压消耗cpu）

1. 浏览器携带支持的解压方式
2. 浏览器与nginx的交互

```sh
location ~ /(.*)\.(html|js|css|jpg|jpeg|png|gif)$ {#覆盖/re/a.htm路径
		gzip on; # 启用gzip压缩，默认是off，不启用
		
		# 对js、css、jpg、png、gif格式的文件启用gzip压缩功能
		gzip_types application/javascript text/css image/jpeg image/png image/gif;
		gzip_min_length 1024; # 所压缩文件的最小值，小于这个的不会压缩
		gzip_buffers 4 1k; # 设置压缩响应的缓冲块的大小和个数，默认是内存一个页的大小
		gzip_comp_level 1; # 压缩水平，默认1。取值范围1-9，取值越大压缩比率越大，但越耗cpu时间
		
		root html/gzip;
	}
```

#### Https

nginx配置https的时候，需要两个东西：（此两样需购买）  		
一个key，私钥。放在nginx服务器里面，仅此一份		
一个证书，公钥，供浏览器去下载。		

> nginx -V 需要有--with-http_ssl_module,表示安装了https模块。否则需要重新安装。

配置如下：

```sh
ssl_certificate      /etc/nginx/conf.d/server.crt;
ssl_certificate_key  /etc/nginx/conf.d/server_nopass.key;
```










