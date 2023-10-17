## 3 模块配置介绍

### 3.1 kernel menuconfig 配置

#### 3.1.1 linux-4.9 版本下

在命令行中进入内核根目录(kernel/linux-4.9)，执行make ARCH=arm64(arm) menuconfig(32 位系统为make ARCH=arm menuconfig) 进入配置主界面(linux-5.4 内核版本在longan 目录下执行:./build.sh menuconfig 进入配置主界面)，并按以下步骤操作：
首先，选择Device Drivers 选项进入下一级配置，如下图所示：

![image-20221216114515324](https://photos.100ask.net/Tina-Sdk/Linux_RTC_DevGuide_image-20221216114515324.png)

选择Real Time Clock，进入下级配置，如下图所示：

![image-20221216114527372](https://photos.100ask.net/Tina-Sdk/Linux_RTC_DevGuide_image-20221216114527372.png)

选择Allwinner sunxi RTC，如下图所示：

![image-20221216114542981](https://photos.100ask.net/Tina-Sdk/Linux_RTC_DevGuide_image-20221216114542981.png)

由于在关机过程中，RTC 一般都是独立供电的，因此在RTC 电源域中的寄存器不会掉电且RTC寄存器的值也不会恢复为默认值。利用此特性，Sunxi 平台支持reboot 命令的一些扩展功能和
假关机功能，但需要打开support ir fake poweroff 和Sunxi rtc reboot Feature 选项，RTC驱动才能支持这些扩展功能。

#### 3.1.2 linux-5.4 版本下

在命令行中进入longan 顶层目录，执行./build.sh config，按照提示配置平台、板型等信息（如果之前已经配置过，可跳过此步骤）。
然后执行./build.sh menuconfig，进入内核图形化配置界面，并按以下步骤操作：
选择Device Driver选项进入下一级配置，如下图所示：

![image-20221216114823615](https://photos.100ask.net/Tina-Sdk/Linux_RTC_DevGuide_image-20221216114823615.png)

选择Real Time Clock进入下一级配置，如下图所示：

![image-20221216114840997](https://photos.100ask.net/Tina-Sdk/Linux_RTC_DevGuide_image-20221216114840997.png)

选择Allwinner sunxi RTC配置，如下图所示。

![image-20221216114857679](https://photos.100ask.net/Tina-Sdk/Linux_RTC_DevGuide_image-20221216114857679.png)

由于在关机过程中，RTC 一般都是独立供电的，因此在RTC 电源域中的寄存器不会掉电且RTC寄存器的值也不会恢复为默认值。利用此特性，Sunxi 平台支持reboot 命令的一些扩展功能，但需要打开Sunxi rtc reboot flag和Sunxi rtc general register save bootcount选项，RTC 驱动才能支持这些扩展功能。

### 3.2 device tree 源码结构和路径

SoC 级设备树文件（sun*.dtsi）是针对该SoC 所有方案的通用配置：

```
• 对于ARM64 CPU 而言，SoC 级设备树的路径为：arch/arm64/boot/dts/sunxi/sun*.dtsi
• 对于ARM32 CPU 而言，SoC 级设备树的路径为：arch/arm/boot/dts/sun*.dtsi
```

板级设备树文件（board.dts）是针对该板型的专用配置：

```
• 板级设备树路径：device/config/chips/{IC}/configs/{BOARD}/board.dts
```

板级设备树文件（board.dts）是针对该板型的专用配置：
• 板级设备树路径：device/config/chips/{IC}/configs/{BOARD}/board.dts

#### 3.2.1 linux-4.9 版本下

device tree 的源码结构关系如下：

```
board.dts
 └--------sun*.dtsi
 |------sun*-pinctrl.dtsi
 └------sun*-clk.dtsi
```

#### 3.2.2 linux-5.4 版本下

device tree 的源码结构关系如下：

```
board.dts
 └--------sun*.dtsi
```

### 3.3 device tree 对RTC 控制器的通用配置

#### 3.3.1 linux-4.9 版本下

```
1 / {
2 rtc: rtc@07000000 {
3 compatible = "allwinner,sunxi-rtc"; //用于probe驱动
4 device_type = "rtc";
5 auto_switch; //支持RTC使用的32k时钟源硬件自动切换
6 wakeup-source; //表示RTC是具备休眠唤醒能力的中断唤醒源
7 reg = <0x0 0x07000000 0x0 0x200>; //RTC寄存器基地址和映射范围
8 interrupts = <GIC_SPI 104 IRQ_TYPE_LEVEL_HIGH>; //RTC硬件中断号
9 gpr_offset = <0x100>; //RTC通用寄存器的偏移
10 gpr_len = <8>; //RTC通用寄存器的个数
11 gpr_cur_pos = <6>;
12 };
13 }
```

说明
对于linux-4.9 内核，当RTC 结点下配置auto_switch 属性时，RTC 硬件会自动扫描检查外部32k 晶体振荡器的起振情
况。当外部晶体振荡器工作异常时，RTC 硬件会自动切换到内部RC16M 时钟分频出来的32k 时钟，从而保证RTC 工作正
常。当没有配置该属性时，驱动代码中直接把RTC 时钟源设置为外部32k 晶体的，当外部32K 晶体工作异常时，RTC 会工
作异常。因此建议配置上该属性。

#### 3.3.2 linux-5.4 版本下

```
1 / {
2 rtc: rtc@7000000 {
3 compatible = "allwinner,sun50iw10p1-rtc"; //用于probe驱动
4 device_type = "rtc";
5 wakeup-source; //表示RTC是具备休眠唤醒能力的中断唤醒源
6 reg = <0x0 0x07000000 0x0 0x200>; //RTC寄存器基地址和映射范围
7 interrupts = <GIC_SPI 108 IRQ_TYPE_LEVEL_HIGH>; //RTC硬件中断号
8 clocks = <&r_ccu CLK_R_AHB_BUS_RTC>, <&rtc_ccu CLK_RTC_1K>; //RTC所用到的时钟
9 clock-names = "r-ahb-rtc", "rtc-1k"; //上述时钟的名字
10 resets = <&r_ccu RST_R_AHB_BUS_RTC>;
11 gpr_cur_pos = <6>; //当前被用作reboot-flag的通用寄存器的序号
12 };
13 }
```

在Device Tree 中对每一个RTC 控制器进行配置, 一个RTC 控制器对应一个RTC 节点, 节点属性的含义见注释。

### 3.4 board.dts 板级配置

board.dts用于保存每个板级平台的设备信息(如demo 板、demo2.0 板等等)。board.dts路径如下：

```
device/config/chips/{IC}/configs/{BOARD}/boar d.dts
```

在board.dts中的配置信息如果在*.dtsi(如sun50iw9p1.dtsi等) 中存在，则会存在以下覆盖规则：

在board.dts中的配置信息如果在*.dtsi(如sun50iw9p1.dtsi等) 中存在，则会存在以下覆盖规则：

1. 相同属性和结点，board.dts的配置信息会覆盖*.dtsi中的配置信息
2. 新增加的属性和结点，会添加到编译生成的dtb 文件中