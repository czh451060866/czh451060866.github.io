---
title: android开机启动
author: 阿呆
date: 2020-10-09
categories: android
tags: 开机启动
---

## 思路一

注册一个静态广播去监听开机启动完毕的广播，然后在接听到开机广播完毕后启动Activity或者Service。
首先注册一个静态广播，监听action为“android.intent.action.BOOT_COMPLETED”类型的广播
```java
<receiver android:name=".BootCompletedReceiver">
        <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
        <category android:name="android.intent.category.HOME" />
        </intent-filter>
</receiver>
```
然后在manifest.xml中申请权限
```java
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```
最后声明一个receiver处理广播
```java
public class BootCompletedReceiver extends BroadcastReceiver {
 
    private static final String ACTION_BOOT_COMPLETED = "android.intent.action.BOOT_COMPLETED";
 
    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent.getAction().equals(ACTION_BOOT_COMPLETED)){
            Intent intent=new Intent(context,MainActivity.class);
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            context.startActivity(intent);
        }
    }
```
这个方法存在的缺陷：如果应用安装后必须手动运行一次app，才能收到android.intent.action.BOOT_COMPLETED广播

然而以上的解决思路在Android 3.1之后有了变更；

APP接收不到BOOT_COMPLETED广播可能的原因，有以下几种：

· BOOT_COMPLETED对应的action和uses-permission没有一起添加
· 应用安装到了sd卡内，安装在sd卡内的应用是收不到BOOT_COMPLETED广播的
· 系统开启了Fast Boot模式，这种模式下系统启动并不会发送BOOT_COMPLETED广播
· 应用程序安装后重来没有启动过，这种情况下应用程序接收不到任何广播，包括BOOT_COMPLETED、ACTION_PACKAGE_ADDED、CONNECTIVITY_ACTION等等。

Android3.1之后，系统为了加强了安全性控制，应用程序安装后或是(设置)应用管理中被强制关闭后处于stopped状态，在这种状态下接收不到任何广播，除非广播带有FLAG_INCLUDE_STOPPED_PACKAGES标志，而默认所有系统广播都是FLAG_EXCLUDE_STOPPED_PACKAGES的，所以就没法通过系统广播自启动了。

所以Android3.1之后:

应用程序无法在安装后自己启动
没有ui的程序必须通过其他应用激活才能启动，如它的Activity、Service、Content Provider被其他应用调用。
不过，存在一种例外，就是应用程序被adb push you.apk /system/app/下是会自动启动的，不处于stopped状态。解决方式就是将APK推送到/system/app目录下, 或者打包系统时，将APK放置到/system/app中打包

## 思路二

通过其他应用（命名为应用A）发送特定广播信号，我只需监控应用A发送的广播信号即可实现自启动了