## 2 模块介绍

### 2.1 模块功能介绍

Linux 提供了 MMC 子系统来实现对各种 SD/MMC/EMMC/SDIO 设备访问，MMC 子系统由上到下可以分为三层，MMC/SD card 层，MMC/SD core 层以及 MMC/SD host 层，它们之间的层次关系如下所示。

MMC/SD card 层负主要是按照 LINUX 块设备驱动程序的框架实现一个卡的块设备驱动。负责块设备请求的处理，以及请求队列的管理。

MMC/SD core 层负责通信协议的处理，包括 SD/MMC/eMMC/SDIO，为上一层提供具体读写接口，同时为下一层提供 host 端接口。

MMC/SD host 层是实现对 SD/MMC 控制器相关的操作，直接操作硬件，也是主要实现部分。



### 2.2 相关术语介绍

#### 2.2.1 硬件术语

| 术语  | 解释说明                             |
| ----- | ------------------------------------ |
| Sunxi | 指 Allwinner 的一系列 SOC 硬件平台。 |
| SD    | Secure Digital Memory Card           |
| MMC   | Multimedia Card                      |
| eMMC  | Embedded MultiMediaCard              |
| host  | 指具体的 SD/MMC 控制器               |



#### 2.2.2 软件术语

无



### 2.3 模块配置介绍

#### 2.3.1 sys_config.fex 配置说明

**[card0_boot_para]**

```
card_ctrl = 0
card_high_speed = 1
card_line = 4
sdc_d1 = port:PF0<2><1><3><default>
sdc_d0 = port:PF1<2><1><3><default>
sdc_clk = port:PF2<2><1><3><default>
sdc_cmd = port:PF3<2><1><3><default>
sdc_d3 = port:PF4<2><1><3><default>
sdc_d2 = port:PF5<2><1><3><default>
```

各个配置项的意义如下：

| **配置项**      | **配置项含义**               |
| --------------- | ---------------------------- |
| card_ctrl       | 0：选择卡量产相关的控制器    |
| card_high_speed | 速度模式 0 为低速，1 为高速  |
| card_line       | 代表卡总线宽度，分别有 1,4,8 |
| sdc_d1          | sdc data1 的 GPIO 配置       |
| sdc_d0          | sdc data0 的 GPIO 配置       |
| sdc_clk         | sdc clk 的 GPIO 配置         |
| sdc_cmd         | sdc cmd 的 GPIO 配置         |
| sdc_d3          | sdc data3 的 GPIO 配置       |
| sdc_d2          | sdc data2 的 GPIO 配置       |



**[card2_boot_para]**

```
card_ctrl = 2
card_high_speed = 1
card_line = 8
sdc_clk = port:PC5<3><1><3><default>
sdc_cmd = port:PC6<3><1><3><default>
sdc_d0 = port:PC10<3><1><3><default>
sdc_d1 = port:PC13<3><1><3><default>
sdc_d2 = port:PC15<3><1><3><default>
sdc_d3 = port:PC8<3><1><3><default>
sdc_d4 = port:PC9<3><1><3><default>
sdc_d5 = port:PC11<3><1><3><default>
sdc_d6 = port:PC14<3><1><3><default>
sdc_d7 = port:PC16<3><1><3><default>
sdc_emmc_rst = port:PC1<3><1><3><default>
sdc_ds = port:PC0<3><2><3><default>
sdc_ex_dly_used = 2
sdc_io_1v8 = 1
```



各个配置项的意义如下：

