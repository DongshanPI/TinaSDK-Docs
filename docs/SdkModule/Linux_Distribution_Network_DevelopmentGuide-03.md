# 3 测试说明

## 3.1 网测apk 获取途径

配网使用的手机app 可以在tina SDK 的以下路径获取到：package/allwinner/wireless/wifimanager2.0/app

### 3.2 蓝牙配网测试

1. 板子通过串口连接PC 与开发板，系统起来，进入Linux shell；
2. 执行wifi_deamon 命令，启动wifimanager-v2.0 的后台进程。
3. 执行wifi -p ble 命令，启动蓝牙配网模式。
4. 启动手机蓝牙配网app Blink。
5. 点击SCAN 按钮后可以扫到蓝牙配对热点aw_bt_blink。
6. 点击aw_bt_blink 配对热点进行连接，并发送想要板子连接的ssid 和passwd。
7. 板子收到ssid 和passwd 后会进行路由的连接，连接上获取到ip 后就可以执行ping 测试了。

![image-20230104105423666](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina-Linux_configNet_image-20230104105423666.png)

## 3.3 softap 配网

1. 板子通过串口连接PC 与开发板，系统起来，进入Linux shell；
2. 执行wifi_deamon 命令，启动wifimanager-v2.0 的后台进程。
3. 执行wifi -p softap 命令，启动softap 配网模式。
4. 此时手机可以扫描到Aw-wifimg-Test 热点，手机连接上。
5. 手机利用ckysoftAPDemo 　发送想要板子连接的ssid 和passwd。
6. 板子收到ssid 和passwd 后会进行路由的连接，连接上获取到ip 后就可以执行ping 测试了。

![image-20230104105436185](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina-Linux_configNet_image-20230104105436185.png)