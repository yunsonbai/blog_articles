---
title: 我对python装饰器的使用
date: 2017-06-30 14:27:22
tags:
    - python装饰器
    - 装饰器
    - python
    - 闭包
categories: python
---

## 引言

``` bash
    趁着闲暇之际总结了关于装饰器的知识，并结合前人的总结，整理了一篇关于python
    装饰器的小文章，接下来分享一下我对python装饰器的理解：简单来说，装饰器是可
    调用的对象，它的参数就是一个函数。要么装饰器处理这个函数，然后把它返回，或
    者把原来的函数替换成另外的可调用对象。
```

<!--more-->
## python中的装饰器是什么
* 直观:
    以@开发放在某个函数之上的可调用对象

* 我的理解:
    [装饰器](http://yunsonbai.github.io/2017/06/30/python%E4%B8%AD%E7%9A%84%E8%A3%85%E9%A5%B0%E5%99%A8/)本身是python的一个函数(也可以说是对象，因为python中一切皆是对象),
    它能方便的让我们在不动已经写好的代码的基础上获取额外的功能，最重要的是它
    能帮我抽取出几乎一样的代码出来，尤其是当不同函数中有相同的部分的时候，装
    饰器能干净利落的让我们完成需求，比如鉴权功能/日志功能等等。也就是在进入
    真正的函数之前就做了一些其他的事儿

## 理解一下什么是闭包
* 问题的来源
    在学习之初总是听到或看到闭包这个词，我想很多直接学习python的同学肯定也是
    经常听到这个词了
* 什么是闭包
    我个人的理解时闭包是能访问定义体之外的非全局变量，或者说这个函数的作用域被
    延伸了(因为它居然访问了并不是自己定义的非全局变量，注意这个非全局变量很重
    要要是全局变量就没意义了)

* 来个例子吧

```python
def list_append():
    l = []  # 1

    def worker(new_e):  # 2
        l.append(new_e)  # 3，l在这成了自由变量(w.__code__.co_freevars)
        return l   # 4

    return worker


if __name__ == '__main__':
    w = list_append()
    for i in range(3):
        print(i, ':', w(i))

执行结果：
0 : [0]
1 : [0, 1]
2 : [0, 1, 2]

闭包： #1 到 #4之间的代码
```

## 写个简单的装饰器
### 代码
```python
def get_result(func):
    def warp(*args, **kwargs):
        print('haha warp')
        result = func(*args, **kwargs)
        print('result:', result)
        return result
    return warp


@get_result
def add(a, b):
    print('func add:', a + b)
    return a + b


if __name__ == '__main__':
    add(1, 2)

```
### 执行结果及解释
```python
执行结果：
    haha warp
    func add: 3
    result: 3

解释：
    add会作为参数func传给get_result函数,然后get_result返回warp函数。
    如果你执行add.__name__你会发现明明是add名字却变成了warp,其实这是
    python解释器在背后吧warp赋值给了add，然后add就保存了warp的索引，每
    次调用add(a, b)都是在执行warp(a, b), 还记的前边说的把函数转变成了
    其他的可调用对象，然后二者接收同样的参数。可能有时我们并不想这么干，因
    为毕竟只是装饰一下我的函数，结果给我变了属性，怎么解决后边会说。不管怎
    么说我们还是实现了一个简单的装饰器。
```

## 解决name属性被修改的问题
### 代码
```python
from functools import wraps


def get_result(func):
    @wraps(func)
    def warp(*args, **kwargs):
        print('haha warp')
        result = func(*args, **kwargs)
        print('result:', result)
        return result
    return warp


@get_result
def add(a, b):
    print('func add:', a + b)
    return a + b


if __name__ == '__main__':
    a = add
    a(1, 2)
    print('name:', a.__name__)



if __name__ == '__main__':
    a = add
    a(1, 2)
    print(a.__name__)
```

### 执行结果及解释
```python
执行结果：
    haha warp
    func add: 3
    result: 3
    name: add
解释：
    我们用到了functools.warps它的作用就是协助构建良好的装饰器，就是把被修饰的函数对象
    的指定属性复制给包装函数对象。可以看一下wraps的部分代码(太占篇幅注释去了)：
    WRAPPER_ASSIGNMENTS = (
        '__module__', '__name__', '__qualname__', '__doc__',
        '__annotations__')
    WRAPPER_UPDATES = ('__dict__',)
    # 看到WRAPPER_ASSIGNMENTS、WRAPPER_UPDATES和注释1/2应该已经很明了了
    def update_wrapper(wrapper, wrapped, assigned = WRAPPER_ASSIGNMENTS,
                       updated = WRAPPER_UPDATES):
        # 注释1
        for attr in assigned:
            try:
                value = getattr(wrapped, attr)
            except AttributeError:
                pass
            else:
                setattr(wrapper, attr, value)
        # 注释2
        for attr in updated:
            getattr(wrapper, attr).update(getattr(wrapped, attr, {}))
        wrapper.__wrapped__ = wrapped
        return wrapper

    def wraps(wrapped, assigned = WRAPPER_ASSIGNMENTS,
              updated = WRAPPER_UPDATES):
        return partial(
            update_wrapper, wrapped=wrapped, assigned=assigned, updated=updated)
小结：
    当我们需要保留被修饰的函数的属性时，wraps装饰器非常有用，以至于我们在写装饰器的时候
    绝大部分情况时需要用到它的，另外上边的例子其实很容易变形成记录日志功能的装饰器，可以
    试一下。
```

## 鉴权装饰器
### 场景说明
```python
我们在进行接口开发的过程中，总是免不了对调用方进行鉴权的，毕竟资源时很宝贵的，不能随随
便便让别人拿到。我们又不可能去所有的接口去实现同样的代码部分，这时候装饰器是很好的选择
[参考](https://yunsonbai.top/2016/05/26/decorator/)
```

### 代码
```
from functools import wraps


def user_auth(func):
    @wraps(func)
    def warp(self, data, *args, **kwargs):
        token = data.get('token', False)
        if not token:
            return {}
        return func(self, data, *args, **kwargs)
    return warp


class UserInfo(object):

    @user_auth
    def get_info(self, data):
        '''
        data: 调用方传递的用户的凭证信息包括token
        '''
        info = {'age': 18}
        return info


if __name__ == '__main__':
    user_info = UserInfo()
    data = {'token': 'token'}
    print('has token:', user_info.get_info(data))
    print('no token:', user_info.get_info({}))
```

### 执行结果及解释
```python
结果：
    has token: {'age': 18}
    no token: {}
解释：
    这个修士器可以对接口进行鉴权工作,而且不用每个接口中加同样的代码,直接在函数上加上
    修饰器即可,另外一定要记得添加self参数,正如前边所说装饰器使得返回的函数和被修饰的
    函数接收一样的参数,而用过wraps之后返回的函数被重新赋予了被修饰函数的一些属性.另
    外也可以在这附上self.user_id等属性方便后边使用.
```

## 传参装饰器
### 怎么生成传参装饰器
```python
    装饰器工厂函数,在学习python过程中经常听到这个词,因为这个函数的存在我们能很容易的
    写出传参装饰器,简单来说,这个函数的功能就是把参数传给这个装饰器工厂函数,然后返回装
    饰器,再来装饰要装饰的函数.可能不太好捋清楚,来个例子
```

### 代码
```python
from functools import wraps


def user_auth(has_token=True):
    def decorator(func):
        @wraps(func)
        def warp(self, data, *args, **kwargs):
            if has_token:
                token = data.get('token', False)
                if not token:
                    return {}
            else:
                uid = data.get('uid', False)
                passwd = data.get('passwd', False)
                if (bool(uid) and bool(passwd)) is False:
                    return {}
            return func(self, data, *args, **kwargs)
        return warp
    return decorator


class UserInfo(object):

    @user_auth(has_token=True)
    def get_info(self, data):
        '''
        data: 调用方传递的用户的凭证信息包括token
        '''
        info = {'age': 18}
        return info

    @user_auth(has_token=False)
    def login(self, data):
        '''
        data: 登录信息
        '''
        info = {'age': 18}
        return info


if __name__ == '__main__':
    user_info = UserInfo()

    print('-------------login------------')
    data = {'uid': 'uid1', 'passwd': 'sss'}
    print('has uid passwd:', user_info.login(data))
    print('no uid passwd:', user_info.login({}))
    print('-------------get_info------------')
    data = {'token': 'token'}
    print('has token:', user_info.get_info(data))
    print('no token:', user_info.get_info({}))
```

### 执行结果及解释
```python
结果：
    -------------login------------
    has uid passwd: {'age': 18}
    no uid passwd: {}
    -------------get_info------------
    has token: {'age': 18}
    no token: {}
解释：
    这个用户鉴权装饰器应对于登陆和登陆后的鉴权,如参数has_token,当为False时,说明
    是登陆,反之说明已经登陆成功,应携带token参数请求接口,其他的逻辑应该很简单.
```