| 配置项                | 配置项含义                                                   |
| --------------------- | ------------------------------------------------------------ |
| card_ctrl             | 0：选择卡量产相关的控制器                                    |
| card_high_speed       | 速度模式 0 为低速，1 为高速                                  |
| card_line             | 代表卡总线宽度，分别有 1,4,8                                 |
| sdc_clk               | sdc clk 的 GPIO 配置                                         |
| sdc_cmd               | sdc cmd 的 GPIO 配置                                         |
| sdc_d0                | sdc data0 的 GPIO 配置                                       |
| sdc_d1                | sdc data1 的 GPIO 配置                                       |
| sdc_d3                | sdc data3 的 GPIO 配置                                       |
| sdc_d4                | sdc data4 的 GPIO 配置                                       |
| sdc_d5                | sdc data5 的 GPIO 配置                                       |
| sdc_d6                | sdc data6 的 GPIO 配置                                       |
| sdc_d7                | sdc data7 的 GPIO 配置                                       |
| sdc_emmc_rst          | emmc 复位信号的 GPIO 配置                                    |
| sdc_ds                | sdc ds 线的 GPIO 配置                                        |
| sdc_ex_dly_used       | 采样模式控制，2: tune 采样点；1：固定采样点方式，烧写阶段和启动阶段，通过 sys_config 配置采样点；其它值：烧写阶段和启动阶段使用预设的采样点，通常用 2，不建议修改 |
| sdc_io_1v8            | 1: 表示 eMMC IO 电平是 1.8V ，需要根据实际 emmc io 供电配置  |
| sdc_force_boot_tuning | 1: 强制启动 tuning                                           |
| sdc_tm4_win_th        | Tune 采样最小可选窗口                                        |
| sdc_dis_host_caps     | 禁止某一种速度模式，bit1：Host 不支持 HS-SDR；Bit3：Host不支持 8 线模式；bit6:Host 不支持 HS-DDR;bit7:Host 不支持HS200;bit8:Host 不支持 HS400；其他值，不建议使用 |



#### 2.3.2 Device Tree 配置说明

##### 2.3.2.1 1.uboot 阶段

放到板级目录下面：uboot-board.dts



##### 2.3.2.1.1（1）sdc0

```
&sdc0_pins_a {
    allwinner,pins = "PF0", "PF1", "PF2",
	    "PF3", "PF4", "PF5";
    allwinner,function = "sdc0";
    allwinner,muxsel = <2>;
    allwinner,drive = <3>;
    allwinner,pull = <1>;
};
&card0_boot_para {
    /* reg = <0x0 0x2 0x0 0x0>; */
    device_type = "card0_boot_para";
    card_ctrl = <0x0>;
    card_high_speed = <0x1>;
    card_line = <0x4>;
    pinctrl-0 = <&sdc0_pins_a>;
};
```

| 配置项          | 配置项含义                                                   |
| --------------- | ------------------------------------------------------------ |
| card_ctrl       | 0：选择卡量产相关的控制器                                    |
| card_high_speed | 速度模式 0 为低速，1 为高速                                  |
| card_line       | 代表卡总线宽度，分别有 1,4,8                                 |
| pinctrl-0       | 代表卡的 pin 设置                                            |
| sdc0_pins_a     | 具体卡的 pin 设置，allwinner,pins 代表具体的 pin 名字，allwinner,function 表示 pin 选择的功能，这里选择 sdc0，allwinner,muxsel 代表 sdc0 对应 spec 里面的功能值，allwinner,drive 代表去掉能力，allwinner,pull 代表上下拉 |



##### 2.3.2.1.2 （2）sdc2

```
&sdc2_pins_a {
    allwinner,pins = "PC1", "PC5", "PC6",
                    "PC8", "PC9", "PC10", "PC11",
                    "PC13", "PC14", "PC15", "PC16";
    allwinner,function = "sdc2";
    allwinner,muxsel = <3>;
    allwinner,drive = <3>;
    allwinner,pull = <1>;
};
&sdc2_pins_b {
    allwinner,pins = "PC0", "PC1", "PC5", "PC6",
                    "PC8", "PC9", "PC10", "PC11",
                    "PC13", "PC14", "PC15", "PC16";
    allwinner,function = "io_disabled";
    allwinner,muxsel = <7>;
    allwinner,drive = <1>;
    allwinner,pull = <1>;
};
&sdc2_pins_c {
    allwinner,pins = "PC0";
    allwinner,function = "sdc2";
    allwinner,muxsel = <3>;
    allwinner,drive = <3>;
    allwinner,pull = <2>;
};
&card2_boot_para {
    /*reg = <0x0 0x3 0x0 0x0>; */
    device_type = "card2_boot_para";
    card_ctrl = <0x2>;
    card_high_speed = <0x1>;
    card_line = <0x8>;
    pinctrl-0 = <&sdc2_pins_a &sdc2_pins_c>;
    sdc_ex_dly_used = <0x2>;
    sdc_io_1v8 = <0x1>;
    sdc_tm4_win_th = <0x08>;
    sdc_tm4_hs200_max_freq = <150>;
    sdc_tm4_hs400_max_freq = <100>;
    sdc_type = "tm4";
};
```

