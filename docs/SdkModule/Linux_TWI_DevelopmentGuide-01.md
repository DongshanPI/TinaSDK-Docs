# Linux_TWI_开发指南
## 1 前言

### 1.1 文档简介

介绍 Sunxi 平台上 TWI 驱动接口与调试方法，为 TWI 模块开发提供参考。



### 1.2 目标读者

TWI 模块内核层以及应用层的开发、维护人员。



### 1.3 适用范围

​															*表1-1：适用产品列表*

| 内核版本  | 驱动文件    |
| --------- | ----------- |
| Linux-4.9 | i2c-sunxi.c |
| Linux-5.4 | i2c-sunxi.c |



## 2 模块介绍

### 2.1 模块功能介绍

全志公司的 twi 总线兼容 i2c 总线协议，是一种简单、双向二线制同步串行总线。它只需要两根线即可在连接于总线上的器件之间传送信息。TWI 控制器支持的标准通信速率为 100kbps，最高通信速率可以达到 400kbps。全志的 twi 控制器支持一下功能：



*•* 支持主机模式和从机模式；

*•* 主机模式下支持 dma 传输；

*•* 主机模式下在多个主机的模式下支持总线仲裁；

*•* 主机模式下支持时钟同步，位和字节等待；

*•* 从机模式下支持地址检测中断；

*•* 支持 7bit 从机地址和 10bit 从机地址；

*•* 支持常规的 i2c 协议模式和自定义传输模式；



sunxi 平台支持多路 TWI，包含 TWI 与 S_TWI。



### 2.2 相关术语介绍

#### 2.2.1 硬件术语

​		                                                                 表 2-1: 硬件术语

| 相关术语 | 解释说明                                                  |
| -------- | --------------------------------------------------------- |
| TWI      | Two Wire Interface，全志平台兼容 I2C 标准协议的总线控制器 |



#### 2.2.2 软件术语

​                                                                 表 2-2: 软件术语

| 相关术语      | 相关术语                                                     |
| ------------- | ------------------------------------------------------------ |
| Sunxi         | 全志科技使用的 linux 开发平台                                |
| I2C_dapter    | linux 内核中 I2C 总线适配器的抽象定义.IIC 总线的控制器，在物理上连接若干个 I2C 设备 |
| I2C_algorithm | linux 内核中 I2C 总线通信的抽象定义。描述 I2C 总线适配器与 I2C 设备之间的通信方法 |
| I2C Client    | linux 内核中 I2C 设备的抽象定义                              |
| I2C Driver    | linux 内核中 I2C 设备驱动的抽象定义                          |



### 2.3 模块配置介绍

在不同的 Sunxi 硬件平台中，TWI 控制器的数目不同；但对于同一块板子上的每一个 TWI 控制器来说, 模块配置类似，本小节展示 Sunxi 平台上的 TWI0 控制器配置（其他 TWI 控制器配置类似）。



#### 2.3.1 device tree 默认配置

设备树中存在的是该类芯片所有平台的模块配置，设备树文件的路径为：{linux-ver}/arch/arm64（32 位平台为 arm）/boot/dts/sunxi(32 位系统无这目录)/xxxx.dtsi(CHIP 为研发代号，如sun50iw10p1 等), TWI 总线的设备树配置如下所示：

```
twi0: twi@0x05002000{
	#address-cells = <1>;
	#size-cells = <0>;
	compatible = "allwinner,sun50i-twi"; //具体的设备，用于驱动和设备的绑定 
	device_type = "twi0";                //设备节点名称，用于sys_config.fex匹配 
	reg = <0x0 0x05002000 0x0 0x400>;    //TWI0总线寄存器配置 
	interrupts = <GIC_SPI 6 IRQ_TYPE_LEVEL_HIGH>; //TWI0总线中断号、中断类型 
	clocks = <&clk_twi0>;       //设备使用的时钟 
	clock-frequency = <400000>; //TWI0控制器的时钟频率
	pinctrl-names = "default", "sleep"; //TWI0控制器使用的Pin脚名称，其中default为正常通 信时的引脚配置，sleep为睡眠时的引脚配置
	pinctrl-0 = <&twi0_pins_a>; //TWI0控制器default时使用的pin脚配置
	pinctrl-1 = <&twi0_pins_b>; //TWI0控制器sleep时使用的pin脚配置
	twi_drv_used = <1>;  //使用DMA传输数据
	status = "disabled"; //TWI0控制器是否使能
};
```



在 linux-5.4 中，TWI 的配置与 linux-4.9 内核配置有些不同，区别主要体现在 clock 和 dma 的配置上：

