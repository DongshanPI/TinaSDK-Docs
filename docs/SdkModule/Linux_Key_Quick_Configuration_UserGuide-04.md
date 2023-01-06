## 4 ADC-Key

ADC-Key 一共有两种adc 检测方式，分别是LRADC 和GPADC。LRADC 分辨率为6 位，GPADC 分辨率为12 位，因此后者精度会更高。

下面将分别讲述两种不同的方式。

### 4.1 4.4 以及4.9 内核

#### 4.1.1 LRADC-Key

LRADC-Key 有如下特性：

1.Support voltage input range between 0 to 2V。
2.Support sample rate up to 250Hz，可以配置为32Hz,62Hz,125Hz,250Hz,R328 sdk中默认配置为250Hz。

3.当前lradc key 最大可以支持到13 个按键(0.15V 为一个档)，通常情况下，建议lradc key最大不要超过8 个(0.2V 为一档)，否则由于采样误差、精度等因素存在，

会很容易出现误判的情况。

如下图所示,lradc-key 检测原理是当有按键被按下或者抬起时，LRADC 控制器(6bit) 检测到电压变化后，经过LRADC 内部的逻辑控制单元进行比较运算后，产生相

应的中断给CPU, 同时电压的变化值会通过LRADC 内部的data register 的值(0~0x3f) 来体现。

![Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228144650067](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228144650067.png)

<center>图4-1: LRADC 按键原理图</center>

下面以R328 为例进行说明，驱动文件：

```
linux-4.9/drivers/input/keyboard/sunxi-keyboard.c
linux-4.9/arch/arm/boot/dts/sun8iw18p1.dtsi
```



1. 如果使用adc-key , 请先确保softwinner KEY BOARD support 有被选择上。

  ```
  Device Drivers
      └─>Input device support
          └─>Keyboards
              └─>softwinnner KEY BOARD support
  ```

  ![Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228144827271](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228144827271.png)

<center>图4-2: linux4.4/4.9 LRADC 按键配置图</center>

2. 在lradc-key 驱动中涉及两个重要的结构体

```
static unsigned char keypad_mapindex[64] = {
    0,0,0,0,0,0,0, 					/* key 1, 0-7 */
    1,1,1,1,1,1, 					/* key 2, 8-14 */
    2,2,2,2,2,2, 					/* key 3, 15-21 */
    3,3,3,3,3, 						/* key 4, 22-27 */
    4,4,4,4,4, 						/* key 5, 28-33 */
    5,5,5,5,5, 						/* key 6, 34-39 */
    6,6,6,6,6,6,6,6,6, 				/* key 7, 40-49 */
    7,7,7,7,7,7,7,7,7,7,7,7,7 		/* key 8, 50-63 */
};
static unsigned int sunxi_scankeycodes[KEY_MAX_CNT] = {
    [0 ] = KEY_VOLUMEUP,
    [1 ] = KEY_VOLUMEDOWN,
    [2 ] = KEY_HOME,
    [3 ] = KEY_ENTER,
    [4 ] = KEY_MENU,
    [5 ] = KEY_RESERVED,
    [6 ] = KEY_RESERVED,
    [7 ] = KEY_RESERVED,
    [8 ] = KEY_RESERVED,
    [9 ] = KEY_RESERVED,
    [10] = KEY_RESERVED,
    [11] = KEY_RESERVED,
    [12] = KEY_RESERVED,
};
```

keypad_mapindex[] 数组的值跟LRADC_DATA0(0x0C) 的值是对应的，表示的具体意义是：key_val(lradc_data0 寄存器的值) 在0~7 范围内时，表示的是操作

key1，依此类推，key_val为8~14 的范围之内时，表示的操作key2。正常来说，按照R16 推荐的硬件设计，该数组内无需任何更改，当个别情况下，如遇到adc-

key input 上报的keycode 有异常时，需要依次按下每个按键同时读取key_val 的值来修正keypad_mapindex[]。

sunxi_scankeycodes[]: 该数组的意义为　sunxi_scankeycodes[*] 标示的是具体的某一个key，用户可以修改其中的keycode。

例如，key_val 等于25, 则根据keypad_mapindex 得到scancode 为4, 再由sunxi_scankeycodes得到键值为KEY_MENU。

