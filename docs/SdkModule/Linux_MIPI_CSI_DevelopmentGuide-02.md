## 2 模块介绍

### 2.1 模块功能介绍

1. Video input 主要由接口部分（CSI/MIPI）和图像处理单元（ISP/VIPP）组成;

2. CSI/MIPI 部分主要实现视频数据的捕捉;

3. ISP 实现 sensor raw data 数据的处理，包括 lens 补偿、去坏点、gain、gamma、de-mosaic、de-noise、color matrix 等以及一些 3A 的统计;

4. VIPP 能对将图进行缩小、和打水印处理。VIPP 支持 bayer raw data 经过 ISP 处理后再缩小，也支持对一般的 YUV 格式的 sensor 图像直接缩小。



### 2.2 相关术语介绍

<center>表 2-1: 软件术语</center>

| 相关术语 | 解释说明                                                     |
| -------- | ------------------------------------------------------------ |
| ISP      | Image Signal Processor 图像信号处理                          |
| VIPP     | Video Input Post Processor 图像输入后处理                    |
| MIPI     | Mobile Industry Processor Interface 移动工业处理接口         |
| CCI      | Camera Control Interface 摄像头控制接口                      |
| TDM      | Time division multiplexing ISP 时分复用                      |
| MCLK     | Master clock（From AP to camera）摄像头主时钟                |
| PCLK     | Pixel clock（From camera to AP，Sampling clock for data-bus）像素时钟 |
| YUV      | Color Presentation（Y for luminance，U&V for Chrominance）图像数据格式 |



### 2.3 驱动框架介绍

![](https://photos.100ask.net/Tina-Sdk/LinuxMIPICSIDevelopmentGuide_001.png)

​																		图 2-1: 驱动框图

VIN 驱动可以分为 Kernel 层、Video Input Framework、Device Driver 层。



#### 2.3.1 Kernel 层

1. V4l2 Framework；

2. Linux 内核视频驱动第二版（Video for Linux Two ）；

3) 适用于收音机、视频编解码、视频捕获以及视频输出设备驱动；

4) 提供/dev/videoX 节点，应用通过该节点进行相应视频流和控制操作；

5. Media Device Framework；

6. Linux 多媒体设备框架；

7) 适用于管理设备拓扑结构；

8) 提供/dev/mediaX 节点，通过该节点应用可以获取媒体设备拓扑结构，并能够通过 API 控制子设备间数据流向。



#### 2.3.2 Video Input Framework 层

1. Video Control : 视频命令处理（分辨率协商，数据格式处理，Buffer 管理等）；

2. Runtime Handle : 运行时管理（Pipeline 管理，系统资源管理，中断调度等）；

3. Event Process : 事件管理（如上层调用，中断等事件的接收与分发）；

4. Config Handle : 配置管理（如硬件拓扑结构，模组自适应列表等）。



#### 2.3.3 Device Driver 层

1. Camera Modules : 模组驱动（图像传感器，对焦电机，闪光灯等驱动）；

2. Camera Interfac : 接口驱动（MIPI、Sub-Lvds 、HiSpi、Bt656、Bt601、Bt1120、DC等）；

3. Image Signal Processor : 图像处理器驱动（基本处理模块驱动，3A 统计驱动）；

4. Video Input Post Processor : 视频输入后处理（Scaler，OSD 等）。



### 2.4 模块配置介绍

#### 2.4.1 kernel menuconfig 配置

