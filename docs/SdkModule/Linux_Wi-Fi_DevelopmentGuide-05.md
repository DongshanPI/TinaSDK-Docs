## 6 常见问题

### 6.1 编译问题

#### 6.1.1 找不到wowlan 变量

1. 现象：

  ```
  drivers/net/wireless/xr829/umac/main.c:870:17: error: 'struct wiphy' has no member named 'wowlan'
  if ((hw->wiphy->wowlan->flags || hw->wiphy->wowlan->n_patterns)
  ```

  

2. 原因：
   wowlan成员变量受CONFIG_PM控制，没有打开导致的．休眠唤醒的依赖。

3. 解决方案：

  ```
  在内核配置
  -Power management options --->
  Device power management core functionality
  ```

#### 6.1.2 找不到xxx.ko

1. 现象：

   ```
   sunxi_wlan_get_bus_index...xradio_core.ko undefined!
   ```

2. 原因：
   缺少配置misc。

3. 解决方案：
   在内核配置

   ```
   m kernel_menuconfig-->
      Device drivers-->
      	Misc　devices-->
      		Allwinner rfkill driver
   ```



#### 6.1.3 mmc_xxx undefined

1. 现象：

  ```
  drivers/built-in.o: In function scan_device_store':
  lichee/linux-4.9/drivers/misc/sunxi-rf/sunxi-wlan.c:309: undefined reference
  tosunxi_mmc_rescan_card'
  lichee/linux-4.9/drivers/misc/sunxi-rf/sunxi-wlan.c:309:(.text+0x5fc40): relocation
  truncated to fit:
  R_AARCH64_CALL26 against undefined symbol `sunxi_mmc_rescan_card'
  ```

2. 原因：
   没有配置mmc。

3. 解决方案

  ```
  Device Drivers --->
  	<*> MMC/SD/SDIO card support --->
  	<*> Allwinner sunxi SD/MMC Host Controller support
  ```

  

#### 6.1.4 缺少依赖库

1. 现象：

   ```
   Package kmod-net-xr829 is missing dependencies for the following libraries:
   cfg80211.ko
   mmc_core.ko
   sunxi-wlan.ko
   ```

   

2. 原因：
   依赖库需要编译进内核，不能以模块方式编译进去。

3. 解决方案。

   ```
   在内核配置如下模块时，配置成y
   CONFIG_RFKILL =y
   CONFIG_CFG80211=y
   CONFIG_MMC=y
   CONFIG_MAC80211=y
   ```

   

### 6.2 驱动加载问题

#### 6.2.1 R16 博通模组联网时提示：No such device.

1. 现象：

```
root@TinaLinux:/# wifi_add_network_test ssid passwd 1

*********************************

***Start wifi connect ap test!***

*********************************