| 配置项                   | 配置项含义                                                   |
| ------------------------ | ------------------------------------------------------------ |
| card_ctrl                | 0：选择卡量产相关的控制器                                    |
| card_high_speed          | 速度模式 0 为低速，1 为高速                                  |
| card_line                | 代表卡总线宽度，分别有 1,4,8                                 |
| sdc_ex_dly_used          | 采样模式控制，2: tune 采样点；1：固定采样点方式，烧写阶段和启动阶段，通过 sys_config 配置采样点；其它值：烧写阶段和启动阶段使用预设的采样点，通常用 2 |
| sdc_io_1v8               | 1: 表示 eMMC IO 电平是 1.8V, 需要根据实际 emmc io 供电配置   |
| sdc_tm4_win_th           | Tune 采样最小可选窗口                                        |
| sdc_tm4_hs200_max_freq/  | 代表 emmc 的 hs200/hs400 最大频率设置, 不建议修改            |
| sdc_tm4_hs400_max_freq   |                                                              |
| sdc2_pins_a, sdc2_pins_c | 具体卡的 pin 设置，allwinner,pins 代表具体的 pin 名字，allwinner,function 表示 pin 选择的功能，这里选择 sdc0，allwinner,muxsel 代表sdc0 对应 spec 里面的功能值，allwinner,drive代表去掉能力，allwinner,pull 代表上下拉 |



##### 2.3.2.2 2. 内核阶段

存放在 board.dts 或者内核目录下面 arch/armXX/boot/dts/sunxi/sunxiXiwXpX 中在不同的 Sunxi 硬件平台中，SD/MMC 控制器的数目也不一定相同，但对于每一个 SD/MMC 控制器来说，在 board.dts 中配置的参数相似。各个项目的意义如下



##### 2.3.2.2.1 [sdc0] 通常用作 SD 卡

```
sdc0: sdmmc@04020000 {
    device_type = "sdc0";
    cd-used-24M;
    disable-wp;
    pinctrl-0 = <&sdc0_pins_a>;
    bus-width = <4>;
    cd-gpios = <&pio PF 6 6 1 3 0xffffffff>;
    /*non-removable;*/
    /*broken-cd;*/
    /*cd-inverted*/
    /*data3-detect;*/
    cap-sd-highspeed;
    sd-uhs-sdr50;
    sd-uhs-ddr50;
    sd-uhs-sdr104;
    no-sdio;
    no-mmc;
    sunxi-power-save-mode;
    /*sunxi-dis-signal-vol-sw;*/
    max-frequency = <150000000>;
    ctl-spec-caps = <0x8>;
    vmmc-supply = <&reg_dldo1>;
    vqmmc33sw-supply = <&reg_dldo1>;
    vdmmc33sw-supply = <&reg_dldo1>;
    vqmmc18sw-supply = <&reg_aldo1>;
    vdmmc18sw-supply = <&reg_aldo1>;
    status = "okay";
}；
```

各个配置项的意义如下：

| **配置项**              | 配置项含义                                        |
| ----------------------- | ------------------------------------------------- |
| device_type             | phy 索引值的选择                                  |
| cd-used-24M             | 使用 24M 时钟检测插拔卡中断                       |
| disable-wp              | 卡设置写保护，ro                                  |
| pinctrl-0               | 第一组 pin 脚的 GPIO 配置                         |
| bus-width               | bus-width                                         |
| cd-gpios                | 卡检测的 GPIO 配置                                |
| non-removable           | 不可移除                                          |
| broken-cd               | sd 卡检测方式：轮训                               |
| cd-inverted             | 卡检测的高电平有效还是低电平有效                  |
| data3-detect            | data3 线检测卡                                    |
| cap-sd-highspeed        | SD 卡的 High speed                                |
| sd-uhs-sdr50            | SD 卡的 uhs-sdr50                                 |
| sd-uhs-ddr50            | SD 卡的 uhs-ddr50                                 |
| sd-uhs-sdr104           | SD 卡的 uhs-sdr104                                |
| no-sdio                 | 无 sdio                                           |
| no-mmc                  | 无 mmc                                            |
| sunxi-power-save-mode   | 发送数据或者命令才有时钟输出                      |
| sunxi-dis-signal-vol-sw | 关闭电压切换                                      |
| max-frequency           | 最大频率                                          |
| ctl-spec-caps           | 控制 spec 能力                                    |
| vmmc-supply             | 供电电压、工作电压，需要根据实际 pmu 供电方案修改 |
| vqmmc33sw-supply        | 3.3V 的 IO 电压，需要根据实际 pmu 供电方案修改    |
| vdmmc33sw-supply        | 3.3V 的卡检测电压，需要根据实际 pmu 供电方案修改  |
| vqmmc18sw-supply        | 1.8V 的 IO 电压，需要根据实际 pmu 供电方案修改    |
| vdmmc18sw-supply        | 1.8V 的卡检测电压，需要根据实际 pmu 供电方案修改  |
| status                  | 设备树的状态                                      |



