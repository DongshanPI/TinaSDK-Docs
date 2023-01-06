## 2 模块介绍

### 2.1 模块功能介绍

SPI 是一种高速、高效率的串行接口技术。通常由一个主模块和一个或多个从模块组成，主模块选择一个从模块进行同步通信，从而完成数据的交换，被广泛应用于 ADC、LCD 等设备与 MCU 之间。全志的 spi 控制器支持以下功能：

*•* 全双工同步串行接口。

*•* 支持 5 种时钟源选择。

*•* 支持 master 和 slave 两种配置。

*•* 四个 cs 片选支持。

*•* 8bit 宽度和 64 字节 fifo 深度。

*•* cs 和 clk 的极性和相位可配置。

*•* 支持使用 DMA。 

*•* 支持四种通信模式。

*•* 批量生产支持最大的 io 速率 100MHz。 

*•* 支持 3 线、4 线 SPI 模式。

*•* 支持可编程串行行数据帧长：0~32bits。 

*•* 支持 Standard SPI/Dual-Output/Dual-input SPI/Dual i/O SPI/ 和 Quad-Output/Quad Input SPI。



### 2.2 相关术语介绍

#### 2.2.1 硬件术语

​																			表 2-1: 硬件术语

| 术语 | 解释说明                                      |
| ---- | --------------------------------------------- |
| SPI  | Serial Peripheral Interface，同步串行外设接口 |



#### 2.2.2 软件术语

​																	 	表 2-2: 软件术语

| 术语       | 解释说明                           |
| ---------- | ---------------------------------- |
| Sunxi      | 指 Allwinner 的一系列 SOC 硬件平台 |
| SPI Master | SPI 主设备                         |
| SPI Device | 指 SPI 外部设备                    |



### 2.3 模块配置介绍

#### 2.3.1 device tree 配置说明

在不同的 Sunxi 硬件平台中，SPI 控制器的数目也不同，但对于每一个 SPI 控制器来说，在设备树中配置参数相似，平台设备树文件的路径为：kernel/内核版本/arch/arm64（32 位平台arm）/boot/dts/sunxi/CHIP.dtsi(CHIP 为研发代号，如 sun50iw10p1 等)，对于配置 SPI1而言，如下：

```
spi1: spi@05011000 {
    #address-cells = <1>;
    #size-cells = <0>;
    compatible = "allwinner,sun50i-spi";          //具体的设备，用于驱动和设备的绑定
    device_type = "spi1";                         //设备节点名称
    reg = <0x0 0x05011000 0x0 0x1000>;            //总线寄存器配置
    interrupts = <GIC_SPI 13 IRQ_TYPE_LEVEL_HIGH>;//总线中断号、中断类型
    clocks = <&clk_pll_periph0>, <&clk_spi1>;     //设备使用的时钟
    clock-frequency = <100000000>;                //控制器的时钟频率
    pinctrl-names = "default", "sleep";           //控制器使用的Pin脚名称
    pinctrl-0 = <&spi1_pins_a &spi1_pins_b>;      //控制器使用的pin脚配置
    pinctrl-1 = <&spi1_pins_c>;                   //控制器使用的pin脚配置
    spi1_cs_number = <1>;                         //控制器cs脚数量
    spi1_cs_bitmap = <1>;                       /* cs0- 0x1; cs1-0x2, cs0&cs1-0x3. */
    status = "disabled";                          //控制器是否使能
};
```

在 Linux-5.4 版本内核中，与 Linux-4.9 内核配置有稍许差异，主要在于 clock 和 dma 的配置上：

```
spi1: spi@4026000 {
    #address-cells = <1>;
    #size-cells = <0>;
    compatible = "allwinner,sun20i-spi";  //具体的设备，用于驱动和设备的绑定
    reg = <0x0 0x04026000 0x0 0x1000>;    //设备节点名称 
    interrupts-extended = <&plic0 32 IRQ_TYPE_LEVEL_HIGH>;//总线中断号、中断类型
    clocks = <&ccu CLK_PLL_PERIPH0>, <&ccu CLK_SPI1>, <&ccu CLK_BUS_SPI1>;//设备使用 的时
    clock-names = "pll", "mod", "bus";  //设备使用的时钟名称
    resets = <&ccu RST_BUS_SPI1>;       //设备的reset时钟
    clock-frequency = <100000000>;      //控制器的时钟频率
    spi1_cs_number = <1>;               //控制器cs脚数量
    spi1_cs_bitmap = <1>;               /* cs0- 0x1; cs1-0x2, cs0&cs1-0x3. */
    dmas = <&dma 23>, <&dma 23>;        //控制器使用的dms通道号
    dma-names = "tx", "rx";             //控制器使用通道号对应的名字
    status = "disabled";                //控制器是否使能
};
```

为了在 SPI 总线驱动代码中区分每一个 SPI 控制器，需要在 Device Tree 中的 aliases 节点中为每一个 SPI 节点指定别名：

```
aliases {
    soc_spi0 = &spi0;
    soc_spi1 = &spi1;
    ...
};
```