1. 首先，进入 Device Drivers，选择 Multimedia support ，然后依次打开 Cameras/video grabbers support 、Media Controller support 和 SUNXI platform devices, 如下图所示。

   ​									![](https://photos.100ask.net/Tina-Sdk/LinuxMIPICSIDevelopmentGuide_002.png)

   ​															图 2-2: Device Drivers 选项配置

2. 其次，进入 SUNXI platform devices，选择 sunxi video input (camera csi/mipi isp vipp)driver 和 v4l2 new driver for SUNXI，如下图所示。

   ![](https://photos.100ask.net/Tina-Sdk/LinuxMIPICSIDevelopmentGuide_003.png)

   ​                                                            图 2-3: Device Drivers 选项配置

3. 最后，sunxi video input (camera csi/mipi isp vipp)driver 目录下的其他选项需要根据实际产品需求进行开关，如：使用闪光灯、对焦马达、打开 vin log、使用 IOMMU 如下图所示。

   ![](https://photos.100ask.net/Tina-Sdk/LinuxMIPICSIDevelopmentGuide_004.png)

   ​															图 2-4: Device Drivers 选项配置



#### 2.4.2 Device Tree 配置说明

*•* 设备树文件的配置是该 SoC 所有方案的通用配置，对于 ARM64 CPU 而言，设备树的路径为：kernel/{KERNEL_VERSION}/arch/arm64/boot/dts/sunxi/sun*.dtsi。 

*•* 设备树文件的配置是该 SoC 所有方案的通用配置，对于 ARM32 CPU 而言，设备树的路径为：kernel/{KERNEL_VERSION}/arch/arm/boot/dts/sun*.dtsi。 

*•* 板级设备树 (board.dts) 路径：

/device/config/chips/{IC}/configs/{BOARD}/KERNEL_VERSION/board.dts。 

在 sun*.dtsi* 文件中，配置了该 *SoC* 的 *CSI* 控制器的通用配置信息，一般不建议修改，由 *CSI* 驱动维护者维护，如果需要修改配置请修改板级设备树 *board.dts*，板级设备树里面的内容会覆盖*sun*.dtsi 对应的信息。



*•* vind 配置

```
&vind0 {
    vind0_clk = <336000000>;
    vind0_isp = <300000000>;
    status = "okay";
    
    tdm0:tdm@0 {
    	work_mode = <0>;
    };
    
    isp00:isp@0 {
    	work_mode = <0>;
    };
    
    scaler00:scaler@0 {
    	work_mode = <0>;
    };
    
    scaler10:scaler@4 {
    	work_mode = <0>;
    };
    
    scaler20:scaler@8 {
    	work_mode = <0>;
    };
    
    scaler30:scaler@12 {
    	work_mode = <0>;
    };
    
    actuator0:actuator@0 {
        device_type = "actuator0";
        actuator0_name = "ad5820_act";
        actuator0_slave = <0x18>;
        actuator0_af_pwdn = <>;
        actuator0_afvdd = "afvcc-csi";
        actuator0_afvdd_vol = <2800000>;
        status = "disabled";
    };
    flash0:flash@0 {
        device_type = "flash0";
        flash0_type = <2>;
        flash0_en = <>;
        flash0_mode = <>;
        flash0_flvdd = "";
        flash0_flvdd_vol = <>;
        status = "disabled";
    };
    sensor0:sensor@0 {
        device_type = "sensor0";
        sensor0_mname = "gc2053_mipi";
        sensor0_twi_cci_id = <1>;
        sensor0_twi_addr = <0x6e>;
        sensor0_mclk_id = <0>;
        sensor0_pos = "rear";
        sensor0_isp_used = <1>;
        sensor0_fmt = <1>;
        sensor0_stby_mode = <0>;
        sensor0_vflip = <0>;
        sensor0_hflip = <0>;
        sensor0_iovdd-supply = <&reg_aldo2>;
        sensor0_iovdd_vol = <1800000>;
        sensor0_avdd-supply = <&reg_bldo2>;
        sensor0_avdd_vol = <2800000>;
        sensor0_dvdd-supply = <&reg_dldo2>;
        sensor0_dvdd_vol = <1200000>;
        sensor0_power_en = <>;
        sensor0_reset = <&pio PA 18 1 0 1 0>;
        sensor0_pwdn = <&pio PA 19 1 0 1 0>;
        sensor0_sm_hs = <>;
        sensor0_sm_vs = <>;
        flash_handle = <&flash0>;
        act_handle = <&actuator0>;
        status = "okay";
    };
    sensor1:sensor@1 {
        device_type = "sensor1";
        sensor1_mname = "imx386_mipi_2";
        sensor1_twi_cci_id = <0>;
        sensor1_twi_addr = <0x20>;
        sensor1_mclk_id = <1>;
        sensor1_pos = "front";
        sensor1_isp_used = <1>;
        sensor1_fmt = <1>;
        sensor1_stby_mode = <0>;
        sensor1_vflip = <0>;
        sensor1_hflip = <0>;
        sensor1_iovdd-supply = <&reg_aldo2>;
        sensor1_iovdd_vol = <1800000>;
        sensor1_avdd-supply = <&reg_bldo2>;
        sensor1_avdd_vol = <2800000>;
        sensor1_dvdd-supply = <&reg_dldo2>;
        sensor1_dvdd_vol = <1200000>;
        sensor1_power_en = <>;
        sensor1_reset = <&pio PA 20 1 0 1 0>;
        sensor1_pwdn = <&pio PA 21 1 0 1 0>;
        sensor1_sm_hs = <>;
        sensor1_sm_vs = <>;
        flash_handle = <>;
        act_handle = <>;
        status = "okay";
    };
    vinc00:vinc@0 {
        vinc0_csi_sel = <0>;
        vinc0_mipi_sel = <0>;
        vinc0_isp_sel = <0>;
        vinc0_isp_tx_ch = <0>;
        vinc0_tdm_rx_sel = <0>;
        vinc0_rear_sensor_sel = <0>;
        vinc0_front_sensor_sel = <0>;
        vinc0_sensor_list = <0>;
        work_mode = <0x0>;
        status = "okay";
    };
    vinc01:vinc@1 {
        vinc1_csi_sel = <2>;
        vinc1_mipi_sel = <0xff>;
        vinc1_isp_sel = <1>;
        vinc1_isp_tx_ch = <1>;
        vinc1_tdm_rx_sel = <1>;
        vinc1_rear_sensor_sel = <0>;
        vinc1_front_sensor_sel = <0>;
        vinc1_sensor_list = <0>;
        status = "disabled";
    };
    vinc02:vinc@2 {
        vinc2_csi_sel = <2>;
        vinc2_mipi_sel = <0xff>;
        vinc2_isp_sel = <2>;
        vinc2_isp_tx_ch = <2>;
        vinc2_tdm_rx_sel = <2>;
        vinc2_rear_sensor_sel = <0>;
        vinc2_front_sensor_sel = <0>;
        vinc2_sensor_list = <0>;
        status = "disabled";
    };
    vinc03:vinc@3 {
        vinc3_csi_sel = <0>;
        vinc3_mipi_sel = <0xff>;
        vinc3_isp_sel = <0>;
        vinc3_isp_tx_ch = <0>;
        vinc3_tdm_rx_sel = <0>;
        vinc3_rear_sensor_sel = <1>;
        vinc3_front_sensor_sel = <1>;
        vinc3_sensor_list = <0>;
        status = "disabled";
    };
    …………
};
```

其中：

> status 是 vin 驱动的总开关，对应的是 media 设备，使用 vin 时必须设为 okay；
>
> vind0_clk 是 vin 模块的时钟，实际使用时可以根据 sensor 的帧率和分辨率来设置；
>
> vind0_isp 是 isp 模块时钟，实际使用时可以根据 sensor 的帧率和分辨率来设置；
>
> vind0_clk 表示 csi clk，计算公式：帧率 x （vts）x （hts）x 1(wdr 则为 2) / 8 / 1(双 pixel则为 2) / 1000000，向上取整，单位为 MH；vind0_isp 表示 isp clk，计算公式：帧率 x 宽 x 高 x 1.2 / 1000000，向上取整，单位为 MH；其中有些 ic 是没有 isp_clk，csi_clk 和isp_clk 都是设置在 vind0_clk。那么 vind0_clk 设置为 csi_clk 和 isp_clk 中最大的数值；



> work_mode: 0:online mode 1:offline mode, 根据使用需求配置；



> flash0_type: 0:FLASH_RELATING, 1:FLASH_EN_INDEPEND, 2:FLASH_POWER
>
> flash0_en: flash enable gpio, type = 0 of 1
>
> flash0_mode: flash mode gpio, type = 0 of 1
>
> flash0_flvdd: flash module io power handle string, pmu power supply, type = 2
>
> flash0_flvdd_vol: flash module io power voltage, pmu power supply, type = 2
>
> status: 是否使用 flash, disable 代表关，okay 代表开



> actuator0_name: vcm name
>
> actuator0_slave: vcm iic slave address
>
> actuator0_af_pwdn: vcm power down gpio
>
> actuator0_afvdd: vcm power handle string, pmu power supply
>
> actuator0_afvdd_vol: vcm power voltage, pmu power supply
>
> status: vcm if used, disable 代表关，okay 代表开



> device_type: sensor type sensor0_mname: sensor name
>
> sensor0_twi_cci_id：sensor 所使用的 twi 或者 cci 的 id。
>
> sensor0_twi_addr：sensor 的 twi 地址
>
> sensor0_mclk_id：sensor 所使用的 mclk 的 id。
>
> sensor0_pos：sensor 的位置，前置还是后置，主要用在平板上。
>
> sensor0_isp_used: not use isp 1:use isp
>
> sensor0_fmt: 0:yuv 1:bayer raw rgb
>
> sensor0_stby_mode: not shut down power at standby 1:shut down power at standby
>
> sensor0_vflip: flip in vertical direction 0:disable 1:enable
>
> sensor0_hflip: flip in horizontal direction 0:disable 1:enable
>
> sensor0_iovdd-supply: camera module io power handle string, pmu power supply
>
> sensor0_iovdd_vol: camera module io power voltage, pmu power supply
>
> sensor0_avdd-supply: camera module analog power handle string, pmu power supply
>
> sensor0_avdd_vol: camera module analog power voltage, pmu power supply
>
> sensor0_dvdd-supply: camera module core power handle string, pmu power supply
>
> sensor0_dvdd_vol: camera module core power voltage, pmu power supply
>
> sensor0_power_en: camera module power enable gpio
>
> sensor0_reset: camera module reset gpio
>
> sensor0_pwdn: camera module pwdn gpio sensor0_sm_hs: camera module sm_hs
>
> gpio sensor0_sm_vs: camera module sm_vs gpio status: open or close sensor de
>
> vice flash/actautor/sensor 节点用于对应的外设的开关和配置。这些节点的配置一般需要参
>
> 考对应方案的原理图和外设的 data sheet 来完成。



> vinc0_csi_sel：表示该 pipeline 上 parser 的 id，必须配置，且为有效 id。
>
> vinc0_mipi_sel：表示该 pipeline 上 mipi（sublvds/hispi）的 id，不使用时配置为 0xff。
>
> vinc0_isp_sel：表示该 pipeline 上 isp 的 id，必须配置，当 isp 为空时，这个 isp 只是表示路由不做 isp 的效果处理。
>
> vinc0_isp_tx_ch 表示该 pipeline 上 isp 的 ch，必须配置，默认为 0。当 sensor 是 bt656 多通道或者 WDR 出 RAW 时，该 ch 可以配置 0～3 的值。
>
> vinc0_tdm_rx_sel: 表示该 pipeline 上 tdm rx 的 ch，必须配置，默认为 0。当不使用 tdm功能时，配置为 0xff；
>
> vinc0_rear_sensor_sel 表示该 pipeline 上使用的后置 sensor 的 id。
>
> vinc0_front_sensor_sel 表示该 pipeline 上使用的前置 sensor 的 id。
>
> vinc0_sensor_list 表示是否使用 sensor_list 来时适配不同的模组，1 表示使用，0 表示不使用。
>
> work_mode: 0:online mode 1:offline mode, 根据使用需求配置；只有 vinc0/4/8/12 可以配置。
>
> status：vipp 的使能开关，okay or disable。





### 2.5 源码模块结构

驱动路径位于 drivers/media/platform/sunxi-vin 目录。

```
sunxi-vin:.
├── Kconfig
├── Makefile
├── modules
│   ├── actuator
│   │   ├── actuator.c     ；vcm driver的一般行为
│   │   ├── actuator.h     ；vcm driver的头文件
│   │   ├── ad5820_act.c   ；具体vcm driver型号实现
│   │   ├── an41908a_act.c ；具体vcm driver型号实现
│   │   ├── dw9714_act.c   ；具体vcm driver型号实现
│   │   ├── Makefile       ；编译文件
│   ├── flash
│   │   ├── flash.c ；led补光灯控制实现
│   │   ├── flash.h ；led补光灯驱动头文件
│   └── sensor
│   ├── ar0238.c      ；具体的sensor驱动
│   ├── camera_cfg.h  ；camera ioctl扩展命令头文件
│   ├── camera.h      ；camera公用结构体头文件
│   ├── gc030a_mipi.c ；具体的sensor驱动
│   ├── gc0310_mipi.c ；具体的sensor驱动
│   ├── gc5024_mipi.c ；具体的sensor驱动
│   ├── imx179_mipi.c ；具体的sensor驱动
│   ├── imx214.c      ；具体的sensor驱动
│   ├── imx219.c      ；具体的sensor驱动
│   ├── imx317_mipi.c ；具体的sensor驱动
│   ├── Makefile      ；驱动的编译文件
│   ├── nvp6134     ；具体的dvp sensor驱动
│   │   ├── acp.c
│   │   ├── acp_firmup.c
│   │   ├── acp_firmup.h
│   │   ├── acp.h
│   │   ├── common.h
│   │   ├── csi_dev_nvp6134.c
│   │   ├── csi_dev_nvp6134.h
│   │   ├── eq.c
│   │   ├── eq_common.c
│   │   ├── eq_common.h
│   │   ├── eq.h
│   │   ├── eq_recovery.c
│   │   ├── eq_recovery.h
│   │   ├── Makefile
│   │   ├── nvp6134c.c ；具体的sensor驱动实现
│   │   ├── type.h
│   │   ├── video.c
│   │   └── video.h
│   ├── nvp6158 ；具体的dvp sensor驱动
│   │   ├── audio.c ；音频部分实现
│   │   ├── audio.h ；音频部分头文件接口
│   │   ├── coax_protocol.c
│   │   ├── coax_protocol.h
│   │   ├── coax_table.h
│   │   ├── common.h
│   │   ├── Makefile
│   │   ├── modules.builtin
│   │   ├── modules.order
│   │   ├── motion.c
│   │   ├── motion.h
│   │   ├── nvp6158c.c ；具体的sensor驱动实现
│   │   ├── nvp6158_drv.c
│   │   ├── nvp6158_drv.h
│   │   ├── nvp6168_eq_table.h
│   │   ├── video_auto_detect.c
│   │   ├── video_auto_detect.h
│   │   ├── video.c
│   │   ├── video_eq.c
│   │   ├── video_eq.h
│   │   ├── video_eq_table.h
│   │   ├── video.h
│   ├── rn6854m_mipi.c ；具体的sensor驱动实现
│   ├── sensor-compat-ioctl32.c
│   ├── sensor_helper.c ；驱动函数接口的实现
│   ├── sensor_helper.h ；驱动函数接口的定义
├── modules.builtin
├── modules.order
├── platform
│   ├── platform_cfg.h ；vin平台配置文件
│   ├── sun50iw10p1_vin_cfg.h ；不同平台配置文件
│   ├── sun50iw3p1_vin_cfg.h ；不同平台配置文件
│   ├── sun50iw6p1_vin_cfg.h ；不同平台配置文件
│   ├── sun50iw9p1_vin_cfg.h ；不同平台配置文件
│   ├── sun8iw12p1_vin_cfg.h ；不同平台配置文件
│   ├── sun8iw15p1_vin_cfg.h ；不同平台配置文件
│   ├── sun8iw16p1_vin_cfg.h ；不同平台配置文件
│   └── sun8iw19p1_vin_cfg.h ；不同平台配置文件
├── top_reg.c
├── top_reg.h
├── top_reg_i.h
├── top_reg.o
├── utility
│   ├── bsp_common.c
│   ├── bsp_common.h
│   ├── bsp_common.o
│   ├── cfg_op.c ;读取ini文件的实现函数
│   ├── cfg_op.h ;读取ini文件的实现函数
│   ├── config.c ;sensor电压、通道选择、i2c地址等信息读取函数
│   ├── config.h ;sensor电压、通道选择、i2c地址等信息读取函数头文件
│   ├── vin_io.h ;vin模块寄存器操作头文件
│   ├── vin_os.c
│   ├── vin_os.h
│   ├── vin_supply.c
│   ├── vin_supply.h
├── vin.c
├── vin-cci
│   ├── bsp_cci.c ；底层cci bsp函数
│   ├── bsp_cci.h ；底层cci bsp函数头文件
│   ├── cci_helper.c ；cci 帮助函数，供sensor驱动调用
│   ├── cci_helper.h ；cci 帮助函数头文件
│   ├── csi_cci_reg.c ；cci硬件底层实现
│   ├── csi_cci_reg.h ；cci硬件底层实现头文件
│   ├── csi_cci_reg_i.h ；cci 寄存器资源头文件
│   ├── Kconfig
│   ├── sunxi_cci.c ；cci 平台驱动源文件
│   ├── sunxi_cci.h ；cci 平台驱动头文件
├── vin-csi
│   ├── parser_reg.c   ；CSI控制函数
│   ├── parser_reg.h   ；CSI控制函数头文件
│   ├── parser_reg_i.h ；CSI 寄存器值
│   ├── sunxi_csi.c    ；csi 子模块驱动原文件
│   ├── sunxi_csi.h    ；csi 子模块驱动头文件
├── vin.h
├── vin-isp
│   ├── isp500
│   │   ├── isp500_reg_cfg.c
│   │   ├── isp500_reg_cfg.h
│   │   ├── isp500_reg_cfg.o
│   │   └── isp500_reg.h
│   ├── isp520
│   │   ├── isp520_reg_cfg.c
│   │   ├── isp520_reg_cfg.h
│   │   └── isp520_reg.h
│   ├── isp521
│   │   ├── isp521_reg_cfg.c
│   │   ├── isp521_reg_cfg.h
│   │   └── isp521_reg.h
│   ├── isp522
│   │   ├── isp522_reg_cfg.c
│   │   ├── isp522_reg_cfg.h
│   │   └── isp522_reg.h
│   ├── isp_default_tbl.h
│   ├── sunxi_isp.c
│   ├── sunxi_isp.h
│   └── sunxi_isp.o
├── vin-mipi
│   ├── bsp_mipi_csi.c ；底层mipi bsp函数
│   ├── bsp_mipi_csi.h ；底层mipi bsp函数头文件
│   ├── bsp_mipi_csi_null.c ；底层mipi bsp空函数
│   ├── bsp_mipi_csi_v1.c   ；底层mipi bsp函数--v1
│   ├── combo_common.h
│   ├── combo_csi
│   │   ├── combo_csi_reg.c
│   │   ├── combo_csi_reg.h
│   │   └── combo_csi_reg_i.h
│   ├── combo_rx
│   │   ├── combo_rx_reg.c
│   │   ├── combo_rx_reg.h
│   │   ├── combo_rx_reg_i.h
│   │   └── combo_rx_reg_null.c
│   ├── dphy
│   │   ├── dphy.h ；mipi dphy头文件
│   │   ├── dphy_reg.c ；mipi dphy底层实现函数
│   │   ├── dphy_reg.h ；mipi dphy底层实现函数头文件
│   │   └── dphy_reg_i.h ；mipi dphy 寄存器资源头文件
│   ├── protocol
│   │   ├── protocol.h ；mipi协议层头文件
│   │   ├── protocol_reg.c ；mipi协议层底层实现
│   │   ├── protocol_reg.h ；mipi协议层底层实现头文件
│   │   └── protocol_reg_i.h
│   ├── protocol.h
│   ├── sunxi_mipi.c
│   ├── sunxi_mipi.h
├── vin-stat
│   ├── vin_h3a.c ；3A控制接口函数
│   ├── vin_h3a.h ；3A控制接口函数头文件
├── vin-tdm
│   ├── tdm_reg.c ；TDM寄存器控制函数
│   ├── tdm_reg.h
│   ├── tdm_reg_i.h
│   ├── vin_tdm.c
│   └── vin_tdm.h
├── vin_test
│   ├── mplane_image
│   │   ├── csi_test_mplane.c ；camera抓图测试用例
│   │   └── Makefile ；测试用例编译文件
│   ├── sunxi_camera_v2.h
│   └── sunxi_display2.h
├── vin-video
│   ├── dma_reg.c ；csi dma寄存器控制函数
│   ├── dma_reg.h ；csi dma寄存器控制函数
│   ├── dma_reg_i.h ；csi dma 寄存器值定义头文件
│   ├── vin_core.c ；vin模块核心
│   ├── vin_core.h ；vin模块核心头文件
│   ├── vin_video.c ； 数据格式处理、pipe通道选择、Buffer管理等函数
│   ├── vin_video.h ；数据格式处理、pipe通道选择、Buffer管理等函数头文件
└── vin-vipp
    ├── sunxi_scaler.c ；图像压缩处理函数
    ├── sunxi_scaler.h ；图像压缩处理函数头文件
    ├── vipp_reg.c ；vipp寄存器控制函数
    ├── vipp_reg.h ；vipp寄存器控制函数头文件
    ├── vipp_reg_i.h ；vipp寄存器具体描述头文件
```


