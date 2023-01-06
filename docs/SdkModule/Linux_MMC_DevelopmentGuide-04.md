## 5 FAQ

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxMMCDevelopmentGuide_004.png)

1、问：sd 卡和 wifi 经常出现 retry：start 等打印的原因和原理；

答：出现这种情况是因为 sdio 通信失败了，一般是数据 crc 校验错误；对于这种错误主要怀疑，

硬件信号受到干扰；而这个 retry 机制是通过重发改变相位等方法来规避单次通信失败；在通信

无法回复的情况下，会出现大量的 retrylog，此时请检查硬件信号，供电等关键信息。



### 5.1 调试方法

#### 5.1.1 调试工具

#### 5.1.2 调试节点

##### 5.1.2.1 1. 寄存器信息

linux5.4 内核

a.sdc2

(1).sdc2 gpio 寄存器信息

cat /sys/devices/platform/soc@2900000/4022000.sdmmc/sunxi_dump_gpio_register

(2).sdc2 ccmu 寄存器信息

cat /sys/devices/platform/soc@2900000/4022000.sdmmc/sunxi_dump_ccmu_register

(3).sdc2 host 寄存器信息

cat /sys/devices/platform/soc@2900000/4022000.sdmmc/sunxi_dump_host_register

b.sdc0

(1).sdc0 gpio 寄存器信息

cat /sys/devices/platform/soc@2900000/4020000.sdmmc/sunxi_dump_gpio_register

(2).sdc0 ccmu 寄存器信息

cat /sys/devices/platform/soc@2900000/4020000.sdmmc/sunxi_dump_ccmu_register

(3).sdc0 host 寄存器信息

cat /sys/devices/platform/soc@2900000/4020000.sdmmc/sunxi_dump_host_register

（4）手动扫描卡接口

echo 1 > /sys/devices/platform/soc@2900000/4020000.sdmmc/sunxi_insert

c.sdc1

(1).sdc1 gpio 寄存器信息

cat /sys/devices/platform/soc@2900000/4021000.sdmmc/sunxi_dump_gpio_register

(2).sdc1 ccmu 寄存器信息

cat /sys/devices/platform/soc@2900000/4021000.sdmmc/sunxi_dump_ccmu_register

(3).sdc1 host 寄存器信息

cat /sys/devices/platform/soc@2900000/4021000.sdmmc/sunxi_dump_host_register

linux4.9 内核

a.sdc2

(1).sdc2 gpio 寄存器信息

cat /sys/devices/platform/soc/sdc2/sunxi_dump_gpio_register

(2).sdc2 ccmu 寄存器信息

cat /sys/devices/platform/soc/sdc2/sunxi_dump_ccmu_register

(3).sdc2 host 寄存器信息

cat /sys/devices/platform/soc/sdc2/sunxi_dump_host_register

b.sdc0

(1).sdc0 gpio 寄存器信息

cat /sys/devices/platform/soc/sdc0/sunxi_dump_gpio_register

(2).sdc0 ccmu 寄存器信息

cat /sys/devices/platform/soc/sdc0/sunxi_dump_ccmu_register

(3).sdc0 host 寄存器信息

cat /sys/devices/platform/soc/sdc0/sunxi_dump_host_register

（4）手动扫描卡接口

echo 1 > /sys/devices/platform/soc/sdc0/sunxi_insert

c.sdc1

(1).sdc1 gpio 寄存器信息

cat /sys/devices/platform/soc/sdc1/sunxi_dump_gpio_register

(2).sdc1 ccmu 寄存器信息

cat /sys/devices/platform/soc/sdc1/sunxi_dump_ccmu_register

(3).sdc1 host 寄存器信息

cat /sys/devices/platform/soc/sdc1/sunxi_dump_host_register



##### 5.1.2.2 2.emmc 信息

（1）获取路径：cd /sys/block/mmcblk0/device. 这里包含了大部分的 emmc 信息

