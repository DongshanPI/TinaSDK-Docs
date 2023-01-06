## 2 模块介绍

### 2.1 模块功能介绍

PMU，负责系统各个模块供电、按键开关机、电池充放电管理。

### 2.2 相关术语介绍

<center>表2-1: 术语简介</center>

| 术语                        | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| PMU                         | 电源管理单元，主要包括regulator、power supply、gpio、power key 这四个子功能部分。 |
| AXP                         | 全志PMU 平台的系列名称，如AXP803、AXP717 等。                |
| LDO                         | 是low dropout regulator，意为低压差线性稳压器。线性稳压器使用<br/>在其线性区域内运行的晶体管或FET，从应用的输入电压中减去超额的<br/>电压，产生经过调节的输出电压。 |
| DC-DC                       | 是直流变直流，即不同直流电源值之间的转换，只要符合这个定义都可<br/>以叫DC-DC 转换器，也包括LDO。但是一般的说法是把直流变直流由 开关方式实现的器件叫DCDC。 |
| regulator                   | Linux 内核对LDO、DC-DC 的管理核心。                          |
| USB-Power-<br/>Supply       | USB 接口对系统的供电。                                       |
| ACIN-Power-<br/>Supply<br/> | 适配器ACIN 对系统的供电。                                    |
| BAT-Power-<br/>Supply<br/>  | 电池BAT 对系统的供电。                                       |
| Power-Supply                | Linux 内核对USB、ACIN、BAT 供电的管理核心。                  |
| MFD                         | Multi Function Device，Linux 内核对多功能设备PMU 的管理核心  |
| regmap                      | Linux 内核用于管理片外模块寄存器的方法。                     |

### 2.3 模块配置介绍

#### 2.3.1 Device Tree 配置说明

在Tina 系统中，有两种dts 文件。一是用于保存芯片所有平台的模块配置${CHIP}.dtsi，二是保存每一个板级平台的设备信息的board.dts。两者的区别主要是：前

者主要保存芯片相关的配置，不管芯片外围换什么硬件，芯片配置还是保持不变的。而后者是用于保存不同版型之间差异化的配置。
PMU 模块的dts 配置是在board.dts 中，dtsi 中无用户可用配置。

##### 2.3.1.1 board.dts 配置说明

board.dts 路径为：${ROOT_DIR}/device/config/chips/${PLATFORM}/configs/${TARGET}/board.dts
**说明**
	${ROOT_DIR}：是tina SDK 根目录
	${PLATFORM}：是芯片型号，如r818
	${TARGET}：是版级型号，如evb1

**技巧**
	board.dts 所在在版级配置目录，tina SDK 环境中，在source build/envsetup.sh 后，可通过“cconfigs” 命令直接跳转过去配置目录。
	board.dts 就在版本配置目录的上一级，再通过“cd ..” 即可到达board.dts 所在的目录。

PMU 一共包含了regulator，power supply，power key，PMU 在内核中的设备是多个设备同时存在，并存在层次对应关系。

```
PMU主设备(MFD)
|
+------> regulator device
|
+------> power key device
|
+------> power supply device
|
+------> wdt device
```

**说明**

​	AXP717 在内核中使用的是名为axp2202 的软件框架，因此在dts 文件中。型号采用软件框架的名字，而不配置为axp717。同理，在sysconfig.fex 和kernel 

​	menuconfig 中也一样。

