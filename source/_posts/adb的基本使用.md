---
title: adb的基本使用
tag: []
categories: [Android]
date: 2020-09-14 10:41:12
---

## adb连接设备

### 开启/关闭 adb 服务

> adb start-server
>
> adb kill-server

### 查看设备

> adb devices

### 通过 WIFI 进行远程连接调式

需要手机与本地电脑连接同个路由器

1. 指定 adb 远程调试端口（默认端口 5555）

   > adb tcpip [port]

2. 连接

   > adb connect [ip]:[port]

3. 断开

   > adb disconnect [ip]

## 基本使用

