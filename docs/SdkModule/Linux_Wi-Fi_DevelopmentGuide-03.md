## 3 Wi-Fi 模组移植

![Tina_Linux_Wi-Fi_Development_Guide-image-20230103102835768](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_Wi-Fi_Development_Guide-image-20230103102835768.png)

<center>图3-1: 主控与Wi-Fi 硬件连接简图</center>

Wi-Fi 模组工作的条件，如上图，需要满足以下几个条件:
• 供电：一般有两路供电，其中VCC-Wi-Fi 为主电源，VCCIO-Wi-Fi 为IO 上拉电源。
• 使能：要能正常工作，需要WL-REG-ON 给高电平。
• SDIO：与SOC 的通信有通过USB，SDIO 等，这里以SDIO 为例，其中SDIO 0~3 为SDIO 的4 条数据线。
• 唤醒主控：当系统休眠时，Wi-Fi 模组可通过WL-WAKE-AP 通过中断的方式唤醒主控，有些模组也通过该引脚来作为主控接收数据的中断。
• 24/26MHz 时钟信号.
• 32.768KHz 信号：根据模组而定，有些模组内部通过（5）中的输入的clk 进行分频得到，有些需要外部单独输入该信号。



对于Wi-Fi 模组移植，重点围绕以上的几个条件进行开展，对于以上几个工作条件allwinner已经提供了对应的driver，根据总线设备驱动模型，只需要根据各个平

台配置device 即可，allwinner device 除了可以可以通过dts 外(linux-3.4 内核无dts)，还可通过sys_config.fex的方式，sys_config.fex 的优先级高于dts，一般情

况下，直接配置sys_config.fex 即可。



说明：

```
- Tina3.5.0及之前sys_config.fex的路径：tina/target/allwinner/xxx(cowbell_perf1)/configs/
- Tina3.5.1及之后sys_config.fex的路径：tina/device/config/chips/xxx(r328)/configs/xxx(perf1)/
- Tina3.5.0及之前dts的路径：tina/lichee/linux-xxx/arch/arm/boot/dts/
- Tina3.5.1及之后dts的路径：tina/device/config/chips/xxx(r328)/configs/xxx(perf1)/board.dts
```



linux 3.4 device（sys_config.fex 配置）

```
[rf_para]
module_num = 10 /*用于区分模组型号：默认8~10*/
module_power1 = "axp22_dldo1" /*power1使用的电源树：dld01*/
module_power1_vol = 3300000 /*power1使用的电源树电压设置为3.3v*/
module_power2 = "axp22_dldo2" /*power2使用的电源树：dld02*/
module_power2_vol = 3300000 /*power2使用的电源树电压设置为3.3v*/
module_power3 = "axp22_aldo3" /*power3使用的电源树：dld03*/
module_power3_vol = 3300000 /*power3使用的电源树电压设置为3.3v*/
/*以上，是使用pmu供电的配置，如果直接供电，可省略*/
power_switch =
chip_en = /*电源使能，备用于使用gpio高低点平打开电源*/
lpo_use_apclk = "losc_out" /*32khz的时钟使能*/
[wifi_para]
wifi_used = 1 /*是否使用Wi-Fi*/
wifi_sdc_id = 1 /*Wi-Fi所使用的sdio 卡号*/
wl_reg_on = port:PL06<1><default><default><0> /*使能脚所使用的gpio*/
wl_host_wake = port:PL07<4><default><default><0> /*唤醒主控所使用的gpio*/
[mmc1_para]
sdc_used = 1
sdc_detmode = 4
sdc_buswidth = 4
sdc_clk = port:PG00<2><1><1><default> /*sdio所使用的gpio号*/
sdc_cmd = port:PG01<2><1><1><default>
sdc_d0 = port:PG02<2><1><1><default>
sdc_d1 = port:PG03<2><1><1><default>
sdc_d2 = port:PG04<2><1><1><default>
sdc_d3 = port:PG05<2><1><1><default>
sdc_det =
sdc_use_wp = 0
sdc_wp =
sdc_isio = 1
sdc_regulator = "none"
```

