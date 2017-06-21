---
title: 如何提高django的并发能力
date: 2017-06-15 18:43:54
tags:
	- 提高django的并发能力
	- gthread
    - gevent
	- gunicorn+django
	- gunicorn+gthread+django
	- gunicorn+gevent+django
categories: python
---
[原文连接](https://yunsonbai.top/2017/06/15/gunicorn-django/)
## 引言

``` bash
    手头上的项目有一些采用django框架编写, 如果说并发量比较小的时候简单的
    runserver是可以应对的，那么当并发达到一两千的时候，该怎么提高django
    的并发能力呢？
```

<!--more-->
## Overview

* 环境说明：
	* python: 3.5
	* django: 1.8.2
	* gunicorn: 19.7.1
* 系统：
	* 服务器: centos 4核
	* 压测机器: centos 4核
* 压测环境
	* siege
	* 4核centos测试机
* 为什么用django
	* 开发效率高
	* 好上手
* 关于gunicorn
	* Gunicorn 'Green Unicorn' is a Python WSGI HTTP Server for UNIX.It's a pre-fork worker model. The Gunicorn server is broadly compatible with various web frameworks, simply implemented, light on server resources, and fairly speedy.(这是官方给出的回答)


## 压测方式及命令
* 压测方式：![lc](https://yunsonbai.github.io/images/django/lc.png)
* 压测命令：siege -c255 -t200S -v -b 'http://B_ip:8080/test POST appid=111'



## 本次实验业务场景
![lc](https://yunsonbai.github.io/images/django/db_c.png)


## 代码展示
### settings部分
```python
# 这里我们用mysql，其他配置都是默认
DATABASES = {
	'default': {
		'ENGINE': 'django.db.backends.mysql',
		'NAME': 'ce',
		'USER': 'root',
		'PASSWORD': '',
		'HOST': '192.168.96.95',
		'PORT': '3306',
		# 'CONN_MAX_AGE': 600,
	}
}
```

### models部分
```python
class Test(models.Model):
	url = models.CharField(max_length=228, blank=True, null=True)
	img_url = models.CharField(max_length=228, blank=True, null=True)
	title = models.CharField(max_length=228, blank=True, null=True)
	content = models.CharField(max_length=228, blank=True, null=True)

	class Meta:
		db_table = 'test'
		verbose_name = "test表"

	def __unicode__(self):
		return self.id
```

### views部分
```python
class Test(APIView):

	def post(self, requsts):
		Test.objects.create(
			**{'url': str(1000000 * time.time())})
		return Response({"status": 200})
```

## 开始压测
### 数据说明
```
目前数据库test表的数据量是, 其中id是自增主键
MySQL [ce]> select id from test order by id desc limit 2;
+--------+
| id     |
+--------+
| 627775 |
| 627774 |
+--------+
```

### runserver方式压测结果
```python
Lifting the server siege...      done.

Transactions:		       24041 hits
Availability:		       99.93 %
Elapsed time:		       99.60 secs
Data transferred:	        0.32 MB
Response time:		        1.03 secs
Transaction rate:	      241.38 trans/sec  # 并发量只有241
Throughput:		        0.00 MB/sec
Concurrency:		      248.94
Successful transactions:       24041
Failed transactions:	          16
Longest transaction:	       32.55
Shortest transaction:	        0.05
```

### gunicorn+gevent(4个worker)
```python
Lifting the server siege...      done.

Transactions:		       23056 hits
Availability:		      100.00 %
Elapsed time:		       99.49 secs
Data transferred:	        0.31 MB
Response time:		        1.09 secs
Transaction rate:	      231.74 trans/sec # 并发量只有231
Throughput:		        0.00 MB/sec
Concurrency:		      252.95
Successful transactions:       23056
Failed transactions:	           0
Longest transaction:	        8.21
Shortest transaction:	        0.01
```

### gunicorn+gthread(4个worker, --threads=50)
#### 启动方式
[官方有相应说明]((http://docs.gunicorn.org/en/latest/settings.html)
```python
gunicorn --env DJANGO_SETTINGS_MODULE=ce.settings ce.wsgi:application -w 4 -b 0.0.0.0:8080 -k gthread --threads 40 --max-requests 4096 --max-requests-jitter 512
```
#### 压测结果
```python
启动方式：

done.
siege aborted due to excessive socket failure; you
can change the failure threshold in $HOME/.siegerc

Transactions:		       28231 hits
Availability:		       95.67 %
Elapsed time:		       30.71 secs
Data transferred:	        0.41 MB
Response time:		        0.27 secs
Transaction rate:	      919.28 trans/sec  # 提高了不少吧，能不能在提高？
Throughput:		        0.01 MB/sec
Concurrency:		      251.06
Successful transactions:       28231
Failed transactions:	        1278        # 但是失败的有些多
Longest transaction:	        8.06
Shortest transaction:	        0.01
```

### gunicorn+gthread+CONN_MAX_AGE(4个worker, --threads=50)
[关于CONN_MAX_AGE](https://docs.djangoproject.com/en/1.11/ref/databases/)
```python
CONN_MAX_AGE: 复用数据库链接

Lifting the server siege...      done.

Transactions:		      110289 hits
Availability:		       99.62 %
Elapsed time:		       99.65 secs
Data transferred:	        1.47 MB
Response time:		        0.23 secs
Transaction rate:	     1106.76 trans/sec  # 这次又提升了不少啊
Throughput:		        0.01 MB/sec
Concurrency:		      253.84
Successful transactions:      110289
Failed transactions:	         422
Longest transaction:	        3.85
Shortest transaction:	        0.01
```

### 能不能gunicorn+gevent+CONN_MAX_AGE(4个worker)
```python
这里我不建议使用，这样的话你的数据库连接数会飚的很高，服务会挂的很惨, 毕竟数据库是不会允许无休止的建立连接的
```


## 如何再次增加并发量
### 采用nginx做负载
![nginx负载](https://yunsonbai.github.io/images/django/nf.png)

### 去掉自增主键

```python
原因很简单,因为自增主键的存在写库存在抢锁, 可以利用全局id生成器提前生成
id直接写入数据库
```


### 换成异步任务去写库

```python
如果数据只是存在mysql中做备份，建议使用异步的方式写入库，先把数据写到缓
存下发给用户，之后在利用后台异步任务一点点的写入，例如聊天系统可以这样干
```


### 换成更高效的框架或者语言

```python
可以试试tornado, 如果tornado依然无法满足，可以尝试使用golango，毕竟
golang是以高并发著称, 而且是编译语言，而且基于它的web框架也很容易上手，
性能很可观，例如Iris
```
[Iris官方网站](http://iris-go.com)