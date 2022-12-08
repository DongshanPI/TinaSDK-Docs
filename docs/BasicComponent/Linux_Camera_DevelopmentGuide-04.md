# Linux_Camera 模块配置
## 4 模块配置

### 4.1 Tina 配置

Tina 中主要是修改平台的modules.mk 配置，modules.mk 主要完成两个方面：

1.拷贝相关的ko 模块到小机rootfs 中

2.rootfs 启动时，按顺序自动加载相关的ko 模块。

由于内核框架的不一样，需要区分vfe 和vin 进行相应的配置。

#### 4.1.1 vfe 框架

modules.mk 配置路径(以R40 平台的为例)：

```
target/allwinner/r40-common/modules.mk
```

其中的r40-common 为R40 平台共有的配置文件目录，相应的修改对应平台的modules.mk即可。

```
define KernelPackage/sunxi-vfe
    SUBMENU:=$(VIDEO_MENU)
    TITLE:=sunxi-vfe support
    FILES:=$(LINUX_DIR)/drivers/media/v4l2-core/videobuf2-core.ko
    FILES+=$(LINUX_DIR)/drivers/media/v4l2-core/videobuf2-memops.ko
    FILES+=$(LINUX_DIR)/drivers/media/v4l2-core/videobuf2-dma-contig.ko
    FILES+=$(LINUX_DIR)/drivers/media/v4l2-core/videobuf2-v4l2.ko
    FILES+=$(LINUX_DIR)/drivers/media/platform/sunxi-vfe/vfe_io.ko
    FILES+=$(LINUX_DIR)/drivers/media/platform/sunxi-vfe/device/ov5640.ko (详见1)
    FILES+=$(LINUX_DIR)/drivers/media/platform/sunxi-vfe/vfe_v4l2.ko
    AUTOLOAD:=$(call AutoLoad,90,videobuf2-core videobuf2-memops videobuf2-dma-contig
    videobuf2-v4l2 vfe_io ov5640 vfe_v4l2) (详见2)
endef
    define KernelPackage/sunxi-vfe/description
    	Kernel modules for sunxi-vfe support
endef
详注：
    1.由具体使用的模组确定，需要确定内核路径中这个驱动是否被编译出来。
    2.AUTOLOAD为小机rootfs挂载后自动加载的机制，vfe_v4l2.ko必须在最后加载，其它ko可以按照上面的相对顺序加载。必须修改相应的sensor ko才会开启自加载。
```

#### 4.1.2 vin 框架

modules.mk 配置路径(以R30 平台的为例)：

```
target/allwinner/r30-common/modules.mk
```

其中的r30-common 为R30 平台共有的配置文件目录，相应的修改对应平台的modules.mk即可。

```
define KernelPackage/sunxi-vin
    SUBMENU:=$(VIDEO_MENU)
    TITLE:=sunxi-vin support
    FILES:=$(LINUX_DIR)/drivers/media/v4l2-core/videobuf2-core.ko
    FILES+=$(LINUX_DIR)/drivers/media/v4l2-core/videobuf2-memops.ko
    FILES+=$(LINUX_DIR)/drivers/media/v4l2-core/videobuf2-dma-contig.ko
    FILES+=$(LINUX_DIR)/drivers/media/v4l2-core/videobuf2-v4l2.ko
    FILES+=$(LINUX_DIR)/drivers/media/platform/sunxi-vin/vin_io.ko
    /*对焦马达驱动加载*/
    FILES+=$(LINUX_DIR)/drivers/media/platform/sunxi-vin/modules/actuator/actuator.ko
    FILES+=$(LINUX_DIR)/drivers/media/platform/sunxi-vin/modules/actuator/dw9714_act.ko(详见
    3)
    FILES+=$(LINUX_DIR)/drivers/media/platform/sunxi-vin/modules/sensor/ov5640.ko (详见1)
    FILES+=$(LINUX_DIR)/drivers/media/platform/sunxi-vin/vin_v4l2.ko
    AUTOLOAD:=$(call AutoLoad,90,videobuf2-core videobuf2-memops videobuf2-dma-contig
    videobuf2-v4l2 vin_io actuator dw9714_act ov5640 vin_v4l2) (详见2)
endef
    define KernelPackage/sunxi-vin/description
    	Kernel modules for sunxi-vin support
endef
详注：
    1.由具体使用的模组确定，需要确定内核路径中这个驱动是否被编译出来。
    2.AUTOLOAD为小机rootfs挂载后自动加载的机制，vin_v4l2.ko必须在最后加载，其它ko可以按照上面的相对顺序加载。
    3.对焦马达驱动加载顺序必须在sensor驱动加载之前，具体驱动型号根据模组规格书进行确认。
```

V 系列平台在完成modules.mk 配置后，还需要完成.ko 挂载脚本S00mpp 的配置，S00mpp

配置路径（以V853 平台为例）：

```
target/allwinner/v853-perf1/busybox-init-base-files/etc/init.d
```

其中的v853-perf1 为V 系列平台共有的配置文件目录，相应的修改对应平台的S00mpp 即可。