linux 3.4 driver 路径，详情请参考以下代码路径

```
tina/lichee/linux-3.4/drivers/misc/rf_pm
```

linux3.4 以上，device（sys_config.fex 配置）

```
[sdc1]
sdc1_used = 1
bus-width = 4
sdc1_clk = port:PG00<2><1><3><default>
sdc1_cmd = port:PG01<2><1><3><default>
sdc1_d0 = port:PG02<2><1><3><default>
sdc1_d1 = port:PG03<2><1><3><default>
sdc1_d2 = port:PG04<2><1><3><default>
sdc1_d3 = port:PG05<2><1><3><default>
sd-uhs-sdr50 =
sd-uhs-ddr50 =
sd-uhs-sdr104 =
cap-sdio-irq =
keep-power-in-suspend =
ignore-pm-notify =
max-frequency = 150000000 /*sdio所使用的最大扫卡频率*/
mix-frequency = 150000000
[wlan]
wlan_used = 1
compatible = "allwinner,sunxi-wlan"
clocks = "losc_out"
wlan_power_num = 2 /*使用pmu供电，所使用的电源数量*/
wlan_power1 = "vcc-wifi1" /*使用pmu供电，使用的power1标示*/
wlan_power2 = "vcc-wifi2" /*使用pmu供电，使用的power2标示*/
wlan_io_regulator = "vcc-io-wifi" /*使用pmu供电，使用的io标示*/
/*上面4行表示使用pmu供电的配置项，对应后面的regulator选项。如果不使用pmu供电，可忽略。*/
wlan_busnum = 1
wlan_regon = port:PE06<1><1><1><0>
wlan_hostwake = port:PE05<6><default><default><default>
[regulator0]
compatible = "axp221s-regulator"
regulator_count = 20
......
regulator2 = "axp221s_dcdc2 none vdd-cpua"
regulator3 = "axp221s_dcdc3 none vdd-sys vdd-gpu"
regulator4 = "axp221s_dcdc4 none"
regulator5 = "axp221s_dcdc5 none vcc-dram"
regulator6 = "axp221s_rtc none vcc-rtc"
regulator7 = "axp221s_aldo1 none vcc-25 csi-avdd"
regulator8 = "axp221s_aldo2 none vcc-ephy0"
regulator9 = "axp221s_aldo3 none avcc vcc-pll"
regulator10 = "axp221s_dldo1 none vcc-io-wifi vcc-pg " /*io power 挂在dld01上*/
regulator11 = "axp221s_dldo2 none vcc-wifi1" /*power1 挂在dld02上*/
regulator12 = "axp221s_dldo3 none vcc-wifi2" /*power2 挂在dld03上*/
regulator13 = "axp221s_dldo4 none vdd-sata-25 vcc-pf"
regulator14 = "axp221s_eldo1 none vcc-pe csi-iovcc"
```

linux 3.4 以上driver，详情请参考以下代码路径

```
tina/lichee/linux-XXX/drivers/misc/sunxi-rf
```

### 3.1 模组移植的步骤

下面总结一款新模组移植到Tina 平台的步骤。

#### 3.1.1 修改模组厂提供的Wi-Fi driver

模组厂提供过来的driver，适配到Tina 平台，主要修改的地方是调用Tina 平台提供的有上下电，扫卡函数，修改firmware 的download 路径，配置Kconfig 和

Makefile 等。

下面先说明Tina 平台提供给driver 的函数，其中linux 3.4 跟其他内核稍微有些区别。

linux 3.4

```
#include <mach/sys_config.h>
#include <linux/gpio.h>
/*
*函数功能： sdio扫卡
*参数id： 卡号，（sdio 0 or 1 ...）
*参数insert： 0，卡插入，进行扫卡； 1，卡拔出。
*返回值：无
*/
extern void sunxi_mci_rescan_card(unsigned id, unsigned insert);
/*
*函数功能： Wi-Fi模组上电，使能。
*参数on： 0，上电；1，掉电。
*返回值： 无
*/
extern void wifi_pm_power(int on);
```