keypad_mapindex 这个数组可以通过sys_config 进行配置：

```
[key_para]
key_used = 1
key_cnt = 5
key1_vol = 300
key2_vol = 600
key3_vol = 1000
key4_vol = 1500
key5_vol = 1800
```

这个配置下转换得到的keypad_mapindex 数组是：

```
static unsigned char keypad_mapindex[64] = {
    0,0,0,0,0,0,0,0,0,0 /* key 1, 0-9 */
    1,1,1,1,1,1,1,1,1,1, /* key 2, 10-19 */
    2,2,2,2,2,2,2,2,2,2,2,2,2, /* key 3, 20-32 */
    3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3, /* key 4, 33-48 */
    4,4,4,4,4,4,4,4,4,4, /* key 5, 49-58 */
    5,5,5,5,5, /* key 6, 59-63 */
};
```

3. 设备树文件sun8iw18p1.dtsi

```
keyboard0:keyboard{
    compatible = "allwinner,keyboard_1350mv";
    reg = <0x0 0x05070800 0x0 0x400>;
    interrupts = <GIC_SPI 50 IRQ_TYPE_NONE>;
    status = "okay";
    key_cnt = <5>;
    key0 = <164 115>;
    key1 = <415 114>;
    key2 = <646 139>;
    key3 = <900 28>;
    key4 = <1157 102>;
    wakeup-source;
};
```

• compatible：匹配驱动。

• reg：LRADC 模块的基地址。

• interrupts：LRADC 中断源。

• status：是否加载驱动。

• key_cnt：按键数量。

• key0：第一个按键，前面数字的是电压范围，后者是input 系统的键值。

• wakeup-source：LRADC 作为唤醒源。

驱动初始化时，会读取设备树中相关属性，其中key_cnt 表示配置的按键个数，keyX 配置对应的ADC 值以及键值。根据这些属性，会重新设置keypad_mapindex 

数组，以及sunxi_scankeycodes 数组。

keypad_mapindex[] 数组的值跟ADC 值是对应的, 6bitADC, 范围0~63，表示的具体意义是：key_val 在0~190 范围内时，表示的是操作key0, 以此类推，key_val 

为191~390 范围内时，表示的是操作key1。

sunxi_scankeycodes[]: 该数组的意义为　sunxi_scankeycodes[*] 表示的是具体的某一个key，用户可以修改设备树来改变keycode。

例如，默认配置下，key_val 等于300, 则根据keypad_mapindex 得到scancode 为1, 再由sunxi_scankeycodes 得到键值114。

#### 4.1.2 GPADC-Key

GPADC 有多个通道，例如在R328S3 中有4 个通道。

通道有三种工作模式：

一、是在指定的通道完成一次转换，转换数据更新在对应通道的数据寄存器中；

二、在所有指定的通道连续转换直到软件停止，转换数据更新在对应通道的数据寄存器中；

三、可以在指定的通道进行adc 的采样和转换，并将结果顺序地存入FIFO，FIFO 深度为64 并支持中断控制。

检测原理是当按键被按下或抬起时，指定的通道的控制器检测到电压变化，然后经过逻辑控制单元进行比较运算后，产生相应的中断给CPU。R328S3 的GPADC 

的框图如下图所示。

![Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228145351260](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228145351260.png)

<center>图4-3: GPADC 按键硬件图</center>

以R328S3 为例，驱动文件：

```
linux-4.9/drivers/input/sensor/sunxi_gpadc.c
linux-4.9/drivers/input/sensor/sunxi_gpadc.h
```

1. 如果使用gpadc-key，请先确保SENSORS_GPADC 有被选上。

```
Device Drivers
    └─>Device Drivers
        └─>Input device support
            └─>Sensors
                └─>SUNXI GPADC
```

![Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228145601077](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228145601077.png)

<center>图4-4: linux4.4/4.9 GPADC 按键配置图</center>

2. 在gpadc 中与lradc 一样涉及两个重要的结构体

