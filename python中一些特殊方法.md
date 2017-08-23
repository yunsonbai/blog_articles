---
title: python中一些特殊方法的自定义及作用
date: 2017-06-17 14:27:22
tags:
    - python
    - __bool__,__len__
    - 自定义算术运算符
    - __repr__
    - __add__,__mul__
categories: python
---
[**原文连接**](https://yunsonbai.top/2017/06/17/python%E4%B8%AD%E4%B8%80%E4%BA%9B%E7%89%B9%E6%AE%8A%E6%96%B9%E6%B3%95/)

## 引言

``` bash
   聊一聊__add__,__mul__,__len__,__bool__的作用和自定义,以及__len__和
   __bool__是怎么影响if/while的条件判断，另外说一说怎么把一个对象以字符串的
   形式表现。
```

<!--more-->
## 自定义运算符
### 使用场景
```python
    自定义一个类, 类中有两个属性分别为x和y, 实现a+b  a*b的功能，要求a+b实现a.x+b.x
    a.y+b.y; a*b实现a.x*b.x a.y*b.y
```
### 实现
```python
class V(object):

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, v):
        print('run __add__')
        return V(x=v.x + self.x, y=v.y + self.y)

    def __mul__(self, v):
        print('run __mul__')
        return V(x=v.x * self.x, y=v.y * self.y)


v = V(1, 2)
print('---------------add----------')
new_v = v + v
print(new_v.x)

print('---------------mul----------')
new_v = v * v
print(new_v.y)
```

## 自定义bool值
### 关于if v
```python
默认情况下我们自己定义的类的实例总是被认为时真的，但是有的时候这个默认值不是我们想要的，
除非我们自己定义__bool__或者__len__才能实现我们想要的结果，我们还以刚才的类V举例，
我想你总不希望当x和y都是0的时候还希望if v成立吧
```

### 关于bool()
```python
像if v/if not v等，python会调用bool(v)方法，而bool(v)背后的调用时尝试调用
v.__bool__(),如果没有__bool__方法，就会尝试调用__len__方法，若这是返回0则
是False否则为True,这两个方法都没有的话只能按照默认处理了
```

### 实现
#### __bool__和__len__都没有
```python
class V(object):

    def __init__(self, x, y):
        self.x = x
        self.y = y

v = V(1, 2)

print('--------------if----------')
if v:
    print('\tif True')
else:
    print('\tif False')

print('-------------while----------')
while v:
    print('\twhile True')
    break
    
执行结果：
--------------if----------
    if True
-------------while----------
    while True

```

#### 只有__len__
```python
class V(object):

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __len__(self):
        print('run __len__')
        return self.x + self.y

v = V(1, 2)

print('--------------if----------')
if v:
    print('\tif True')
else:
    print('\tif False')

print('-------------while----------')
while v:
    print('\twhile True')
    break

执行结果：
    --------------if----------
    run __len__
        if True
    -------------while----------
    run __len__
        while True

```

#### __bool__与__len__都有
```python
class V(object):

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __bool__(self):
        print('run __bool__')
        # 显然以下代码可以使if v效率更高
        return bool(self.x or self.y)

    def __len__(self):
        print('run __len__')
        return self.x + self.y

v = V(1, 2)

print('--------------if----------')
if v:
    print('\tif True')
else:
    print('\tif False')

print('-------------while----------')
while v:
    print('\twhile True')
    break

执行结果：
--------------if----------
run __bool__
    if True
-------------while----------
run __bool__
    while True
```

## 如何使对象以字符串的形式表现

### 为什么要以字符串的形式
```python
我们把一个对象以字符串的形式表现出来,是为了更好的辨认，方便后续处理或让用户觉得更加直观,
最普遍的是在django的admin中，我们希望看到一行数据的谁的数据，而不是希望看到一个class
或其他无法辨识的表现形式。
```

### 只用__str__实现
```python
class Student(object):

    def __init__(self, name):
        self.name = name

    def __str__(self):
        print('\t-------------call __str__-------------\n')
        return self.name


s = Student('baisong')
print('------ s is:', s)

print('---str--- s is:', str(s))

print('---repr---s is:', repr(s))


执行结果：
------ s is:    -------------call __str__-------------

baisong

-------------call __str__-------------

---str--- s is: baisong

---repr---s is: <__main__.Student object at 0x7f64d6fedd30>
```

### 只用__repr__实现
```python
class Student(object):

    def __init__(self, name):
        self.name = name

    def __repr__(self):
        print('\tcall __repr__')
        print('\t-------------call __str__-------------\n')
        return self.name


s = Student('baisong')
print('------ s is:', s)

print('---str--- s is:', str(s))

print('---repr---s is:', repr(s))



执行结果：
------ s is:    call __repr__
-------------call __str__-------------
baisong


call __repr__
-------------call __str__-------------
---str--- s is: baisong


call __repr__
-------------call __str__-------------
---repr---s is: baisong
```

### __repr__与__str__的不同
```python
从上边例子可以看出来__repr__与__str__的区别在于，后者实在str()被使用, 或者在
print的时候调用, __str__返回的字符串对于终端用户更友好。而__repr__是由python
的内置函数repr调用, 如果说你只想用其中的一个,建议使用__repr__, 因为python解释
器会在没有__str__的时候用__repr__代替(前边的例子print已经展示出来), 读者可以自
己试试当__repr__与__str__都存在的输出结果。
```