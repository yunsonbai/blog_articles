---
title: nginx实现https
date: 2016-06-09 22:27:22
tags:
	- nginx
    - https
    - nginx实现https
    - ssl
    - ios访问https的1004问题补充
categories: nginx
---

## 引言

``` bash
    nginx是一个高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP服务器，下边说一下nginx关于https的配置
```

<!--more-->
## 使用ssl生成证书
### 生成命令
``` bash
	# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout 
    /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt
```

### 输入完命令后的提示
``` bash
	提示如下，一步一步完成即可（也可以使用默认，也可随意）：
	Country Name (2 letter code) [AU]:China
    State or Province Name (full name) [Some-State]: Beijing
    Locality Name (eg, city) []:Beijing
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:Bouncy Castles, Inc.
    Organizational Unit Name (eg, section) []:Ministry of Water Slides
    Common Name (e.g. server FQDN or YOUR name) []:your_domain.com
    Email Address []:admin@your_domain.com
```

## 配置nginx
### 打开.conf文件
``` bash
	cd /etc/nginx、nginx.conf文件
    vim nginx.conf
```

### 添加如下配置
``` bash

	server{
	     listen 80;
	     server_name www.xxx.xxx;
	}
	server{
		listen 443 ssl;
	    ssl_certificate /etc/nginx/ssl/nginx.crt;
	    ssl_certificate_key /etc/nginx/ssl/nginx.key;
	    location /static/ {
	    	alias   /test/static/;
	        expires 24h;
	    } 
	}

```

## 测试
### 启动nginx
``` bash
	nginx -c /etc/nginx/nginx.conf
```

### 创建静态页
```
	cd /test/static
	touch test.html
	echo 'test https' > test.html
```

### 访问
``` bash
	在浏览器中访问https:127..0.1/static/test.html
```

## 补充
``` bash
	最近发现了一个很奇怪的问题，就是使用python在访问https下的链接的时候一切正常
	然而在ios9下访问就会时不时的出现1004问题（访问不到）,最后的解决办法是使用最
	新的nginx解决
```

[nginx常用配置](http://www.yunsonbai.top/2016/06/09/nginx%E5%B8%B8%E7%94%A8%E9%85%8D%E7%BD%AE/)