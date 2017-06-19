---
title: fiddler代理利器
date: 2016-05-12 18:38:02
tags:
    - fiddler
    - 代理
    - 调试
categories: fiddler
---
## 引言
```
    fiddler在前端开发工程师那可以说是利器,同样后端工程师也可能用到尤其是在利用app调试api的时候，
    也许你就会用到fiddler
```
<!--more-->
### 只代理某个url
#### 应用
```
比如把线上某个url代理到本地/或者调试某个接口,便于调试
```
#### 配置
``` bash
点击AotuResponder,之后把三个复选框全选上,然后点击add ruler
加入:
    regex:.*yunsonbai.com /activity/(.*)
    http://127.0.0.1:8000/activity/$1

```

### 设置host
#### 应用 
```
把某个域全部代理到本地
```
#### 配置
``` bash
点击Tools->HOSTS->选上复选框
加入:
    127.0.0.1:8000 yunsonbai.com
```

### 如何让手机也走设置好的代理
#### 应用
```
调试移动端应用
```
#### 配置
``` bash
设置wifi代理为手动；
servername: 你电脑的ip/或域名
port: 8888（fiddler默认是启动8888端口）
```

More info: [Generating](http://www.telerik.com/fiddler)
