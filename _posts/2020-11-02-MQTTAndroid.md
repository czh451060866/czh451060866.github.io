---
title: 使用ActiveMQ测试mqtt
author: 阿呆
date: 2020-11-02
categories: 网络协议
tags: mqtt协议
---

## 后台配置
使用Appache服务器，支持mqtt协议。

1.下载：从ActiveMQ官网下载ActiveMQ，地址：http://activemq.apache.org/download.html
2.安装：下载下来的是一个压缩包，解压即安装，直接解压到一个目录即可；mac直接下载tar包
3.初次运行：（在启动ActiveMQ前，请先要已经安装和配置好JDK）在mac版本的activemq中在activeMQ/bin下直接运行 ./activemq start，注意权限，chmod 755
4.检测是否启动：ActiveMQ默认使用61616端口提供JMS服务（Java消息服务指的是两个应用程序之间进行异步通信的API），使用8161端口提供管理控制台服务，我们可以直接访问activemq的管理控制台网页来确定是否已经开始服务：localhost:8161/admin，默认的用户名和密码都是admin。很坑的是，在使用MqttAndroidClient编写程序时，需要使用1883端口。（todo：看看为啥）

在后台创建一个topic。
![](/screenshot/MQTT-1.png)

一些tips：
如果你想在模拟器simulator上面访问你的电脑，那么就使用android内置的IP 10.0.2.2 吧， 10.0.2.2 是模拟器设置的特定ip，是你的电脑的别名alias

记住，在模拟器上用10.0.2.2访问你的电脑本机

activeMQ服务器，jetty.xml中的host应该改为 0.0.0.0，真机才能访问

编写程序时，MQTT URL必须以tcp协议开头。

## 使用
首先添加依赖
```java
compile 'org.eclipse.paho:org.eclipse.paho.client.mqttv3:1.1.0'
compile 'org.eclipse.paho:org.eclipse.paho.android.service:1.1.1'
```

添加权限
```java
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
```

注册service
```java
 <service android:name="org.eclipse.paho.android.service.MqttService" />
```
这个service是sdk中使用到的，必须声明

创建MqttAndroidClient对象，设置callback，在断开连接，消息到来，消息送达会回调
```java
mClient = new MqttAndroidClient(getApplicationContext(), MQTT_URL, getClientId());
mClient.setCallback(mMqttCallback);
```
其中MQTT_URL为 tcp://ip:1883；至于为什么是1883端口，我再看看

创建option，输入用户名密码
```java
		mOptions = new MqttConnectOptions();
        mOptions.setMqttVersion(MqttConnectOptions.MQTT_VERSION_3_1_1);
        mOptions.setAutomaticReconnect(false);
        mOptions.setCleanSession(true);
        mOptions.setKeepAliveInterval(KEEP_ALIVE_INTERVAL);
        mOptions.setUserName("admin");
        mOptions.setPassword("admin".toCharArray());
        mOptions.setMaxInflight(100);
```
连接服务器
```java
mClient.connect(mOptions, MQTTApplication.getApplication(), mIMqttConnectListener);
```
在onSuccess中注册消息
```java
mClient.subscribe(mTopic, 0, MQTTApplication.getApplication(), mIMqttSubscribeListener);
```

## 结果
从后台下发一个消息
![](/screenshot/MQTT-2.png)
客户端会收到消息并回调messageArrived

向服务器发送一个消息
```java
mClient.publish(mTopic, "publish msg".getBytes(), 0, false, MQTTApplication.getApplication(), new IMqttActionListener());
```

如果客户机发布至它也预订了的某个主题，那么它将接收到它自己的发布的副本。
如果使用值为 1 或 2 的 QoS 发送消息，那么该消息将由 MqttClientPersistence 类存储，之后 MQTT 客户机将调用 messageArrived