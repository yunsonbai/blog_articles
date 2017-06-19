---
title: rest_framework.authtoken 定制
date: 2016-07-11 14:27:22
tags:
	- rest_framework
    - rest framework
    - authtoken
    - 返回码定制
    - rest_framework.authtoken 定制
categories: django
---

## 引言

``` bash
    前段时间研究了一下rest_framework.authtoken，它可以实现对用户的token鉴权除了可以使用默认的配置以外，可以
    实现定制。想快速实现一套用户系统的可以看看
```

<!--more-->
## 为什么要使用
``` bash
    如果你急于实现一套用户系统，考虑到完全自己开发所花的时间成本（有可能费劲开发完了并不适用），不妨试试稍加改动
    django自带的用户系统为自己所用，django自带的用户系统还是比较完备的，而且还有丰富的第三方库。如果想用token
    鉴权该怎么办呢，这时候rest_framework.authtoken就用上派场了，rest_framework.authtoken用起来十分简单。
```
## 在django中使用
### 添加app
``` bash
	在INSTALLED_APPS中加入rest_framework.authtoken即可
    INSTALLED_APPS = (
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'rest_framework',
        'rest_framework.authtoken',
    )
```

### view中使用
``` bash
	authentication_classes = (BasicAuthentication, TokenAuthentication)
    permission_classes = (AllowAny,)
```
[rest_framework](http://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication)

## 定制返回码
### 配置
``` bash
	settings中添加：
		 REST_FRAMEWORK = {
            'EXCEPTION_HANDLER': 'ink_user.utils.rest_framework_views.exception_handler',
         }
```

### rest_framework_views.py
``` bash
	# coding=utf-8
    from __future__ import unicode_literals

    from django.core.exceptions import PermissionDenied
    from django.http import Http404
    from django.utils import six
    from django.utils.translation import ugettext_lazy as _

    from rest_framework import exceptions, status
    from rest_framework.compat import set_rollback
    from rest_framework.response import Response


    def exception_handler(exc, context):
        """
        Returns the response that should be used for any given exception.

        By default we handle the REST framework `APIException`, and also
        Django's built-in `Http404` and `PermissionDenied` exceptions.

        Any unhandled exceptions may return `None`, which will cause a 500 error
        to be raised.
        """
        if isinstance(exc, exceptions.APIException):
            headers = {}
            if getattr(exc, 'auth_header', None):
                headers['WWW-Authenticate'] = exc.auth_header
            if getattr(exc, 'wait', None):
                headers['Retry-After'] = '%d' % exc.wait

            if isinstance(exc.detail, (list, dict)):
                data = exc.detail
            else:
                data = {'detail': exc.detail}
                # if exc.detail == 'Invalid token.':
                if exc.status_code == 401:
                    data = {'status': 401.1, 'msg': exc.detail}
            set_rollback()
            return Response(data)
            # return Response(data, status=exc.status_code, headers=headers)

        elif isinstance(exc, Http404):
            msg = _('Not found.')
            data = {'detail': six.text_type(msg)}

            set_rollback()
            return Response(data, status=status.HTTP_404_NOT_FOUND)

        elif isinstance(exc, PermissionDenied):
            msg = _('Permission denied.')
            data = {'detail': six.text_type(msg)}

            set_rollback()
            return Response(data, status=status.HTTP_403_FORBIDDEN)

        # Note: Unhandled exceptions will raise a 500 error.
        return None
```

## note
``` bash
	重点在于：
    	settings中的配置；
        rest_framework_views.py 重写：
        	if exc.status_code == 401:
                # 此处定义自己的消息体
                data = {'status': 401.1, 'msg': exc.detail}
```