---
title: 让自己的博客支持https
date: 2017-01-12 10:23:14
tags:
    - https
    - 博客支持https
    - 免费https
categories: https
---

## 引言

``` bash
    昨天把博客迁到了https下，现在博客支持https访问了。本文适合自己用机器搭建的博客。
```

<!--more-->
## 起因
```python
	首先之前就已经听说从2017年1月份起，Chrome浏览器将会把采用HTTP协议的网站标记为“不安全”网站；另外最近自己手头的很多项目都开始往https下迁移；还有就是用谷歌浏览器看自己的博客一直提示感叹号；最后最让我郁闷的是某运营商的劫持，种种这些原因促使自己的让博客支持https访问。
```

## https请求流程

![Catch8D7B](https://yunsonbai.github.io/images/https/lc.jpg)

## 实现方案
* 简单方式： 把博客部署在github上，直接就能https访问
	例如[yunsonbai_github](https://yunsonbai.github.io/)
* 稍微麻烦点的方式： 自己购买服务器部署上博客后，找认证机构办法签证（我采用的这个方式）
	例如[yunsonbai](https://yunsonbai.top)
## 认证机构
* 收费的
	* geotrust
* 免费的
	* Let's Encrypt(本次介绍如何使用这个)

## 具体实现
* 创建一个临时文件ssl(存放过程中需要用到的临时文件)
	$ mkdir ssl
	$ cd ssl

* 创建一个 RSA 私钥用于 Let's Encrypt 识别你的身份
	$ openssl genrsa 4096 > account.key

* 生成 CSR
	* 创建 RSA 私钥
		$ openssl genrsa 4096 > domain.key
		$ openssl req -new -sha256 -key domain.key -out domain.csr(基本都能看懂，都是跟地址相关的东西)

* 配置nginx(针对nginx)

```python
	location ^~ /.well-known/acme-challenge/ {
        alias /usr/local/nginx/html/.well-known/acme-challenge/;
        try_files $uri =404;
    }
	
	为什么要用到.well-known/acme-challenge/文件，Let's Encrypt在验证网站是否归你所有的时候会访问该目录。

```

* 获取网站证书
	
    * 利用Let's Encrypt官网提供的脚本工具
	* 利用开源项目acme_tiny(我用的这个，不错)
    	$ wget https://raw.githubusercontent.com/diafygi/acme-tiny/master/acme_tiny.py
    	$ python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir /usr/local/nginx/html/.well-known/acme-challenge/ > ./signed.crt

    	```
    	一切正常的话不会报任何错误，切记/usr/local/nginx/html/.well-known/acme-challenge/路径是你的博客相关路径
    	```

## 配置nginx
```python
    到目前为止拿到了证书
```
* 把刚才ssl文件移到你认为比较靠谱的目录下(例如 /usr/ssl)
* 配置nginx

```python
	server{
		listen 443 ssl;
		ssl_certificate /usr/ssl/signed.crt;
		ssl_certificate_key /usr/ssl/domain.key;
		location / {
				root   html;
				index  index.html index.htm;
			}
	}
```

也可以参考[nginx实现https](http://yunsonbai.github.io/2016/06/09/nginx-https%E9%85%8D%E7%BD%AE/)

* 将http请求转到https下

```python
	server {
		listen       80;
        server_name  xxx.xx;
		location / {
			rewrite ^/(.*)$ https://xxx.xx/$1 permanent;
		}
	}
```


## 需要注意的地方
```python
	Let's Encrypt的证书只有90天有效期，所以你要美90天续签一次，最简单的方式就是利用crontab来做这个是，说白了定期执行这条python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir /usr/local/nginx/html/.well-known/acme-challenge/ > ./signed.crt命令
```


## 大功告成
```python
尝试访问一下https://yoursite.com
```