```
pmu0: pmu@34 {
    compatible = "x-powers,axp2202";
    reg = <0x34>;
    interrupts = <0 IRQ_TYPE_LEVEL_LOW>;
    interrupt-parent = <&nmi_intc>;
    x-powers,drive-vbus-en;
    pmu_reset = <0>;
    pmu_irq_wakeup = <1>;
    pmu_hot_shutdown = <1>;
    wakeup-source;
    //interrupt-controller;
    //#interrupt-cells = <1>;
    usb_power_supply: usb_power_supply {
        compatible = "x-powers,axp2202-usb-power-supply";
        pmu_usbpc_vol = <4600>;
        pmu_usbpc_cur = <500>;
        pmu_usbad_vol = <4000>;
        pmu_usbad_cur = <2500>;
        pmu_boost_vol = <5126>;
        pmu_bc12_en;
        pmu_cc_logic_en = <1>;
        /* pmu_boost_en; */
        pmu_usb_typec_used = <1>;
        wakeup_usb_in;
        wakeup_usb_out;
        status = "okay";
    };
    /* cvin */
    gpio_power_supply: gpio_power_supply {
        compatible = "x-powers,gpio-supply";
        status = "disabled";
        wakeup_gpio;
    };
    bat_power_supply: bat-power-supply {
        compatible = "x-powers,axp2202-bat-power-supply";
        param = <&axp2202_parameter>;
        status = "okay";
        pmu_chg_ic_temp = <1>;
        pmu_battery_rdc= <147>;
        pmu_battery_cap = <1771>;
        pmu_runtime_chgcur = <1000>;
        pmu_suspend_chgcur = <1500>;
        pmu_shutdown_chgcur = <1500>;
        pmu_terminal_chgcur = <128>;
        pmu_init_chgvol = <4200>;
        pmu_battery_warning_level1 = <15>;
        pmu_battery_warning_level2 = <0>;
        pmu_chgled_func = <0>;
        pmu_chgled_type = <0>;
        pmu_bat_para1 = <0>;
        pmu_bat_para2 = <0>;
        pmu_bat_para3 = <0>;
        pmu_bat_para4 = <0>;
        pmu_bat_para5 = <0>;
        pmu_bat_para6 = <0>;
        pmu_bat_para7 = <2>;
        pmu_bat_para8 = <3>;
        pmu_bat_para9 = <4>;
        pmu_bat_para10 = <6>;
        pmu_bat_para11 = <9>;
        pmu_bat_para12 = <14>;
        pmu_bat_para13 = <26>;
        pmu_bat_para14 = <38>;
        pmu_bat_para15 = <49>;
        pmu_bat_para16 = <52>;
        pmu_bat_para17 = <56>;
        pmu_bat_para18 = <60>;
        pmu_bat_para19 = <64>;
        pmu_bat_para20 = <70>;
        pmu_bat_para21 = <77>;
        pmu_bat_para22 = <83>;
        pmu_bat_para23 = <87>;
        pmu_bat_para24 = <90>;
        pmu_bat_para25 = <95>;
        pmu_bat_para26 = <99>;
        pmu_bat_para27 = <99>;
        pmu_bat_para28 = <100>;
        pmu_bat_para29 = <100>;
        pmu_bat_para30 = <100>;
        pmu_bat_para31 = <100>;
        pmu_bat_para32 = <100>;
        pmu_bat_temp_enable = <0>;
        pmu_bat_charge_ltf = <1105>;
        pmu_bat_charge_htf = <121>;
        pmu_bat_shutdown_ltf = <1381>;
        pmu_bat_shutdown_htf = <89>;
        pmu_bat_temp_para1 = <2814>;
        pmu_bat_temp_para2 = <2202>;
        pmu_bat_temp_para3 = <1737>;
        pmu_bat_temp_para4 = <1381>;
        pmu_bat_temp_para5 = <1105>;
        pmu_bat_temp_para6 = <890>;
        pmu_bat_temp_para7 = <722>;
        pmu_bat_temp_para8 = <484>;
        pmu_bat_temp_para9 = <332>;
        pmu_bat_temp_para10 = <233>;
        pmu_bat_temp_para11 = <196>;
        pmu_bat_temp_para12 = <166>;
        pmu_bat_temp_para13 = <141>;
        pmu_bat_temp_para14 = <121>;
        pmu_bat_temp_para15 = <89>;
        pmu_bat_temp_para16 = <66>;
        wakeup_bat_out;
        /* wakeup_bat_in; */
        /* wakeup_bat_charging; */
        /* wakeup_bat_charge_over; */
        /* wakeup_low_warning1; */
        /* wakeup_low_warning2; */
        /* wakeup_bat_untemp_work; */
        /* wakeup_bat_ovtemp_work; */
        /* wakeup_bat_untemp_chg; */
        /* wakeup_bat_ovtemp_chg; */
    };
    powerkey0: powerkey@0 {
        compatible = "x-powers,axp2101-pek";
        pmu_powkey_off_time = <6000>;
        pmu_powkey_off_func = <0>;
        pmu_powkey_off_en = <1>;
        pmu_powkey_long_time = <1500>;
        pmu_powkey_on_time = <512>;
        wakeup_rising;
        wakeup_falling;
        status = "okay";
    };
    regulator0: regulators@0 {
        reg_dcdc1: dcdc1 {
            regulator-name = "axp2202-dcdc1";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <1540000>;
            regulator-ramp-delay = <2500>;
            regulator-enable-ramp-delay = <1000>;
            regulator-boot-on;
            regulator-always-on;
        };
        reg_dcdc2: dcdc2 {
            regulator-name = "axp2202-dcdc2";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <3400000>;
            regulator-ramp-delay = <2500>;
            regulator-enable-ramp-delay = <1000>;
            regulator-boot-on;
            regulator-always-on;
        };
        reg_dcdc3: dcdc3 {
            regulator-name = "axp2202-dcdc3";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <1840000>;
            regulator-ramp-delay = <2500>;
            regulator-enable-ramp-delay = <1000>;
            regulator-always-on;
        };
        reg_dcdc4: dcdc4 {
            regulator-name = "axp2202-dcdc4";
            regulator-min-microvolt = <1000000>;
            regulator-max-microvolt = <3700000>;
            regulator-ramp-delay = <2500>;
            regulator-enable-ramp-delay = <1000>;
        };
        reg_rtcldo: rtcldo {
            /* RTC_LDO is a fixed, always-on regulator */
            regulator-name = "axp2202-rtcldo";
            regulator-min-microvolt = <1800000>;
            regulator-max-microvolt = <1800000>;
            regulator-boot-on;
            regulator-always-on;
        };
        reg_aldo1: aldo1 {
            regulator-name = "axp2202-aldo1";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <3500000>;
            regulator-enable-ramp-delay = <1000>;
        };
        reg_aldo2: aldo2 {
            regulator-name = "axp2202-aldo2";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <3500000>;
            regulator-enable-ramp-delay = <1000>;
        };
        reg_aldo3: aldo3 {
            regulator-name = "axp2202-aldo3";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <3500000>;
            regulator-enable-ramp-delay = <1000>;
            regulator-always-on;
            regulator-boot-on;
        };
        reg_aldo4: aldo4 {
            regulator-name = "axp2202-aldo4";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <3500000>;
            regulator-enable-ramp-delay = <1000>;
            regulator-always-on;
            regulator-boot-on;
        };
        reg_bldo1: bldo1 {
            regulator-name = "axp2202-bldo1";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <3500000>;
            regulator-enable-ramp-delay = <1000>;
        };
        reg_bldo2: bldo2 {
            regulator-name = "axp2202-bldo2";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <3500000>;
            regulator-enable-ramp-delay = <1000>;
            regulator-boot-on;
            regulator-always-on;
        };
        reg_bldo3: bldo3 {
            regulator-name = "axp2202-bldo3";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <3500000>;
            regulator-enable-ramp-delay = <1000>;
        };
        reg_bldo4: bldo4 {
            regulator-name = "axp2202-bldo4";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <3500000>;
            regulator-enable-ramp-delay = <1000>;
        };
        reg_cldo1: cldo1 {
            regulator-name = "axp2202-cldo1";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <3500000>;
            regulator-enable-ramp-delay = <1000>;
        };
        reg_cldo2: cldo2 {
            regulator-name = "axp2202-cldo2";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <3500000>;
            regulator-enable-ramp-delay = <1000>;
        };
        reg_cldo3: cldo3 {
            regulator-name = "axp2202-cldo3";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <3500000>;
            regulator-ramp-delay = <2500>;
            regulator-enable-ramp-delay = <1000>;
            regulator-boot-on;
        };
        reg_cldo4: cldo4 {
            regulator-name = "axp2202-cldo4";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <3500000>;
            regulator-enable-ramp-delay = <1000>;
        };
        reg_cpusldo: cpusldo {
            /* cpus */
            regulator-name = "axp2202-cpusldo";
            regulator-min-microvolt = <500000>;
            regulator-max-microvolt = <1400000>;
            regulator-boot-on;
            regulator-always-on;
        };
        reg_drivevbus: drivevbus {
            regulator-name = "axp2202-drivevbus";
            regulator-enable-ramp-delay = <1000>;
        };
        };
        virtual-dcdc1 {
            compatible = "xpower-vregulator,dcdc1";
            dcdc1-supply = <&reg_dcdc1>;
        };
        virtual-dcdc2 {
        compatible = "xpower-vregulator,dcdc2";
        dcdc2-supply = <&reg_dcdc2>;
        };
        virtual-dcdc3 {
            compatible = "xpower-vregulator,dcdc3";
            dcdc3-supply = <&reg_dcdc3>;
        };
        virtual-dcdc4 {
            compatible = "xpower-vregulator,dcdc4";
            dcdc4-supply = <&reg_dcdc4>;
        };
        virtual-rtcldo {
            compatible = "xpower-vregulator,rtcldo";
            rtcldo-supply = <&reg_rtcldo>;
        };
        virtual-aldo1 {
            compatible = "xpower-vregulator,aldo1";
            aldo1-supply = <&reg_aldo1>;
        };
        virtual-aldo2 {
            compatible = "xpower-vregulator,aldo2";
            aldo2-supply = <&reg_aldo2>;
        };
        virtual-aldo3 {
            compatible = "xpower-vregulator,aldo3";
            aldo3-supply = <&reg_aldo3>;
        };
        virtual-aldo4 {
            compatible = "xpower-vregulator,aldo4";
            aldo4-supply = <&reg_aldo4>;
        };
        virtual-bldo1 {
            compatible = "xpower-vregulator,bldo1";
            bldo1-supply = <&reg_bldo1>;
        };
        virtual-bldo2 {
            compatible = "xpower-vregulator,bldo2";
            bldo2-supply = <&reg_bldo2>;
        };
        virtual-bldo3 {
            compatible = "xpower-vregulator,bldo3";
            bldo3-supply = <&reg_bldo3>;
        };
        virtual-bldo4 {
            compatible = "xpower-vregulator,bldo4";
            bldo4-supply = <&reg_bldo4>;
        };
        virtual-cldo1 {
            compatible = "xpower-vregulator,cldo1";
            cldo1-supply = <&reg_cldo1>;
        };
        virtual-cldo2 {
            compatible = "xpower-vregulator,cldo2";
            cldo2-supply = <&reg_cldo2>;
        };
        virtual-cldo3 {
            compatible = "xpower-vregulator,cldo3";
            cldo3-supply = <&reg_cldo3>;
        };
        virtual-cldo4 {
            compatible = "xpower-vregulator,cldo4";
            cldo4-supply = <&reg_cldo4>;
        };
        virtual-cpusldo {
            compatible = "xpower-vregulator,cpusldo";
            cpusldo-supply = <&reg_cpusldo>;
        };
        virtual-drivevbus {
            compatible = "xpower-vregulator,drivevbus";
            drivevbus-supply = <&reg_drivevbus>;
        };
        axp_gpio0: axp_gpio@0 {
            gpio-controller;
            #size-cells = <0>;
            #gpio-cells = <6>;
            status = "okay";
            };
        };
        /{
        axp2202_parameter:axp2202-parameter {
            select = "battery-model";
            battery-model {
                parameter = /bits/ 8 <0x01 0xf5 0x40 0x00 0x1b 0x1e 0x28 0x0f
                0x0c 0x1e 0x32 0x02 0x14 0x05 0x0a 0x04
                0x74 0xfb 0xc8 0x0d 0x43 0x10 0x36 0xfb
                0x46 0x01 0xea 0x0d 0x2a 0x06 0x36 0x05
                0xf4 0x0a 0xb5 0x0f 0x42 0x0e 0xe6 0x09
                0x9a 0x0e 0x42 0x0e 0x3b 0x04 0x2d 0x04
                0x23 0x09 0x18 0x0e 0x09 0x0e 0x04 0x08
                0xf7 0x0d 0xda 0x0d 0xd0 0x03 0xbb 0x03
                0x9d 0x08 0x7f 0x0d 0x6a 0x0d 0x55 0x07
                0xc2 0x57 0x2b 0x27 0x1e 0x0d 0x14 0x08
                0xc5 0x98 0x7e 0x66 0x4e 0x44 0x38 0x1a
                0x12 0x0a 0xf6 0x00 0x00 0xf6 0x00 0xf6
                0x00 0xfb 0x00 0x00 0xfb 0x00 0x00 0xfb
                0x00 0x00 0xf6 0x00 0x00 0xf6 0x00 0xf6
                0x00 0xfb 0x00 0x00 0xfb 0x00 0x00 0xfb
                0x00 0x00 0xf6 0x00 0x00 0xf6 0x00 0xf6>;
    	};
    };
};
```