linux 3.4 以上

```
#include <linux/mmc/host.h>
#include <linux/sunxi-gpio.h>
#include <linux/power/aw_pm.h>
/*
*函数功能： 获取所使用的sdio卡号，对应sysconfig.fex中的wlan_busnum
*返回值： sdio 卡号
*/
extern int sunxi_wlan_get_bus_index(void);
/*
*函数功能： sdio 扫卡
*参数id： 卡号，（sdio 0 or 1 ...）
*返回值： 无
*/
extern void sunxi_mmc_rescan_card(unsigned ids);
/*
*函数功能： Wi-Fi模组上电，使能。
*参数on： 0，上电；1，掉电。
*返回值： 无
*/
extern void sunxi_wlan_set_power(bool on);
/*
*函数功能：获取gpio wlan hostwake pin的申请的中断号
*参数: void
*返回值：irq number
*说明：　部分模组，主控接收数据通过hostwake pin产生中断来触发，
* 所以需要主控这边提供获取到中断号。
*/
extern int sunxi_wlan_get_oob_irq(void);
/*
*函数功能：获取host wake pin设置中断的标志位
*参数: void
*返回值： irq flag
*/
extern int sunxi_wlan_get_oob_irq_flags(void);
```

首先是将Wi-Fi driver 放到linux-4.9/drivers/net/wireless，填充对应的上电，扫卡等函数。

linux 3.4 的驱动请参考：

esp8089模组：

```
tina/lichee/linux-3.4/drivers/net/wireless/esp8089/sdio_stub.c
```

xr819模组：

```
tina/lichee/linux-3.4/drivers/net/wireless/xradio/wlan/platform.c
```

linux 3.4 以上的驱动请参考：

```
tina/lichee/linux-4.9/drivers/net/wireless/rtl8723ds/platform/platform_ARM_SUNnI_sdio.c
```

其次是增加内核的menuconfig 配置以及编译，只需要修改以下地方即可。

```
tina/lichee/linux-4.9/drivers/net/wireless/Kconfig
example：
+source "drivers/net/wireless/xr829/Kconfig"
```

```
tina/lichee/linux-4.9/drivers/net/wireless/Makefile
example：
+obj-$(CONFIG_XR829_WLAN) += xr829/
```

配置完成后，可执行make kernel_menuconfig 中选上，编译的时候，就会把指定的driver 编译。

```
Device Drivers --->
    [*] Network device support --->
        [*] Network device support --->
            [*] Wireless LAN --->
            	[] xxx模块
```



#### 3.1.2 添加make munconfig 的配置

该步骤主要将kernel 中编译的ko 文件以及firmware 拷贝到跟文件系统中。
首先是配置firmware。firmware 文件一般以模组文件名存放在如下，并需要新增一个mk 文件，使其在make munconfig 中可见。

```
tina/package/firmware/linux-firmware/XXX模组
tina/package/firmware/linux-firmware/XXX模组/XXX.mk
example:
tina/package/firmware/linux-firmware/xr829
make munuconfig
Firmware --->
< > xr829-firmware..................................... Xradio xr829 firmware
```

其次是配置ko。

```
tina/target/allwinner/xxx方案/modules.mk
example:
Tina-3.5.0及以前：
tina/target/allwinner/cowbell-perf1/modules.mk
Tina-3.5.1及以后：
tina/target/allwinner/r328s2-perf1/modules.mk
make munuconfig
    Kernel modules --->
        Wireless Drivers --->
        	XXX 模块
```



#### 3.1.3 配置sys_config.fex

前面已经阐述，见第3 章节开头描述。

#### 3.1.4 验证

按照前面的配置好，make kernel_menuconfig 选上对应模块,make menuconfig 选项对应
firmware 和模块，同时，make munconfig 新增选上如下，即可进行验证了。

