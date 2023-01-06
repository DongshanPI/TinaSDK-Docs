## 2 Wi-Fi 简介

### 2.1 Wi-Fi 工作的几种模式

目前Tina 平台上的Wi-Fi 一般可处于3 种工作模式，分别是STATION，AP，MONITOR。
• STATION：连接无线网络的终端，大部分无线网卡默认都处于该模式，也是常用的一种模式。
• AP：无线接入点，常称热点，比如路由器功能。
• MONITOR：也称为混杂设备监听模式，所有数据包无过滤传输到主机。

### 2.2 Tina Wi-Fi 软件结构

![Tina_Linux_Wi-Fi_Development_Guide-image-20230103102646913](http://photos.100ask.net/tina-docs/Tina_Linux_Wi-Fi_Development_Guide-image-20230103102646913.png)

<center>图2-1: Tina 软件结构图</center>

• wifimanager-v2.0：包含了wifimanager-v1.0 的功能（用于STATION 模式，提供Wi-Fi连接扫描等功能）外，还集成了softap（启动AP 功能）和smartlink（多种配网模式）的功能，做到了一个应用集成了多种wifi 功能，方便客户使用和管理。
• wpa_supplicant: 开源的无线网络配置工具，主要用来支持WEP，WPA/WPA2 和WAPI 无线协议和加密认证的，实际上的工作内容是通过socket 与驱动交互上报数据给用户。
• hostapd: 是一个用户态用于AP 和认证服务器的守护进程。
• monitor: Wi-Fi 处于混杂设备监听模式的处理应用。

### 2.3 Wi-Fi 常用命令介绍

#### 2.3.1 station 模式

详情请看Tina_linux_wifimanger2.0_ 开发指南。

执行下面的命令前请确保wifi_deamon后台进程已启动，若没有启动请先启动wifi_deamon后台进程。

```
wifi -o sta 以sta模式打开wifimanager
wifi -s 扫描周围网络
wifi -c ssid [passwd] 以加密或非加密的方式连接指定网络
wifi -d 断开已经连接的网络
wifi -l [all] 列出保存的网络
wifi -a [enable/disable] 重连断开的网络
wifi -r [ssid/all] 移除保存的指定网络
```

```
注:
ssid 网络名
passwd 秘钥
```



#### 2.3.2 ap 模式

执行下面的命令前请确保wifi_deamon后台进程已启动，若没有启动请先启动wifi_deamon后台进程。

```
wifi -o ap [ssid] [passwd] 以ap模式打开wifimanager
wifi -l 列出连接到ap热点的sta信息
```

```
注:
ssid 网络名
passwd 秘钥
ap模式和station模式在不同模组上不一定能共存,详情看第5节介绍。
```



#### 2.3.3 monitor 模式

执行下面的命令前请确保wifi_deamon后台进程已启动，若没有启动请先启动wifi_deamon后台进程。

```
wifi -o monitor 以monitor模式打开wifimanager
```

```
注:
没有
```

#### 2.3.4 额外功能

执行下面的命令前请确保wifi_deamon后台进程已启动，若没有启动请先启动wifi_deamon后台进程。

```
wifi -f 关闭wifimanager
wifi -p [softap/ble/xconfig/soundwave] 使用softap/ble/xconfig/soundwave进行配网
wifi -D [error/warn/info/debug/dump/exce] 设置打印等级
wifi -g 获取mac地址信息
wifi -m [macaddr] 设置mac地址
wifi -h 打印wifimanager使用说明
```

```
注:
配网模式并不是所有模组都支持，要看具体的模组。
mac地址设置只能进行临时性设置。
```

