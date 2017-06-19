---
title: 在django中只使用celery
date: 2016-07-18 14:27:22
tags:
	- django celery
    - 异步任务
    - 在django中只使用celery
    - django中不使用djcelery只使用celery
    - celery多配置
categories: python
---

## 引言

``` bash
    前段时间分享了一篇在django中使用djcelery来完成异步任务，使用的小伙伴会发现djcelery会在mysql创建很多表，
    如果你不使用mysql作为BROKER的话,数据显得很脏，另外如果你用了mysql作为BROKER又会带来性能问题，毕竟数据库
    的读写速度不如缓存。接下来就分享一下如何在django中不使用djcelery，而是纯粹的使用celery来实现异步任务。
```

<!--more-->
## 先看一下目录结构
{% img  https://yunsonbai.github.io/images/celery/celery_django.png %}
``` bash
	注意看celery.py 和tasks.py两个文件，这个是celery官方给出的例子
```

## 代码示例及启动
### celery.py
``` bash
	from __future__ import absolute_import
    import os

    from celery import Celery

    # set the default Django settings module for the 'celery' program.
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'proj.settings')

    from django.conf import settings  # noqa

    app = Celery('proj')

    # Using a string here means the worker will not have to
    # pickle the object when using Windows.
    app.config_from_object('django.conf:settings')
    app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)

    @app.task(bind=True)
    def debug_task(self):
        print('Request: {0!r}'.format(self.request))
```

### __init__.py
```bash
	在与celery.py同级的__init__.py文件中加入(这也都是官方给出的)
    	from __future__ import absolute_import
        # This will make sure the app is always imported when
        # Django starts so that shared_task will use this app.
        from .celery import app as celery_app  # noqa
```

### tasks.py
``` bash
	（这是自己造的）
	from __future__ import absolute_import
    from proj import celery_app
    import time

    @celery_app.task(bind=True)
    def mul(*args, **kwargs):
        f = open('/home/baisong/a.txt', 'a+')
        f.write(str(time.time()))
        f.write('/n')
        f.close()
```

### 启动
``` bash
	beat:
    	celery -A proj beat -l info
    worker:
    	celery -A proj worker -l info
```

## 提问
``` bash
	前边几乎全是官方给出的，下边是有些人会遇到的，多配置文件怎么
    办， 可以看看下边的目录结构
```
{% img  https://yunsonbai.github.io/images/celery/moresettings.jpg %}

## 解决
### 重写celery.py
``` bash
	from __future__ import absolute_import
    import os
    from celery import Celery
    import sys
    from django.conf import settings
    app_name = 'proj'
    defauke_config = 'proj.settings'
    try:
        defauke_config = sys.argv[-1].split('--config=')[1]
    except:
        pass
    # set the default Django settings module for the 'celery' program.
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', str(defauke_config))

    app = Celery(app_name)

    # Using a string here means the worker will not have to
    # pickle the object when using Windows.
    app.config_from_object('django.conf:settings')
    app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)
```

### 启动
``` bash
	celery -A proj beat -l info --config=proj.settings.settings1
    celery -A proj worker -l info --config=proj.settings.settings1
```

## 链接
[celery 异步任务](https://www.yunsonbai.top/2016/07/12/celery/)
