---
title: 使用rsyslog收集Django的日志
date: 2016-06-08 22:53:20
tags:
	- 使用rsyslog收集Django的日志
    - Django
    - rsyslog
    - 日志收集
categories: django
---

## 引言

``` bash
    日志收集对项目很重要,在前边我写过一篇如何利用sentry收集django程序运行时的日志,我们的项目中一般用sentry
    来收集error日志，因为打的比较详细，可观性也比较强。但是我们如果把info日志达到sentry上不方便观看，所以不
    妨使用rsyslog来收集Django中的一些info日志，当然只是打我们想要的东西。
```

<!--more-->
## 环境说明
``` bash
	ubuntu12.04
```

## 搭建rsyslog环境
### 安装
``` bash
    在Ubuntu中默认已经装好了rsyslog，当然如果有问题你可以自己安装（使用apt-get 
    install rsyslog即可，你也可以使用编译安装）
```

### 配置（/var/log/mylog/forum.log）
``` bash
    你可以添加类似这样一条：
	local4.*        /var/log/mylog/forum.log
```

### 重启rsyslog
``` bash
    service rsyslog restart
```

## 在Django中使用
### 修改Djangosettings.py

``` bash
    from logging.handlers import SysLogHandler

	LOGGING = {
	    'version': 1,
	    'disable_existing_loggers': False,
	    'formatters': {
	        'verbose': {
	            'format': "iSync|%(levelname)s|%(asctime)s|%(module)s|%(process)d|%(thread)d|%(message)s",
	        },
	        'simple': {
	            'format': "iSync|%(levelname)s|%(asctime)s|%(message)s",
	        },
	    },
	    'handlers': {
	        'null': {
	            'level': 'INFO',
	            'class': 'logging.NullHandler',
	        },
	        'console': {
	            'level': 'DEBUG',
	            'class': 'logging.StreamHandler',
	            'formatter': 'verbose',
	        },
	        'syslog': {
	            'level': 'INFO',
	            'class': 'logging.handlers.SysLogHandler',
	            'formatter': 'simple',
	            'facility': SysLogHandler.LOG_LOCAL4, #此处一定要和rsyslog中的local4一致
	            'address': '/dev/log', # 机器上一定要有该文件（srw-rw-rw-）
	        },
	    },
	
	    'loggers': {
	        'django.request': {
	            'handlers': ['console', 'null'],
	            'level': 'ERROR',
	            'propagate': True,
	        },
	        'rsyslog': {
	            'handlers': ['syslog', ],
	            'level': 'INFO',
	            'propagate': True,
	        },
	    },
	
	}

```

### 在view或其他地方调用，将日志打到rsyslog

``` bash
    rsysloger = logging.getLogger('rsyslog')
    rsysloger.info('sdsdsdsds')
	# 之后你可以去/var/log/mylog/forum.log 看结果了

```

## [利用sentry收集Django日志](https://www.yunsonbai.top/2016/05/30/django-sentry/)