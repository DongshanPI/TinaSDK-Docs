## 2 模块介绍

### 2.1 模块功能介绍

USB 有主机功能和从设备功能。做主机时，能连接 U 盘、USB 鼠标等 USB 设备；做从设备时，具有 ADB 调试等从设备功能。

### 2.2 相关术语介绍

​																			表 2-1: 术语介绍

| 术语   | 说明                                                     |
| ------ | -------------------------------------------------------- |
| USB    | Universal Serial Bus, 通用串行总线                       |
| OTG    | On-The-Go                                                |
| ADB    | Android Debug Bridge，Android 调试桥                     |
| Gadget | 小配件                                                   |
| HCD    | Host Controller Driver，主机控制器驱动                   |
| UDC    | USB Device Controller, USB 设备控制器                    |
| HCI    | Host Controller Interface，主机控制器接口                |
| EHCI   | Enhanced Host Controller Interface，增强型主机控制器接口 |
| OHCI   | Open Host Controller Interface，开放式主机控制器接口     |



### 2.3 模块配置介绍

#### 2.3.1 Device Tree 配置说明

设备树中存在的是该类芯片所有平台的模块配置，设备树文件的路径为：kernel/linux-4.9/arch/arm64（32 位平台为 arm）/boot/dts/sunxi/xxx.dtsi（xxx 为具体芯片型号，如 sun50iw10p1 等）, 设备树配置如下所示：

*•* USB0 配置

```
usbc0:usbc0@0 {
    device_type = "usbc0";
    compatible = "allwinner,sunxi-otg-manager";
    usb_port_type = <2>;
    usb_detect_type = <1>;
    usb_id_gpio;
    usb_det_vbus_gpio;
    usb_regulator_io = "nocare";
    usb_wakeup_suspend = <0>;
    usb_luns = <3>;
    usb_serial_unique = <0>;
    usb_serial_number = "20080411";
    rndis_wceis = <1>;
    status = "okay";
};

udc:udc-controller@0x05100000 {
    compatible = "allwinner,sunxi-udc";
    reg = <0x0 0x05100000 0x0 0x1000>,      /*udc base*/
    	  <0x0 0x00000000 0x0 0x100>,       /*sram base*/
    	  <0x0 0x05200000 0x0 0x1000>;      /*usb1 base, for common circuit*/
    interrupts = <GIC_SPI 32 IRQ_TYPE_LEVEL_HIGH>; /*设备使用的中断*/
    clocks = <&clk_usbphy0>, <&clk_usbotg>, <&clk_usbehci1>, <&clk_usbphy1>; /*设备使用的时钟*/
    status = "okay"; /*是否使能该设备*/
};

ehci0:ehci0-controller@0x05101000 {
    compatible = "allwinner,sunxi-ehci0";
    reg = <0x0 0x05101000 0x0 0xFFF>, /*hci0 base*/
    	  <0x0 0x00000000 0x0 0x100>,
    	  <0x0 0x05100000 0x0 0x1000>,
    	  <0x0 0x07010250 0x0 0x10>,    /*prcm base, for usb standby*/
    	  <0x0 0x05200000 0x0 0x1000>;  /*usb1 base, for common circuit*/
    interrupts = <GIC_SPI 30 IRQ_TYPE_LEVEL_HIGH>;
    clocks = <&clk_usbphy0>, <&clk_usbehci0>, <&clk_usbehci1>, <&clk_usbphy1>;
    hci_ctrl_no = <0>; /*主机控制器的序列*/
    status = "okay";
};

ohci0:ohci0-controller@0x05101400 {
    compatible = "allwinner,sunxi-ohci0";
    reg = <0x0 0x05101000 0x0 0xFFF>, /*hci0 base*/
	      <0x0 0x00000000 0x0 0x100>,
          <0x0 0x05100000 0x0 0x1000>,
    	  <0x0 0x07010250 0x0 0x10>, /*prcm base, for usb standby*/
    	  <0x0 0x05200000 0x0 0x1000>;/*usb1 base, for common circuit*/
    interrupts = <GIC_SPI 31 IRQ_TYPE_LEVEL_HIGH>;
    clocks = <&clk_usbphy0>, <&clk_usbohci0>, <&clk_usbohci1>, <&clk_usbphy1>;
    hci_ctrl_no = <0>;
    status = "okay";
};
```