```
#!/bin/sh
#
# Load mpp modules....
#
MODULES_DIR="/lib/modules/`uname -r`"
start() {
    printf "Load mpp modules\n"
    insmod $MODULES_DIR/videobuf2-core.ko
    insmod $MODULES_DIR/videobuf2-memops.ko
    insmod $MODULES_DIR/videobuf2-dma-contig.ko
    insmod $MODULES_DIR/videobuf2-v4l2.ko
    insmod $MODULES_DIR/vin_io.ko
    # insmod $MODULES_DIR/sensor_power.ko
    insmod $MODULES_DIR/gc4663_mipi.ko
    insmod $MODULES_DIR/vin_v4l2.ko
    insmod $MODULES_DIR/sunxi_aio.ko
    insmod $MODULES_DIR/sunxi_eise.ko
    # insmod $MODULES_DIR/vipcore.ko
}
stop() {
    printf "Unload mpp modules\n"
    # rmmod $MODULES_DIR/vipcore.ko
    rmmod $MODULES_DIR/sunxi_eise.ko
    rmmod $MODULES_DIR/sunxi_aio.ko
    rmmod $MODULES_DIR/vin_v4l2.ko
    rmmod $MODULES_DIR/gc4663_mipi.ko
    # rmmod $MODULES_DIR/sensor_power.ko
    rmmod $MODULES_DIR/vin_io.ko
    rmmod $MODULES_DIR/videobuf2-v4l2.ko
    rmmod $MODULES_DIR/videobuf2-dma-contig.ko
    rmmod $MODULES_DIR/videobuf2-memops.ko
    rmmod $MODULES_DIR/videobuf2-core.ko
}
case "$1" in
        start)
        start
        ;;
        stop)
        stop
        ;;
        restart|reload)
        stop
        start
        ;;
    *)
    echo "Usage: $0 {start|stop|restart}"
    exit 1
esac
exit $?
```

### 4.2 CSI 板级配置

Tina 平台根据不同的平台差异分别使用sys_config.fex 或board.dst 配置camera CSI，具体的对应关系如下表，下面将分别介绍sys_config.fex 和board.dts 中关

于camera CSI 配置。



<center>表4-1: 平台配置方式对应表</center>

| 平台  | CSI 使用的配置方式 |
| ----- | ------------------ |
| F35   | sys_config.fex     |
| R16   | sys_config.fex     |
| R18   | sys_config.fex     |
| R30   | sys_config.fex     |
| R40   | sys_config.fex     |
| R311  | sys_config.fex     |
| MR133 | sys_config.fex     |
| R818  | board.dts          |
| MR813 | board.dts          |
| R528  | board.dts          |
| V536  | sys_config.fex     |
| V533  | board.dts          |
| V831  | board.dts          |
| V833  | board.dts          |
| V851  | board.dts          |
| V853  | board.dts          |

#### 4.2.1 sys_config.fex 平台配置

sys_config.fex 配置camera CSI，CSI sys_config.fex 部分对应的字段为：[csi0]。通过举例R40 平台说明在实际使用中应该如何配置：假如使用一个并口camera

 模组需要配置[csi0] 的公用部分和[csi0] 的vip_dev0_(x) 部分，另外[csi0] 中vip_used 设置为1，[csi1]中vip_used 设置为0。

下面给出一个ov5640 模组的参考配置：其中[csi0] 为并口的配置。具体填写方法请参照以下说明：

