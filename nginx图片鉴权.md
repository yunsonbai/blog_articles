---
title: nginx图片鉴权
date: 2016-08-09 13:05:19
tags:
	- nginx
    - nginx图片鉴权
    - nginx静态资源鉴权
    - internal
    - nginx django internal
categories: nginx
---

## 引言

``` bash
    在开发过程中总离不开用户系统，有了用户系统就免不了有些资源需要和用户关联上,
	所以就无法避免的有了需要资源私有化的问题，比如某张图片或某个字体只能允许所
	有者才能访问，该怎么做鉴权防止其他用户访问，昨天试了一下nginx的internal
	相信应该能满足这样的需求。
```

<!--more-->
## 需求
``` bash
	隐藏图片的原地址，并能实现鉴权只有拥有者才能访问。
```

## 环境说明
``` bash
	系统：ubuntu
	app：django
	web：nginx
```

## 流程说明
![Catch8D7B](/images/nginx/nginx_django图片鉴权.png)

## 实现
### nginx设置
``` bash
	server {
        listen 80;
        server_name www.example.com;
        client_max_body_size 150m;
        access_log /home/baisong/project/yunsonbai/log/nginx/test.log main;
        error_log /home/baisong/project/yunsonbai/log/nginx/test_error.log;
        location / {
            proxy_pass    http://127.0.0.1:8000/;
        }
        location /private/ {
            internal;
            # http://www.yunsonbai.top/images/avtar.jpg
            proxy_pass    http://www.yunsonbai.top/;
        }
    }
```

### hosts配置
```bash
	怎么配置就不说了，配置这项就是为了测试方便
```

### django程序(views.py)
``` bash
	class Test(APIView):

    '''
    test
    '''

    def get(self, request):
        user_id = request.query_params.get('user_id', None)
        if user_auth(user_id):
			# 这里写相对路径
            url = '/private/images/avtar.jpg'
            response = HttpResponse()
            response['Content-Type'] = ""
            response['X-Accel-Redirect'] = url
            return response
        else:
            return HttpResponseForbidden()
```

## 效果
### 用户合法
![Catch8D7B](/images/nginx/合法.jpg)
```bash
	可以看到隐藏了真正的资源链接
```

### 用户不合法
![Catch8D7B](/images/nginx/不合法.jpg)
```bash
	可以看到直接返回403,并没有暴露资源地址
```

## 总结
``` bash
	该方法适用除了图片私有化其他资源也可以采用，例如表情等，当然不是唯一办法，文章有不
	合适的地方可以一起讨论。
```
