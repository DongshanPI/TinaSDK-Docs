# 2 配置

## 2.1 TinaTest 配置

在tina/目录下执行”make menuconfig” 进行配置:

![image-20230104120106024](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_MassProductionTest_image-20230104120106024.png)

选择TestTools->tinatest->System Config->global->outlog，这里选择DragonMAT：

![image-20230104120117902](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_MassProductionTest_image-20230104120117902.png)

DragonMAT 有三个子项可供选择：

| 配置项              | 含义                                           |
| ------------------- | ---------------------------------------------- |
| wait_till_connected | 等待dragonMAT 连接上设备，再进行测试。         |
| exit_when_end       | 当测试结束时退出dragonMAT。                    |
| exit_call           | 在DragonMAT 测试通过，结束前调用执行对应的文件 |

这里选中wait_till_connected，exit_when_end，exit_call 根据需要选择。

![image-20230104120131531](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_MassProductionTest_image-20230104120131531.png)

Exit 到TestTools->tinatest 界面，选择base，进行量产测试用例的选择：

![image-20230104120141328](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_MassProductionTest_image-20230104120141328.png)

选择base 下的production，该选项下的所有测试用例都是量产测试用例，可根据测试需求进行选择。其名称格式为：+ “tester”。
例如：cameratester 就是测试camera 的测试用例。

![image-20230104120154071](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_MassProductionTest_image-20230104120154071.png)

打开每一个测试用例，能够对用例进行配置。每一个测试用例的具体配置请参考“2.2 用例配置”。在对tinatest 及其测试用例进行配置后，即可选择Save，点击OK 保存配置，进行固件的编译或者ipk 包的编译。

![image-20230104120204971](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_MassProductionTest_image-20230104120204971.png)

## 2.2 用例配置

一般来说，只要在base->production 下选中测试用例，使用默认配置即可。

但对于某些特殊的测试用例（硬件相关＆特殊需求），请根据实际情况更改配置，以确保测试的准确性。测试用例的正确执行有两个前提：1. 测试用例及其依赖被正确安装；2. 测试用例被正确配置。

在menuconfig 中选中测试用例后，进入该测试用例的配置菜单。例如：pmutester 的配置菜单。

![image-20230104120214224](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_MassProductionTest_image-20230104120214224.png)

测试用例配置项分为普通配置项和高级配置项：
普通配置项用于修改测试用例的测试参数，例如上图第二行的axp_name，则修改pmu 测试用例的芯片名为axp803。

高级配置项用于控制测试用例的测试行为，只有在使能了Advanced 时才会显示。例如
run_times 配置执行次数，command 配置脚本执行命令等。

一般情况下，修改普通配置项即可完成测试，不需要使用高级配置项。高级配置项每一项的含义请参考文档《Tina Linux Tinatest 测试使用指南》。

以下用例配置都是在base->production 下选中了对应测试用例的情况下进行的配置。

### 2.2.1 cameratester

测试camera 模块功能：加水印、连拍、改分辨率。
a. 安装
在命令行中进入内核根目录，执行make menuconfig 进入配置主界面，并按以下配置路径操作
选择编译camera 相关模块：

```
Kernel modules
└─>Video Support
	└─>kmod-sunxi-vfe
```

首先选择Kernel modules 进入下一项配置，如下图所示：

![image-20230104120232198](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_MassProductionTest_image-20230104120232198.png)

一些平台由于框架不同（如V853），选择video 的modules 会有不同，如下：

```
Kernel modules
└─>Video Support
	└─>kmod-vin-v4l2
```

![image-20230104120240043](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_MassProductionTest_image-20230104120240043.png)

**说明**
**要选择当前要测试的板子上对应的camera 模块。**

b. 私有配置
无需额外配置。

### 2.2.2 sdcardtester

测试sd 卡功能。
a. 安装
base->production 中选中sdcardtester 即可。
b. 私有配置
无需额外配置。

### 2.2.3 nandtester

测试nand flash 功能。

a. 安装
base->production 中选中nandtester 即可。
b. 私有配置
无需额外配置。

### 2.2.4 tptester

测试触摸屏功能。
a. 安装
在tina 根目录执行make menuconfig 进入配置主界面，并按以下配置路径操作选择触摸屏模块：

```
Kernel modules
└─>Input modules
	└─>kmod-touchscreen-gt82x
```

![image-20230104120250639](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_MassProductionTest_image-20230104120250639.png)

**说明**
**根据当前所用触摸屏选择对应的模块，例如当前选用的是gt82x 触摸屏。**

b. 私有配置
tp_name: 触摸屏的名称。
touch_times: 触摸次数。

### 2.2.5 pmutester