```
make menuconfig
    Allwinner --->
        wireless --->
            <*> wifimanager-v2.0................................... Tina wifimanager-v2.0
            <*> wifimanager-v2.0-demo..................... Tina wifimanager-v2.0 app demo
```

验证命令

```
查看模块是否加载：lsmod
模块卸载： rmmod
ps查看wifi_deamon后台进程是否已起来，若没有起来先启动wifi_deamon后台进程
连接路由命令：wifi -c ssid passwd
扫描周围热点：wifi -s
```

3.1.5 模组移植示例

以RTL8723DS 为例：

1. 获取资料
   1.1建议从RTL原厂获取最新版本的完整资料，包括驱动，文档，工具。（也可以从其他内核已适配版本获取驱动）

2. 内核适配

```
2.1将整个驱动SDK拷贝到tina/lichee/linux-xxx/drivers/net/wireless/
2.2驱动重命名为rtl8723ds
2.3在tina/lichee/linux-xxx/drivers/net/wireless/目录修改Kconfig和Makefile
Kconfig:
+source "drivers/net/wireless/rtl8723ds/Kconfig"
Makefile:
+obj-$(CONFIG_RTL8723DS) += rtl8723ds/ (注意：这里命名一定要匹配)
2.4修改驱动原生代码
2.4.1驱动的Makefile(tina/lichee/linux-xxx/drivers/net/wireless/rtl8723ds/Makefile)
+CONFIG_RTW_ANDROID = 0 (# CONFIG_RTW_ANDROID - 0: no Android, 4/5/6/7/8/9/10 : Android
version)
+CONFIG_PLATFORM_I386_PC = n
+CONFIG_PLATFORM_ARM_SUNxI = y
2.4.2替换适配sunxi的平台文件(tina/lichee/linux-xxx/drivers/net/wireless/rtl8723ds/platform)
可以从已经适配过的其他模组获取：platform_ARM_SUNxI_sdio.c
```

3. Tina module 适配

   ```
   3.1.从其他任意已经支持的IC方案中拷贝module的配置
   define KernelPackage/net-rtl8723ds
   SUBMENU:=$(WIRELESS_MENU) //make menuconfig的菜单位置，一般不更改。
   TITLE:=RTL8723DS support (staging) //make menuconfig的提示
   DEPENDS:= +r8723ds-firmware +@IPV6 +@USES_REALTEK +@PACKAGE_realtek-rftest +
   @PACKAGE_rtk_hciattach //添加tina依赖，可以理解为select
   FILES:=$(LINUX_DIR)/drivers/net/wireless/rtl8723ds/8723ds.ko
   KCONFIG:=\ //添加内核依赖可以理解位select
   ...
   AUTOLOAD:=$(call AutoProbe,8723ds)
   endef
   define KernelPackage/net-rtl8723ds/description //make menuconfig的描述
   Kernel modules for RealTek RTL8723DS support
   endef
   $(eval $(call KernelPackage,net-rtl8723ds))
   一个完整的module
   注：建议直接添加在平台的通用配置中：tina/target/allwinner/xxx-common/modules.mk
   3.2.firmware的配置
   /package/firmware/linux-firmware/rtl8723ds/ //更新驱动时更新firmware文件(如果有最新的)
   3.3.sys_config.fex/board.dts的配置
   rfkill: rfkill@0 {
   compatible = "allwinner,sunxi-rfkill";
   chip_en;
   power_en;
   status = "okay";
   wlan: wlan@0 {
   compatible = "allwinner,sunxi-wlan";
   pinctrl-0 = <&wlan_pins_a>;
   pinctrl-names = "default";
   clock-names = "32k-fanout1";
   clocks = <&ccu CLK_FANOUT1_OUT>;
   wlan_busnum = <0x1>;
   wlan_regon = <&pio PE 17 GPIO_ACTIVE_HIGH>;
   wlan_hostwake = <&pio PG 10 GPIO_ACTIVE_HIGH>;
   /*wlan_power = "VCC-3V3";*/
   /*wlan_power_vol = <3300000>;*/
   /*interrupt-parent = <&pio>;
   interrupts = < PG 10 IRQ_TYPE_LEVEL_HIGH>;*/
   wakeup-source;
   };
   ...
   }
   ```

   4. 整体编译烧写
   5. 验证排查

