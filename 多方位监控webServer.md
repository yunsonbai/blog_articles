---
title: 多方位监控webServer
date: 2016-07-02 14:27:22
tags:
	- 监控
    - api监控
    - 监控系统
    - 系统监控
    - webServer监控
categories: api或系统监控
---

## 引言

``` bash
    最近趁着开发需求较少，着手整理了一段时间关于系统监控的东西，系统监控的目的就是为了快速的发现系统问题、找到问题所在、并
    解决它。下边分享一下
```

<!--more-->
## 初识监控
### 懵懂
``` bash
	记得当初来公司的时候，师傅让我在代码关键的部位打日志，初来乍到，以为日志嘛，还按照上边的写的例子接着写得了。学习期间
	知道该方法是为了帮助排错用的，但是该怎么用，怎么打并没有好好研究过，甚至还有怀疑，既然测试人员已经测试好了还有必要再在
	业务部分加吗，这样不会降低系统的处理效率吗？
```

### 第一次使用日志
``` bash
	好景不长，没多久就出了一个很奇怪的问题，大部分情况下系统正常，但是总有意外发生（而且是事后一段时间后才发现，因为当时
	没有加报错邮件），很是烦恼，苦恼时刻想到了曾经师傅让打的日志，开始用grep过滤这个意外的问题，很快找到了问题所在，多亏
	日志是仿照前辈打的（当时完全不知道人家为啥要那样打）。从那感受到了日志的魅力。
```

### 如何打
``` bash
	原则，不能因为添加监控而导致服务质量大幅度下滑, 打关键点不乱打
```

### 开始谈这段时间的结果
``` bash
	上边说的有点废话，但是对于刚毕业的学生来讲，很多人会遇到这样的情况，下边开始说说这段时间是怎么来监控我们的系统的。
```

## 报错邮件的添加
### 重要性
``` bash
	我们知道在稳定的系统，都有可能因为这样或那样的问题（硬件、网络等问题）导致出错，尤其是在系统中调用第三方接口的代
	码段，第三方的接口是不可控的，换句话说就是这些地方极易出现问题。所以这些地方需要有报警邮件及时通知。
```