**说明**
	PMU 为I2C 设备，PMU 设备配置需要写在i2c 节点内。

• PMU 属性配置

```
reg <u32>
	i2c寄存器地址
interrupts <args>
	中断配置，参考内核中断配置文档
interrupt-parent <phandler>
	上级中断控制器结点
wakeup-source <bool>
	是否作为唤醒源
    0:disable
    1:enable
x-powers,drive-vbus-en <bool>
	set N_VBUSEN pin as an output pin to control an external regulator to drive VBus
pmu_reset <bool>
    when power key press longer than 16s, PMU reset or not.
    0: not reset
    1: reset
pmu_irq_wakeup
    press irq wakeup or not when sleep or power down.
    0: not wakeup
    1: wakeup
pmu_hot_shutdown
    when PMU over temperature protect or not.
    0: disable
    1: enable
```

• power_supply 配置

power supply 属性配置，包括usb-power-supply 、gpio-power-supply 和battery-powersupply。

对于usb-power-supply 属性配置如下:

```
pmu_usbpc_vol <u32>
usb pc输入电压限制值，单位为mV
pmu_usbpc_cur <u32>
usb pc输入电流限制值，单位为mA
pmu_usbad_vol <u32>
usb adaptor输入电压限制值(vimdpm)，单位为mV
pmu_usbad_cur <u32>
usb adaptor输入电流限制值，单位为mA
pmu_boost_vol <u32>
打开boost给usb口供电的电压值，单位为mV
pmu_bc12_en <bool>
是否打开BC1.2协议功能
pmu_cc_logic_en <bool>
是否打开cc协议功能
pmu_boost_en <bool>
是否在初始化时打开boost功能
pmu_usb_typec_used <bool>
usb接口为type-c
wakeup_usb_in <bool>
usb插入唤醒使能
wakeup_usb_out <bool>
usb拔出唤醒使能
```