#### 3.1.6 模组移植总结

主要就是以下几点：
• 修改模组厂提供的driver，填充相应的上电，扫卡等函数。
• 增加make kernel_menuconfig 和make menuconfig 选项, 涉及到firmware,makefile,ko。
• 配置sys_config.fex。
• 验证。

目前Tina 平台的linux 内核版本有linux_3.4,linux_3.10,linux_4.4,linux_4.9,linux_5.4，由于历史原因，很有可能内核版本之间的配置有些不一样，主要体现在device：sys_config.fex 以及driver：sunxi-rf。用户在模组移植时，可参考对应各个内核版本进行参考。

Tina-3.5.0 及以前：

| 产品名称 | 内核版本  | sys_config.fex                                           |
| -------- | --------- | -------------------------------------------------------- |
| R16      | Linux-3.4 | tina/target/allwinner/astarparrot/configs/sysconfig.fex  |
| R18      | Linux-4.4 | tina/target/allwinner/tulip_d1/configs/sysconfig.fex     |
| R328     | Linux-4.9 | tina/target/allwinner/cowbell_demo/configs/sysconfig.fex |

Tina-3.5.1 及以后：

| 产品名称 | 内核版本  | 配置文件                                                     |
| -------- | --------- | ------------------------------------------------------------ |
| R16      | Linux-3.4 | tina/device/config/chips/r16/configs/parrot/sys_config.fex   |
| R18      | Linux-4.4 | tina/device/configs/chips/r18/configs/d1/sys_config.fex      |
| R328     | Linux4.9  | tina/device/config/chips/r328s2/configs/perf1/sys_config.fex(board.dts) |
| R528     | Linux-5.4 | tina/device/config/chips/r528/configs/evb1/sys_config.fex(board.dts) |

### 3.2 Tina 平台已经移植的模组

Tina 平台上已经移植过多款Wi-Fi 模组，支持列表如下：

```
BCM ： AP6212,AP6212A,AP6255,AP6256,AP6335等。
Realtek： RTL8723DS（linux 3.4/4.9/5.4),RTL8821cs(linux 4.9/5.4),RTL8822cs(linux 4.9),RTL8189FTV(linux4.9)
Xradio ： XR819(linux 3.4/4.4/4.9),xr829(linux 3.4/4.4/4.9/5.4)
Esp ： esp8089（linux 3.4）
.....
```

对于以上已经移植的模组，用户大多情况只需要在kernel_menuconfig 和menuconfig 选上对应的配置即可。如果按照在对应的menuconfig 中选上，还是不

能工作，就按照3.1 小节的步骤依次排查原因。同时，如果有些方案可能在make menuconfig 中无法显示相应的Kernel modules，这是因为在方案下的

modules.mk 文件中没有添加，可按照3.1.2 小节的方式进行添加。

以下列出各个模组kernel_menuconfig 以及menuconfig 的选项。

#### 3.2.1 BCM 系列的模组

make kernel_menuconfig 配置

```
Device Drivers --->
	Network device support --->
		Wireless LAN --->
			<M> Broadcom FullMAC wireless cards support
                (/lib/firmware/fw_bcmdhd.bin) Firmware path
                (/lib/firmware/nvram.txt) NVRAM path
PS： BCM系列的模组，如AP6212，AP6255，AP6256..都是用的同一份driver
```

make menuconfig 配置

```
Kernel modules--->
	Wireless Drivers--->
	<*> kmod-net-broadcom
Firmware--->
	<*> ap6212-firmware. ---根据模组型号选择
```

#### 3.2.2 XR 系列的模组

（1）XR819
make kernel_menuconfig 配置

```
Device Drivers --->
	Network device support --->
		Wireless LAN --->
			<M> XR819 WLAN support --->
		/*or*/<M> XRadio WLAN support --->
```

make menuconfig 配置