```
/* 下面部分的CSI配置适用4.9内核之前的平台*/
;--------------------------------------------------------------------------------
;csi (COMS Sensor Interface) configuration
;csi(x)_dev(x)_used: 0:disable 1:enable
;csi(x)_dev(x)_isp_used 0:not use isp 1:use isp
;csi(x)_dev(x)_fmt: 0:yuv 1:bayer raw rgb
;csi(x)_dev(x)_stby_mode: 0:not shut down power at standby 1:shut down power at standby
;csi(x)_dev(x)_vflip: flip in vertical direction 0:disable 1:enable
;csi(x)_dev(x)_hflip: flip in horizontal direction 0:disable 1:enable
;csi(x)_dev(x)_iovdd: camera module io power handle string, pmu power supply
;csi(x)_dev(x)_iovdd_vol: camera module io power voltage, pmu power supply
;csi(x)_dev(x)_avdd: camera module analog power handle string, pmu power supply
;csi(x)_dev(x)_avdd_vol: camera module analog power voltage, pmu power supply
;csi(x)_dev(x)_dvdd: camera module core power handle string, pmu power supply
;csi(x)_dev(x)_dvdd_vol: camera module core power voltage, pmu power supply
;csi(x)_dev(x)_afvdd: camera module vcm power handle string, pmu power supply
;csi(x)_dev(x)_afvdd_vol: camera module vcm power voltage, pmu power supply
;fill voltage in uV, e.g. iovdd = 2.8V, csix_iovdd_vol = 2800000
;fill handle string as below:
;axp22_eldo3
;axp22_dldo4
;axp22_eldo2
;fill handle string "" when not using any pmu power supply
;--------------------------------------------------------------------------------
[csi0]
csi0_used = 1
csi0_sensor_list = 0
csi0_pck = port:PE00<2><default><default><default>
csi0_mck = port:PE01<2><default><default><default>
csi0_hsync = port:PE02<2><default><default><default>
csi0_vsync = port:PE03<2><default><default><default>
csi0_d0 = port:PE04<2><default><default><default>
csi0_d1 = port:PE05<2><default><default><default>
csi0_d2 = port:PE06<2><default><default><default>
csi0_d3 = port:PE07<2><default><default><default>
csi0_d4 = port:PE08<2><default><default><default>
csi0_d5 = port:PE09<2><default><default><default>
csi0_d6 = port:PE10<2><default><default><default>
csi0_d7 = port:PE11<2><default><default><default>
csi0_sck = port:PE12<2><default><default><default>
csi0_sda = port:PE13<2><default><default><default>
[csi0/csi0_dev0]
csi0_dev0_used = 1
csi0_dev0_mname = "ov5640" ;必须和sensor驱动中的SENSOR_NAME一致
csi0_dev0_twi_addr = 0x78 ；请参考实际模组的8bit ID填写
csi0_dev0_twi_id = 2
csi0_dev0_pos = "rear"
csi0_dev0_isp_used = 0 ；YUV格式填0，RAW格式填1
csi0_dev0_fmt = 0 ；YUV格式填0，RAW格式填1
csi0_dev0_stby_mode = 0
csi0_dev0_vflip = 0
csi0_dev0_hflip = 0
csi0_dev0_iovdd = "csi-iovcc" ；电源请参考实际原理图填写，同时参考
sys_config.fex 的regulator 配置，确认该字段有效
csi0_dev0_iovdd_vol = 2800000 ；电压值参考datasheet
csi0_dev0_avdd = "csi-avdd" ；电源请参考实际原理图填写，同时参考
sys_config.fex 的regulator 配置，确认该字段有效
csi0_dev0_avdd_vol = 2800000 ；电压值参考datasheet
csi0_dev0_dvdd = "csi-dvdd" ；电源请参考实际原理图填写，同时参考
sys_config.fex 的regulator 配置，确认该字段有效
csi0_dev0_dvdd_vol = 1500000 ；电压值参考datasheet
csi0_dev0_afvdd = "csi-afvcc" ；电源请参考实际原理图填写，同时参考
sys_config.fex 的regulator 配置，确认该字段有效
csi0_dev0_afvdd_vol = 2800000 ；电压值参考datasheet
csi0_dev0_power_en =
csi0_dev0_reset = port:PE14<1><0><1><0> ；io选取参照实际原理图
csi0_dev0_pwdn = port:PE15<1><0><1><0> ；io选取参照实际原理图
csi0_dev0_flash_used = 0
csi0_dev0_flash_type = 2
csi0_dev0_flash_en =
csi0_dev0_flash_mode =
csi0_dev0_flvdd = ""
csi0_dev0_flvdd_vol =
csi0_dev0_af_pwdn =
csi0_dev0_act_used = 0
csi0_dev0_act_name = "ad5820_act"
csi0_dev0_act_slave = 0x18
/* 下面部分的CSI配置适用4.9内核平台*/
;--------------------------------------------------------------------------------
;csi (COMS Sensor Interface) configuration
;csi(x)_dev(x)_used: 0:disable 1:enable
;csi(x)_dev(x)_isp_used 0:not use isp 1:use isp
;csi(x)_dev(x)_fmt: 0:yuv 1:bayer raw rgb
;csi(x)_dev(x)_stby_mode: 0:not shut down power at standby 1:shut down power at standby
;csi(x)_dev(x)_vflip: flip in vertical direction 0:disable 1:enable
;csi(x)_dev(x)_hflip: flip in horizontal direction 0:disable 1:enable
;csi(x)_dev(x)_iovdd: camera module io power handle string, pmu power supply
;csi(x)_dev(x)_iovdd_vol: camera module io power voltage, pmu power supply
;csi(x)_dev(x)_avdd: camera module analog power handle string, pmu power supply
;csi(x)_dev(x)_avdd_vol: camera module analog power voltage, pmu power supply
;csi(x)_dev(x)_dvdd: camera module core power handle string, pmu power supply
;csi(x)_dev(x)_dvdd_vol: camera module core power voltage, pmu power supply
;csi(x)_dev(x)_afvdd: camera module vcm power handle string, pmu power supply
;csi(x)_dev(x)_afvdd_vol: camera module vcm power voltage, pmu power supply
;fill voltage in uV, e.g. iovdd = 2.8V, csix_iovdd_vol = 2800000
;fill handle string as below:
;axp22_eldo3
;axp22_dldo4
;axp22_eldo2
;fill handle string "" when not using any pmu power supply
;--------------------------------------------------------------------------------
[vind0]
vind0_used = 1
[vind0/csi_cci0]
csi_cci0_used = 1 ;配置是否使用CCI，如果使用CCI，需要使能该配置并配置下面的CCI引脚
csi_cci0_sck = port:PE01<2><default><default><default>
csi_cci0_sda = port:PE02<2><default><default><default>
[vind0/flash0]
flash0_used = 0
flash0_type = 2
flash0_en =
flash0_mode =
flash0_flvdd = ""
flash0_flvdd_vol =
[vind0/actuator0]
actuator0_used = 0
actuator0_name = "ad5820_act"
actuator0_slave = 0x18
actuator0_af_pwdn =
actuator0_afvdd = "afvcc-csi"
actuator0_afvdd_vol = 2800000
[vind0/sensor0]
sensor0_used = 0
sensor0_mname = "gc8034_mipi"
sensor0_twi_cci_id = 0
sensor0_twi_addr = 0x6e
sensor0_pos = "rear"
sensor0_isp_used = 1
sensor0_fmt = 1
sensor0_stby_mode = 0
sensor0_vflip = 0
sensor0_hflip = 0
sensor0_cameravdd = ""
sensor0_cameravdd_vol = 3300000
sensor0_iovdd = "iovdd-csi"
sensor0_iovdd_vol = 1800000
sensor0_avdd = "avdd-csi-f"
sensor0_avdd_vol = 2800000
sensor0_dvdd = "dvdd-csi"
sensor0_dvdd_vol = 1200000
sensor0_power_en =
sensor0_reset = port:PE06<0><0><1><0>
sensor0_pwdn = port:PE05<0><0><1><0>
[vind0/sensor1]
sensor1_used = 1
sensor1_mname = "gc8034_mipi" ;必须要和驱动的SENSOR_NAME 一致
sensor1_twi_cci_id = 0 ;配置使用的TWI id，如果使用TWI，则不使用CCI
sensor1_twi_addr = 0x6e ;配置sensor的i2c地址
sensor1_pos = "front"
sensor1_isp_used = 1 ;配置是否使用isp
sensor1_fmt = 1
sensor1_stby_mode = 0
sensor1_vflip = 0
sensor1_hflip = 0
sensor1_cameravdd = ""
sensor1_cameravdd_vol = 3300000
sensor1_iovdd = "iovdd-csi"
sensor1_iovdd_vol = 1800000
sensor1_avdd = "avdd-csi-f"
sensor1_avdd_vol = 2800000
sensor1_dvdd = "dvdd-csi"
sensor1_dvdd_vol = 1200000
sensor1_power_en =
sensor1_reset = port:PE06<0><0><1><0>
sensor1_pwdn = port:PE05<0><0><1><0>
[vind0/vinc0] ;配置video0 的数据链路
vinc0_used = 1
vinc0_csi_sel = 0
vinc0_mipi_sel = 0
vinc0_isp_sel = 0
vinc0_rear_sensor_sel = 1 ;配置使用sensor1 输出图像数据到video0
vinc0_front_sensor_sel = 1 ;配置使用sensor1 输出图像数据到video0
vinc0_sensor_list = 0
[vind0/vinc1]
vinc1_used = 0
vinc1_csi_sel = 0
vinc1_mipi_sel = 0
vinc1_isp_sel = 0
vinc1_rear_sensor_sel = 1
vinc1_front_sensor_sel = 1
vinc1_sensor_list = 0
```