**说明**

在使用type-c 时，还需将usb 驱动中的“usb_detect_type” 节点同步配置成2，才能使用typec 识别usb 设备的功能。

对于gpio-power-supply：是控制ac in 的配置，在AXP717 上默认不使用。

对于battery-power-supply 属性配置如下：

```
param <string>
电池参数，与axp2202_parameter对应
pmu_chg_ic_temp <u32>
1: TS current source always on
0: TS current source off
pmu_battery_rdc <u32>
电池内阻，单位为mΩ
pmu_battery_cap <u32>
电池容量，单位为mAh
pmu_runtime_chgcur <u32>
运行时constant充电电流限制，单位为mA
pmu_suspend_chgcur <u32>
休眠时constant充电电流限制，单位为mA
pmu_shutdown_chgcur <u32>
关机时constant充电电流限制，单位为mA
pmu_terminal_chgcur <u32>
截止电流，停止充电的标志位之一，单位为mA
pmu_init_chgvol <u32>
电池满充电压，单位为mV
pmu_battery_warning_level1 <u32>
5-20 5% - 20% warning level1
电池低电量警告，当芯片检测电池电量从高到低跌到了设置的level1的值，capacity < warning_level1，就会触
发warning_level1中断，从而再进行对应操作。
如果capacity ≥ warning_level1则会清掉该中断。
如AXP717，当触发warning1中断时，默认是不发生操作，需要修改对应代码进行定制化操作。
pmu_battery_warning_level2 <u32>
0-15 0% - 15% warning level2
意义同level1，当电池电量从从高到低跌落，capacity < warning_level2，就会触发warning_level2中断。
当电池电量warning_level2 ≤ capacity < warning_level1 则会清掉level2中断。
pmu_chgled_func <u32>;
CHGKED pin control
0: controlled by pmu
1: controlled by Charger
pmu_chgled_type <u32>
CHGLED Type select when pmu_chgled_func is 0
0: display with type A function
1: display with type B function
3: output controlled by the register of chgled_out_ctrl
pmu_bat_para1 <u32>
pmu_bat_para2 <u32>
...
pmu_bat_para32 <u32>
电池曲线参数
**电池参数根据使用的电池不同，通过仪器测量出来**
pmu_bat_temp_enable <u32>
设置电池温度检测、ntc是否使能
pmu_bat_charge_ltf <u32>
触发电池低温停充的TS pin电压阈值，单位：mV
默认：1105mV
范围：0-8160mV
pmu_bat_charge_htf <u32>
触发电池高温停充的TS pin电压阈值，单位：mV
默认：121mV
范围：0-510mV
pmu_bat_shutdown_ltf <u32>
非充电模式下，触发电池低温中断的TS pin电压阈值，单位：mV
默认：1381mV
pmu_bat_shutdown_htf <u32>
默认：89mV
范围：0-510mV
pmu_bat_temp_para1 <u32>
电池包-25度对应的TS pin电压，单位：mV
pmu_bat_temp_para2 <u32>
电池包-15度对应的TS pin电压，单位：mV
pmu_bat_temp_para3 <u32>
电池包-10度对应的TS pin电压，单位：mV
pmu_bat_temp_para4 <u32>
电池包-5度对应的TS pin电压，单位：mV
pmu_bat_temp_para5 <u32>
电池包0度对应的TS pin电压，单位：mV
pmu_bat_temp_para6 <u32>
电池包5度对应的TS pin电压，单位：mV
pmu_bat_temp_para7 <u32>
电池包10度对应的TS pin电压，单位：mV
pmu_bat_temp_para8 <u32>
电池包20度对应的TS pin电压，单位：mV
pmu_bat_temp_para9 <u32>
电池包30度对应的TS pin电压，单位：mV
pmu_bat_temp_para10 <u32>
电池包40度对应的TS pin电压，单位：mV
pmu_bat_temp_para11 <u32>
电池包45度对应的TS pin电压，单位：mV
pmu_bat_temp_para12 <u32>
电池包50度对应的TS pin电压，单位：mV
pmu_bat_temp_para13 <u32>
电池包55度对应的TS pin电压，单位：mV
pmu_bat_temp_para14 <u32>
电池包60度对应的TS pin电压，单位：mV
pmu_bat_temp_para15 <u32>
电池包70度对应的TS pin电压，单位：mV
pmu_bat_temp_para16 <u32>
电池包80度对应的TS pin电压，单位：mV
**不同电池包的温敏电阻特性不一样，根据电池包的TS温敏电阻手册，找到pmu_bat_temp_para[1-16]对应温度点
的电阻阻值，将阻值除以20得到的电压数值（单位：mV），将电压数值填进pmu_bat_temp_para[1-16]的节点中即
可**
wakeup_bat_out <bool>
电池拔出唤醒使能
wakeup_bat_charging <bool>
电池充电唤醒使能
wakeup_bat_charge_over <bool>
电池充电结束唤醒使能
wakeup_low_warning1 <bool>
电池低电量告警唤醒使能
wakeup_low_warning2 <bool>
电池低电量告警2唤醒使能
wakeup_bat_untemp_chg <bool>
电池低温充电唤醒使能
wakeup_bat_ovtemp_chg <bool>
电池超温充电唤醒使能
wakeup_bat_untemp_work <bool>
电池低温工作唤醒使能
wakeup_bat_ovtemp_work <bool>
电池高温工作唤醒使能
```

