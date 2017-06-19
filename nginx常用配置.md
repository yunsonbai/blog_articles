---
title: nginx常用配置
date: 2016-06-09 22:27:22
tags:
	- nginx
    - 配置
    - nginx常用配置
    - 反向代理
    - 设置cookie
categories: nginx
---

## 引言

``` bash
    nginx是一个高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP服务器，下边说一下这断时间用到的nginx配置
```

<!--more-->
## 反向代理
### 第一种
``` bash
	upstream forum  {
		server 127.0.0.1:8000 weight 1;
		server 127.0.0.1:8001 weight 1;
	}
	server{
        listen 80;
        location / {
            proxy_pass         http://forum;
            proxy_set_header   Host             $host;
            proxy_set_header   X-Real-IP        $remote_addr;
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
	}
```

### 第二种
``` bash
	server{
		location /static/ {
			proxy_set_header Host test.xx.com;
			proxy_set_header Origin http://test.xx.com;
			proxy_pass    http://test.xx.com/static1/; #可以不是代理的url
		}
	}
```

## 设置与打印cookies
### 设置
``` bash
	server{
        listen 80;
        server_name test.xx.com;
        add_header Set-Cookie "name=baisong";
	}
```

### 打印
``` bash
	log_format main	'$http_cookie,...'
```

## 静态代理
``` bash
	server {
		location /static/ {
        	alias   /test/static/;
			expires 24h;
		} 
	}
```

## 其他
### 进程数指定 
``` bash
	worker_processes 8; #最好和cpu数量一致
```

### 进程连接数指定
``` bash
	worker_connections 65535;
	# 每个进程允许的最多连接数， 理论上每台nginx 服务器的最大连接数为worker_processes*worker_connections。
    keepalive_timeout 60;keepalive 超时时间。
```

### 头部的缓冲区大小
``` bash
	 open_file_cache max=65535 inactive=60s;
	 #这个将为打开文件指定缓存，默认是没有启用的，max 指定缓存数量，建议和打开文件数一致，inactive 是指经过多长时间文件没被请求后删除缓存。
```
### 缓存检查
``` bash
	open_file_cache_valid 80s;
	# 这个是指多长时间检查一次缓存的有效信息。
```

### 控制缓冲区溢出攻击
``` bash
	client_body_buffer_size  1K;
	client_header_buffer_size 1k;
    client_max_body_size 1k;
    large_client_header_buffers 2 1k; 
```

### if语句
``` bash
	# 限制可用的请求方法
	if ($request_method !~ ^(GET|HEAD|POST)$ ) {
         return 403;
	}
```

### 防止图片盗链
``` bash
	只有www.example1.com example2.com 能访问
    location /images/ {
         valid_referers none blocked www.example1.com example2.com;
         if ($invalid_referer) {
              return   403;
         }
    }
```

[nginx实现https](http://www.yunsonbai.top/2016/06/09/nginx-https%E9%85%8D%E7%BD%AE/)