别名形式为字符串 “spi” 加连续编号的数字，在 SPI 总线驱动程序中可以通过 of_alias_get_id() 函数获取对应 SPI 控制器的数字编号，从而区别每一个 SPI 控制器。

其中内核版本为 Linux-4.9 的 spi1_pins_a, spi1_pins_b 的配置文件路径为 kernel/linux-4.9/arch/arm64（32 位平台为 arm）/boot/dts/sunxi/xxx-pinctrl.dtsi，具体配置如下所示：

```
spi1_pins_a: spi1@0 {
    allwinner,pins = "PH4", "PH5", "PH6";
    allwinner,pname = "spi1_sclk", "spi1_mosi", 
                      "spi1_miso"; 
    allwinner,function = "spi1";
    allwinner,muxsel = <2>;
    allwinner,drive = <1>;
    allwinner,pull = <0>;
};

spi1_pins_b: spi1@1 {
    allwinner,pins = "PH3";
    allwinner,pname = "spi1_cs0";
    allwinner,function = "spi1";
    allwinner,muxsel = <2>;
    allwinner,drive = <1>;
    allwinner,pull = <1>; // only CS should be pulled up
};

spi1_pins_c: spi1@2 {
    allwinner,pins = "PH3", "PH4", "PH5", "PH6";
    allwinner,function = "io_disabled";
    allwinner,muxsel = <7>;
    allwinner,drive = <1>;
    allwinner,pull = <0>;
};
```

内核版本为 Linux-5.4 的 spi1_pins_a, spi1_pins_b 的具体配置如下所示：

```
spi1_pins_a: spi1@0 {
    pins = "PD11", "PD12", "PD13";
    function = "spi1";
    drive-strength = <10>;
};

spi1_pins_b: spi1@1 {
    pins = "PD10"; 
    function = "spi1";
    drive-strength = <10>;
    bias-pull-up; /* only CS should be pulled up */
};

spi1_pins_c: spi1@2 {
    pins = "PD10", "PD11", "PD12", "PD13";
    function = "gpio_in";
};
```



#### 2.3.2 board.dts 配置说明

board.dts 用于保存每一个板级平台设备差异化的信息的补充（如 demo 板，demo2.0 板，ver1 板等等），里面的配置信息会覆盖上面的 device tree 默认配置信息。

board.dts 的路径为/device/config/chips/{IC}/configs/{BOARD}/board.dts, 其中 SPI1 的具体配置如下：

说明

**在** **Linux-5.4** **内核版本中对** **board.dts** **语法做了修改，不再支持同名节点覆盖，使用** **“&”** **符号引用节点。**

```
&spi1 {
    clock-frequency = <100000000>;
    pinctrl-0 = <&spi1_pins_a &spi1_pins_b>;
    pinctrl-1 = <&spi1_pins_c>;
    pinctrl-names = "default", "sleep";
    spi_slave_mode = <0>;
    status = "disabled";

    spi_board1@0 {
        device_type = "spi_board1";
        compatible = "rohm,dh2228fv";
        spi-max-frequency = <0x5f5e100>;
        reg = <0x0>;
        spi-rx-bus-width = <0x4>;
        spi-tx-bus-width = <0x4>;
        status = "disabled";
    };
};
```

注意，如果要使用 spi slave 模式，请把 spi_slave_mode = <0> 修改为：spi_slave_mode = <1>。

spi_board1 还有一些可配置参数，如：

*•* spi-cpha 和 spi-cpol：配置 spi 的四种传输模式。

*•* spi-cs-high：配置 cs 引脚有效状态时的电平。



spi1_pins_a, spi1_pins_b 、spi1_pins_c 的具体配置如下所示：

```
spi1_pins_a: spi1@0 {
    pins = "PD11", "PD12", "PD13","PD14", "PD15"; /*clk mosi miso hold wp*/
    function = "spi1";
    drive-strength = <10>;
};

spi1_pins_b: spi1@1 {
    pins = "PD10";
    function = "spi1";
    drive-strength = <10>;
    bias-pull-up; // only CS should be pulled up
};

spi1_pins_c: spi1@2 {
    allwinner,pins = "PD10", "PD11", "PD12", "PD13","PD14", "PD15";
    allwinner,function = "gpio_in";
    allwinner,muxsel = <0>;
    drive-strength = <10>;
};
```





#### 2.3.3 menuconfig 配置说明

在命令行中进入内核 linux 目录，执行 make ARCH=arm64 menuconfig(32 位系统为 make ARCH=arm menuconfig) 进入配置主界面 (Linux-5.4 内核版本执行：./build.sh menuconfig)，并按以下步骤操作。

选择 Device Drivers 选项进入下一级配置，如下图所示。