```
static unsigned char keypad_mapindex[128] = {
    0, 0, 0, 0, 0, 0, 0, 0, 0, /* key 1, 0-8 */
    1, 1, 1, 1, 1, /* key 2, 9-13 */
    2, 2, 2, 2, 2, 2, /* key 3, 14-19 */
    3, 3, 3, 3, 3, 3, /* key 4, 20-25 */
    4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, /* key 5, 26-36 */
    5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, /* key 6, 37-39 */
    6, 6, 6, 6, 6, 6, 6, 6, 6, /* key 7, 40-49 */
    7, 7, 7, 7, 7, 7, 7 /* key 8, 50-63 */
};
u32 scankeycodes[KEY_MAX_CNT];
```

keypad_mapindex[] 数组的定义与LRADC 的相似，但是要注意的是，GPADC 的数组大小为128，而LRADC 的是64，所以说GPADC 的精度更高。在linux4.9 内核

中已经不在数组里配置了，程序中该数组只是初始化的。后面会根据sys_config 或者dts 的配置来生成所要使用的keypad_mapindex[]。

3. 设备树文件这里示例的是R328S3 的方案级设备树文件：

```
lichee/linux-4.9/arch/arm/boot/dts/sun8iw18p1.dtsi
```

详细GPADC 配置如下：

```
gpadc:gpadc{
    compatible = "allwinner,sunxi-gpadc";
    reg = <0x0 0x05070000 0x0 0x400>;
    interrupts = <GIC_SPI 48 IRQ_TYPE_NONE>;
    clocks = <&clk_gpadc>;
    status = "disable";
};
```

• compatible：匹配驱动。
• reg：GPADC 寄存器基地址。
• interrupts：GPADC 中断源。
• clocks：GPADC 依赖的时钟。
• status：是否加载设备。

4.sys_config.fex 文件

```
[gpadc]
gpadc_used = 1
channel_num = 1
channel_select = 0x01
channel_data_select = 0
channel_compare_select = 0x01
channel_cld_select = 0x01
channel_chd_select = 0
channel0_compare_lowdata = 1700000
channel0_compare_higdata = 1200000
key_cnt = 4
key0_vol = 164
key0_val = 115
key1_vol = 415
key1_val = 114
key2_vol = 700
key2_val = 139
key3_vol = 1000
key3_val = 28
```

在sys_config 中，配置含义如下：

• channel_num：表示平台上支持的最大通道数；
• channel_select：表示通道启用设置，通道0:0x01 通道1:0x02 通道2:0x04 通道3:0x08；
• channel_data_select：表示通道数据启用，通道0:0x01 通道1:0x02 通道2:0x04 通道3:0x08；
• channel_compare_select；表示比较功能启用，通道写法跟前面一致；
• channel0_compare_lowdata：表示的低于该电压可以产生中断，这里是1.7V；
• channel0_compare_higdata：则是高于该电压可以产生中断，这里是1.2V；
• key_cnt：表示的是按键个数；
• key*_vol：表示第* 个按键的电压值；
• key*_val：表示的是第* 个按键的键值。

在软件上，当key_vol 在0~164 范围时，表示的是操作key0，以此类推；在硬件上，164 表示的是164mV，通过计算keyn_vol = keyn_vol + ( key(n+1)_vol - 

keyn_vol) / 2来得到范围。比如说第一个范围为key0_vol = 164 + (415 - 164) / 2 = 289.5，即电压在0~289.5 在，adc 检测到为操作的key0，以此类推。然后通过

scankeycodes 得到键值115。

5.board.dts 文件

方案级设备树文件只写这个模块的配置，而详细的按键配置一般需要写在板级的board.dts 中。

```
/*
resistance gpadc configuration
channel_num: Maxinum number of channels supported on the platform.
channel_select: channel enable setection. channel0:0x01 channel1:0x02 channel2:0x04
channel3:0x08
channel_data_select: channel data enable. channel0:0x01 channel1:0x02 channel2:0x04
channel3:0x08.
channel_compare_select: compare function enable channel0:0x01 channel1:0x02 channel2:0
x04 channel3:0x08.
channel_cld_select: compare function low data enable setection: channel0:0x01 channel1:0
x02 channel2:0x04 channel3:0x08.
channel_chd_select: compare function hig data enable setection: channel0:0x01 channel1:0
x02 channel2:0x04 channel3:0x08.
*/
gpadc:gpadc{
    channel_num = <1>;
    channel_select = <0x01>;
    channel_data_select = <0>;
    channel_compare_select = <0x01>;
    channel_cld_select = <0x01>;
    channel_chd_select = <0>;
    channel0_compare_lowdata = <1700000>;
    channel0_compare_higdata = <1200000>;
    channel1_compare_lowdata = <460000>;
    channel1_compare_higdata = <1200000>;
    key_cnt = <5>;
    key0_vol = <210>;
    key0_val = <115>;
    key1_vol = <410>;
    key1_val = <114>;
    key2_vol = <590>;
    key2_val = <139>;
    key3_vol = <750>;
    key3_val = <28>;
    key4_vol = <880>;
    key4_val = <102>;
    status = "okay";
};
```

