---
title: 创建自己的pypi库
date: 2016-05-23 15:36:06
tags:
    - python
    - pypi库
    - 创建自己的pypi库
    - pypiserver
    - pypi-server
categories: python
---
## 引言
``` 
    在做项目的时候你会发现有好多重复性的工作在做着，如果我们能抽出公共部分的话那样你的工作就会事半功倍，那抽出
    来的公共部分就需要存放在自己的pypi库（也不排除你还有其他的办法）
```
<!--more-->
## 搭建服务端
### 创建环境
``` bash
    如果你不想混乱自己的环境可以利用virtualenv
    virtualenv3.5 pypi
```
### 安装需要的库
``` bash
    yum install apache2-utils
    pip install pypiserver
    pip install passlib
```
### 创建packages文件夹和配置文件.htaccess
``` bash
    mkdir packages
    htpasswd -sc .htaccess user（之后输入密码，例如123456）
```
### 启动服务
``` bash
   pypi-server -p 3141 -P ./.htaccess ./packages
```
## 客户端上传公共库
### 在用户根目录下添加配置文件.pypirc
``` bash
    比如你是root用户：
        cd
        vim .pypirc
        加入内容如下：（distutils处一定要换行）
        [distutils]
        index-servers =
          local

        [local]
        repository:http://xx.xx.xx.xx:3141
        username:user
        password:123456

```
### 上传包
``` bash
    python setup.py sdist upload -r local
```
### 安装
``` bash
    pip install -i http://localhost:3134/simple/ some-package
```