关于电源的配置，根据板子的原理图，了解需要sensor 驱动配置哪几路电，然后在sys_config.fex中进行配置：比如说sensor0 有个“CSI-IOVCC” 连接到AXP 

的“LDO4”，那么，在sys_config.fex 中搜索LDO4 ，然后在其后面增加“csi-iovcc” ，这样，在sensor 端就可以使用该标号配置sensor0_iovdd。

```
regulator14 = "pmu1736_bldo2 none csi-iovdd"
sensor0_iovdd = "csi-iovdd"
```

同时关于mr133/R311 平台，sys_config.fex 中的vinc0_rear_sensor_sel 和vinc0_front_sensor_sel



配置决定着使用哪路sensor 输入数据，该配置与硬件连接相关，可参考本文档最后的其他注意事项章节。

#### 4.2.2 board.dts 平台配置

当前MR813/R818/R528 平台的摄像头配置不再使用sys_config.fex 而使用board.dts，文件存放在tina/device/config/chips/mr813(R818、R528)/configs/

< 方案> 目录下，摄像头相关的配置如下：

```
    vind0:vind@0 {
        vind0_clk = <336000000>;
        vind0_isp = <327000000>;
        status = "okay";
        actuator0:actuator@0 {
        device_type = "actuator0";
        actuator0_name = "ad5820_act";/*必须要和驱动的SUNXI_ACT_NAME一致*/
        actuator0_slave = <0x18>;/*必须和驱动的SUNXI_ACT_ID一致*/
        actuator0_af_pwdn = <>;
        actuator0_afvdd = "afvcc-csi";
        actuator0_afvdd_vol = <2800000>;/*af模块的配电不在此处，在sensor配置中*/
        status = "disabled";/*使能开关，当使用AF功能时，status = "okay"*/
    };
    flash0:flash@0 {
        device_type = "flash0";
        flash0_type = <2>;
        flash0_en = <>;
        flash0_mode = <>;
        flash0_flvdd = "";
        flash0_flvdd_vol = <>;
        device_id = <0>;
        status = "disabled";
    };
    sensor0:sensor@0 {
        device_type = "sensor0";
        sensor0_mname = "imx278_mipi"; /* 必须要和驱动的 SENSOR_NAME 一致 */
        sensor0_twi_cci_id = <2>;
        sensor0_twi_addr = <0x20>;
        sensor0_mclk_id = <0>;
        sensor0_pos = "rear";
        sensor0_isp_used = <1>; /* R528 没有isp，该项需要配置为0 */
        sensor0_fmt = <1>;
        sensor0_stby_mode = <0>;
        sensor0_vflip = <0>;
        sensor0_hflip = <0>;
        /* sensor iovdd 连接的 ldo，根据硬件原理图的连接，
        * 确认是连接到 axp 哪个 ldo，假设 iovdd 连接到 aldo3，
        * 则 sensor0_iovdd-supply = <&reg_aldo3>，其他同理。
        */
        sensor0_iovdd-supply = <&reg_dldo2>;
        sensor0_iovdd_vol = <1800000>;
        sensor0_avdd-supply = <&reg_dldo3>;
        sensor0_avdd_vol = <2800000>;
        sensor0_dvdd-supply = <&reg_eldo2>;
        sensor0_afvdd-supply = <&reg_aldo3>;/*根据硬件原理图，确定配的哪路电*/
        sensor0_afvdd_vol = <2800000>;/*根据硬件原理图，确认工作电压*/
        sensor0_dvdd_vol = <1200000>;
        sensor0_power_en = <>;
        /* 根据板子实际连接，修改 reset、pwdn 的引脚即可 */
        /* GPIO 信息配置：pio 端口 组内序号 功能分配 内部电阻状态 驱动能力 输出电平状态 */
        sensor0_reset = <&pio PE 9 1 0 1 0>;
        sensor0_pwdn = <&pio PE 8 1 0 1 0>;
        status = "okay";
    };
    sensor1:sensor@1 {
        device_type = "sensor1";
        sensor1_mname = "imx386_mipi";
        sensor1_twi_cci_id = <3>;
        sensor1_twi_addr = <0x20>;
        sensor1_mclk_id = <1>;
        sensor1_pos = "front";
        sensor1_isp_used = <1>;
        sensor1_fmt = <1>;
        sensor1_stby_mode = <0>;
        sensor1_vflip = <0>;
        sensor1_hflip = <0>;
        sensor1_iovdd-supply = <&reg_dldo2>;
        sensor1_iovdd_vol = <1800000>;
        sensor1_avdd-supply = <&reg_dldo3>;
        sensor1_avdd_vol = <2800000>;
        sensor1_dvdd-supply = <&reg_eldo2>;
        sensor1_dvdd_vol = <1200000>;
        sensor0_power_en = <>;
        sensor1_reset = <&pio PE 7 1 0 1 0>;
        sensor1_pwdn = <&pio PE 6 1 0 1 0>;
        status = "okay";
    };
    /* 一个 vinc 代表一个 /dev/video 设备 */
    vinc0:vinc@0 {
        vinc0_csi_sel = <0>; /* 代表选择的 csi，MR813/R818 有两个 csi 接口
        */
        vinc0_mipi_sel = <0>; /* 代表选择的 mipi 接口，MR813/R818 有两个 mipi
        接口 */
        vinc0_isp_sel = <0>;
        vinc0_isp_tx_ch = <0>; /* 表示 ISP 的通道数，一般配置为 0 */
        vinc0_tdm_rx_sel = <0>; /* 与 isp_sel 保持一致即可 */
        vinc0_rear_sensor_sel = <0>; /* 该 video 可以选择从哪个 sensor 输入图像数据
        */
        vinc0_front_sensor_sel = <1>;
        vinc0_sensor_list = <0>;
        status = "okay";
    };
    vinc1:vinc@1 {
        vinc1_csi_sel = <0>;
        vinc1_mipi_sel = <0>; /* R528没有mipi，该项配置为0xff */
        vinc1_isp_sel = <0>; /* R528没有isp，该项配置为0 */
        vinc1_isp_tx_ch = <0>; /* R528没有isp，该项配置为0 */
        vinc1_tdm_rx_sel = <0>; /* R528没有isp，该项配置为0xff */
        vinc1_rear_sensor_sel = <0>;
        vinc1_front_sensor_sel = <1>;
        vinc1_sensor_list = <0>;
        status = "okay";
    };
    vinc2:vinc@2 {
        vinc2_csi_sel = <1>;
        vinc2_mipi_sel = <1>;
        vinc2_isp_sel = <1>;
        vinc2_isp_tx_ch = <0>;
        vinc2_tdm_rx_sel = <1>;
        vinc2_rear_sensor_sel = <1>;
        vinc2_front_sensor_sel = <1>;
        vinc2_sensor_list = <0>;
        status = "okay";
    };
    vinc3:vinc@3 {
        vinc3_csi_sel = <1>;
        vinc3_mipi_sel = <1>;
        vinc3_isp_sel = <1>;
        vinc3_isp_tx_ch = <0>;
        vinc3_tdm_rx_sel = <1>;
        vinc3_rear_sensor_sel = <1>;
        vinc3_front_sensor_sel = <1>;
        vinc3_sensor_list = <0>;
        status = "okay";
    };
    };
    /* 以下将配置两路 sensor 输入，产生 4 个 video 节点，内核配置 CONFIG_SUPPORT_ISP_TDM=n,此时
    * 不同 sensor 输出的节点不能同时使用,比如以下配置的 video0 不可以和 video2 video3 同时使用
    */
    vinc0:vinc@0 {
        vinc0_csi_sel = <0>;
        vinc0_mipi_sel = <0>;
        vinc0_isp_sel = <0>;
        vinc0_isp_tx_ch = <0>;
        vinc0_tdm_rx_sel = <0>;
        vinc0_rear_sensor_sel = <0>;
        vinc0_front_sensor_sel = <0>;
        vinc0_sensor_list = <0>;
        status = "okay";
    };
    vinc1:vinc@1 {
        vinc1_csi_sel = <0>;
        vinc1_mipi_sel = <0>;
        vinc1_isp_sel = <0>;
        vinc1_isp_tx_ch = <0>;
        vinc1_tdm_rx_sel = <0>;
        vinc1_rear_sensor_sel = <0>;
        vinc1_front_sensor_sel = <1>;
        vinc1_sensor_list = <0>;
        status = "okay";
    };
    vinc2:vinc@2 {
        vinc2_csi_sel = <1>;
        vinc2_mipi_sel = <1>;
        vinc2_isp_sel = <0>;
        vinc2_isp_tx_ch = <0>;
        vinc2_tdm_rx_sel = <0>;
        vinc2_rear_sensor_sel = <1>;
        vinc2_front_sensor_sel = <1>;
        vinc2_sensor_list = <0>;
        status = "okay";
    };
    vinc3:vinc@3 {
        vinc3_csi_sel = <1>;
        vinc3_mipi_sel = <1>;
        vinc3_isp_sel = <0>;
        vinc3_isp_tx_ch = <0>;
        vinc3_tdm_rx_sel = <0>;
        vinc3_rear_sensor_sel = <1>;
        vinc3_front_sensor_sel = <1>;
        vinc3_sensor_list = <0>;
        status = "okay";
    };
    /* 以下配置将可以从两路 sensor 同时输入，内核配置 CONFIG_SUPPORT_ISP_TDM=y,但是有个限制，
    * 只能先运行 video0，然后才可以运行 video2，关闭的时候也是如此，先关 video2，再关 video0
    */
    vinc0:vinc@0 {
        vinc0_csi_sel = <0>;
        vinc0_mipi_sel = <0>;
        vinc0_isp_sel = <0>;
        vinc0_isp_tx_ch = <0>;
        vinc0_tdm_rx_sel = <0>;
        vinc0_rear_sensor_sel = <0>;
        vinc0_front_sensor_sel = <0>;
        vinc0_sensor_list = <0>;
        status = "okay";
    };
    vinc1:vinc@1 {
        vinc1_csi_sel = <0>;
        vinc1_mipi_sel = <0>;
        vinc1_isp_sel = <0>;
        vinc1_isp_tx_ch = <0>;
        vinc1_tdm_rx_sel = <0>;
        vinc1_rear_sensor_sel = <0>;
        vinc1_front_sensor_sel = <1>;
        vinc1_sensor_list = <0>;
        status = "okay";
    };
    vinc2:vinc@2 {
        vinc2_csi_sel = <1>;
        vinc2_mipi_sel = <1>;
        vinc2_isp_sel = <1>;
        vinc2_isp_tx_ch = <0>;
        vinc2_tdm_rx_sel = <1>;
        vinc2_rear_sensor_sel = <1>;
        vinc2_front_sensor_sel = <1>;
        vinc2_sensor_list = <0>;
        status = "okay";
    };
    vinc3:vinc@3 {
        vinc3_csi_sel = <1>;
        vinc3_mipi_sel = <1>;
        vinc3_isp_sel = <1>;
        vinc3_isp_tx_ch = <0>;
        vinc3_tdm_rx_sel = <1>;
        vinc3_rear_sensor_sel = <1>;
        vinc3_front_sensor_sel = <1>;
        vinc3_sensor_list = <0>;
        status = "okay";
    };
```

