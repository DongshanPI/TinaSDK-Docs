# 2 wifimanager-v2.0 配网

Tina 目前支持的WiFi 模组有全志Xradio，Broadcom AP 系列模组，RELTEK 的RTL 系列模组, 乐鑫的ESP 系列模组。
为了方便wifi 的管理以及客户配网的简便性，wifimanger-v2.0 除了包含wifimanger1.0 的sta 联网模式外，还支持了ap 和monitor 模式，同时也把配网方式集成进去。用户只需要wifimangere-v2.0 一个应用即可完成联网，开启ap 热点，
使用手机app 进行配网等多种功能。本文档只介绍wifimanager-v2.0 配网部分的功能，其他功能请参考其他文档。
wifimanager-v2.0 支持的配网方式有soundwave（声波）、softap（热点）、以及蓝牙BLE配网。

## 2.1 编译配置



```
make menuconfig
Allwinner --->
wireless --->
<*> wifimanager-v2.0................................... Tina wifimanager-v2.0
<*> wifimanager-v2.0-demo..................... Tina wifimanager-v2.0 app demo
[*] CONFIG_WMG_PROTOCOL_SOFTAP
[*] CONFIG_WMG_PROTOCOL_BLE
[*] CONFIG_WMG_PROTOCOL_SOUNDWAVE
```

## 2.2 demo 使用说明

```
1.执行wifi_deamon命令；该命令是启动wifimanager-v2.0的后台进程。
2.执行wifi -p XXX; 该命令是使用什么方式进行联网。
```

