---
title: decorator 装饰器
date: 2016-05-26 10:10:39
tags:
    - python
    - 装饰器
    - 类方法装饰器
    - decorator
categories: python
---

## 引言
``` bash
    python的装饰器用途非常多，例如鉴权，重要参数前期验证等等，主要用于在进入方法以前做前期工作，
    下边举例针对类方法和函数
```
<!--more-->
### 类方法修饰器
``` bash
    from functools import wraps

    def create_user(func):
    @wraps(func)
    def _warp(self, *args, **kwargs):
            # 这里写你的方法
            pass
            return Response(context)
        return func(self, *args, **kwargs)
    return _warp
```
### 函数修饰器
``` bash
    和类方法修饰器不通的是没有self
    from functools import wraps

    def create_user(func):
    @wraps(func)
    def _warp(*args, **kwargs):
            # 这里写你的方法
            pass
            return Response(context)
        return func(*args, **kwargs)
    return _warp
```