• power key 属性配置

power key 设备为按键设备，具体的说为电源按键设备。power key 属性配置：

```
pmu_powkey_off_time <u32>
控制按下多长时间响应poweroff事件
可选的值为：
4000 4s
6000 6s
8000 8s
10000 10s
pmu_powkey_off_func <u32>
控制power_off事件功能,如果不配置，默认为power-off
0：power_off
1：复位系统
pmu_powkey_off_en <bool>
控制按键关机使能
1：PWRON > OFFLEVEL AS poweroff source enable
0：PWRON > OFFLEVEL as poweroff source disable
pmu_powkey_long_time <u32>
控制ponlevel 寄存器0x27[5:4]
1000 1s
1500 1.5s
2000 2s
2500 2.5s
pmu_powkey_on_time <u32>
控制按钮按下多长时间开机
128 0.128s
512 0.512s
1000 1s
2000 2s
wakeup_rising <bool>
控制是否弹起按钮唤醒系统
wakeup_falling <bool>
控制是否按下按钮唤醒系统
```

• regulator 属性配置

regulator 为系统regulator_dev 设备，每个regulator_dev 代表一路电源，设备通过对regulator_dev 的引用建立regulator，用来实现对电源的电压设置等功能。

regulator 属性配置，参考内核原生regulator 使用文档：Documentation/devicetree/bindings/regulator/regulator.txt。

