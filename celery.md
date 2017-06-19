---
title: celery 异步任务
date: 2016-07-12 14:27:22
tags:
	- celery
    - celery实现异步任务
    - python异步任务
    - djcelery
    - djcelery 异步任务
categories: python
---

## 引言

``` bash
    在开发的过程中，总会遇到异步完成某个操作的需求，选择会很多，说一下在python编程过程中如何实现异步任务，让你优雅的完成
    任务，优化自己的应用。
```

<!--more-->
## 初始celery
``` bash
	任务调度利器,是基于Python开发的分布式任务队列。它支持使用任务队列的方式在分布的机器／进程／线程上执行任务调度。
```
## 单独使用celery
### 需求
``` bash
	定时往文件中写入和在程序中异步添加任务往文件中写入
```
### 编写tasks
``` bash
	# coding=utf-8
    from celery import Celery

    celery = Celery('tasks', broker='redis://localhost:6379/0')

    @celery.task
    def test(sss):
        f = open('/yunsonbai/apps/asyn_task/celery/a.txt','a+')
        f.write(sss)
        f.close()

    if __name__=="__main__":
        sendmail('bas')
```

### celeryconfig配置文件
``` bash
	这个比较重要，关系到启动
    from datetime import timedelta

    BROKER_URL = 'redis://127.0.0.1:6379/0'
    CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/1'

    CELERY_TASK_SERIALIZER = 'json'
    CELERY_RESULT_SERIALIZER = 'json'
    CELERY_ENABLE_UTC = False
    CELERY_TIMEZONE = 'Asia/Shanghai'
    CELERY_IMPORTS = ("tasks",)
    # 定时执行
    # CELERYBEAT_SCHEDULE = {
    #     'runs-every-30-seconds': {
    #         'task': 'tasks.sendmail',
    #         'schedule': timedelta(seconds=30),
    #         'args': ('baisong',),
    #     },
    # }
```

### note
``` bash
	如果使用redis，建议使用CELERY_QUEUES和CELERY_ROUTES两个配置项，可以让你使不用应用的任务区分开，不然如果用同一个
    redis库存储BROKER，多个worker之间会发生干扰。另外就是让tasks和celeryconfig在同一个目录下。
```

### 启动
``` bash
	celery worker --loglevel=debug --config=celeryconfig  负责执行任务
    celery beat --loglevel=info --config=celeryconfig 负责添加任务（针对定时任务，如果没有可以不启动）
```

### 如何在程序中添加任务
``` bash
	from tasks import test
    result = sendmail.apply_async(args =('yusonbaio',))
    # print result

```

## 在django中使用celery
### 需求
``` bash
	同上
```

### 优点
``` bash
	能方便的调用model、settings等，方便数据库操作。
```

### tasks
``` bash
	# coding=utf-8
    from celery import app

    @app.task
    def test(sss):
        f = open('/yunsonbai/apps/asyn_task/celery/a.txt','a+')
        f.write(sss)
        f.close()

    if __name__=="__main__":
        test('bas')
```

### 配置文件
``` bash
	把上边的配置文件复制到settings中，同时在INSTALLED_APPS加入djcelery，记
    得安装djcelery
```

### 添加celety.py文件
``` bash
	在与app同级目录下添加celety.py文件，如下：
    	from __future__ import absolute_import
        import os
        from celery import Celery
        from django.conf import settings

        os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'app.settings')
        app = Celery('app')

        app.config_from_object('django.conf:settings')
        app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)
```

### 程序中调用
``` bash
	同上
```

### 启动
``` bash
	python manage.py celery worker --loglevel=info
    python manage.py celery beat --loglevel=info
```
