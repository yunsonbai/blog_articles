---
title: python+mqtt实现推送
date: 2016-05-24 16:18:30
tags:
    - python
    - mqtt
    - 推送
categories: python
---
## 引言
``` bash
    前段时间趁着空闲时间研究了一下mqtt,自己用python简单的实现了一下,希望日后能用上
```
<!--more-->
## 安装mosquitto并使用命令行
### 环境说明
``` bash
    python2.7
    paho-mqtt 1.1
```

### 安装安装mosquitto
``` bash
    sudo apt-add-repository ppa:mosquitto-dev/mosquitto-ppa
    sudo apt-get update
    sudo apt-get install mosquitto mosquitto-clients python-mosquitto
    或者：
         apt-get install mosquitto
```

### 安装paho-mqtt

``` bash
    pypi上有这个库，可以自行安装
```

### 命令行
``` bash
    启动命令: /usr/sbin/mosquitto -c /etc/mosquitto/mosquitto.conf
    server：mosquitto_pub -t baiyunsong -h 127.0.0.1 -m "{\"pin\":17,\"value\":0}"
    client：mosquitto_sub -v -t baiyunsong -h 127.0.0.1    （先启动）
```

## python代码
### server端
``` bash
    # import paho.mqtt.client as mqtt
    from paho.mqtt.publish import mqtt
    import json
    mqttc = mqtt.Client()
    mqttc.connect('127.0.0.1', port=1883)
    msg = {
        'pin': 17,
        'value': 10
    }
    msg = json.dumps(msg)

    print mqttc.publish('baisong', payload=msg)
    mqttc.loop(2)
```
### client 端
``` bash
    # coding=utf-8
    import paho.mqtt.client as mqtt
    import json
    #
    def on_connect(client, userdata, flags, rc):
        print('Connected with result code ' + str(rc))
        client.subscribe('baisong')


    def on_message(client, userdata, msg):
        print msg.topic + ' ' + str(msg.payload)
        print json.loads(msg.payload)

    if __name__ == '__main__':
        client = mqtt.Client()
        client.on_connect = on_connect
        client.on_message = on_message

        try:
            client.connect('127.0.0.1', port=1883)
            client.loop_forever()
        except KeyboardInterrupt:
            client.disconnect()

```