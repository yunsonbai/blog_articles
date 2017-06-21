---
title: 新版sentry安装配置
date: 2017-03-24 14:48:15
tags:
    - sentry
    - postgreSQL
    - sentry安装配置
    - sentry 8.14.1
    - 监控系统sentry搭建
categories: python
---


## 引言

``` bash
前段时间搭建了sentry 7.x版本,运行了有一段时间了，现在日志来量有点大
遇到了一个突出的问题：清理历史数据十分缓慢。最近在浏览sentry官方文档
发现都已经更新到8.14.1了, 而且不在支持mysql, 官方给的解释:"Due to 
numerous issues over the years and recent discoveries that
nearly all schema migration was broken in MySQL (due to some
behavior in our migration tool), we've made the decision to
no longer support MySQL. It is possible to bring the schema
up to date on a MySQL machine, but Sentry's automated migrations
will likely not work and require DBA assistance.
Postgres is now the only supported production database."
而且现在之前搭建的7.x版本清理数据慢的出奇，故花了些时间来搭建新的sentry,
搭建过程分享一下
```

<!--more-->
## Overview
* 需要安装：
    * python==2.7.5
    * Postgresql==9.2
    * sentry==8.14.1
    * redis==3.2.8
* 系统： 
    * centos6
* 本文重点：
    * Postgresql安装配置
    * sentry安装配置

##　Postgresql安装配置

### 安装
```python
    这里采用yum或者编译安装都可以，关键是配置启动，我们来说yum方式

    # yum install postgresql
    # yum install libpq-dev*
    # yum install python-dev*
    # yum install postgresql-contrib
    # yum install postgresql-client
    # yum install postgresql-devel
    # yum install pgadmin3

    没有意外的话安装应该ok

```

### 配置

#### 启动
```python
首先不能在root下启动postgresql,所以先切到postgres(安装过程中已经创建)

# su postgres

最让人恶心的是它的配置并不在etc下，而是在/var/lib/pgsql/data/(
确定一下用户组和用户是不是postgres，不是改)

$ cd /var/lib/pgsql/data/
$ vim /var/lib/pgsql/data/pg_hba.conf
    尤其是以下三行：
        local   all　　　　all                 trust
        host    all    all  127.0.0.1/32   trust
        host    all    all  ::1/128        ident

启动

$ postgres -D /var/lib/pgsql/data/
```

#### 创建库

```python
postgres初始后第一个用户是postgres默认下是没有密码的，但是如果不把以上
配置配成trust登陆不了

$ psql -U postgres -h 127.0.0.1 -p 5432
postgres=# alter user postgres with password 'new password';

之后更改配置文件成：
    local   all　　　　all                 md5
    host    all    all  127.0.0.1/32   md5
    host    all    all  ::1/128        ident
启动

$ postgres -D /var/lib/pgsql/data/

登陆创建库：
$ psql -U postgres -h 127.0.0.1 -p 5432
postgres=#　create database sentry

```

## 安装sentry

### 前提
```python
安装好python2.7(yum或者自己编译)
安装好redis(直接取官方下载)

```

### 开始安装
```python
这边我们用pip的方式安装
# pip install sentry  (最新版)
```

### 配置
```python
# vim sentry.conf.py

几个关键点：
    
# 连接数据库
DATABASES = {
    'default': {
        'ENGINE': 'sentry.db.postgres',
        'NAME': 'sentry',
        'USER': 'postgres',
        'PASSWORD': '123456',
        'HOST': '127.0.0.1',
        'PORT': '5432',
        'AUTOCOMMIT': True,
        'ATOMIC_REQUESTS': False,
    }
}
# 配置SENTRY_OPTIONS
SENTRY_OPTIONS = {
    'redis.clusters': {
        'default': {
            'hosts': {
                0: {
                    'host': '127.0.0.1',
                    'port': 6379,
                    'password': '',
                    'db': 0
                }
            }
        }
    },
    'system.secret-key': 'DF0d/JSUbdCx7pzq4uh/lW43svXnCGuqIfjS2krkFA7N0BIPxCgcmg==',

    'mail.from': 'xx@xx.com',  # 发报警邮件用
    'mail.host': 'xx.xx.xx.xx',
    'mail.port': 26,
    'mail.username': 'xx@xx.com',
    'mail.password': 'xxxx',
}

SENTRY_WEB_HOST = '0.0.0.0'
SENTRY_WEB_PORT = 9000
SENTRY_WEB_OPTIONS = {
    'workers': 5,  # the number of web workers
    # 'protocol': 'uwsgi',  # Enable uwsgi protocol instead of http
}

```

### 启动
```python
/usr/bin/sentry --config=/baisong/sentry.conf.py run worker
/usr/bin/sentry --config=/baisong/sentry.conf.py django runserver 0.0.0.0:9000
```

### 优化
```python
可以采用nginx代理的方式多,后端多起几个端口
```

## 关于使用
```python
目前sentry已经搭建完毕，关于在django中如何使用，请看下边的连接
```
[利用sentry收集django的日志](https://yunsonbai.top/2016/05/30/django-sentry/)