修改该文件之后，需要重新编译固件再打包，才会更新到dts。同时，如果需要使用双摄，双摄分别使用到两个ISP，那么内核需要选上SUPPORT_ISP_TDM 配置。

### 4.3 menuconfig 配置说明

在命令行进入Tina 根目录，执行命令进入配置主界面：

```
source build/envsetup.sh (详见1)
lunch 方案编号(详见2)
make menuconfig (详见3)
详注：
1.加载环境变量及tina提供的命令；
2.输入编号，选择方案；
3.进入配置主界面(对一个shell而言，前两个命令只需要执行一次)
```

make menuconfig 配置路径：

```
Kernel modules
	└─>Video Support
        └─>kmod-sunxi-vfe(vfe框架的csi camera) (详见1)
        └─>kmod-sunxi-vin(vin框架的csi camera) (详见2)
        └─>kmod-sunxi-uvc(uvc camera) (详见3)
详注：
    1.平台使用vfe框架的csi camera选择该驱动；
    2.平台使用vin框架的csi camera选择该驱动；(该项与vfe框架，在同一个平台只会出现其中一个)
    3.usb camera选择该驱动；
```

在完成sensor 驱动编写，modules.mk 和板级配置后，通过make menuconfig 选上相应的驱动，camera 即可正常使用。下面以R40 平台介绍。首先，选择

