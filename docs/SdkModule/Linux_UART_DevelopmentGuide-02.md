# 模块配置介绍
## 3 模块配置介绍

### 3.1 kernel menuconfig 配置

在 longan 顶层目录，执行./build.sh menuconfig(需要先执行./build.sh config) 进入配置主界面，并按以下步骤操作：首先，选择 Device Drivers 选项进入下一级配置，如下图所示：



​																	图 3-1: 内核 menuconfig 根菜单



选择 Character devices, 进入下级配置，如下图所示：



​																	图 3-2: 内核 menuconfig device drivers 菜单



选择 Serial drivers, 进入下级配置，如下图所示：



​																	

​																   图 3-3: 内核 menuconfig Character drivers 菜单



选择 SUNXI UART Controller 和 Console on SUNXI UART port 选项，如下图：



​																	图 3-4: 内核 menuconfig sunxi uart 配置菜单



如果需要 UART 支持 DMA 传输，则可以打开 SUNXI UART USE DMA 选项。





### 3.2 device tree 源码结构和路径

*•* 设备树文件的配置是该 SoC 所有方案的通用配置，对于 ARM64 CPU 而言，设备树的路径为内核目录下：arch/arm64/boot/dts/sunxi/sun*.dtsi。 

*•* 设备树文件的配置是该 SoC 所有方案的通用配置，对于 ARM32 CPU 而言，设备树的路径为内核目录下：arch/arm/boot/dts/sun*.dtsi。 

*•* 板级设备树 (board.dts) 路径：/device/config/chips/{IC}/configs/{BOARD}/board.dts。device tree 的源码结构关系如下：



device tree 的源码结构关系如下：

```
linux-4.9
board.dts
|--------sun*.dtsi
			|------sun*-pinctrl.dtsi
			|------sun*-clk.dtsi
linux-5.4
board.dts
|-------sun*.dtsi
```





#### 3.2.1 device tree 对 uart 控制器的通用配置

linux-4.9 的通用配置如下: 

```
/ {
    model = "sun*";
    compatible = "arm,sun*";
    interrupt-parent = <&wakeupgen>;
    #address-cells = <2>;
    #size-cells = <2>;
    aliases {
        serial0 = &uart0;
        serial1 = &uart1;
        serial2 = &uart2;
        serial3 = &uart3;
        serial4 = &uart4;
        serial5 = &uart5;
        ...
    };

    uart0: uart@05000000 {
        compatible = "allwinner,sun50i-uart"; /* 用于驱动和设备绑定 */
        device_type = "uart0";                /* 设备类型*/
        reg = <0x0 0x05000000 0x0 0x400>;     /* 设备使用的寄存器基地址以及范围*/
        interrupts = <GIC_SPI 0 IRQ_TYPE_LEVEL_HIGH>; /* 设备使用的硬件中断号*/
        clocks = <&clk_uart0>;                /* 设备使用的时钟*/
        pinctrl-names = "default", "sleep";
        pinctrl-0 = <&uart0_pins_a>;          /* 设备正常状态下使用的pin脚*/
        pinctrl-1 = <&uart0_pins_b>;          /* 设备休眠状态下使用的pin脚*/
        uart0_port = <0>; /* uart控制器对应的ttyS唯一端口号，不能与其他uart控制器重复*/
        uart0_type = <2>; /* uart控制器线数，取值2/4/8*/
        use_dma = <0>;    /* 是否采用DMA 方式传输，0：不启用，1：只启用TX，2：只启用RX，3：启 用TX 与RX*/
        status = "okay"; /* 是否使能该节点*/
};
```



linux-5.4 的通用配置如下: 