```
twi0: twi@0x05002000{
	#address-cells = <1>;
	#size-cells = <0>;
	compatible = "allwinner,sun20i-twi"; //具体的设备，用于驱动和设备的绑定 
	device_type = "twi0";                //设备节点名称，用于sys_config.fex匹配
	reg = <0x0 0x02502000 0x0 0x400>;    //TWI0总线寄存器配置 
	interrupts-extended= <&plic0 25 IRQ_TYPE_LEVEL_HIGH>; //TWI0总线中断号、中断类
	clocks = <&ccu CLK_BUS_I2C0>;//twi控器使用的时钟
	resets = <&ccu RST_BUS_I2C0>;//twi控器使用的reset时钟
	clock-names = "bus";
	clock-frequency = <400000>; //TWI0控制器的时钟频率
	dmas = <&dma 43>, <&dma 43>;//TWI0控制器的dma通道号
	dma-names = "tx", "rx";
	status = "disabled";//TWI0控制器是否使能
};
```



为了在 TWI 总线驱动代码中区分每一个 TWI 控制器，需要在 Device Tree 中的 aliases 节点中为每一个 TWI 节点指定别名：

```
aliases {
	soc_twi0 = &twi0;
	soc_twi1 = &twi1;
	soc_twi2 = &twi2;
	soc_twi3 = &twi3;
	...
};
```

 别名形式为字符串 “twi” 加连续编号的数字，在 TWI 总线驱动程序中可以通过 of_alias_get_id()函数获取对应 TWI 控制器的数字编号，从而区别每一个 TWI 控制器。

其中 twi0_pins_a, twi0_pins_b 为 TWI 的引脚配置的配置节点。linux4.9 中 该 配 置 的 路 径 为 arch/arm64（32 位 平 台 为 arm）/boot/dts/sunxi/xxxxpinctrl.dtsi(CHIP 为研发代号，如 sun50iw10p1 等)，具体配置如下所示： 

```
twi0_pins_a: twi0@0 {
	allwinner,pins = "PD14", "PD15";          //TWI控制器使用的引脚 
	allwinner,pname = "twi0_scl", "twi0_sda"; //TWI控制器的引脚功能说明
	allwinner,function = "twi0"; //引脚功能描述
	allwinner,muxsel = <4>;      //引脚复用功能配置
	allwinner,drive = <0>;       //io驱动能力
	allwinner,pull = <0>;        //内部电阻状态 
};

twi0_pins_b: twi0@1 {
	allwinner,pins = "PD14", "PD15";
	allwinner,function = "io_disabled";
	allwinner,muxsel = <7>;
	allwinner,drive = <1>;
	allwinner,pull = <0>;
};
```

linux-5.4 中该配置的路径为 arch/arm64（32 位平台为 arm）/boot/dts/sunxi/xxxx.dtsi(CHIP为研发代号，如 sun50iw10p1 等)，具体如下所示：

```
twi0_pins_a: twi0@0 {
	pins = "PH0", "PH1";
	function = "twi0";
	drive-strength = <10>;
};
twi0_pins_b: twi0@1 {
	pins = "PH0", "PH1";
	function = "gpio_in";
}
```

另外 clk_twi0 为时钟的配置。

在 linux-4.9 中， 路 径 为 arch/arm64（32 位 平 台 为 arm）/boot/dts/sunxi/XXXXclk.dtsi(CHIP 为研发代号，如 sun50iw10p1 等)，具体配置如下所示：

```
clk_twi0: twi0 {
	#clock-cells = <0>;
	compatible = "allwinner,periph-clock";
	clock-output-names = "twi0"; //指定clock名称，用于匹配clock配置 
};
```

在 linux-5.4 中，无需配置。



#### 2.3.2 board.dts 板级配置

board.dts 用于保存每一个板级平台的设备信息（如 demo 板，perf1 板，ver1 板等等），里面的配置信息会覆盖上面的 device tree 默认配置信息。board.dts 的路径为 longan/device/config/chips/IC/configs/BOARD/board.dts,

在 linux-4.9 中，对应 board.dts 里面 TWI0 的具体配置如下：

```
twi0_pins_a: twi0@0 {
	allwinner,pins = "PA0", "PA1";
	allwinner,pname = "twi0_scl", "twi0_sda";
	allwinner,function = "twi0";
	allwinner,muxsel = <4>;
	allwinner,drive = <1>;
	allwinner,pull = <0>;
};

twi0_pins_b: twi0@1 {
	allwinner,pins = "PA0", "PA1";
	allwinner,function = "io_disabled";
	allwinner,muxsel = <7>;
	allwinner,drive = <1>;
	allwinner,pull = <0>;
};

twi0: twi@0x05002000{
	clock-frequency = <400000>; //i2c时钟频率为400K
	pinctrl-0 = <&twi0_pins_a>;
	pinctrl-1 = <&twi0_pins_b>;
	status = "okay"; //使能TWI0
};
```



在 linux-5.4 中，对应 board.dts 里面 TWI0 的具体配置如下：

