---
title: ELK日志平台搭建
date: 2016-07-01 14:27:22
tags:
	- ELK
    - 日志平台搭建
    - ELK搭建
categories: api或系统监控
---

## 引言

``` bash
    在系统问题排查时，尤其是在第三方接口出问题的时候，貌似是自己写的服务异常，其实不然，这样很是浪费自己时间，错误在第三方
    接口。所以日志收集整理，很重要，能帮助我们排错。
```

<!--more-->
## 可选择的日志收集方式
### rsyslog收集
``` bash
	走514端口，收集远程服务器打来的日志，可以根据local级别（0-7）过滤，也可以根据内容（不太方便），最终要使用grep去服务
    器上过滤排查
```

### syslog-ng收集
``` bash
	和rsyslog差不多，在根据内容过滤上，本人认为还是比较方便的，但是同样得用grep去机器上过滤想要的东西。
```

### ELK
``` bash
	何为ELK：
		e:
          elasticsearch，搜索引擎，创建索引，存储数据
     	l:
          logstash， 日志收集，过滤，传输日志到elasticsearch
     	k:
	这个不错，可以在web页面查看，还能方便的过滤，所以可以尝试一下这个。
```

## ELK的搭建
``` bash
	网上有很多类似的，下边是我一边搭建一边整理的，没有意外是可以的。
	环境：
		centos6
```

### elasticsearch
#### 版本说明
``` bash
	建议去官网下载，我使用的是1.4.4下载elasticsearch-1.4.4.tar.gz后解压
```

#### 配置
``` bash
	vim elasticsearch-1.4.4/config/elasticsearch.yml
   	添加如下：
		cluster.name: elasticsearch  （不能用root用户）
		node.name: "node1"
		node.master: true
     	node.data: true
     	network.bind_host: 192.168.xx.xx
```

#### 启动
``` bash
	sh elasticsearch-1.4.4/bin/elasticsearch 
    （所有的服务通过后建议写脚本采用后台运行）
	看到9200和9300端口说明启动成功
```

### logstash
``` bash
	原则，不能因为添加监控而导致服务质量大幅度下滑。
```

#### 版本说明
``` bash
	官方下载，我使用的是logstash-1.5.4。
```

#### 配置
``` bash
	vim test.conf
	input {
    	file {
	    	path => "/var/log/messages"
	    	type => "syslog"
		}
		# 如果用远程的收集话，要使用udp方式
	}

	filter{

    	#test
    	if [message] =~ /TEST/ {
        	mutate{
				# 可取官方文档中找，添加你想要的参数
			}
    	}
	}
	output {
    	#test
    	if [message】 =~ /TEST$/{
        	elasticsearch { host =>  "192.168.xx.xx"
            	protocol => "http"
            	index => "test-%{+YYYY.MM}"
            	manage_template  => false
        	}
    	}
	}
```

#### 启动
``` bash
	sh logstash-1.5.4/bin/logstash
```

### kibana
#### 版本说明
``` bash
	官网下载，我使用的是4.1.1
```

#### 配置
``` bash
	vim config/kibana.yml
   	添加如下：
		elasticsearch_url: "http://192.168.xx.xx:9200"
		其他的不动
```

#### 启动
``` bash
	sh bin/kibana
    看到5601就行了
```

## 使用
``` bash
	往配置的文件中打印日志，在kibana的setting中配置一条[text-]YY.MM，
    就能看到你的日志了，还可以根据需求定制显示哪一部分
```