```
uart0: uart@5000000 {
    compatible = "allwinner,sun50i-uart";
    device_type = "uart0";
    reg = <0x0 0x05000000 0x0 0x400>;
    interrupts = <GIC_SPI 0 IRQ_TYPE_LEVEL_HIGH>;
    sunxi,uart-fifosize = <64>;
    clocks = <&ccu CLK_BUS_UART0>; /* 设备使用的时钟 */
    clock-names = "uart0";
    resets = <&ccu RST_BUS_UART0>; /* 设备reset时钟 */
    pinctrl-names = "default", "sleep";
    pinctrl-0 = <&uart0_pins_a>;
    pinctrl-1 = <&uart0_pins_b>;
    uart0_port = <0>;
    uart0_type = <2>;
    dmas = <&dma 14>; /* 14表示DRQ */
    dma-names = "tx";
    use_dma = <0>; /* 是否采用DMA 方式传输，0：不启用，1：只启用TX，2：只启用RX，3：启用TX 与RX */
};
```

在 Device Tree 中对每一个 UART 控制器进行配置, 一个 UART 控制器对应一个 UART 节点, 节点属性的含义见注释。为了在 UART 驱动代码中区分每一个 UART 控制器，需要在 Device Tree 中的 aliases 节点中未每一个。UART 节点指定别名，如上 aliases 节点所示。别名形式为字符串 “serial” 加连续编号的数字，在 UART 驱动程序中可以通过 of_alias_get_id() 函数获取对应的 UART 控制器的数字编号，从而区分每一个 UART 控制器。



#### 3.2.2 board.dts 板级配置

board.dts 用于保存每个板级平台的设备信息 (如 demo 板、demo2.0 板等等)，board.dts 路径如下：/device/config/chips/{IC}/configs/{BOARD}/board.dts。 

在 board.dts 中的配置信息如果在 *.dtsi(如 sun50iw9p1.dtsi 等) 存在，则会存在以下覆盖规则：

1. 相同属性和结点，board.dts 的配置信息会覆盖 *.dtsi 中的配置信息。

2. 新增加的属性和结点，会添加到编译生成的 dtb 文件中。

uart 在 board.dts 的简单配置如下：

```
soc@03000000 {
    ...
    &uart0 {
        uart-supply = <reg_dcdc1>; /* IO使用的电 */
        status = "okay";
    };

    &uart1 {
        status = "okay";
    };
    ...
}
```



#### 3.2.3 uart dma 模式配置

1. 在内核配置菜单打开 CONFIG_SERIAL_SUNXI_DMA 配置，如下图所示:



​																图 3-5: 内核 menuconfig sunxi uart 配置菜单

2. 在对应 dts 配置使用 dma，如下所示:

​	Linux-4.9 配置如下:

```
uart1: uart@05000400 {
    ...
    use_dma = <3>; /* 是否采用DMA 方式传输，0：不启用，1：只启用TX，2：只启用RX，3：启用TX 与RX */
    status = "okay";
};
```



​	linux-5.4 配置如下: 

```
uart0: uart@5000000 {
    ...
    dmas = <&dma 14>; /* 14表示DRQ, 查看dma spec */
    dma-names = "tx";
    use_dma = <0>; /* 是否采用DMA 方式传输，0：不启用，1：只启用TX，2：只启用RX，3：启用TX 与RX
    */
};
```



#### 3.2.4 设置其他 uart 为打印 conole

1. 从 dts 确保设置为 console 的 uart 已经使能，如 uart1。

```
uart1: uart@05000400 {
    compatible = "allwinner,sun50i-uart";
    device_type = "uart1";
    reg = <0x0 0x05000400 0x0 0x400>;
    interrupts = <GIC_SPI 1 IRQ_TYPE_LEVEL_HIGH>;
    clocks = <&clk_uart1>;
    pinctrl-names = "default", "sleep";
    pinctrl-0 = <&uart1_pins_a>;
    pinctrl-1 = <&uart1_pins_b>;
    uart1_port = <1>;
    uart1_type = <4>;
    status = "okay"; /* 确保该uart已经使能 */
};
```

2. 修改方案使用的 env*.cfg 文件，如下所示：

```
console=ttyS1,115200
说明:
ttyS0 <===> uart0
ttyS1 <===> uart1
...
```



3.2.5 设置 uart 波特率

在不同的 Sunxi 硬件平台中，UART 控制器的时钟源选择、配置略有不同，总体上的时钟关系如下：



​														            图 3-6: 时钟说明



UART 控制器会对输入的时钟源进行分频，最终输出一个频率满足（或近似）UART 波特率的时钟信号。UART 常用的标准波特率有：

