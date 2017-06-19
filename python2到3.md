---
title: python2到3项目迁移
date: 2016-05-20 13:40:05
tags:
    - python2
    - python3
    - 迁移
    - 不同点
    - python项目迁移
categories: python
---

## 引言
``` 
    目前项目组逐步将项目迁移到python3，下边说一下在迁移的过程中遇到的一些问题和注意事项
```

<!--more-->
## 安装python3
``` bash
    起初我采用的是如下编译方式：
        $ ./configure --prefix=/usr/local/python3.5/
        make install
    好景不长，收到了如下错误：
        $ No module named _sqlite3
    解决办法：
        $ yum -y install sqlite-devel
        $ ./configure --enable-loadable-sqlite-extensions --prefix=/usr/local/python3.5/
        $ make install
```

## 迁移中需要注意的点
### urllib2改变
``` bash
    python2.7：
        avatar_img = urllib2.urlopen(avatar_url).read()
    python3.5：
        import urllib.request
        avatar_img = urllib.request.urlopen(avatar_url).read()
```

### redis配置改变
``` bash
    redis_conf = {
        'host': ip,
        'port': port,
        'db': 1,
        'password': password,
        'socket_timeout': None,
        'encoding': 'utf-8',
        'encoding_errors': 'strict',
        'decode_responses': True    此处要注意
    }
```

### CStringIO与StringIO不能再用
``` bash
   io.BytesIO
   io.StringIO 
```

### try和except的改变
``` bash
    python2.7：
        try:
            pass
        except Exception, e:
            pass
    python3.5:
        try：
            pass
        except Exception as e:
            pass
```

### 默认Image取消
``` bash
    python2.7：
        支持：import Image
    python3.5：
        pip3.5 install Pillow
        from PIL import Image
```

### hash操作必须encode
``` bash
    sig_str = sig_string + appkey
    sig_str = sig_str.encode('utf-8')
    sig_md5 = hashlib.md5()
    sig_md5.update(sig_str)
```

### xrange取消
```
    由range代替
```

### startwith第一个参数必须是bytes或者bytes组成的tuple
```
    startwith(b'GIF89')(python3.5强制加b)
```

### raw_input()改成input()
```
    python2.7：
        a = raw_input(input a:)
    python3.5:
        a = input('input a:')
```

### 有人可能用supervisor
```
    目前supervisor不支持python3，
    但是git上已久supervisor4，支持python3，
    我想不就得将来supervisor马上也就能在python3上使用了
```

### python与yum
```
    建议不要将python3软连在/usr/bin/python上，这样有可能导
    yum不能使用，
    如果非要这样软连，你需要修改：
    /usr/bin/yum
        #!/usr/bin/python
        改成：
        #!/usr/bin/python2.7
```

### 提示bytes-like object is required, not 'str'
```
    解决办法：
        1、加b
        2、encode('utf-8')
```