##### 2.3.2.2.2 [sdc1] 通常用作 SDIO WIFI

```
sdc1: sdmmc@04021000 {
    pinctrl-0 = <&sdc0_pins_a>;
    bus-width = <4>;
    cap-sd-highspeed;
    sd-uhs-sdr50;
    sd-uhs-ddr50;
    sd-uhs-sdr104;
    no-sd;
    no-mmc;
    /*sunxi-power-save-mode;*/
    /*sunxi-dis-signal-vol-sw;*/
    cap-sdio-irq;
    keep-power-in-suspend;
    ignore-pm-notify;
    max-frequency = <150000000>;
    ctl-spec-caps = <0x8>;
    sunxi-dly-208M = <1 1 0 0 0 1>;
    vmmc-supply = <&reg_dldo1>;
    vqmmc33sw-supply = <&reg_dldo1>;
    vdmmc33sw-supply = <&reg_dldo1>;
    vqmmc18sw-supply = <&reg_aldo1>;
    vdmmc18sw-supply = <&reg_aldo1>;
    status = "okay";
}；
```

各个配置项的意义如下：

| **配置项**              | **配置项含义**                                               |
| ----------------------- | ------------------------------------------------------------ |
| pinctrl-0               | 第一组 pin 脚的 GPIO 配置                                    |
| bus-width               | 线宽                                                         |
| cap-sd-highspeed        | SDIO 卡的 High speed                                         |
| sd-uhs-sdr50            | SDIO 卡的 uhs-sdr50                                          |
| sd-uhs-ddr50            | SDIO 卡的 uhs-ddr50                                          |
| sd-uhs-sdr104           | SDIO 卡的 uhs-sdr104                                         |
| no-sd                   | 无 sd                                                        |
| no-mmc                  | 无 mmc                                                       |
| sunxi-power-save-mode   | 发送数据或者命令才有时钟输出                                 |
| sunxi-dis-signal-vol-sw | 关闭电压切换                                                 |
| cap-sdio-irq            | 开启 SDIO 中断                                               |
| keep-power-in-suspend   | 休眠时保持电源                                               |
| ignore-pm-notify        | 忽略电源管理的通知                                           |
| max-frequency           | 最大频率                                                     |
| ctl-spec-caps           | 控制 spec 能力                                               |
| sunxi-dly-208M          | <1(cmd driver phase) 1(data driver phase) 00 0(data sample phase) 0(cmd sample phase)> |
| vmmc-supply             | 供电电压、工作电压，需要根据实际 pmu 供电方案修改            |
| vqmmc33sw-supply        | 3.3V 的 IO 电压，需要根据实际 pmu 供电方案修改               |
| vdmmc33sw-supply        | 3.3V 的卡检测电压，需要根据实际 pmu 供电方案修改             |
| vqmmc18sw-supply        | 1.8V 的 IO 电压，需要根据实际 pmu 供电方案修改               |
| vdmmc18sw-supply        | 1.8V 的卡检测电压，需要根据实际 pmu 供电方案修改             |
| status                  | 设备树的状态                                                 |





##### 2.3.2.2.3 [sdc2]  通常用作 eMMC