测试电源管理模块功能。
a. 安装
base->production 中选中pmutester 即可。
b. 私有配置
axp_name: 设备端所使用的电源管理芯片。

板子与axp_name 对应关系如下：

| 板子  | axp_name    |
| ----- | ----------- |
| R16   | axp22_board |
| R40   | axp221s     |
| R18   | axp803      |
| R818  | axp803      |
| MR813 | axp803      |
| v853  | axp2101     |

### 2.2.6 keytester

测试按键功能。
a. 安装
在tina 根目录中执行make kernel_menuconfig 进入配置主界面，并按以下配置路径操作选择编译：

```
Device Drivers
└─>Input device support
	└─>Keyboards
		└─>softwinner KEY BOARD support
```

![image-20230104125135875](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_MassProductionTest_image-20230104125135875.png)

b. 私有配置
number_of_keys: 根据实际板子的按键情况，配置按键数目，一般开发板上adc 按键都是5个，而测试keytester 默认配置是2 个。

### 2.2.7 rtctester

测试rtc 功能。
a. 安装
base->production 中选中rtctester 即可。
b. 私有配置
无需额外配置。

### 2.2.8 wifitester

测试wifi 是否正常启动。

a. 安装

1. base->production->wifi 中选中wifitester。
2. 配置。
   内核配置：
   1) AP6212/AP6212A 等芯片，在Tina 根目录下执行：

```
$ make kernel_menuconfig
```

选择编译Broadcom 无线网卡驱动为模块
wifi:(编译成模块)

```
Device Drivers --->
			Network device support --->
				Wireless LAN --->
						<M> Broadcom FullMAC wireless cards support
						(/lib/firmware/fw_bcmdhd.bin) Firmware path
						(/lib/firmware/nvram.txt) NVRAM path
```

2) RTL8188EU，在Tina 目录下执行：

```
make kernel_menuconfig
```

选择编译RTL8188EU 为模块

```
Device Drivers --->
	Network device support --->
		Wireless LAN --->
			<M> Realtek 8188E USB WIFI
```

3) XR819 在Tina 目录下执行：

```
$ make kernel_menuconfig
```

选择编译XRadio 无线网卡驱动为模块

```
wifi:(编译成模块)
Device Drivers --->
		Network device support --->
			Wireless LAN --->
				<M> XRadio WLAN support --->
```

Tina 配置：
1) AP6212/AP6212A 等芯片，在Tina 目录下执行：

```
$ make menuconfig
```

以AP6216 为例，选中使用AP6212，系统就会将AP6212 的驱动模块拷贝到制定位置，使得系统固件烧写后在Tina 系统中保存，并且在系统启动时能够自动加载。以下配置实现WIFI 驱动拷贝以及开机自动加载：

```
Kernel modules--->
	Wireless Drivers--->
		<*> kmod-net-broadcom
```

以下配置编译拷贝wifi 的firmware：

```
Firmware--->
	<*> ap6212-firmware.
```

2) RTL8188EU，内核选定之后，Tina 进行相关配置，在Tina 目录下执行：

```
$ make menuconfig
```

选中使用RTL8188EU，系统就会将RTL8188EU 的驱动模块拷贝到制定位置，使得系统固件烧写后在Tina 系统中保存，并且在系统启动时能够自动加载。以下配置实现wifi 驱动的拷贝以及开机自动加载：

```
Kernel modules--->
	Wireless Drivers--->
		<*> kmod-net-rtl8188eu
```

以下配置编译拷贝wifi 的firmware

```
Firmware--->
	<*> r8188eu-firmware.
```

3) XR819 在Tina 目录下执行：

```
$ make menuconfig
```

选中使用XR819，系统就会将XR819 的驱动模块拷贝到制定位置，使得系统固件烧写后在Tina系统中保存，并且在系统启动时能够自动加载。以下配置实现WIFI 驱动拷贝以及开机自动加载：

```
Kernel modules--->
	Wireless Drivers--->
		-*- komd-cfg8021
			...
		<*> kmod-xradio-xr819
```

以下配置编译拷贝wifi 的firmware：

```
Firmware--->
	<*> xr819-firmware.
```

b. 私有配置
max_test_times: 最大测试次数。

### 2.2.9 emmctester

测试emmc 功能。
a. 安装
base->production 中选中emmctester 即可。
b. 私有配置
can_format: 是否可以格式化。

### 2.2.10 satatester

测试sata 功能。
a. 安装
base->production 中选中satatester 即可。
b. 私有配置
format: 是否可以格式化。

### 2.2.11 batterytester

测试电池功能。
a. 安装
base->production 中选中batterytester 即可。
b. 私有配置
无需额外配置。

### 2.2.12 ledarraytester

测试mic 板上led 阵列的功能。
a. 安装

