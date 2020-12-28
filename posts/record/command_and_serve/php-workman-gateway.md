---
title: php-workman-gateway
date: 2017-06-27 00:00:00
tags: ["php"]
categories: ["记录"]
draft: true
---

# php-gateway

> 在[自己的项目](https://github.com/yiiouo/js-vod-system)中，用到了php的gateway，gateway是基于workerman再次开发的一个网络框架，看下[手册](http://doc2.workerman.net)和DEMO，就可以基本的使用了。

## 如何使用

### 下载，增加php环境变量

我电脑是Windows，所以下载的是Windos版本。在官网下载文件后（[下载地址](http://www.workerman.net/download)），随意找个地方解压就好了。然后在系统->高级系统设置->环境变量->用户变量中的path，编辑->添加你的php所在的目录地址。

我用的是wamp，所以php的地址是：D:\wamp\bin\php\php5.3.10  

最后打开解压后的文件，会有个start_for_win.bat的批处理，点击打开它就OK了。这样就会打开服务，可以使用。

### 使用Websocket

Gateway文件夹，大部分文件我们不用理的，只要关注Applications/YourApp/文件中的Events.php 和start_gateway.php这两个文件就行了。

Events.php是处理业务代码。start_gateway.php是处理使用什么协议，端口是多少。

因为我用的是Websocket，在start_gateway.php中，将协议改成webscoket,并写上ip地址。

    $gateway = new Gateway("websocket://127.0.0.1:8282");

这里ip地址如果是`127.0.0.1`，只能让本机使用，`本机ip地址`，能让内网中的ip使用。

## 开发功能

### Events.php

在Events.php中，大概结构是这样的：

    class Events{
       public static function onConnect($client_id){}
       public static function onMessage($client_id,$message){}
       public static function onClose($client_id){}
    }

* onConnect：当链接连接成功后的回调函数
* onMessage：当客户端发送信息后的回调函数
* onClose：当链接关闭后的回调函数

主要用到的是`onMessage`，客户端通过websocket发送信息，就会触发此函数，我们就可以通过gateway给的一些方法，把要处理的数据，发送给全部用户或者特定的用户。

每个函数的参数功能，都如同它的名字一样。`$client_id`是发送数据的用户id，这个id由Gateway生成的，不用我们是理。`$message`是用户发送的数据，发送的数据格式，最好就是字符串，所以前端后端，要把数据发送出去，都要先将它转成字符串。

    public static function onConnect($client_id){
    	//当客户端连接成功后，就发送一条当前用户的id给所有连接此连接的客户端
        Gateway::sendToAll("$client_id");
    }

### 前端

前端的代码也是很简单

	<div>send data</div>
	<script>
		//这里的地址要和start_gateway.php中注册的Gateway地址一样
		var ws = new WebSocket('ws:127.0.0.1:8282')
		ws.addEventListener('open', function(e){
			console.log(e)
		}, false)
		ws.addEventListener('message', function(e){
			console.log(e)
		}, false)
		document.querySelector('div').addEventListener('click',function(){
			ws.send('我发了信息，111')
		},false)
	</script>

页面打开后，连接成功，就会收到了一条信息。如果我们将再新建一个页面，同样的代码，两个页面都会收到信息。

同样，我们主要写的代码，也是放在`message`的回调函数里。

### Gateway常用的方法

这里列出我用到的，详细大家可以看手册。

* sendToAll：对所有用户发送数据，`Gateway::sendToAll($data)`
* sendToClient：对某个用户发送数据，`Gateway::sendToClient($client_id,$data)`
* joinGroup：添加用户到某个组，`Gateway::joinGroup($group_name,$client_id)`
* sendToGroup：对某个组内的所有用户发送数据,`Gateway::sendToGroup($group_name,$data)`

我现在暂时用到这4个，其他还没研究。这里说下Gateway的group。一个组可以含有多个用户，一个用户可以加入多个组，而且用户断线后，会自动从组里删除。这个是非常好用的，指定某个群组的人，发送数据。