regulator 配置如下：

```
reg\_aldo1: aldo1{
    regulator-name = "axp2101-dcdc1";
    为电源设备的名称
    regulator-min-microvolt = <1500000>;
    电源的最小值，单位：uV
    regulator-max-microvolt = <3400000>;
    电源的最大值，单位：uV
    regulator-ramp-delay = <2500>;
    电源的调压延时，单位：us
    regulator-enable-ramp-delay = <1000>;
    电源从关闭到开启的使能延时，单位：us
    regulator-boot-on;
    电源从启动时开启，在内核启动时，就按照dts配置加载了该路regulator。同时，进入系统后通过软件读取也能
    获知该路regulator被加载了。
    regulator-always-on;
    电源保持常开，不会由于调用regulator等API接口而关闭。
};
```

##### 2.3.1.2 sys_config.fex 配置

在sysconfig 中定义了PMU 的regulator 输出信息及板型PMU 类型，在boot0 和uboot 会通过解析这部分属性来执行调压等操作。

```
;----------------------------------------------------------------------------------
;[target] system bootup configuration
;boot_clock = CPU boot frequency, Unit: MHz
;storage_type = boot medium, 0-nand, 1-card0, 2-card2, -1(defualt)auto scan
;advert_enable = 0-close advert logo 1-open advert logo (只有多核启动下有效)
;power_mode = axp_type, 0:axp81X, 1:dummy, 2:axp806, 3:axp2202, 4:axp858
;----------------------------------------------------------------------------------
[target]
boot_clock = 1008
storage_type = -1
advert_enable = 0
burn_key = 1
dragonboard_test= 0
power_mode = 3
;----------------------------------------------------------------------------------
; system configuration
; ?
;dcdc1_vol ---set dcdc1 voltage,mV
,500-1200,10mV/step
;
1220-3400,20mV/step
;dcdc2_vol ---set dcdc2 voltage,mV
,500-1200,10mV/step
;
1220-1540,20mV/step
;aldo1_vol ---set aldo1 voltage,mV
,500-3500,100mV/step
;dldo1_vol ---set dldo1 voltage,mV
,500-3500,100mV/step
;----------------------------------------------------------------------------------
[power_sply]
dcdc3_vol = 1001200
aldo3_vol = 1003300
aldo4_vol = 1001800
bldo2_vol = 1002500
cldo1_vol = 1001800
cldo3_vol = 1003300
cpusldo_vol = 100900
dcdc1_mode = 1
dcdc2_mode = 1
;----------------------------------------------------------------------------------
; gpio_bias
; set gpio group withstand voltage
; pc_bias = 1800 is emmc
; pc_bias = 3300 is nand
;----------------------------------------------------------------------------------
[gpio_bias]
device_type = "gpio_bias"
pl_bias = 3300
pl_supply = "aldo3_vol"
pc_bias = 1800
pc_supply = "cldo1_vol"
;pc_bias = 3300
;pc_supply = "cldo3_vol"
[power_delay]
device_type = "power_delay"
aldo3_vol_delay = 20000
```

