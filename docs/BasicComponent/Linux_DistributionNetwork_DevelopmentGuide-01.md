# Linux 配网开发指南
> Tina Linux wifimanager-v2.0 配网开发指南

## 1. 概述
### 1.1 编写目的
介绍Allwinner 平台上基于wifimanager-v2.0 的WiFi 配网方式，包括softap(WiFi ap 模式
热点配网),soundwave(声波配网),BLE(蓝牙低功耗配网)。

### 1.2 适用范围
• allwinner 软件平台tina v5.0 版本及以上，wifimanger 版本在2.0 版本以上；
• allwinner 硬件平台r 系列（r6，r11，r16，R18，R30，R40，R328，R331, R329,R818, R528…）。
• allwinner 硬件平台mr 系列（mr133, mr813…）。
• allwinner 硬件平台h 系列（h133…）。
• allwinner 硬件平台v 系列（v853…）。


### 1.3 相关人员

## 2. wifimanager-v2.0 配网

Tina 目前支持的WiFi 模组有全志Xradio，Broadcom AP 系列模组，RELTEK 的RTL 系列
模组, 乐鑫的ESP 系列模组。
为了方便wifi 的管理以及客户配网的简便性，wifimanger-v2.0 除了包含wifimanger1.0 的
sta 联网模式外，还支持了
ap 和monitor 模式，同时也把配网方式集成进去。用户只需要wifimangere-v2.0 一个应用即
可完成联网，开启ap 热点，
使用手机app 进行配网等多种功能。本文档只介绍wifimanager-v2.0 配网部分的功能，其他功
能请参考其他文档。
wifimanager-v2.0 支持的配网方式有soundwave（声波）、softap（热点）、以及蓝牙BLE
配网。


### 2.1 编译配置

```shell
make menuconfig
Allwinner --->
wireless --->
<*> wifimanager-v2.0................................... Tina wifimanager-v2.0
<*> wifimanager-v2.0-demo..................... Tina wifimanager-v2.0 app demo
[*] CONFIG_WMG_PROTOCOL_SOFTAP
[*] CONFIG_WMG_PROTOCOL_BLE
[*] CONFIG_WMG_PROTOCOL_SOUNDWAVE
```

### 2.2 demo 使用说明

1.执行wifi_deamon命令；该命令是启动wifimanager-v2.0的后台进程。
2.执行wifi -p XXX; 该命令是使用什么方式进行联网。

## 3. 测试说明
### 3.1 网测apk 获取途径

配网使用的手机app 可以在tina SDK 的以下路径获取到：
package/allwinner/wireless/wifimanager2.0/app

### 3.2 蓝牙配网测试
1. 板子通过串口连接PC 与开发板，系统起来，进入Linux shell；
2. 执行wifi_deamon 命令，启动wifimanager-v2.0 的后台进程。
3. 执行wifi -p ble 命令，启动蓝牙配网模式。
4. 启动手机蓝牙配网app Blink。
5. 点击SCAN 按钮后可以扫到蓝牙配对热点aw_bt_blink。
6. 点击aw_bt_blink 配对热点进行连接，并发送想要板子连接的ssid 和passwd。
7. 板子收到ssid 和passwd 后会进行路由的连接，连接上获取到ip 后就可以执行ping 测试
了。

[图3-2: 蓝牙配网](./images/3-2.jpg)


### 3.3 softap 配网
1. 板子通过串口连接PC 与开发板，系统起来，进入Linux shell；
2. 执行wifi_deamon 命令，启动wifimanager-v2.0 的后台进程。
3. 执行wifi -p softap 命令，启动softap 配网模式。
4. 此时手机可以扫描到Aw-wifimg-Test 热点，手机连接上。
5. 手机利用ckysoftAPDemo 　发送想要板子连接的ssid 和passwd。
6. 板子收到ssid 和passwd 后会进行路由的连接，连接上获取到ip 后就可以执行ping 测试
了。

[图3-3: softap 配网](./images/3-3.jpg)

