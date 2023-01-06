## 3 GPIO-Key

### 3.1 4.4 以及4.9 内核

4.4 以及4.9 内核的按键相关配置是一样的。涉及到的驱动文件位于如下位置：

其中，64 位平台和32 位平台的dts 文件位置是不一样的。

```
lichee/linux-*/arch/arm/boot/dts/平台代号.dtsi //32位平台的dts文件位置
lichee/linux-*/arch/arm64/boot/dts/sunxi/平台代号.dtsi //64位平台的dts文件位置
```

其中drivers/input/keyboard/目录下的相关文件为驱动文件，而平台名称.dtsi 为设备树文件,例如R328 的dts 文件sun8iw18p1.dtsi，下面以R328 为例进行说明。

支持interrupt-key,poll-key 驱动文件如下:

```
lichee/linux-*/drivers/input/keyboard/gpio-keys-polled.c //gpio poll key
lichee/linux-*/drivers/input/keyboard/gpio-keys.c //interrupt key
```

R328 的dts 文件：

```
lichee/linux-4.9/arch/arm/boot/dts/sun8iw18p1.dtsi
```

配置这种类型的gpio-key 时，请先查询，当前的gpio 是否可以用作中断功能，如果该引脚可以用作中断功能，则采用interrupt 触发方式获取按键信息，若不能用

作中断功能，则只能采用轮询的方式查询按键的状态信息。

GOIO-Key 的硬件配置图参考下图所示：

![Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228142536350](http://photos.100ask.net/tina-docs/Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228142536350.png)

<center>图3-1: GPIO-Key 配置图</center>

#### 3.1.1 普通GPIO 采用poll 方式

1. 修改设备树文件

根据原理图arch/arm/boot/dts/sun8iw18p1.dtsi 文件中添加对应的gpio。例如音量加减键分别用到PH5,PH6 这两个GPIO，则修改方法如下：

```
gpio-keys {
    compatible = "gpio-keys";
    status = "okay";
    vol-down-key {
        gpios = <&pio PH 5 1 2 2 1>;
        linux,code = <114>;
        label = "volume down";
        debounce-interval = <10>;
        wakeup-source = <0x1>;
    };
    vol-up-key {
        gpios = <&pio PH 6 1 2 2 1>;
        linux,code = <115>;
        label = "volume up";
        debounce-interval = <10>;
    };
};
```

• compatible：用于匹配驱动。
• status：是否加载设备。
• vol-down-key：每一个按键都是单独的一份配置，需要分别区分开来。
• gpios：GPIO 口配置。
• linux,code：这个按键对应的input 键值。
• label：单个按键对应的标签。
• debounce-interval：消抖时间，单位为us。
• wakeup-source：是否作为唤醒源，配置了这个项的按键可以作为唤醒源唤醒系统。

2. 确认驱动是否被选中

确认gpio_keys_polled.c 是否编译进系统，在tina 目录下执行make kernel_menuconfig，需要将Polled GPIO buttons 选成“[*]”。

```
Device Drivers
    └─>Input device support
        └─>Keyboards
            └─>Polled GPIO buttons
```

![Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228142913846](http://photos.100ask.net/tina-docs/Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228142913846.png)

<center>图3-2: linux4.4/4.9 轮询按键配置图</center>

#### 3.1.2 普通GPIO 采用中断方式

跟采用poll 方式一样，也需要完成如下步骤：

1. 修改设备树文件
   可见普通gpio 采用poll 方式，不一样的是这里gpio 采用中断的方式。

```
gpio-keys {
    compatible = "gpio-keys";
    status = "okay";
    vol-down-key {
        gpios = <&pio PH 5 6 2 2 1>;
        linux,code = <114>;
        label = "volume down";
        debounce-interval = <10>;
        wakeup-source = <0x1>;
    };
    vol-up-key {
        gpios = <&pio PH 6 6 2 2 1>;
        linux,code = <115>;
        label = "volume up";
        debounce-interval = <10>;
    };
};
```

• compatible：用于匹配驱动。

• status：是否加载设备。

• vol-down-key：每一个按键都是单独的一份配置，需要分别区分开来。

• gpios：GPIO 口配置。

• linux,code：这个按键对应的input 键值。

• label：单个按键对应的标签。

• debounce-interval：消抖时间，单位为us。

• wakeup-source：是否作为唤醒源，配置了这个项的按键可以作为唤醒源唤醒系统。

2. 确认驱动是否被选中
   在tina 目录下执行make kernel_menuconfig，确保GPIO Buttons 被选上。

```
Device Drivers
    └─>Input device support
        └─>Keyboards
            └─>GPIO Buttons
```

![Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228143516377](http://photos.100ask.net/tina-docs/Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228143516377.png)

<center>图3-3: linux4.4/4.9 中断按键配置图</center>

### 3.2 5.4 内核

Linux-5.4 内核相对4.4/4.9 来说，GPIO 子系统有所变化，因此dts 的配置也不大一样。

dts 文件位置：

```
lichee/linux-5.4/arch/arm/boot/dts/平台代号.dtsi //32位平台的dts文件位置
lichee/linux-5.4/arch/riscv/boot/dts/sunxi/平台代号.dtsi //risc-v架构平台的dts文件位置
```

其中drivers/input/keyboard/目录下的相关文件为驱动文件。

支持interrupt-key,poll-key 驱动文件如下:

```
lichee/linux-5.4/drivers/input/keyboard/gpio-keys-polled.c //gpio poll key
lichee/linux-5.4/drivers/input/keyboard/gpio-keys.c //interrupt key
```

#### 3.2.1 普通GPIO 采用poll 方式

1. 修改设备树文件

```
gpio-keys {
    compatible = "gpio-keys";
    status = "okay";
    vol-down-key {
        gpios = <&pio PH 5 GPIO_ACTIVE_LOW>;
        linux,code = <114>;
        label = "volume down";
        debounce-interval = <10>;
        wakeup-source = <0x1>;
    };
    vol-up-key {
        gpios = <&pio PH 6 GPIO_ACTIVE_LOW>;
        linux,code = <115>;
        label = "volume up";
        debounce-interval = <10>;
    };
};
```

• compatible：用于匹配驱动。
• status：是否加载设备。
• vol-down-key：每一个按键都是单独的一份配置，需要分别区分开来。
• gpios：GPIO 口配置。
• linux,code：这个按键对应的input 键值。
• label：单个按键对应的标签。
• debounce-interval：消抖时间，单位为us。
• wakeup-source：是否作为唤醒源，配置了这个项的按键可以作为唤醒源唤醒系统。

与4.4/4.9 相比，主要是gpios 配置方式变化了。

2. 选上驱动
   GPIO-Key 轮询驱动kernel_menuconfig 路径：

```
Device Drivers
    └─>Input device support
        └─>Keyboards
        	└─>Polled GPIO buttons
```

#### 3.2.2 普通GPIO 采用中断方式

1. 修改设备树文件

```
gpio-keys {
    compatible = "gpio-keys";
    status = "okay";
    vol-down-key {
        gpios = <&pio PH 5 GPIO_ACTIVE_LOW>;
        linux,code = <114>;
        label = "volume down";
        debounce-interval = <10>;
        wakeup-source = <0x1>;
    };
    vol-up-key {
        gpios = <&pio PH 6 GPIO_ACTIVE_LOW>;
        linux,code = <115>;
        label = "volume up";
        debounce-interval = <10>;
    };
};
```

• compatible：用于匹配驱动。
• status：是否加载设备。
• vol-down-key：每一个按键都是单独的一份配置，需要分别区分开来。
• gpios：GPIO 口配置。
• linux,code：这个按键对应的input 键值。
• label：单个按键对应的标签。
• debounce-interval：消抖时间，单位为us。
• wakeup-source：是否作为唤醒源，配置了这个项的按键可以作为唤醒源唤醒系统。

2. 选上驱动
   GPIO-Key 中断驱动kernel_menuconfig 路径：

```
Device Drivers
    └─>Input device support
        └─>Keyboards
            └─>GPIO Button
```

3.2.3 矩阵键盘
当需要使用大量按键的时候，如果单独给每一个按键配一个GPIO 的话，那GPIO 是远远不够的。这时，可以使用矩阵键盘的方式，使用N 个GPIO，就可以最大支持N*N 个按键。矩阵按键的硬件原理图如下所示：

![Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228144144489](http://photos.100ask.net/tina-docs/Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228144144489.png)

<center>图3-4: 矩阵按键硬件原理图</center>

**警告**
**矩阵键盘的GPIO 建议使用SOC 自带的IO 口，不使用扩展IO 芯片的IO 口。因为矩阵键盘扫描按键的时间比较短，而扩展IO 芯片的IO 是通过I2C/UART 等等的总线去修改IO 状态的，修改一次状态时间比较长，可能会导致矩阵按键扫描按键检测失败的。**



这里以R528 为示例，dts 为：

```
lichee/linux-5.4/arch/arm/boot/dts/sun8iw20p1.dtsi
```

驱动文件为：

```
lichee/linux-5.4/drivers/input/keyboard/matrix_keypad.c
```

1. 修改设备树文件
   根据R528 原理图来添加对应行和列的gpio，分别写在row-gpios 和col-gpios，详细设备树文件为：

```
matrix_keypad:matrix-keypad {
    compatible = "gpio-matrix-keypad";
    keypad,num-rows = <6>;
    keypad,num-columns = <7>;
    row-gpios = <&pio PG 17 GPIO_ACTIVE_LOW
        &pio PG 18 GPIO_ACTIVE_LOW
        &pio PG 1 GPIO_ACTIVE_LOW
        &pio PG 2 GPIO_ACTIVE_LOW
        &pio PG 3 GPIO_ACTIVE_LOW
        &pio PG 4 GPIO_ACTIVE_LOW>;
    col-gpios = <&pio PG 0 GPIO_ACTIVE_LOW
        &pio PG 5 GPIO_ACTIVE_LOW
        &pio PG 12 GPIO_ACTIVE_LOW
        &pio PG 14 GPIO_ACTIVE_LOW
        &pio PG 6 GPIO_ACTIVE_LOW
        &pio PG 8 GPIO_ACTIVE_LOW
        &pio PG 10 GPIO_ACTIVE_LOW>;
    linux,keymap = <
        0x00000001 //row 0 col0
        0x00010002
        0x00020003
        0x00030004
        0x00040005
        0x00050006
        0x00060007
        0x01000008 //row 1 col0
        0x01010009
        0x0102000a
        0x0103000b
        0x0104000c
        0x0105000d
        0x0106000e
        0x0200000f //row 2 col0
        0x02010010
        0x02020011
        0x02030012
        0x02040013
        0x02050014
        0x02060015
        0x03000016 //row 3 col0
        0x03010017
        0x03020018
        0x03030019
        0x0304001a
        0x0305001b
        0x0306001c
        0x0400001d //row 4 col0
        0x0401001e
        0x0402001f
        0x04030020
        0x04040021
        0x04050022
        0x04060023
        0x05000024 //row 5 col0
        0x05010025
        0x05020026
        0x05030027
        0x05040028
        0x05050029
        0x0506002a
    >;
    gpio-activelow;
    debounce-delay-ms = <20>;
    col-scan-delay-us = <20>;
    linux,no-autorepeat;
    status = "okay";
};
```

设备树文件参数描述如下：
• keypad,num-rows：行数
• keypad,num-columns：列数
• row-gpios：行对应的gpio 口，从第一行开始
• col-gpios：列对应的gpio 口，从第一列开始
• linux,keymap：每一个按键对应的键值
• gpio-activelow：按键按下时，行线是否为低电平。用于触发中断，必须配置。
• debounce-delay-ms：消抖时间。
• col-scan-delay-us：扫描延时，如果IO 状态转换时间过长可能会导致按键扫描错误。
• linux,no-autorepeat：按键按下时是否重复提交按键, 设1 就是不重复, 设0 重复。

2. 确认驱动是否被选中在tina 目录下执行运行make kernel_menuconfig，确认以下配置：

```
Device Drivers
    └─>Input device support
        └─>Keyboards
            └─>GPIO driven matrix keypad support
```

![Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228144512150](http://photos.100ask.net/tina-docs/Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228144512150.png)

<center>图3-5: 矩阵键盘配置图</center>