下面将按照不同模块来解析sys_config 中各个模块的配置含义。

• target 属性配置：

在此配置下，与PMU 相关的主要是power_mode。power mode 这个节点就是为了告诉平台当前使用的是哪个PMU 的哪种方案（有时在同一SOC 平台同一PMU 

也可能出现电源树配置不一样的情况），在boot0 阶段就会解析出该属性并调用调压接口进行调压。

```
power mode <u32>
属性决定了当前SOC板型使用哪个PMU，需boot0代码支持解析，目前仅R818/MR813方案支持解析该节点。后续别的平台需支持时应根据boot0代码更新sysconfig中的版型选择说明。
    0:axp81X,
    1: dummy
    2: axp806
    3: axp2202
    4: axp858
```

**说明**
AXP717 与其他PMU 使用同一套代码框架，使用AXP717 则选上3: axp2202即可。

• power_sply 属性配置

```
xxxx_vol <u32>
uboot阶段xxxx这路电是否开关及输出电压配置，其中xxxx为供电输出名。属性由前缀(100/000)和后缀组成。未配
置的电在uboot阶段不会进行开关电和调压操作。
前缀： 100，这路电在uboot阶段打开
前缀： 110，这路电在uboot阶段打开，但是烧写时会关闭
前缀： 000，这路电在uboot阶段关闭
后缀： 3300，这路电输出电压设置为3300 mV
dcdcx_mode <u32>
uboot阶段强制将dcdcx设置为fpwm开关模式，提高这路抗负载扰动能力。该属性不配置默认为0。目前仅AXP806/
AXP305/AXP81X/AXP803/AXP2202/AXP717支持该功能。
0： pfm-pwm模式自由切换
1： 强制pwm模式
battery_exist <u32>
强制电池存在状态，uboot阶段根据该属性决定是否做电池状态的相关判断。如果该属性不进行配置，默认为1。适用于
部分无电持方案或factory_mode的无电池场景的调试
0： 强制认为无电池存在，uboot阶段不做电池相关状态判断
1： 认为电池存在，uboot阶段正常进行电池状态的相关判断
charge_mode <u32>
配置充电页面，根据该属性决定是否进入关机充电页面。
如果该属性不进行配置，默认为1。
适用于不需要充电页面，或者适配器唤醒直接开机的需求。
0： 不进入充电页面，适配器唤醒直接进入开机流程
1： 适配器唤醒进入充电流程
```

• power_delay 属性配置

```
xxxx_vol_delay <u32>
uboot阶段xxxx这路电调压后的延时时间，单位us。用于部分调压后需等电压稳定才能操作的模块。
如：twi，在上电时twi的电压不是3.3V，在uboot阶段需要将其升至3.3V，在电压抬升阶段是不能允许twi进行通信，不然就会产生错误。因此需要在调整电压的时候停止twi的功能。
该属性需与**power_sply**中的xxx_vol属性一块使用，当输出需要开关或调压时才需要进行延时。
```

