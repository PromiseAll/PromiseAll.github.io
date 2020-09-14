---
title: adb的基本使用
tag: []
categories: [Android]
date: 2020-09-14 10:41:12
---

## adb连接设备

### 开启/关闭 adb 服务

> adb start-server		（一般无需手动执行此命令，在运行 adb 命令时若发现 adb server 没有启动会自动调起）

> adb kill-server

### 查看版本

> adb version

### 查看设备

> adb devices

### 通过 WIFI 进行远程连接调式

需要手机与本地电脑连接同个路由器

1. 指定 adb 远程调试端口（默认端口 5555）

   > // 使用adb开启（需要先使用usb数据线先连接）
   >
   > adb tcpip [port]
   >
   > 
   >
   > // 以root权限开启
   >
   > su
   > setprop service.adb.tcp.port 5555

2. 连接

   > adb connect [ip]:[port]

3. 断开

   > adb disconnect [ip]



## 基本使用

### 以 root 权限运行 adbd

> adb root



### 安装应用

> adb install [-lrtsdg] [path_to_apk]

| 参数 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| -l   | 将应用安装到保护目录 /mnt/asec                               |
| -r   | 允许覆盖安装                                                 |
| -t   | 允许安装 AndroidManifest.xml 里 application 指定 `android:testOnly="true"` 的应用 |
| -s   | 将应用安装到 sdcard                                          |
| -d   | 允许降级覆盖安装                                             |
| -g   | 授予所有运行时权限                                           |



### 卸载应用

> adb uninstall [-k] [packagename]

`packagename` 表示应用的包名，`-k` 参数可选，表示卸载应用但保留数据和缓存目录



### 清除应用数据与缓存

> adb shell pm clear [packagename]
>



## 详细文档

[awesome-adb](https://mazhuang.org/awesome-adb/)