dts 文件参数的含义可以参考sys_config.fex 的。

### 4.2 5.4 内核

#### 4.2.1 LRADC-Key

5.4 内核的LRADC 驱动功能与4.4/4.9 内核的无明显变化，因此下面只介绍设备树文件与配置教程。

1. 修改设备树文件

这里以D1 为例，设备树文件路径为：

```
lichee/linux-5.4/arch/riscv/boot/dts/sunxi/sun20iw1p1.dtsi
```

dtsi 一般默认已经写好LRADC 的配置：

```
keyboard0: keyboard@2009800 {
    compatible = "allwinner,keyboard_1350mv";
    reg = <0x0 0x02009800 0x0 0x400>;
    interrupts-extended = <&plic0 77 IRQ_TYPE_EDGE_RISING>;
    clocks = <&ccu CLK_BUS_LRADC>;
    resets = <&ccu RST_BUS_LRADC>;
    key_cnt = <5>;
    key0 = <210 115>;
    key1 = <410 114>;
    key2 = <590 139>;
    key3 = <750 28>;
    key4 = <880 172>;
    wakeup-source;
    status = "okay";
};
```

• compatible：匹配驱动。

• reg：LRADC 模块的基地址。

• interrupts：LRADC 中断源。

• status：是否加载驱动。

• key_cnt：按键数量。

• key0：第一个按键，前面数字的是电压范围，后者是input 系统的键值。

• wakeup-source：LRADC 作为唤醒源。



但是D1 方案电路板上实际LRADC 只连接了一个按键，此时可以通过修改board.dts 来进行配置。若实际的电路板上LRADC 按键是符合以上配置的，则不需要修

改。



**说明**
board.dts 的配置最终会覆盖了方案的dtsi 文件配置



board.dts 配置：

```
&keyboard0 {
    key_cnt = <1>;
    key0 = <210 115>;
    status = "okay";
};
```

2. 确认驱动是否被选中在tina 目录下执行make kernel_menuconfig，确认以下配置：

```
Device Drivers
    └─>Input device support
        └─>Keyboards
            └─>softwinnner KEY BOARD support
```

![Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228150327080](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228150327080.png)

<center>图4-5: Linux-5.4 LRADC 按键配置图</center>

#### 4.2.2 GPADC-Key

5.4 内核的GPADC 设备树配置主要是时钟和中断方面有所改动，board.dts 配置可参考4.9 内核。

1. 修改设备树文件

这里以D1 来作为示例，设备树文件为：

```
lichee/linux-5.4/arch/riscv/boot/dts/sunxi/sun20iw1p1.dtsi
```

详细配置为：

```
gpadc: gpadc@2009000 {
    compatible = "allwinner,sunxi-gpadc";
    reg = <0x0 0x02009000 0x0 0x400>;
    interrupts-extended = <&plic0 73 IRQ_TYPE_LEVEL_HIGH>;
    clocks = <&ccu CLK_BUS_GPADC>;
    clock-names = "bus";
    resets = <&ccu RST_BUS_GPADC>;
    status = "okay";
};
```

• compatible：匹配驱动。
• reg：GPADC 寄存器基地址。
• interrupts：GPADC 中断源。
• clocks：GPADC 依赖的时钟。
• status：是否加载设备。

2. 确认驱动是否被选中
   在tina 目录下执行make kernel_menuconfig，确认以下配置：

```
Device Drivers
    └─>Input device support
        └─>Sensors
            └─>sunxi gpadc driver support
```

![Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228150504719](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228150504719.png)

<center>图4-6: Linux-5.4 GPADC 配置图</center>

