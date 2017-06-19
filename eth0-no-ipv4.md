---
title: 当eth0没有ipv4地址的解决办法
date: 2016-06-09 22:27:22
tags:
	- Linux
    - eth0没有ipv4地址
    - ipv4地址
    - 配置eth0
    - ifconfig
categories: Linux
---

## 引言

``` bash
    有时候我们在Linux使用ifconfig后发现没有ipv4地址，下边说一下解决办法
```

<!--more-->
## 打开ubuntu的/etc/network/interfaces
``` bash
	删除已经有的，后根据喜好选择添加如下
	自动：
          auto eth0
          iface eth0 inet dhcp
     静态分配的配置方法：
          auto eth0
          iface eth0 inet static
          address 192.168.205.139
          netmask  255.255.255.0
          gateway  192.168.205.1
```

## 添加域名服务器
``` bash
	打开/etc/resolv.conf文件
    添加这行:nameserver 127.0.0.1
```

## 重启eth0
``` bash
    $/etc/init.d/networking restart(这条命令是重启网卡)
     或者
    $ifdown eth0
    $ifup   eth0（这两条命令是有针对性的重启某个网络接口，因为一个系统可能有多个网络接口）
```