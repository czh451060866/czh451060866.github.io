---
title: websocket学习
author: 阿呆
date: 2020-10-14
categories: 网络协议
tags: websocket
---

## 概述
websocket是应用层协议，他是一种双向通信协议，建立在TCP基础上，握手过程：

1. 浏览器、服务器建立TCP连接，三次握手。这是通信的基础，传输控制层，若失败后续都不执行。
2. TCP连接成功后，浏览器通过HTTP协议向服务器传送WebSocket支持的版本号等信息。（开始前的HTTP握手）
3. 服务器收到客户端的握手请求后，同样采用HTTP协议回馈数据。
4. 当收到了连接成功的消息后，通过TCP通道进行传输通信。

## 和http的关系

相同点：都是基于TCP的，都是可靠传输协议；都是应用层协议  
不同点：websocket是双向通信协议，模拟socket，可以双向发送和接受信息；http是单向的
websocket是需要握手建立连接的  
联系： websocket建立连接时是通过http传输的，建立连接之后不需要http协议

## 和socket的关系
其实就是没关系
socket不是协议，是为了方便使用tcp和udp抽象出来的一层，位于传输控制层，websocket是应用层协议