wpa_suppplicant not running!
Cannot create "/data/misc/wifi/entropy.bin": No such file or directory
Wi-Fi entropy file was not created
ifconfig: SIOCGIFFLAGS: No such device
event_label 0x0
wifi on failed!
wifi on failed event 0xf001
```


2. 原因：

  ```
  firmware选择不匹配，导致驱动加载时下载失败．
  
  - lsmod查看驱动已经正常加载
  - dmesg 查看加载log发现：
    [ 22.336336] dhdsdio_download_code_file: Open firmware file failed /lib/firmware/
    fw_bcm43438a1.bin
    [ 22.346331] _dhdsdio_download_firmware: dongle image file download failed
    表示firmware固件缺失（这里表示缺失fw_bcm43438a1.bin）
    最后发现是在配置firmware时选择了ap6212,正常应该用ap6212a
  ```


3. 解决方案

  ```
  tina配置正确的firmware
  firmware --->
  └─> <*> ap6212a-firmware............................... Broadcom AP6212A firmware
  ```

#### 6.2.2 R329_XR829 模组ifconfig 显示：No such device

1. 现象：

   ```
   ifconfig: SIOCGIFFLAGS: No such device
   ```

   

2. 原因：

   ```
   firmware选择不匹配。
   
   - lsmod查看驱动已经正常加载。
   - dmesg 查看加载log发现：
     [ 195.966066] [XRADIO_ERR] xradio_load_firmware: Wait for wakeup:device is not responding.
     XR829换了40M晶振。
   ```

   

3. 解决方案

   ```
   tina配置选择40M晶振的firmware
   firmware --->
   	[*] xr829 with 40M sdd
   ```

#### 6.2.3 MR133_XR829 can’t open /etc/wifi/xr_wifi.conf, failed

1.现象：

lsmod驱动没有正常加载。

原因：

```
dmesg查看log：
[ 6.802331] [XRADIO_ERR] can't open /etc/wifi/xr_wifi.conf, failed(-30)
[ 6.802338] [XRADIO_ERR] Access_file failed, path:/etc/wifi/xr_wifi.conf!
[ 6.914044] sunxi-mmc sdc1: no vqmmc,Check if there is regulator
[ 7.028376] [XRADIO_ERR] xradio_load_firmware: Wait_for_wakeup: can't read control
register.
busnum配置错误，原理图上使用的是sdc0.
```

2.解决方案

```
board.dts中配置wlan时
busnum = 0;
```



#### 6.2.4 驱动加载问题总结

##### 6.2.4.1 配置问题

```
1.内核驱动，Tina modules, Tina firmware三者必须正确对应同一个模组。
2.注意common下的modules.mk的编写。
3.Sdio的配置一定要根据原理图选择对应busnum。
可能导致：
1.扫卡失败。
2.下载firmware失败。
最终导致驱动加载失败。
```

##### 6.2.4.2 供电问题

```
检查VCC_WIFI和VCC_IO_WIFI两路电。
不同模组对供电时序有一定要求，比如RTL8723ds需要两路电同时供电，针对有AXP的方案，一定要注意供电的配置，特别是enable的时间。
１．硬件方面：主要排查两路电的供电方案，是否是同一路供电，若是分开供电，要考虑两路供电的时序，
例如DCDC1--->VCC_WIFI,LDOA--->VCC_IO_WIFI,那么DCDC1和LDOA的时序就得考虑。
２．软件方面主要是sysconfig.fex或者boart.dts的配置，分开供电的是否需要单独配置。
如：R818硬件设计是两路电分开供电。
可能导致：
1.扫卡失败。
2.下载firmware失败。
3.sdio_clk没有时钟。
4.32k竞争不起振。
最终导致驱动加载失败。
```

##### 6.2.4.3 sdio 问题

```
1.sdio busnum配置错误．
2.驱动WL-REG-ON的方式不对．例如：
XR819模块出现
[SBUS_ERR] sdio probe timeout!
[XRADIO_ERR] sbus_sdio_init_failed
这个问题主要是sdio扫卡失败，跟sdio上电时序有关，可在drivers/net/wireless/xradio/wlan/platform.c中
xradio_wlan_power函数sunxi_wlan_set_power(on)后面加上一段延时。
RLT8723ds需要先高－低－高的方式．
可能导致：
1.扫卡失败。
2.下载firmware失败。
3.sdio_clk没有时钟。
4.32k晶振不起振。
5.WL-REG-ON无法正常被拉高。
最终导致驱动加载失败。
```

### 6.3 起wlan0 网卡问题

#### 6.3.1 R818_RTL8723ds ifconfig wlan0 up: No such device

1.现象：

```
ifconfig: SIOCGIFFLAGS: No such device
```



2.原因：

```
- lsmod查看驱动已经正常加载。
- dmesg查看log未发现异常。
- 排查sdio_clk, regon_on,32k,都正常。
- 两路供电都正常配置。
- 对比其他平台硬件发现，供电方式不一样，两路电采用了分开供电，咨询RTL需要同时上电。
```

3.解决方案
硬件更改，VCC_WIFI/VCC_IO_WIFI用同一路电供电。

#### 6.3.2 R328_RTL8723ds 无法自启动wlan0

1.现象：

```
启动脚本/etc/init.d/wpa_supplicant中会自启动wlan0
但是每次启动启动都自启动失败，然后手动ifconfig wlan0 up正常。
```

2.原因：

```
AP-WAKE_BT引脚被接了上拉电阻，进入测试模式了。
```

3.解决方案
硬件摘除上拉电阻。

#### 6.3.3 起wlan0 网卡问题总结

wlan0启动失败问题目前遇到的都是与硬件相关的，如果不能自加载一般采用ifconfig wlan0 up先手动加载看看打印提
示。
同时让硬件帮忙check一下供电和一些io的上下拉电阻。

### 6.4 supplicant 服务问题

#### 6.4.1 找不到wpa_suplicant.conf 文件

1.现象：
起supplicant失败

```
- ps发现没有supplicant进程．
- 于是手动执行wpa_supplicant -D nl80211 -i wlan0 -c /etc/wpa_supplicant.conf -B
  提示：
  Failed to open config file '/etc/wpa_supplicant.conf', error: No such file or directory
  Failed to read or parse configuration '/etc/wpa_supplicant.conf'.
```

2.原因：
路径错误。

3.解决方案

```
tina正常的路径一般在/etc/wifi/wpa_supplicant.conf
在wifimanage包下面配置正确的路径，保持和启动脚本一致．
```



### 6.5 wifimanager 使用问题

#### 6.5.1 联网时出现：network not exist!

1.现象：

```
wifi_connect_ap_test ssid passwd
network not exist!
```

2.原因：

```
- lsmod查看驱动已经正常加载。
- ifconfig查看wlan0已经正常up。
- ps查看supplicant服务已经正常启动。
- 使用wifi_scan_results_test扫描网络
root@TinaLinux:/# wifi_scan_results_test
*********************************
***Start scan!***
*********************************
bssid / frequency / signal level / flags / ssid
******************************
Wifi get_scan_results: Success!
******************************
没有任何网络扫描到。
```

3.解决方案
一般是信号太多，没有板载天线，尝试外加一根天线。

### 6.6 上层网络应用服务问题

#### 6.6.1 MR133_XR829 ping 压力测试: poll time out

1.现象：

```
ping 压力测试，一段时间后出现poll time out。
```

2.原因：

```
ping的网络性能不好．连接的公司内网可能存在一些未知的限制。
```

3.解决方案

```
尝试连接另外的路由器测试。
```