### 添加报警模块
``` bash
	之前我写过一篇文章，是如何利用sentry（采用Django开发，当然有其他的可以留言讨论）来监控系统，如下是连接，sentry
	可以使全局的监控，也可以是部分代码段的监控，在部分代码段的监控上建议使用try语句监控，这样避免致服务中断，给用户不
	好的体验。
```
[利用sentry收集django的日志](https://www.yunsonbai.top/2016/05/30/django-sentry/)

## Api响应时间的监控
### 必要性
``` bash
	关于api的响应时间监控显然很重要，如果你能实时或准实时的看到你写的接口的响应时间的话，你就会很有目的性的去优化你的
	接口，就能提供更好的服务。在调用第三方接口的地方最好也能加上监控，因为有可能你接口慢就是由于第三方的接口导致的。
```

### 准实时监控
``` bash
	我叫它准实时因为过一小段时间后我们才能知道结果。
```

#### 利用nginx日志
``` bash
	在nginx日志中可以根据$request_time(全程的时间：从用户点击到所有结果返回）和$upstream_response_time（Nginx
	向后端建立连接开始到接受完数据然后关闭连接为止的时间），可以根据需求来利用这两个参数的值。当然你需要写一个日志分析
	脚本，这个不难。
```

#### 利用diamond
``` bash
	diamond是可以实现类似于定期任务的工具,可以监控系统内存、CPU使用情况等等，你可以事先写好代码片段去请求自己的接口，
	然后加到diamond的collectors中，定期执行，它能给你在单位时间内系统的监控参数。可以看一下之前的文章。
```
[使用diamond监控系统或api](https://www.yunsonbai.top/2016/06/13/python-diamond/)

### 实时监控
``` bash
	个人觉得最好只监控api中的关键部位，因为他将会影响api的整体
	响应时间。
```
#### 利用Metrology
``` bash
	import time
	from metrology import Metrology
	from metrology.reporter import GraphiteReporter

	class ApiTimer(object):

		__instance = None

		def __init__(self, key, carbon_host, carbon_port):
			self.timer = Metrology.timer(key)
			self.carbon_host = carbon_host
			self.carbon_port = carbon_port

		def reporte(self):
			# 主要负责往传输统计结果
			reporter = GraphiteReporter(self.carbon_host, self.carbon_port)
			reporter.write()

		def probe(self, func, *arg, **args):
			# 代理执行函数
			st = time.time()
			res = func(*arg, **args)
			self.timer.update(int((time.time() - st) * 1000))
			last_tick = self.timer.meter.last_tick.value
			if last_tick - self.timer.meter.start_time > 2:
				self.reporte()
				self.timer.clear()
			return res


	def fun(a):
		pass
		# time.sleep(random.random())

	if __name__ == '__main__':
		# 以下是测试用例
		# import random
		st = time.time()
		timer = ApiTimer('test1.test', 'xx.xx.xx.xx', xx)
		while time.time() - st < 20:
			timer.probe(fun, 1)
```

## 关键部位info日志添加
### 重要性
``` bash
	有时候你会遇到这样的问题：api响应时间超短，也没有报错邮件，但是就是结果不对。这一般是返回结果出了问题，遇到最多
	的就是在调用第三方接口的时候出了这样的问题，这时候就需要有info日志来帮忙了。打日志最好使用,|:分隔符分开关键的参
	数和返回值，方便过滤。
```

### 在api中关键断码段打印日志
#### 使用sentry
``` bash
	sentry除了能监控error日志，还能收集info日志，你可以尝试使用。但是sentry打的info日志会有很多额外的信息。可以
	去前边的链接中看怎么使用。
```

#### 使用rsyslog
``` bash
	该方法比较通用，不论在什么开发语言和框架下，基本上都支持对rsyslog的
	使用。我前边写了一篇在Django中使用rsyslog，可以看一下。
```
[使用rsyslog收集Django的日志](https://www.yunsonbai.top/2016/06/08/django-rsyslog/)

### 收集远程日志
``` bash
	前边说了日志打印，下边说一下如何收集这些日志。
```

#### 服务器写传输脚本，定期执行
``` bash
	有点low，但也不失为一种办法
```

#### 使用rsyslog
``` bash
	当然如果你使用的是使用的是sentry收集info日志，你不用操心怎么收集，因为sentry都帮你做好了。如果是使用的rsyslog的
	话你就需要收集服务器上的日志了。rsyslog除了可以帮你本地打印日志，还能远程收集日志。可以走默认端口514来收集，当然
	需要把web服务器上的rsyslog配置成远程传输的方式，即通过514端口输出日志到远端rsyslog日志服务器上。
```
#### 使用syslog-ng收集
``` bash
	syslog-ng收集日志的方式和rsyslog一样都是走514端口，但是syslog-ng可以很方便的根据关键词帮你进行日志过滤，这样的
	话你就不用苦恼于web应用太多而local只能从（0-7）总感觉不够用，但是关于rsyslog和syslog-ng的比较大家可以在网上搜
	搜，毕竟这里只是提供方法。
```

#### 使用ELK
``` bash
	使用ELK，何为ELK：
		e:
          elasticsearch，搜索引擎，创建索引，存储数据
     	l:
          logstash， 日志收集，过滤，传输日志到elasticsearch
     	k:
          kibana， web展示
	用这个日至系统的好处就是能直接通过浏览器访问你的日志，免去了grep操作。目前我们有部分系统采用ELK。
	关于如何搭建请点击下边的链接。
```
[ELK系统搭建](https://www.yunsonbai.top/2016/07/01/ELK%E5%AE%9E%E6%97%B6%E6%97%A5%E5%BF%97%E5%B9%B3%E5%8F%B0%E6%90%AD%E5%BB%BA/)

## 总结
``` bash
	至此，从error监控、api响应时间监控、关键代码段响应时间监控、info日志打印四个角度对webServer做了多方位的监控，这
	些监控是为了方便你及时修改系统bug和做优化的。如何选择还是要根据具体情况而定，总之一定要遵循这样的原则：不能因为监控
	系统而拖垮服务，最终目的是为了更好的提供服务。
```