```
Kernel modules--->
	Wireless Drivers--->
		<*> kmod-net-broadcom
Firmware--->
	<*> xr819-firmware..................................... Xradio xr819 firmware
```

（2）XR829
make kernel_menuconfig 配置

```
Device Drivers --->
	Network device support --->
	Wireless LAN --->
	<M> XR829 WLAN support --->
```

make menuconfig 配置

```
Kernel modules--->
Wireless Drivers--->
<*> kmod-net-xr829................................... xr829 support (staging)
Firmware--->
<*> xr829-firmware..................................... Xradio xr829 firmware
```



#### 3.2.3 REALTEK 系列的模组

（1）RTK8723DS
make kernel_menuconfig 配置

```
Device Drivers --->
	Network device support --->
	Wireless LAN --->
	<M> Realtek 8723D SDIO or SPI WiFi
```

make menuconfig 配置

```
Kernel modules--->
    Wireless Drivers--->
    	<*> kmod-net-rtl8723ds........................... RTL8723DS support (staging)
Firmware--->
	<*> r8723ds-firmware.............................. RealTek RTL8723DS firmware
```

（2）RTK8822CS
make kernel_menuconfig 配置

```
Device Drivers --->
    Network device support --->
    	Wireless LAN --->
    		<M> Realtek 8822C SDIO WiFi
```

make menuconfig 配置

```
Kernel modules--->
	Wireless Drivers--->
		<*> kmod-net-rtl8822cs........................... RTL8723CS support (staging)
Firmware--->
		<*> rtl8821cs-firmware............................ RealTek RTL8821CS firmware
```

（3）RTK8189FTV
make kernel_menuconfig 配置

```
Device Drivers --->
	Network device support --->
		Wireless LAN --->
			<M> Realtek 8189F SDIO WiFi
```

make menuconfig 配置

```
Kernel modules--->
	Wireless Drivers--->
		<*> kmod-net-rtl8189fs........................... RTL8189FS support (staging)
注：RTL8189FTV 在tina配置中不需要firmware配置．
```

#### 3.2.4 ESP 系列的模组

（1）ESP8089
make kernel_menuconfig 配置

```
Device Drivers --->
	Network device support --->
		Wireless LAN --->
			<*> Eagle WLAN driver
```

make menuconfig 配置

```
Kernel modules--->
	Wireless Drivers--->
		<*> kmod-esp8089................................... esp8089 support (staging)
Firmware--->
		<*> esp8089-firmware........................................ esp8089 firmware
```

### 3.3 Tina 主要平台模组支持列表

| 硬件平台       | 支持模组                                        | 内核版本  |
| -------------- | ----------------------------------------------- | --------- |
| R6             | XR819                                           | linux3.10 |
| R7/R11         | XR819                                           | linux3.4  |
| R16            | AP6210/AP6212/AP6181/8723BS                     | linux3.4  |
| R18            | AP6212/AP6236/AP6255                            | linux4.4  |
| R30            | AP6181/AP6212/AP6234/AP6330/8723BS              | linux4.4  |
| R40            | AP6212                                          | linux3.10 |
| R58            | AP6212/AP6181/8723BS                            | linux3.4  |
| R311           | AP6212/AP6181/8723BS                            | linux4.9  |
| R328           | XR829/8723DS(单天线)                            | linux4.9  |
| R329           | XR829/8723DS(单双天线)                          | linux4.9  |
| R331/R332/R333 | 8723DS(单天线)                                  | linux4.9  |
| R818           | XR829/8723DS(单双天线)/8189FTV/8821CS(单双天线) | linux4.9  |
| MR133          | AP6212/AP6181/8723BS                            | linux4.9  |
| MR813          | XR829/8723DS(单双天线)/8189FTV                  | linux4.9  |
| T7             | AP6210/AP6212/AP6234                            | linux4.9  |
| R528           | RTL8723ds(单)/RTL8821cs/xr829                   | linux5.4  |
| V853           | XR829/XR819s/RTL8189FS                          | linux4.9  |

