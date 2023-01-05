## 3 模块配置

### 3.1 kernel menuconfig 配置

进入 longan 根目录，执行./build.sh menuconfig

进入配置主界面，并按以下步骤操作:

首先，选择 Device Drivers 选项进入下一级配置，如下图所示：

![](http://photos.100ask.net/tina-docs/LinuxGPIODevelopmentGuide_003.png)

​                                                        图 3-1: 内核 menuconfig 根菜单



选择 Pin controllers, 进入下级配置，如下图所示：

 ![](http://photos.100ask.net/tina-docs/LinuxGPIODevelopmentGuide_004.png)

​                                                      图 3-2: 内核 menuconfig device drivers 菜单



选择 Allwinner SoC PINCTRL DRIVER, 进入下级配置，如下图所示：

![](http://photos.100ask.net/tina-docs/LinuxGPIODevelopmentGuide_005.png)

​                                                    图 3-3: 内核 menuconfig pinctrl drivers 菜单



Sunxi pinctrl driver 默认编译进内核，如下图（以 sun50iw9p1 平台为例，其他平台类似）所示：

![](http://photos.100ask.net/tina-docs/LinuxGPIODevelopmentGuide_006.png)

​                                                  图 3-4: 内核 menuconfig allwinner pinctrl drivers 菜单





### 3.2 device tree 源码结构和路径

对于 Linux4.9： 

*•* 设备树文件的配置是该 SoC 所有方案的通用配置，对于 ARM64 CPU 而言，设备树的路径为：kernel/{KERNEL}/arch/arm64/boot/dts/sunxi/sun*-pinctrl.dtsi。 

*•* 设备树文件的配置是该 SoC 所有方案的通用配置，对于 ARM32 CPU 而言，设备树的路径为：kernel/{KERNEL}/arch/arm32/boot/dts/sun*-pinctrl.dtsi。 

*•* 板级设备树 (board.dts) 路径：/device/config/chips/{IC}/configs/{BOARD}/board.dts

device tree 的源码结构关系如下：

```
board.dts
|--------sun*.dtsi
		     |------sun*-pinctrl.dtsi
		     |------sun*-clk.dtsi
```



对于 Linux5.4:

*•* 设备树文件的配置是该 SoC 所有方案的通用配置，对于 ARM64 CPU 而言，5.4 内核中不再维护单独的 pinctrl 的 dtsi，直接将 pin 的信息放在了：kernel/{KERNEL}/arch/arm32/boot/dts/sun*.dtsi

*•* 设备树文件的配置是该 SoC 所有方案的通用配置，对于 ARM32 CPU 而言，5.4 内核中不再维护单独的 pinctrl 的 dtsi，直接将 pin 的信息放在了：kernel/{KERNEL}/arch/arm32/boot/dts/sun*.dtsi

*•* 板级设备树 (board.dts) 路径：/device/config/chips/{IC}/configs/{BOARD}/board.dts

*•* device tree 的源码包含关系如下：

```
board.dts
    |--------sun*.dtsi
```





#### 3.2.1 device tree 对 gpio 控制器的通用配置

在 kernel/{KERNEL}/arch/arm64/boot/dts/sunxi/sun*-pinctrl.dtsi* 文件中 *(Linux5.4* 直接放在 *sun*.dtsi 中)，配置了该 SoC 的 pinctrl 控制器的通用配置信息，一般不建议修改，有 pinctrl 驱动维护者维护。目前，在 sunxi 平台，我们根据电源域，注册两个 pinctrl 设备：r_pio 设 备 (PL0 后的所有 pin) 和 pio 设备 (PL0 前的所有 pin)，两个设备的通用配置信息如下：

```
r_pio: pinctrl@07022000 {
	compatible = "allwinner,sun50iw9p1-r-pinctrl"; //兼容属性，用于驱动和设备绑定
	reg = <0x0 0x07022000 0x0 0x400>; //寄存器基地址0x07022000和范围0x400
    clocks = <&clk_cpurpio>;          //r_pio设置使用的时钟 
    device_type = "r_pio";            //设备类型属性 
    gpio-controller;                  //表示是一个gpio控制器 
    interrupt-controller;             //表示一个中断控制器，不支持中断可以删除 
    #interrupt-cells = <3>;           //pin中断属性需要配置的参数个数，不支持中断可以删除 
    #size-cells = <0>;                //没有使用，配置0
    #gpio-cells = <6>;                //gpio属性配置需要的参数个数,对于linux-5.4为3

    /*
     * 以下配置为模块使用的pin的配置，模块通过引用相应的节点对pin进行操作
     * 由于不同板级的pin经常改变，建议通过板级dts修改（参考下一小节）
     */
    s_rsb0_pins_a: s_rsb0@0 {
        allwinner,pins = "PL0", "PL1";
        allwinner,function = "s_rsb0";
        allwinner,muxsel = <2>;
        allwinner,drive = <2>;
        allwinner,pull = <1>;
    };

    /*
     * 以下配置为linux-5.4模块使用pin的配置，模块通过引用相应的节点对pin进行操作
     * 由于不同板级的pin经常改变，建议将模块pin的引用放到board dts中
     *（类似pinctrl-0 = <&scr1_ph_pins>;),并使用scr1_ph_pins这种更有标识性的名字）。
     */
    scr1_ph_pins: scr1-ph-pins {
        pins = "PH0", "PH1";
        function = "sim1";
        drive-strength = <10>;
        bias-pull-up;
    };
};

pio: pinctrl@0300b000 {
    compatible = "allwinner,sun50iw9p1-pinctrl"; //兼容属性，用于驱动和设备绑定
    reg = <0x0 0x0300b000 0x0 0x400>; //寄存器基地址0x0300b000和范围0x400
    interrupts = <GIC_SPI 51 IRQ_TYPE_LEVEL_HIGH>, /* AW1823_GIC_Spec: GPIOA: 83-32=51 */
            <GIC_SPI 52 IRQ_TYPE_LEVEL_HIGH>,
            <GIC_SPI 53 IRQ_TYPE_LEVEL_HIGH>,
            <GIC_SPI 54 IRQ_TYPE_LEVEL_HIGH>,
            <GIC_SPI 55 IRQ_TYPE_LEVEL_HIGH>,
            <GIC_SPI 56 IRQ_TYPE_LEVEL_HIGH>,
            <GIC_SPI 57 IRQ_TYPE_LEVEL_HIGH>; //该设备每个bank支持的中断配置和gic中断号，每个中断号对应一个支持中断的bank
    device_type = "pio"; //设备类型属性
    clocks = <&clk_pio>, <&clk_losc>, <&clk_hosc>; //该设备使用的时钟
    gpio-controller;          //表示是一个gpio控制器
    interrupt-controller;     //表示是一个中断控制器
    #interrupt-cells = <3>;   //pin中断属性需要配置的参数个数，不支持中断可以删除
    #size-cells = <0>;        //没有使用
    #gpio-cells = <6>;        //gpio属性需要配置的参数个数,对于linux-5.4为3
    /* takes the debounce time in usec as argument */
}
```



#### 3.2.2 board.dts 板级配置

board.dts 用于保存每个板级平台的设备信息 (如 demo 板、demo2.0 板等等)，以 demo 板为例，board.dts 路径如下：

/device/config/chips/{CHIP}/configs/demo/board.dts

在 board.dts 中的配置信息如果在 *.dtsi 中 (如 sun50iw9p1.dtsi 等) 存在，则会存在以下覆盖规则：

*•* 相同属性和结点，board.dts 的配置信息会覆盖 *.dtsi 中的配置信息。

*•* 新增加的属性和结点，会追加到最终生成的 dtb 文件中。

linux-4.9 上面 pinctrl 中一些模块使用 board.dts 的简单配置如下：

```
pio: pinctrl@0300b000 {
    input-debounce = <0 0 0 0 0 0 0>; /*配置中断采样频率，每个对应一个支持中断的bank，单位us*/
    
    spi0_pins_a: spi0@0 {
        allwinner,pins = "PC0", "PC2", "PC4"; 
        allwinner,pname = "spi0_sclk", "spi0_mosi", "spi0_miso"; 
        allwinner,function = "spi0"; 
    };
};
```

对于 linux-5.4，不建议采用上面的覆盖方式，而是修改驱动 pinctrl-0 引用的节点。

linux-5.4 上面 board.dts 的配置如下：

```
&pio{
    input-debounce = <0 0 0 0 1 0 0 0 0>; //配置中断采样频率，每个对应一个支持中断的bank，单位us
    vcc-pe-supply = <&reg_pio1_8>; //配置IO口耐压值，例如这里的含义是将pe口设置成1.8v耐压值 
};
```