• gpio_bias 属性配置

```
xx_bias <u32>
GPIOx口的耐压值设置，单位：mV。用于调整GPIOx口的耐压值，使其与GPIOx模块挂载的电压匹配，避免IO口损坏，提升信号质量。
xx_supply <char>
GPIOx模块挂载的输出电压名，名字格式需与**power_sply**中的xxx_vol一致。该属性如果配上，在GPIOx挂载的输出改编后，会将对应的GPIOx bias耐压值修改过来。
```

#### 2.3.2 kernel menuconfig 配置说明

在Tina SDK 根目录运行make kernel_menuconfig，进行内核配置修改，进入配置界面按以下步骤进行修改。

AXP717 与AXP2101 公用同一份控制器、按键以及regularotr 的代码，因此在kernel_menuconfig 里选的是AXP2101 的选项。

• PMU 控制器

```
-> Device Drivers
    -> Multifunction device drivers
    	<*> X-Powers AXP2101 PMICs with I2C
```

![Tina_Linux_PMU_Development_Guide-image-20221229171742892](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_PMU_Development_Guide-image-20221229171742892.png)

<center>图2-1: pmu-control-config</center>

• regulator

```
-> Device Drivers
    -> Voltage and Current Regulator Support
    	<*> X-POWERS AXP2101 PMIC Regulators
```

![Tina_Linux_PMU_Development_Guide-image-20221229171817733](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_PMU_Development_Guide-image-20221229171817733.png)

<center>图2-2: regularot-config</center>

• charger

```
-> Device Drivers
-> Power supply class support
<*> AXP2202 power supply driver
```

![Tina_Linux_PMU_Development_Guide-image-20221229171855027](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_PMU_Development_Guide-image-20221229171855027.png)

<center>图2-3: charger-config</center>

• power key

```
-> Device Drivers
    -> Input device support
        -> Miscellaneous devices
            <*> X-Powers AXP2101 power button driver
```

![Tina_Linux_PMU_Development_Guide-image-20221229171925367](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_PMU_Development_Guide-image-20221229171925367.png)

<center>图2-4: power-key-config</center>

• virtual regulator

```
-> Device Drivers
    -> Voltage and Current Regulator Support
    	<*> Virtual regulator consumer support
```

![Tina_Linux_PMU_Development_Guide-image-20221229172053147](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_PMU_Development_Guide-image-20221229172053147.png)

<center>图2-5: virtuaal-config</center>

• acin

```
-> Device Drivers
    -> Power supply class support
        < > AXP2202 power virtual acin
```

![Tina_Linux_PMU_Development_Guide-image-20221229172125375](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_PMU_Development_Guide-image-20221229172125375.png)

<center>图2-6: acin-config</center>

### 2.4 源码结构介绍

• AXP717

```
${ROOT_DIR}/lichee/{KERNEL_VERSION}/
drivers/mfd/axp2101.c
drivers/mfd/axp2101-i2c.c
drivers/regulator/axp2101-regulator.c
drivers/input/misc/axp2101-pek.c
drivers/power/supply/axp2202_battery.c
drivers/power/supply/axp2202_charger.c
drivers/power/supply/axp2202_gpio_power.c
drivers/power/supply/axp2202_usb_power.c
```

### 2.5 模块框架介绍

AXP 的多功能设备驱动采用i2c 总线跟主控进行交互，使用regmap 方式注册访问接口。将AXP按照功能抽象出数个子设备模块，共同使用父设备axp mfd 的资源

（bus、irq）。基本的软件结构图如下图所示。

![Tina_Linux_PMU_Development_Guide-image-20221229172218801](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_PMU_Development_Guide-image-20221229172218801.png)

<center>图2-7: AXP 框架图</center>

将axp 按照功能划分为几个子设备，分别是regulator、charger、powe key、gpio。每个子设备作为一个cell，使用父设备的资源（bus，irq），与不同的内核子

系统交互，实现完整的电源管理功能。

![Tina_Linux_PMU_Development_Guide-image-20221229172322260](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_PMU_Development_Guide-image-20221229172322260.png)

<center>图2-8: AXP 软件框架图</center>

不同款的AXP 型号功能框架也会是不一致的，详情各个型号的功能列表如下表所示。

| 产品名称 | regulator | charger | power key | gpio |
| -------- | --------- | ------- | --------- | ---- |
| AXP717   | 有        | 有      | 有        | 无   |