```
&twi0 {
	clock-frequency = <400000>;
	pinctrl-0 = <&twi0_pins_a>;
	pinctrl-1 = <&twi0_pins_b>;
	pinctrl-names = "default", "sleep";
	status = "disabled";
	eeprom@50 {
		compatible = "atmel,24c16";
		reg = <0x50>;
		status = "disabled";
	};
};
```

其中，TWI 速率由 “clock-frequency” 属性配置，最大支持 400K。

对于 TWI 设备，可以把设备节点填充作为 Device Tree 中相应 TWI 控制器的子节点。TWI 控制器驱动的 probe 函数透过 of_i2c_register_devices() ，自动展开作为其子节点的 TWI 设备。

对于 twi0 中引用的 pin 口，具体的配置如下：

```
twi0_pins_a: twi0@0 {
	pins = "PB10", "PB11"; /*sck sda*/
	function = "twi0";
	drive-strength = <10>;
};

twi0_pins_b: twi0@1 {
	pins = "PB10", "PB11";
	function = "gpio_in";
};
```





#### 2.3.3 kernel menuconfig 配置

在 longan 中，linux-4.9 在 命 令 行 进 入 内 核 根 目 录 (/kernel/linux-4.9)， 执 行 make ARCH=arm64 menuconfig (32 位平台执行：make ARCH=arm menuconfig) 进入配置主界面，并按以下步骤操作 (linux-5.4 在根目录中执行./build.sh menuconfig)在 tina 中，可以直接在根目录里面执行 make kernel_menuconfig 进入 menuconfig 配置界面。

*•* 1. 选择 Device Drivers 选项进入下一级配置，如下图所示。





​                                                                图 2-1: Device Driver



*•* 2. 选择 I2C support 选项，进入下一级配置，如下图所示。





​                                                               图 2-2: I2C support



*•* 3. 配置用户 I2C 接口，选择 I2C device interface，如下图所示。



​                                                              图 2-3: I2C device interface



*•* 4. 选择 I2C HardWare Bus support 选项，进入下一级配置，如下图所示。





​                                                              图 2-4: I2C HardWare Bus support



*•* 5. 选择 SUNXI I2C controller 选项，可选择直接编译进内核，也可编译成模块。如下图所示。





​                                                              图 2-5: SUNXI I2C controller



### 2.4 源码模块结构

I2C 总线驱动的源代码位于内核在 drivers/i2c/busses 目录下：

```
kernel/linux-4.9/drivers/i2c/
├── busses
│   ├── i2c-sunxi.c // Sunxi平台的I2C控制器驱动代码
│   ├── i2c-sunxi.h // 为Sunxi平台的I2C控制器驱动定义了一些宏、数据结构
│   ├── i2c-sunxi-test.c // Sunxi平台的i2c设备测试代码,5.4下暂未适配
├── i2c-core.c // I2C子系统核心文件,提供相关的接口函数
├── i2c-dev.c  // I2C子系统的设备相关文件，用以注册相关的设备文件，方便调试
```



### 2.5 驱动框架介绍





​                                                             图 2-6: TWI 模块结构框图

Linux 中 I2C 体系结构上图所示，图中用分割线分成了三个层次：1. 用户空间，包括所有使用I2C 设备的应用程序；2. 内核，也就是驱动部分；3. 硬件，指实际物理设备，包括了 I2C 控制器和 I2C 外设。

其中，Linux 内核中的 I2C 驱动程序从逻辑上又可以分为 6 个部分：

1. I2C framework 提供一种 “访问 I2C slave devices” 的方法。由于这些 slave devices 由I2C controller 控制，因而主要由 I2C controller 驱动实现这一目标。

2. 经过 I2C framework 的抽象，用户可以不用关心 I2C 总线的技术细节，只需要调用系统的接口，就可以与外部设备进行通信。正常情况下，外部设备是位于内核态的其它 driver（如触摸屏，摄像头等等）。I2C framework 也通过字符设备向用户空间提供类似的接口，用户空间程序可以通过该接口访问从设备信息。

3. 在 I2C framework 内部，有 I2C core、I2C busses、I2C algos 和 I2C muxes 四个模块。

4. I2C core 使用 I2C adapter 和 I2C algorithm 两个子模块抽象 I2C controller 的功能，使用 I2C client 和 I2C driver 抽象 I2C slave device 的功能（对应设备模型中的 device 和 device driver）。另外，基于 I2C 协议，通过 smbus 模块实现 SMBus（System Management Bus，系统管理总线）的功能。

5. I2C busses 是各个 I2C controller drivers 的集合，位于 drivers/i2c/busses/目录下，i2c-sunxi-test.c、i2c-sunxi.c、i2c-sunxi.h。

6. I2C algos 包含了一些通用的 I2C algorithm，所谓的 algorithm，是指 I2C 协议的通信方法，用于实现 I2C 的 read/write 指令，一般情况下，都是由硬件实现，不需要特别关注该目录。