在tina/目录下执行make menuconfig 后，选择Utilities->led_test：

![image-20230104125153054](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_MassProductionTest_image-20230104125153054.png)

b. 私有配置
无需额外配置。

### 2.2.13 displaytester

测试display 模块功能。
a. 安装

1. base->production->displaytester 下根据测试需要选择相应测试用例：

   ```
   hdmitester：测试HDMI功能是否正常，HDMI能否正常输出
   brightnesstester：测试LCD的背光亮度调节功能
   smartbacklighttester：测试智能背光功能是否正常
   fbviewertester：测试能否正常显示bmp，jpeg，png图片在屏幕上
   fbshottester：获取framebuffer信息，并保存成bmp格式的图片
   capturetester：测试截屏功能
   fbtester：测试framebuffer是否正常工作
   yuviewtester：测试yuv格式图片是否显示正常
   smartcolortester：测试smartcolor功能是否正常
   ```

   2. tina/目录下执行make menuconfig，选中Kernel modules->Video Support->kmodsunxi-
      disp & kmod-sunxi-hdmi

   ![image-20230104125213512](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_MassProductionTest_image-20230104125213512.png)

b. 私有配置

```
screen_id是屏幕的id，可赋值0或1，默认是0
hdmitester：
		disp_tv_mode是显示的模式，暂时该值不起作用，默认是9
brightnesstester：
		brightness是初始屏幕背光亮度，可赋值0到200，默认是80
smartbacklighttester：
fbviewertester：
fbshottester：
		fb_id是framebuffer的id，根据在小机端生成的设备节点赋值，默认是0
capturetester：
		layer_id是图层id，可以赋值0到11，默认是0，R16上赋值为3
		channel_id是通道id，可以赋值0到4，默认是0，R11与F35赋值为2
		layer_num是图层数，可以是0到11，默认是1，一般此参数不用修改
fbtester：
yuviewtester：
smartcolortester：
		enhance_enable是否启用smartcolor模式，可赋值0或1，
		0表示不启用，1表示启用，默认是1
		enhance_mode是增强模式，默认是8
		bright是亮度，可赋值0到100，默认是50
		contrast是对比度，可赋值0到100，默认是50
		saturation是饱和度，可赋值0到100，默认是50
		hue是色相，可赋值0到100，默认是50
		window_x，window_y，window_width，window_height
		是窗口坐标与宽高，默认(0,0,800,1280)
		其中bright，contrast，saturationhue，window_x，window_y，
		window_width,window_height参数只在R6，R16平台上有效，其他平台直接设置启
		用smartcolor模式即可
```

### 2.2.14 ledstester

测试板载led 功能。
a. 安装
base->production 下选中ledstester 即可。
b. 私有配置
无需额外配置。

### 2.2.15 otgtester

测试usb otg 功能。
a. 安装
base->production 下选中otgtester 即可。
b. 私有配置
usb_count: 插入usb 数量。
usbctler: usb 控制器数量。

### 2.2.16 hosttester

测试usb 功能。
a. 安装

base->production 下选中hosttester 即可。
b. 私有配置
usb_count: 插入usb 数量。

### 2.2.17 udisktester

测试usb 输入设备功能。
a. 安装
base->production 下选中udisktester 即可。
b. 私有配置
usb_count: 插入usb 数量。

### 2.2.18 uarttester

测试uart 收发功能。
a. 安装
base->production 下选中udisktester，dts 使能对应uart 端口，硬件连接tx、rx。
b. 私有配置

```
uart_port：需要测试的uart端口
uart_baud：uart波特率
test_cycles：测试收发次数
test_bytes_per_cycle：单次收发字节数
```

### 2.2.19 ethtester

测试eth 以太网连接功能。

a. 安装

base->production 下选中ethtester，硬件连接以太网口。

在tina 根目录下运行make kernel_menuconfig，选择：

```
Device Drivers >
Network device support >
Ethernet driversupport >
<*> Allwinner GMAC support
[*] Use extern phy
```



b. 私有配置

无需额外配置。

### 2.2.20 regutester

测试regularot 电压设置读取等功能。

a. 安装

base->production 下选中regutester。

b. 私有配置

无需额外配置。

**说明**
目前该测试只用于R818 和MR813。

### 2.2.21 pintester

测试排针引脚gpio 输入输出功能。

a. 安装

base->production 下选中pintester。

b. 私有配置

```
gpio_num：所以测试的pin组数，比如说要测试一组pin，则是有两个gpio的pin引脚。
input_io：测试输入功能的GPIO，每增加一个GPIO，两个GPIO之间只需用括号相隔即可，如：PA1,PB12
output_io：测试输出能的GPIO，其余同上。
```

