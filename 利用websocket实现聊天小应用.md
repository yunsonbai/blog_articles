---
title: 利用websocket实现聊天小应用
date: 2016-12-18 14:48:15
tags:
	- websocket
    - 聊天demo
    - chat demo
    - python
categories: python
---


## 引言

``` bash
    趁着清闲研究了一下websocket，并利用python和html简单的实现了简单的聊天系统，
	希望对想要了解的有所帮助，有什么好的意见还请大家提出。
```

<!--more-->
## Overview
* 编程语言：python，html
* 实现功能： 在浏览器开启两个对话窗口并进行聊天
* 需要的第三方库：
	* pip install websockets
* 关于websocke:
	* WebSocket protocol 是HTML5一种新的协议
	* 实现了浏览器与服务器全双工通信
* 我为什么要用：
	* 实现web页面的即时通讯
	* 去除轮询带来的诸多缺点（这个网上一搜一大把）

## 关于流程图
* 建立连接过程
![Catch8D7B](https://yunsonbai.github.io/images/push/jl.jpg)
* 发信过程
![Catch8D7B](https://yunsonbai.github.io/images/push/fx.jpg)

## 后端代码
``` python
    # codind=utf-8
	import asyncio
	import websockets
	import json
	import time
	ALLSOKETS = {}


	def add_connection(ws_id, ws):
		global ALLSOKETS
		ALLSOKETS[ws_id] = ws
		print(ALLSOKETS)


	def filter_handle(ws_id):
		return ALLSOKETS.get(ws_id)


	async def handler(websocket, path):
		# print(path)
		while True:
			message = await websocket.recv()
			message = json.loads(message)
			login = message.get('login')
			if login:
				add_connection(message['user_id'], websocket)
			else:
				ws = filter_handle(message['to_user_id'])
				if ws:
					await ws.send(message['msg'])
			print('recv:', message)
			print(time.time())

	start_server = websockets.serve(
		handler, 'localhost', 8765, klass=websockets.WebSocketServerProtocol)
	asyncio.get_event_loop().run_until_complete(start_server)
	asyncio.get_event_loop().run_forever()

```

## 前端代码
* client1

```
	<html>
		<head>
		<title>WebSocket1</title>
		<style>
			html,body{font:normal 0.9em arial,helvetica;}
			#log {width:440px; height:200px; border:1px solid #7F9DB9; overflow:auto;}
			#msg {width:330px;}
		</style>
		<script>
			var socket;
			function init(){
				var host = "ws://127.0.0.1:8765/";
				socket = new WebSocket(host);
				try{
					socket.onopen    = function(msg){ ; };
					socket.onmessage = function(msg){ log(msg.data); };
				}
				catch(ex){
					socket.onclose   = function(msg){ log("Lose Connection!"); };
					log(ex); 
				}
				$("msg").focus();
			}
			function send(){
				var txt,msg;
				txt = $("msg");
				msg = txt.value;
				if(!msg){ 
					alert("Message can not be empty"); 
					return; 
				}
				log(msg)
				msg = '{"to_user_id": "WebSocket2", "msg":"' + msg + '"}'
				txt.value="";
				txt.focus();
				try{ 
					socket.send(msg); 
				} 
				catch(ex){ 
					log(ex); 
				}
			}
			function login(){
				msg = '{"user_id": "WebSocket1", "login": "1"}'
				socket.send(msg); 
			}         

			function $(id){ 
				var id = document.getElementById(id);
				return id;
			}
			function log(msg){ 
				$("log").innerHTML+="<br>"+msg; 
			}
			function onkey(event){ 
				if(event.keyCode==13){ 
					send(); 
				} 
			}
		</script>
		</head>
		<body onload="init()">
			<h3>WebSocket1</h3>
			<br><br>
			<button onclick="login()">login</button>
			<div id="log"></div>
			<input id="msg" type="textbox" onkeypress="onkey(event)"/>
			<button onclick="send()">发送</button>
		</body>
	</html>
```

* client2

```
	与client1差不多只是改下边两处即可
	function send(){
		var txt,msg;
		txt = $("msg");
		msg = txt.value;
		if(!msg){
			alert("Message can not be empty"); 
			return; 
		}
		log(msg)
		msg = '{"to_user_id": "WebSocket1", "msg":"' + msg + '"}'
		txt.value="";
		txt.focus();
		try{ 
			socket.send(msg); 
		} 
		catch(ex){ 
			log(ex); 
		}
	}
	function login(){
		msg = '{"user_id": "WebSocket2", "login": "1"}'
		socket.send(msg); 
	}
```


## 具体操作

* 开启后端代码 python server.py
* 打开浏览器，访问html文件，分别打开client1.html和client2.html
* 相互发信

## 效果图
* clent1
![Catch8D7B](https://yunsonbai.github.io/images/push/c1.jpg)
* clent2
![Catch8D7B](https://yunsonbai.github.io/images/push/c2.jpg)

## 还需要优化的地方
* 添加鉴权
	* header中添加token
	* 或者通过参数等等
* 添加nginx代理, 网上有很多说明，记得以下几个参数
	* proxy_http_version 1.1
	* proxy_set_header Upgrade $http_upgrade
	* proxy_set_header Connection "upgrade"	