Kernel modules 选项进入下一级配置，如下图所示：

![image-20221123105646196](${media}/image-20221123105646196.png)

<center>图4-1: menuconfig</center>

然后，选择Video Support 选项，进入下一级配置，如下图所示：

![image-20221123105728564](${media}/image-20221123105728564.png)

<center>图4-2: video</center>

最后，选择kmod-sunxi-vfe 选项，可选择<*> 表示编译包含到固件，也可以选择表示仅编译不包含在固件。如下图所示：

![image-20221123105750877](${media}/image-20221123105750877.png)

<center>图4-3: sunxi</center>

### 4.4 如何增加ISP 效果配置

在完成ISP 调试之后，将会从ISP 调试工程师中得到相应的头文件配置，添加操作如下：

#### 4.4.1 VFE 框架

1. 将头文件添加到驱动的sunxi-vfe/isp_cfg/SENSOR_H 目录下；

2. 在驱动sunxi-vfe/isp_cfg 目录下，有个isp_cfg.c 文件，这文件中有个isp_cfg_array 数组，在sensor 的ISP 配置文件最下面也有个相应的结构体，在

isp_cfg_array 数组中按照数组的结构，增加sensor 的name 和结构体即可，这样将会在ISP 匹配的时候，将会根据name 匹配到相应的配置；