```
sdc2: sdmmc@04022000 {
    pinctrl-0 = <&sdc0_pins_a &sdc0_pins_c>;
    bus-width = <8>;
    non-removable;
    cap-mmc-highspeed;
    mmc-ddr-1_8v;
    mmc-hs200-1_8v;
    mmc-hs400-1_8v;
    no-sdio;
    no-sd;
    sunxi-power-save-mode;
    sunxi-dis-signal-vol-sw;
    max-frequency = <100000000>;
    ctl-spec-caps = <0x308>;
    vmmc-supply = <&reg_dldo1>;
    vqmmc-supply = <&reg_aldo1>;
    fixed-emmc-driver-type = <0x1>;
    sdc_tm4_sm0_freq0 = <0>;
    status = "disabled";
}；
```

各个配置项的意义如下：

| 配置项                  | 配置项含义                                                   |
| ----------------------- | ------------------------------------------------------------ |
| pinctrl-0               | 第一组 pin 脚的 GPIO 配置                                    |
| bus-width               | 线宽                                                         |
| non-removable           | 不可移除                                                     |
| cap-mmc-highspeed       | MMC 卡的 High speed                                          |
| mmc-ddr-1_8v            | MMC 卡的 ddr50                                               |
| mmc-hs200-1_8v          | MMC 卡的 hs200                                               |
| mmc-hs400-1_8v          | MMC 卡的 hs400                                               |
| no-sdio                 | 无 sdio                                                      |
| no-sd                   | 无 sd                                                        |
| sunxi-power-save-mode   | 发送数据或者命令才有时钟输出                                 |
| sunxi-dis-signal-vol-sw | 关闭电压切换                                                 |
| max-frequency           | 最大频率                                                     |
| vmmc-supply             | 供电电压、工作电压，需要根据实际 pmu 供电方案修改            |
| vqmmc-supply            | IO 电压，需要根据实际 pmu 供电方案修改                       |
| fixed-emmc-driver-type  | 驱动能力等级调整，对应设置 Extended CSD register 中 HS_TIMING |
| sdc_tm4_sm0_freqn       | host timing setting                                          |
| status                  | 设备树的状态                                                 |





#### 2.3.3 kernel menuconfig 配置说明

在命令行中进入内核 longan 根目录，执行./build.sh menuconfig 进入配置界面，并按以下步骤操作：

1.menuconfig 主界面

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxMMCDevelopmentGuide_001.png)

​																图 2-1: menuconfig 主界面

2. 进入 Device Drivers，并选中 MMC/SD/SDIO card support 为 “ * ”

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxMMCDevelopmentGuide_002.png)

​															图 2-2: Device drivers 界面

3. 选择 Allwinner sunxi SD/MMC Host Controller support 为 “ * ”，编译进内核（M 为编译进模块）

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxMMCDevelopmentGuide_003.png)

​															图 2-3: sdmmc 支持界面

2.4 源码结构介绍

SD/MMC 总线驱动的源代码位于内核在 drivers/mmc/host 目录下：drivers/mmc/host

├── sunxi-mmc.c // 集中了所有的的控制器的公共部分以及主要的控制逻辑代码，以及大部分的资源申请和使用，包括中断，pin，ccmu 等

├── sunxi-mmc.h // 为 Sunxi 平台的 SD/MMC 控制器驱动定义了一些公共的宏、数据结构

|—— sunxi-mmc-v4p1x.c // 部分平台的 sdc0、sdc1 的控制器差异部分驱动代码

|—— sunxi-mmc-v4p1x.h // 部分平台的 sdc0、sdc1 的控制器驱动定义了一些的宏、数据结构

|—— sunxi-mmc-v4p5x.c // sdc2 的控制器差异部分驱动代码

|—— sunxi-mmc-v4p5x.h // sdc2 的控制器驱动定义了一些的宏、数据结构

|—— sunxi-mmc-v5p3x.c /部分平台的 sdc0、sdc1 的控制器差异部分驱动代码

|—— sunxi-mmc-v5p3x.h//部分平台的 sdc0、sdc1 的控制器驱动定义了一些的宏、数据结构

|—— sunxi-mmc-debug.c 用于 debug 的代码

|—— sunxi-mmc-export.c 提供给其他模块的独立接口



### 2.5 驱动框架介绍

如源码结构介绍