*•* USB1 配置

```
usbc1:usbc1@0 {
    device_type = "usbc1";
    usb_regulator_io = "nocare";
    usb_wakeup_suspend = <0>;
    status = "okay";
};

ehci1:ehci1-controller@0x05200000 {
    compatible = "allwinner,sunxi-ehci1";
    reg = <0x0 0x05200000 0x0 0xFFF>,
          <0x0 0x00000000 0x0 0x100>,
          <0x0 0x05100000 0x0 0x1000>,
          <0x0 0x07010250 0x0 0x10>;
    interrupts = <GIC_SPI 33 IRQ_TYPE_LEVEL_HIGH>;
    clocks = <&clk_usbphy1>, <&clk_usbehci1>;
    hci_ctrl_no = <1>;
    status = "okay";
};

ohci1:ohci1-controller@0x05200400 {
    compatible = "allwinner,sunxi-ohci1";
    reg = <0x0 0x05200000 0x0 0xFFF>,
          <0x0 0x00000000 0x0 0x100>,
          <0x0 0x05100000 0x0 0x1000>,
          <0x0 0x07010250 0x0 0x10>;
    interrupts = <GIC_SPI 34 IRQ_TYPE_LEVEL_HIGH>;
    clocks = <&clk_usbphy1>, <&clk_usbohci1>, <&clk_usbohci1_12m>, <&clk_osc48md4>, 			 <&clk_hosc>, <&clk_losc>;
    hci_ctrl_no = <1>;
    status = "okay";
};
```



#### 2.3.2 board.dts 配置说明

board.dts 用于保存每一个板级平台的设备信息（如 demo 板，perf1 板等），里面的配置信息会覆盖上面的 Device Tree 默认配置信息。不同 soc、版型及内核版本对应的 board.dts 具体路径如下：device/config/chips/*soc*/*conf igs*/{board}/${内核版本}/board.dts。 

*•* USB0 配置

```
usbc0:usbc0@0 {
    device_type = "usbc0";
    usb_port_type = <0x2>;
    usb_detect_type = <0x1>;
    usb_id_gpio = <&pio PH 8 0 0 0xffffffff 0xffffffff>;
    usb_det_vbus_gpio = "axp_ctrl"; 
    usb_regulator_io = "nocare";
    det_vbus_supply = <&usb_power_supply>;
    usb_wakeup_suspend = <0>;
    usb_luns = <3>;
    usb_serial_unique = <0>;
    usb_serial_number = "20080411";
    rndis_wceis = <1>;
    status = "okay";
};

注：（1）usb_port_type：usb0口默认的模式。
置0：devcie模式；
置1：host模式；
置2：otg模式。
（2）usb_detect_type：usb0口otg检测模式。
置0：不做检测；
置1：vbus/id检测；
置2：id/dpdm检测。
（3）usb_wakeup_suspend：standby模式。
置0：super standby模式；
置1：usb standby模式，支持远程唤醒。

udc:udc-controller@0x51000000 {
	det_vbus_supply = <&usb_power_supply>
}

ehci0:ehci0-controller@0x05101000 {
	drvvbus-supply = <&reg_drivevbus>;
};

ohci0:ohci0-controller@0x05101400 {
	drvvbus-supply = <&reg_drivevbus>;
};
```

说明

**若使用** **usb standby** **模式，需注意如下：**

**1、IC 支持远程唤醒；**

**2、若条件 1 满足，相关硬件部分需严格按照《硬件设计文档》设计；**

**3、若条件 1、2 满足，额外添加属性 “wakeup-source;”, 启用 usb standby 功能。**



*•* USB1 配置