![](http://photos.100ask.net/tina-docs/LinuxSPIDevelopmentGuide_001.png)

​																图 2-1: Device Drivers 配置选项



选择 SPI support 选项，进入下一级配置，如下图所示。

![](http://photos.100ask.net/tina-docs/LinuxSPIDevelopmentGuide_002.png)

​																图 2-2: SPI support 配置选项



选择 SUNXI SPI Controller 选项，可选择直接编译进内核，也可编译成模块。如下图所示。

![](http://photos.100ask.net/tina-docs/LinuxSPIDevelopmentGuide_003.png)

​																图 2-3: SUNXI SPI Controller 配置选项

如果想要放开 spi 的一些调试打印，可以选上 Debug support for SPI drivers。



### 2.4 源码结构介绍

SPI 总线驱动的源代码位于内核在 drivers/spi 目录下：

```
　drivers/spi/
　　├── spi-sunxi.c // Sunxi平台的SPI控制器驱动代码
　　├── spi-sunxi.h // 为Sunxi平台的SPI控制器驱动定义了一些宏、数据结构
```



### 2.5 驱动框架介绍

Linux 中 SPI 体系结构分为三个层次，如下图所示。

![](http://photos.100ask.net/tina-docs/LinuxSPIDevelopmentGuide_004.png)

​																		图 2-4: Linux SPI 体系结构图

#### 2.5.1 用户空间

包括所有使用 SPI 设备的应用程序，在这一层用户可以根据自己的实际需求，将 spi 设备进行一些特殊的处理，此时控制器驱动程序并不清楚和关注设备的具体功能，SPI 设备的具体功能是由用户层程序完成的。例如，和 MTD 层交互以便把 SPI 接口的存储设备实现为某个文件系统，和TTY 子系统交互把 SPI 设备实现为一个 TTY 设备，和网络子系统交互以便把一个 SPI 设备实现为一个网络设备，等等。当然，如果是一个专有的 SPI 设备，我们也可以按设备的协议要求，实现自己的专有协议驱动。同时这部分我们不用关注。



#### 2.5.2 内核空间

内核空间我们同样的会分为一下三部分：



##### 2.5.2.1 SPI 控制器驱动层

考虑到连接在 SPI 控制器上的设备的可变性，在内核没有配备相应的协议驱动程序，对于这种情况，内核为我们准备了通用的 SPI 设备驱动程序，该通用设备驱动程序向用户空间提供了控制 SPI 控制的控制接口，具体的协议控制和数据传输工作交由用户空间根据具体的设备来完成，在这种方式中，只能采用同步的方式和 SPI 设备进行通信，所以通常用于一些数据量较少的简单SPI 设备。

这一层对应于我们内核中的 spidev.c 这个标准的 spi 设备驱动，或者我司的 spi–nand.c，支持 spi 协议的 nand 驱动等。针对特定的 SPI 设备，实现具体的功能，包括 read，write 以及 ioctl 等对用户层操作的接口。SPI 总线驱动主要实现了适用于特定 SPI 控制器的总线读写方法，并注册到 Linux 内核的 SPI 架构，SPI 外设就可以通过 SPI 架构完成设备和总线的适配。但是总线驱动本身并不会进行任何的通讯，它只是提供通讯的实现，等待设备驱动来调用其函数。SPI Core 的管理正好屏蔽了 SPI 总线驱动的差异，使得 SPI 设备驱动可以忽略各种总线控制器的不同，不用考虑其如何与硬件设备通讯的细节。



##### 2.5.2.2 SPI 通用接口封装层

为了简化 SPI 驱动程序的编程工作，同时也为了降低协议驱动程序和控制器驱动程序的耦合程度，内核把控制器驱动和协议驱动的一些通用操作封装成标准的接口，加上一些通用的逻辑处理操作，组成了 SPI 通用接口封装层。这样的好处是，对于控制器驱动程序，只要实现标准的接口回调 API，并把它注册到通用接口层即可，无需直接和协议层驱动程序进行交互。而对于协议层驱动来说，只需通过通用接口层提供的 API 即可完成设备和驱动的注册，并通过通用接口层的API 完成数据的传输，无需关注 SPI 控制器驱动的实现细节。这一层对应于驱动中的 spi.c 文件，是内核原生的文件。



##### 2.5.2.3 SPI 控制器驱动层

为了简化 SPI 驱动程序的编程工作，同时也为了降低协议驱动程序和控制器驱动程序的耦合程度，内核把控制器驱动和协议驱动的一些通用操作封装成标准的接口，加上一些通用的逻辑处理操作，组成了 SPI 通用接口封装层。这样的好处是，对于控制器驱动程序，只要实现标准的接口回调 API，并把它注册到通用接口层即可，无需直接和协议层驱动程序进行交互。而对于协议层驱动来说，只需通过通用接口层提供的 API 即可完成设备和驱动的注册，并通过通用接口层的 API 完成数据的传输，无需关注 SPI 控制器驱动的实现细节。



这一层是我们关注的重点，在后文介绍中会详细的展开进行介绍。



#### 2.5.3 硬件

这一层是实际的物理器件，其中包括我们的 spi 控制器以及与控制器相连的各个 spi 子设备，通过 spi 总线能够与 cpu 进行数据的交互。