block ffu_capable preferred_erase_size cid fwrev prv cmdq_en hwrev raw_rpmb_size_mult csd life_time rca date manfid rel_sectors driver mmcblk0rpmb rev dsr name serial enhanced_area_offset ocr subsystem enhanced_area_size oemid type enhanced_rpmb_supported power uevent erase_size pre_eol_info

（2）cat 需要的信息，

例如获取寿命信息

cat life_time

cat pre_eol_info

获取 emmc 名字

cat namd

获取生产日期

cat data

获取唯一识别码

cat cid

获取制造商 id

cat manfid



##### 5.1.2.3 3、性能验证节点

该节点主要是用来记录底层存储的读写性能，该节点记录的数据不参含调度以及文件系统的影响。

为了描述方便，这里设定 base 目录这一概念，其中 X 代表控制器号；

内核 linux4.9 base=/sys/devices/platform/soc/sdcX

内核 linux5.4 base=/sys/devices/platform/soc@2900000/402X000.sdmmc

(1) 开始测量：echo 1 > /$base/sunxi_host_perf

(2) 进行读写操作

(3) 获取测试结果：cat /$base/sunxi_host_perf

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxMMCDevelopmentGuide_006.png)

​																				图 5-1: sunxi_host_perf

(4) 速度计算：1073741824byte/105978400us = 9.66MB/s

(5) 清除测量数据：echo 0 > /$base/sunxi_host_perf

**动态设置**

以下动态设置的节点均于 base 目录下：

sunxi_host_perf 总开关，打开后下面设置才有效

sunxi_host_filter_w_sector：单笔数据传输的数据大于等于这个数据量，

sunxi_host_filter_w_speed 才生效，单位是扇区

sunxi_host_filter_w_speed：速度低于这个值就打印出来，单位是 B/S

参考

echo 20971520 > /$base/sunxi_host_filter_w_speed

echo 8 > /$base/sunxi_host_filter_w_sector

echo 1 > /$base/sunxi_host_perf

效果

20190322_17:24:37.207 [ 64.922940] c=25,a=0x 3fc00,bs= 2560,t= 105463us,sp=12136KB/s

20190322_17:24:37.586 [ 65.301113] c=25,a=0x 43800,bs= 2560,t= 92740us,sp=13802KB/s

20190322_17:24:37.829 [ 65.544155] c=25,a=0x 46000,bs= 2560,t= 94162us,sp=13593KB/s

20190322_17:24:37.967 [ 65.682744] c=25,a=0x 47400,bs= 2560,t= 77371us,sp=16543KB/s

20190322_17:24:38.041 [ 65.755126] c=25,a=0x 47e00,bs= 2560,t= 64860us,sp=19734KB/s

**开机默认启动**

在 dts 或者 sysconfig.fex 里面加入下面配置

per_enable：总开关，打开后下面设置才有效

fiter_sector ：传输的数据大于等于这个数据量，

fiter_speed 才生效，单位是扇区

fiter_speed：速度低于这个值就打印出来，单位是 B/S

参考

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxMMCDevelopmentGuide_005.png)

​																	图 5-2: sysconfig 性能测试配置

效果

 20190322_17:24:37.207 [ 64.922940] c=25,a=0x 3fc00,bs= 2560,t= 105463us,sp=12136KB/s

 20190322_17:24:37.586 [ 65.301113] c=25,a=0x 43800,bs= 2560,t= 92740us,sp=13802KB/s

 20190322_17:24:37.829 [ 65.544155] c=25,a=0x 46000,bs= 2560,t= 94162us,sp=13593KB/s

 20190322_17:24:37.967 [ 65.682744] c=25,a=0x 47400,bs= 2560,t= 77371us,sp=16543KB/s

 20190322_17:24:38.041 [ 65.755126] c=25,a=0x 47e00,bs= 2560,t= 64860us,sp=19734KB/s



### 5.2 常见问题

参考《MMC 量产问题快速排查指南》《eMMC 硬件排查指南》