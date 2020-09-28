---
title: adb通过root权限安装卸载应用
author: 阿呆
date: 2020-09-27
categories: android
tags: adb root install
---

卸载：
adb root： 切换到root用户
adb remount：将/system部分置于可写模式，默认只读
adb shell
cd /system/app
rm -rf apk
sync：将内存缓冲区的内容同步更新到磁盘
reboot


安装：
adb shell
su
am broadcast -a com.meituan.hardware.ENABLE_CHECK_SIGN -n android/com.android.server.pm.PackageConfigReceiver --ez enable false //告诉系统安装时忽略检查签名

需要root
getprop ro.serialno  //获取sn码
input Text xxx   //输入