```
usbc1:usbc1@0 {
    device_type = "usbc1";
    usb_regulator_io = "nocare";
    usb_wakeup_suspend = <0>;
    status = "okay";
};

ehci1:ehci1-controller@0x05200000 {
	drvvbus-supply = <&reg_usb1_vbus>;
};

ohci1:ohci1-controller@0x05200400 {
	drvvbus-supply = <&reg_usb1_vbus>;
};
```

*•* Vbus 配置

```
reg_usb1_vbus: usb1-vbus {
    compatible = "regulator-fixed";
    gpio = <&pio PH 10 1 2 0 1>;
    regulator-name = "usb1-vbus";
    regulator-min-microvolt = <5000000>;
    regulator-max-microvolt = <5000000>;
    regulator-enable-ramp-delay = <1000>;
    enable-active-high;
};
```



#### 2.3.3 kernel menuconfig 配置说明

进入内核根目录，执行 make ARCH=arm menuconfig（64 位平台为 make ARCH=arm64 menuconfig）进入配置主界面，并按以下步骤操作：

选择 Device Drivers 选项进入下一级配置，如下图所示。

![](https://photos.100ask.net/Tina-Sdk/LinuxUSBDevelopmentGuide_001.png)

​																图 2-1: Device Drivers 选项配置

选择 USB support 选项，进入下一级配置，如下图所示。

![](https://photos.100ask.net/Tina-Sdk/LinuxUSBDevelopmentGuide_002.png)

​																图 2-2: USB Support 选项配置

打开如下两图的选项，如下图所示。

![](https://photos.100ask.net/Tina-Sdk/LinuxUSBDevelopmentGuide_003.png)

​																图 2-3: USB Support 详细配置 1 

![](https://photos.100ask.net/Tina-Sdk/LinuxUSBDevelopmentGuide_004.png)

​																图 2-4: USB Support 详细配置 2



选择 USB Gadget Support，进入下一级配置，如下图所示。

![](https://photos.100ask.net/Tina-Sdk/LinuxUSBDevelopmentGuide_005.png)

​																图 2-5: USB Gadget Support 选项配置	



打开下图的选项，并在对应配置中打开所需的功能性配置, 如: 需要存储功能时, 需打开下图中的 “mass storage” 配置, 如下图所示。

![](https://photos.100ask.net/Tina-Sdk/LinuxUSBDevelopmentGuide_006.png)

​																图 2-6: USB Gadget Support 详细配置



进入 USB Peripheral Controller，并打开下图选项：

![](https://photos.100ask.net/Tina-Sdk/LinuxUSBDevelopmentGuide_007.png)

​																图 2-7: USB Peripheral Controller 详细配置

返回上一级，即 USB support，进入 SUNXI USB2.0 Dual Role controller support，并打开下图选项，如下图所示。

![](https://photos.100ask.net/Tina-Sdk/LinuxUSBDevelopmentGuide_008.png)

​																图 2-8: SUNXI USB2.0 Dual Role Controller Support 详细配置



若需支持 MTP PTP 等功能需开启 TYPEC 配置返回上一级，即 USB support，进入 USB Type-C Support，并打开下图选项，如下图所示：

![](https://photos.100ask.net/Tina-Sdk/LinuxUSBDevelopmentGuide_009.png)

​																图 2-9: USB Type-C Support 详细配置

### 2.4 源码结构介绍

USB 驱动的源代码位于内核 drivers/usb 目录下，如下是 sunxi 平台相关源码：

*•* Host

```
drivers/usb/host/
├── ehci_sunxi.c
├── ohci_sunxi.c
├── sunxi_hci.c
├── sunxi_hci.h
```

*•* UDC 和 Manager

```
drivers/usb/sunxi_usb/
├── include
│   ├── sunxi_hcd.h
│   ├── sunxi_sys_reg.h
│   ├── sunxi_udc.h
│   ├── sunxi_usb_board.h
│   ├── sunxi_usb_bsp.h
│   ├── sunxi_usb_config.h
│   ├── sunxi_usb_debug.h
│   └── sunxi_usb_typedef.h
├── Kconfig
├── Makefile
├── manager
│   ├── usbc0_platform.c
│   ├── usbc_platform.h
│   ├── usb_hcd_servers.c
│   ├── usb_hcd_servers.h
│   ├── usb_hw_scan.c
│   ├── usb_hw_scan.h
│   ├── usb_manager.c
│   ├── usb_manager.h
│   ├── usb_msg_center.c
│   └── usb_msg_center.h
├── misc
│   └── sunxi_usb_debug.c
├── udc
│   ├── sunxi_udc_board.c
│   ├── sunxi_udc_board.h
│   ├── sunxi_udc.c
│   ├── sunxi_udc_config.h
│   ├── sunxi_udc_debug.c
│   ├── sunxi_udc_debug.h
│   ├── sunxi_udc_dma.c
│   └── sunxi_udc_dma.h
└── usbc
    ├── usbc.c
    ├── usbc_dev.c
    ├── usbc_i.h
    └── usbc_phy.c
```



### 2.5 驱动框架介绍

Linux 内核提供了完整的 USB 驱动程序框架。USB 总线采用树形结构，在一条总线上只能有唯一的主机设备。Linux 内核从主机和设备两个角度观察 USB 总线结构。下图是 Linux 内核从主机和设备两个角度观察 USB 总线结构的示意图。

![](https://photos.100ask.net/Tina-Sdk/LinuxUSBDevelopmentGuide_0010.png)

​																		图 2-10: USB 驱动总体结构

USB 子系统主要任务包括：

a. 注册和管理设备驱动；

b. USB 设备寻找驱动，并初始化和配置设备；

c. 内核中表现设备的树形结构；

d. 与设备交互。



### 2.6 Gadget 配置

Gadget 是指具有 USB 设备控制器的 USB 设备，根据具体的功能配置，连接到 PC 后可以作为 mass storage、uac 等设备。Linux 有原生 gadget 框架，通用的配置流程可参考下文。



#### 2.6.1 打开内核配置

需在 “USB functions configurable through configfs” 下选择需要的功能。

![](https://photos.100ask.net/Tina-Sdk/LinuxUSBDevelopmentGuide_0011.png)

​													   图 2-11: linux-4.x usb gadget 配置选择



#### 2.6.2 linux-4.x/linux-5.4 USB Gadget 配置流程

Linux-4.x/Linux-5.4 使用 configfs 框架实现 composite gadget 功能。具体流程如下：

*•* 挂载 configs：

```
mount -t configfs none /sys/kernel/config
```

挂载完成之后在/sys/kernel/config 目录下就会生成 usb_gadget/目录。

*•* 建立 gadgets：

```
mkdir /sys/kernel/config/usb_gadget/g1
```

创建g1/目录之后，该目录下会生成很多配置目录，这里的g1表示 gadget 1，一个 UDC 对应一个 gadget，如果你的 SOC 上有多个 gadget，可以创建多个gx目录。



*•* 写入 gadget 的 PID、VID、序列号等信息：

```
echo "VID" > /sys/kernel/config/usb_gadget/g1/idVendor
echo "PID" > /sys/kernel/config/usb_gadget/g1/idProduct
mkdir /sys/kernel/config/usb_gadget/g1/strings/0x409
echo "manufacturer" > /sys/kernel/config/usb_gadget/g1/strings/0x409/manufacturer
echo "product" > /sys/kernel/config/usb_gadget/g1/strings/0x409/product
```



*•* 建立 gadget 相关配置 configurations

```
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1
echo 0xc0 > /sys/kernel/config/usb_gadget/g1/configs/c.1/bmAttributes
echo 500 > /sys/kernel/config/usb_gadget/g1/configs/c.1/MaxPower
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1/strings/0x409
```



*•* 建立功能 functions

```
mkdir /sys/kernel/config/usb_gadget/g1/functions/<name>.<instance name>
```

：function name ：任意字符串



*•* 建立功能和配置的链接

```
ln -s /sys/kernel/config/usb_gadget/g1/functions/<name>.<instance name> /sys/kernel/config/
usb_gadget/g1/configs/c.1
```



*•* 使能 gadget

```
echo <udc name> > UDC
```

常见 Gadget 功能的配置方式见附录。



### 2.7 端点配置

在 Gadget 配置使用过程中，可能出现端点的默认配置方式无法满足需求的情况，故需对端点进行修改满足需求。可参考现有的端点进行修改。譬如将批量端点改成中断端点，参考现有的中断端点进行修改即可。改动内容包括端点 fifo 大小，端点属性，端点方向。



#### 2.7.1 端点 fifo 大小

```
以4k平台为例：
static const struct sw_udc_fifo ep_fifo[] = {
    {ep0name, 0, 512, 0},/*name， fifo_addr， fifo_size, double_fifo*/
    {ep1in_bulk_name, 512, 512, 0},
    {ep1out_bulk_name, 1024, 512, 0},
    {ep2in_bulk_name, 1536, 512, 0},
    {ep2out_bulk_name, 2048, 512, 0},
    {ep3_iso_name, 2560, 1024, 0},
    {ep4_int_name, 3584, 512, 0},
};
```



#### 2.7.2 端点的属性

```
.ep[2] = {
    .num = 1,
    .ep = {
        .name      = ep1out_bulk_name,
        .ops       = &sunxi_udc_ep_ops,
        .maxpacket = SW_UDC_EP_FIFO_SIZE,
        .maxpacket_limit = SW_UDC_EP_FIFO_SIZE,
        .caps = USB_EP_CAPS(USB_EP_CAPS_TYPE_BULK,
            USB_EP_CAPS_DIR_OUT),
    },
    .dev              = &sunxi_udc,
    .bEndpointAddress = (USB_DIR_OUT | 1),
    .bmAttributes     = USB_ENDPOINT_XFER_BULK,
},
```



#### 2.7.3 定义端点的方向

```
/**
 * ep_fifo_in[i] = {n} i: the physic ep index, n: ep_fifo's index for the ep
 *
 * eg: ep_fifo_in[2] = {3} ===> ep2_in is in ep_fifo[3]
 * 
 * ep3_iso_name and ep4_int_name cannot be tx or rx simultaneously. 
 * 
 */
static const int ep_fifo_in[] = {0, 1, 3, 5, 6, 7};
static const int ep_fifo_out[] = {0, 2, 4, 5, 6, 8};
```



### 2.8 调试方法

#### 2.8.1 调试节点

##### 2.8.1.1 USB0 调试节点

查看 USB0 当前 Role 

```
cat /sys/devices/platform/soc/usbc0/otg_role
```



手动切换到 Host 模式

```
cat /sys/devices/platform/soc/usbc0/usb_host
```



手动切换到 Device 模式

```
cat /sys/devices/platform/soc/usbc0/usb_device
```



##### 2.8.1.2 USB1 调试节点

卸载主机驱动

通过下述命令找到主机驱动节点及对应路径

```
find -name ehci_enable
find -name ohci_enable
```

然后根据上述结果，按如下命令卸载主机驱动 (以 t5 平台为例)

```
echo 0 > sys/devices/platform/soc/5200000.ehci1-controller/ehci_enable
echo 0 > sys/devices/platform/soc/5200000.ohci1-controller/ohci_enable
```

加载主机驱动

通过下述命令找到主机驱动节点及对应路径

```
find -name ehci_enable
find -name ohci_enable
```

然后根据上述结果，按如下命令加载主机驱动 (以 t5 平台为例)

```
echo 1 > sys/devices/platform/soc/5200000.ehci1-controller/ehci_enable
echo 1 > sys/devices/platform/soc/5200000.ohci1-controller/ohci_enable
```



#### 2.8.2 眼图测试

##### 2.8.2.1 USB Device 眼图测试

```
获取otg_ed_test的路径path
find /sys/ -name otg_ed_test
测试眼图命令
echo test_pack > path/otg_ed_test
```



##### 2.8.2.2 USB Host 眼图测试

```
获取ed_test的路径path
find /sys/ -name ed_test
测试眼图命令
echo test_pack > path/ed_test
```