#### 4.4.2 VIN 框架

#### 4.4.2.1 R 系列

1. vin 框架的操作也是类似的，只是更换了位置。vin 的ISP 配置在tina/package/allwinner/libAWIspApi 目录下，其中R311、MR133 在src/isp520，而R818、

MR813 在src/isp522。在libisp/isp_cfg/SENSOR 目录下增加相应的头文件，然后在上一层目录的isp_ini_parse.c 文件增加头文件以及修改相应的isp_cfg_array cfg_arr 数组匹配即可。

2. VIN 使用ISP，需要在camerademo 中make menuconfig 的时候，选择上Choosewhether to use VIN ISP (YES)。同时VIN 的需要注意，当自己开发camera 

  HAL 层时，需要自己运行camera ISP service，具体实现可参考camerademo 的实现。添加正确时，在运行camerademo 将会输出相应的sensor 配置信息，

  比如：

```
[ISP]find imx278_mipi_2048_1152_60_0 [imx278_mipi_default_ini_mr813] isp config
```

上述表示正确查找到imx278_mipi 这个sensor 2048*1152 60fps 的ISP 配置，其他的sensorISP 配置移植正确也将会有类似的打印，输出信息分别是sensor name 、分辨率、帧率，确认这些信息一致即可。

#### 4.4.2.2 V 系列

1. V 系列ISP 库目录，V533、V83x 平台位于：softwinner/eyesee-mpp/middleware/sun8iw19p1/media/V536 平台位于：softwinner/eyesee-

  mpp/middleware/v316/media/LIBRARY/libisp/，V85x 平台位于：external/eyesee-mpp/middleware/sun8iw21/media/LIBRARY/libisp/

2. 修改libisp/isp_cfg/isp_ini_parse.c，将ISP 效果.h 包含进来，并修改struct isp_cfg_arraycfg_arr[] 结构体；其中参数定义：

  （1）Sensor 模块名称：sensor_mipi
  （2）ISP 效果头文件名称
  （3）分辨率宽
  （4）分辨率高
  （5）帧率
  （6）红外IR 模式标志位
  （7）WDR 模式标志位
  （8）ISP 参数结构体

![image-20221123113519900](${media}/image-20221123113519900.png)

<center>图4-4: sunxi</center>

### 4.5 如何输出RAW 数据

在ioctl 的VIDIOC_S_FMT 命令，将其参数pixelformat 设置为RAW 格式即可，RAW 格式如下：

```
    V4L2_PIX_FMT_SBGGR8
    V4L2_PIX_FMT_SGBRG8
    V4L2_PIX_FMT_SGRBG8
    V4L2_PIX_FMT_SRGGB8
    V4L2_PIX_FMT_SBGGR10
    V4L2_PIX_FMT_SGBRG10
    V4L2_PIX_FMT_SGRBG10
    V4L2_PIX_FMT_SRGGB10
    V4L2_PIX_FMT_SBGGR12
    V4L2_PIX_FMT_SGBRG12
    V4L2_PIX_FMT_SGRBG12
    V4L2_PIX_FMT_SRGGB12
```

