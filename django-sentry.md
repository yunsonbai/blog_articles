---
title: 利用sentry收集django的日志
date: 2016-05-30 10:21:48
tags:
    - django
    - sentry
    - 日志收集
    - 邮件发送报错日志
categories: django
---
## 引言

``` bash
    日志收集对项目很重要,利用sentry可以手机django程序运行时的日志,同时适当的配置还能发送报警邮件
```

<!--more-->
## 搭建sentry环境
### 创建独立环境

``` bash
    virtualenv sentry
    安装好MySQL 和 redis
```

### 安装sentry
``` bash
    pip install sentry
```

### 配置(/env/sentry/conf/sentry.conf.py)
#### 数据库配置

``` bash
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'sentry',
            'USER': 'root',
            'PASSWORD': '123456',
            'HOST': '127.0.0.1',
            'PORT': '3306',
        }
    }
```

#### redis配置

``` bash
    SENTRY_REDIS_OPTIONS = {
        'hosts': {
            0: {
                'host': '127.0.0.1',
                'port': 6379,
            }
        }
    }
    CELERY_ALWAYS_EAGER = False
    BROKER_URL = 'redis://localhost:6379/2'

```

#### 端口配置

``` bash
    SENTRY_URL_PREFIX = 'http://127.0.0.1:9000'
    SENTRY_WEB_HOST = '0.0.0.0'
    SENTRY_WEB_PORT = 9000
    SENTRY_WEB_OPTIONS = {
        # 'workers': 3,  # the number of gunicorn workers
        # 'secure_scheme_headers': {'X-FORWARDED-PROTO': 'https'},
    }
```

#### 语言配置

``` bash
    LANGUAGES = (
        ('en', gettext_noop('English')),
        ('zh-cn', gettext_noop('Simplified Chinese')),
        # ('zh-cn', gettext_noop('Traditional Chinese')),
    )

```

#### 邮箱配置

``` bash
    EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
    EMAIL_HOST = 'mail.test.com'
    EMAIL_HOST_PASSWORD = 'xxxxxx'
    EMAIL_HOST_USER = 'xxxx@test.com'
    EMAIL_PORT = xx
    # EMAIL_USE_TLS = False
    # The email address to send on behalf of
    SERVER_EMAIL = 'xxxx@test.com'
```

## 建立django项目并使用sentry
### 建立

``` bash
    django-admin startproject test_sentry
```

### 配置修改

``` bash
    INSTALLED_APPS = (
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'raven.contrib.django.raven_compat',
    )
    # sentry
    RAVEN_CONFIG = {
        'dsn': 'http://0e084efb47d24bedb60d24bedb6abd95ed7:d92981b0d24bedb6abd92981b05d7@xx.xx.xx.xx:9000/1',
    }
```

### 触发

```
    故意让程序出错就OK了
```

### 收集info日志
#### 配置

``` 
    LOGGING = {
        'version': 1,
        'disable_existing_loggers': False,
        'formatters': {
            'verbose': {
                'format': "INK|%(levelname)s|%(asctime)s|%(module)s|%(process)d|%(thread)d|%(message)s",
            },
        },
        'handlers': {
            'console': {
                'level': 'DEBUG',
                'class': 'logging.StreamHandler',
                'formatter': 'verbose',
            },
            'info': {
                'level': 'INFO',
                'formatter': 'verbose',  # 使用哪种formatters日志格式
                'class': 'raven.contrib.django.raven_compat.handlers.SentryHandler',
            },
        },
        'loggers': {
            'django.request': {
                'handlers': ['console'],
                'level': 'DEBUG',
                'propagate': True,
            },
            'info': {
                'handlers': ['info', ],
                'level': 'INFO',
                'propagate': True,
            },
        },
    }
```

#### 调用

```
    import logging
    logger = logging.getLogger('info')

    logger.info('test')
```
