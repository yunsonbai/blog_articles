---
title: apns2 based on http2
date: 2016-10-20 10:34:02
tags:
	- apns
    - http2
    - python
categories: apns
---

## 引言

``` bash
    最近项目中需要用到apns，做了一下apns的功课，旧APNs有点反人类，而基于 HTTP/2 的
    全新APNs 协议则有巨大的优势，于是自己封装了一个基于http2协议的apns。
    欢迎大家提出修改意见。
```
## 项目地址
[apns2](https://github.com/yunsonbai/apns2)

<!--more-->
## 项目简介
* 应用：iso消息推送
* 版本：0.2
* 主要功能和特点:
	* 基于http2
    * 支持批量发送
* 安装
	* git clone git@github.com:yunsonbai/apns2.git
	* python setup.py install


## 例子
```python
from apns2.client import APNsClient
from apns2.payload import Payload
import uuid


def push():
    token_hexs = [
        'f9dcb885edbfcddsdaf64960bed56b19c362b3c617565126b085af2c8829467a',
        '9dc9662fa9dsd390fc6b3d3ec637a764867241f516d847b54d24ce7e37963b6e',
        'd52c31a293aa79c61222626b0d025866ssssa4d0c79a2b2bf729cd48c61f14a2',
        '8c51ed081211d19d90f4b34b403c39f5ceeeeec368cfd9277812a7af778fbf5f',
        '3df3c6bc4a98c38d5d13cbb932f4969f5f0efhhha8e4cd32270777f273dc0e41',
        'a25fbf0b5f84632406f33a78398138c87d13iiiiiiiiiiic466c11be9de22011',
        '3df3c6bc4a98c38d5d13cbb932f4969f5f0efyyyy8e4cd32270777f273dc0e41',
        'aef8a3d0b6d38807ad8ab485665d36edc305a9793jjj269214bcd3357f60e07a']
    print(len(token_hexs))
    payload = Payload(alert="Hello World!", sound="default")
    # custom = {
    #     'custom': {
    #         'type': 1
    #     }
    # }
    # payload = Payload(custom=custom, content_available=True)
    client = APNsClient(
        'Dev_Cer-key.pem',
        # 'Cer_Key.pem',
        use_sandbox=True,
        use_alternative_port=False)
    response = client.send_notification_multiple(token_hexs, payload)
    return response


def response_handler(response):
	# do something: delete badtoken and so on
    pass

if __name__ == '__main__':
    response = push()
	response_handler(response)
```