将pixelformat 设置为上述的其一即可输出RAW 数据， 而如何选择上述的操作， 这个根据sensor 驱动选择， 如果驱动中sensor_formats 的mbus_code 设置为

MEDIA_BUS_FMT_SBGGR10_1X10，则在输出RAW 数据时将pixelformat 设置为V4L2_PIX_FMT_SBGGR10



当前camerademo 已经支持输出RAW 数据，可参照本文档《camerademo 输出RAW 数据》章节。



### 4.6 如何计算实际曝光时间

该部分为使用到ISP 的RAW sensor 配置信息。曝光时间的计算和曝光控制寄存器、hts、pclk这些配置相关，这些配置都在sensor 驱动，ISP 将会根据sensor 驱

动中的设置计算相应的曝光时间，所以驱动中的配置必须正确，否则在调试ISP 效果可能会遇到其他的一些问题。

```
static struct sensor_win_size sensor_win_sizes[] = {
    ...
    .hts = 928,
    .vts = 1720,
    .pclk = 48 * 1000 * 1000,
    ...
}
```

在sensor 的驱动中有以上的一些配置，曝光时间在驱动中是以曝光行为计算单位的，即在sensor_s_exp() 函数中设置的参数为曝光行，部分sensor 是以16 为一

倍的，所以在计算实际的曝光行时，需要将上述函数参数除以16。

曝光时间= 曝光行× hts / pclk

一般的pclk 都是M 级别的，所以时间单位为us，部分sensor 的曝光行为参数的十六分之一，需要除以十六，同时，曝光行不能大于vts 的值，否则将会出现降

帧、没有正常输出图像等问题。

### 4.7 如何脱离isp tuning 工具微调图像亮度

在isp 配置文件中，有类似以下的信息：

```
.ae_cfg = {
	256, 555, 256, 555, 31, 22, 22, 25, 3, 130, 16, 60, 1, 2
},
```

上述ae_cfg 参数的倒数第4 个数值（130）即是控制图像亮度的阀门（期望亮度），该值越大，图像亮度越高。ae_cfg 一共14 个，分别对应着不同的环境亮度

（Lux）。如何确定AE 当前处于哪组ae_cfg 参数呢？修改isp 配置文件中的isp_log_param = 0x1，然后重新编译运行相机应用，留意应用中关于isp 的打印信息：

[ISP_DEBUG]: isp0 ae_target 92, pic_lum 0, weight_lum 0, delta_exp_idx 138, ae_delay 0, AE_TOLERANCE

5

从上述信息可以看到当前的目标亮度是92，这时可以查看isp 配置文件ae_cfg 中哪组的阀门处于92 这个范围，如果需要增加亮度，则提高相应的阀门；降低亮度

则降低阀门。相应的根据实际调试情况修改即可。调试之后，记得将isp_log_param 参数还原为0。

### 4.8 VIN 如何设置裁剪和缩放

裁剪修改sensor 驱动：在驱动有类似以下的配置

```
static struct sensor_win_size sensor_win_sizes[] = {
    {
        .width = VGA_WIDTH,
        .height = VGA_HEIGHT,
        .hoffset = 0,
        .voffset = 0,
        .hts = 878,
        .vts = 683,
        .pclk = 72 * 1000 * 1000,
        .mipi_bps = 720 * 1000 * 1000,
        .fps_fixed = 120,
        .bin_factor = 1,
        .intg_min = 1 << 4,
        .intg_max = (683) << 4,
        .gain_min = 1 << 4,
        .gain_max = 16 << 4,
        .regs = sensor_VGA_120fps_regs,
        .regs_size = ARRAY_SIZE(sensor_VGA_120fps_regs),
        .set_size = NULL,
    },
};
```

上述的width height 表示经过isp 输出之后的数据，如果需要裁剪，修改width、height、hoffset 和voffset。裁剪之后的输出width = sensor_output_src - 

2hoffset height = sensor_height_src - 2voffset 注意，上述的hoffset voffset 必须为双数。

所以，假设sensor 输出的是640 × 480，我们想裁剪为320 × 240 的，则上述配置修改为：

```
static struct sensor_win_size sensor_win_sizes[] = {
    {
        .width = 320,
        .height = 240,
        .hoffset = 160,
        .voffset = 120,
        .hts = 878,
        .vts = 683,
        .pclk = 72 * 1000 * 1000,
        .mipi_bps = 720 * 1000 * 1000,
        .fps_fixed = 120,
        .bin_factor = 1,
        .intg_min = 1 << 4,
        .intg_max = (683) << 4,
        .gain_min = 1 << 4,
        .gain_max = 16 << 4,
        .regs = sensor_VGA_120fps_regs,
        .regs_size = ARRAY_SIZE(sensor_VGA_120fps_regs),
        .set_size = NULL,
    },
};
```

缩放配置：使用硬件缩放，可以在应用层通过VIDIOC_S_FMT 设置分辨率的时候，直接设置分辨率的大小为缩放的分辨率即可。

```
	fmt.fmt.pix_mp.width = 320;
	fmt.fmt.pix_mp.height = 240;
```

上述的操作，将会使用硬件完成相应的缩放输出。