```
#define B4800 0000014
#define B9600 0000015
#define B19200 0000016
#define B38400 0000017
#define B57600 0010001
#define B115200 0010002
#define B230400 0010003
#define B460800 0010004
#define B500000 0010005
#define B576000 0010006
#define B921600 0010007
#define B1000000 0010010
#define B1152000 0010011
#define B1500000 0010012
#define B2000000 0010013
#define B2500000 0010014
#define B3000000 0010015
#define B3500000 0010016
#define B4000000 0010017
```

UART 时钟的分频比是 16 的整数倍，分频难免会有误差，所以输出 UART Device 通信的波特率是否足够精准，很大程度取决于输入的时钟源频率。考虑到功耗，UART 驱动中一般默认使用 24M 时钟源，但是根据应用场景我们有时候需要切换别的时钟源，基于两个原因：

1. 24MHz/16=1.5MHz，这个最大频率满足不了 1.5M 以上的波特率应用；

2. 24M 分频后得到波特率误差可能太大，也满足不了某些 UART 外设的冗余要求（一般要求 2% 或 5% 以内，由外设决定）。UART 时钟源来自 APB2，APB2 的时钟源有两个，分别是 24MHz（HOSC）和 PLL_PERIPH（即驱动中的 PLL_PERIPH_CLK），系统默认配置 APB2 的时钟源是 24M，如果要提高UART 的时钟就要将 APB2 的时钟源设置为 PLL_PERIPH。同时要注意到 APB2 也是 TWI 的时钟源，所以需要兼顾 TWI 时钟。

各个 uart 波特率对应频点关系如下：



​																	图 3-7: 波特率关系



例如需要配置 uart2 的波特率为 460800，在上述关系表中可以看出，对应的时钟为 30M、37.5M、42.857M、46.153M 和 50M 等，所以需要在设备树里修改 uart2 时钟：

linux-4.9 修改波特率如下:

```
    device_type = "uart2";
    reg = <0x0 0x05000800 0x0 0x400>;
    interrupts = <GIC_SPI 2 IRQ_TYPE_LEVEL_HIGH>;
- 	clocks = <&clk_uart2>;
+ 	clocks = <&clk_uart2>, <&clk_apb2>, <&clk_psi>;
+ 	clock-frequency = <50000000>;
	pinctrl-names = "default", "sleep";
```

有的情况下，APB2 不一定能准确分出 30M 或者 37.5M 等时钟，所以这里我们选择配置为 50M 时钟。



如果我们修改了 APB2 时钟，可能会对 uart0 有影响，启动串口会出现乱码，那么我们最好也将 uart0 的时钟配置为 50M 时钟。 

```
	device_type = "uart0";
	reg = <0x0 0x05000000 0x0 0x400>;
	interrupts = <GIC_SPI 0 IRQ_TYPE_LEVEL_HIGH>;
- 	clocks = <&clk_uart0>;
+	clocks = <&clk_uart0>, <&clk_apb2>, <&clk_psi>;
+ 	clock-frequency = <50000000>;
	pinctrl-names = "default", "sleep";
```

linux-5.4 修改波特率如下:

```
	device_type = "uart2";
	reg = <0x0 0x05000800 0x0 0x400>;
	interrupts = <GIC_SPI 2 IRQ_TYPE_LEVEL_HIGH>;
- 		clocks = <&clk_uart2>;
+ 		clocks = <&ccu CLK_BUS_UART0>,
+ 				 <&ccu CLK_APB2>,
+ 				 <&ccu CLK_PSI_AHB1_AHB2>;
+ 		clock-frequency = <50000000>;
	pinctrl-names = "default", "sleep";
```

说明

**修改** **APB2** **总线时钟，会影响到使用到** **APB2** **总线时钟的相应模块。**



#### 3.2.6 支持 cpus 域的 uart

在不同的 Sunxi 硬件平台中，UART 控制器根据电源域划分了 CPUX 域和 CPUS 域，系统默认对 CPUX 域的 uart 控制器都会默认配置上，但对 CPUS 域的 uart 控制器可能没有支持上，当希望通过 CPUX 使用 CPUS 域的 uart 时，请参考以下步骤进行支持:

1. 通过 datasheet 或者 spec，获取 CPUS 域 uart 控制器的 pin 脚、clk、控制器地址等信息。

2. 确保 cpus 运行的系统没有使用到 CPUS 域的 uart 控制器，只有 CPUX 域在使用。

3. 在 r_pio 域配置 pin 的 dtsi 文件 (*.-pinctrl.dtsi)。

```
/ {
	soc@03000000{
		r_pio: pinctrl@07022000 {
			...
			/* 配置CPUS域uart pin信息 */
			s_uart0_pins_a: s_uart0@0 {
                allwinner,pins = "PL2", "PL3";
                allwinner,function = "s_uart0";
                allwinner,muxsel = <2>;
                allwinner,drive = <1>;
                allwinner,pull = <1>;
			};
			...
		};
	...
};	
```

4. 在 clk dtsi 配置时钟信息 (*-clk.dtsi)。

```
clk_suart0: suart0 {
    #clock-cells = <0>;
    compatible = "allwinner,periph-cpus-clock";
    clock-output-names = "suart0";
};
```

5. 在 dtsi 配置 uart 控制器信息 (*.dtsi)。

```
aliases {
    serial0 = &uart0;
    ...
    serial5 = &uart5; //添加uart5的别名，必须要添加
};
uart0:uart@05000000 {
};
....
/* 添加uart控制器信息 ，必须要添加，具体属性含义，见上文 */
uart5: uart@07080000 {
    compatible = "allwinner,sun8i-uart";
    device_type = "uart5";
    reg = <0x0 0x07080000 0x0 0x400>;
    interrupts = <GIC_SPI 111 IRQ_TYPE_LEVEL_HIGH>;
    clocks = <&clk_suart0>;
    pinctrl-names = "default", "sleep";
    pinctrl-1 = <&s_uart0_pins_a>;
    uart5_port = <5>;
    uart5_type = <2>;
    status = "okay";
};
```



6. 检查或添加 pin 描述符。

   有可能 pinctrl 驱动里，没有把 CPUS 域 uart 控制器的 pin 描述出来，因此要的 pinctrl 驱动里检查或添加 pin 的描述信息，查看 pinctrl-*-r.c 文件。

```
static const struct sunxi_desc_pin *_r_pins[] = {
    SUNXI_PIN(SUNXI_PINCTRL_PIN(L, 0),
    ...
    SUNXI_PIN(SUNXI_PINCTRL_PIN(L, 2),
    ...
    SUNXI_FUNCTION(0x2, "s_uart0"), //没有则要添加
    ...
    SUNXI_PIN(SUNXI_PINCTRL_PIN(L, 3),
    ...
    SUNXI_FUNCTION(0x2, "s_uart0"), //没有则要添加
    ...
};
```



7. 检查 clk 的描述信息。

   有可能 clk 驱动根据情况，也没有把 CPUS 域的 uart 控制器的时钟信息描述出来，因此要 clk 驱动里检查或添加 clk 的描述信息，查看 clk-*.c* 和 *clk-*.h 文件。

```
/* clk-*.h */
#define CPUS_UART_GATE 0x018C

/* clk-*.c */
static const char *suart_parents[] = {"cpurapbs2"};

SUNXI_CLK_PERIPH(suart0, 0, 0, 0, 0,
                0, 0, 0, 0, 0, 0,
                CPUS_UART_GATE, CPUS_UART_GATE, 0, 0, 16, 0,
                0, &clk_lock, NULL, 0);
                
struct periph_init_data sunxi_periphs_cpus_init[] = {
    ...
    {"suart0", 0, suart_parents,
    ARRAY_SIZE(suart_parents), &sunxi_clk_periph_suart0 },
};
```



8. 检查 uart 驱动支持的 uart 数量是否足够，查看 sunxi-uart.h 文件。

   通过 SUNXI_UART_NUM 宏确认支持 uart 的数量。

   

9. 修改完毕后，启动小机，查看 ttySn 设备是否生成，通过该设备测试 uart 是否正常使用即可。
