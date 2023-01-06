## 2 模块介绍

### 2.1 模块功能介绍

*•* 休眠唤醒指系统进入低功耗和退出低功耗模式，一般称之为 Standby。standby 分为 super standby 和 normal standby，区别是 cpu 是否掉电。

*•* 假关机是类似 standby 的一种低功耗模式。进入假关机，系统会先复位，再进入低功耗模式，等待唤醒源；检测到唤醒源，系统退出假关机，直接从低功耗模式复位重启。适用于 OTT 类产品代替常规的关机，实现红外/蓝牙开机功能。



### 2.2 相关术语介绍

​																			表 2-1: 术语介绍

| 术语           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| Super standby  | Vdd_cpu 掉电或 Core 掉电，dram 进入 self refresh 状态        |
| Normal standby | CPUX WFI，dram 进入 self refresh 状态                        |
| Fake Poweroff  | 假关机，类似 standby，主要区别是系统退出假关机会重启，而不是唤醒 |
| SCP/CPUS       | 全志平台辅助进行电源管理的协处理器                           |



### 2.3 模块配置介绍

#### 2.3.1 Device Tree 配置说明

设备树中存在的是该类芯片所有平台的模块配置，设备树文件的路径为：kernel/linux-4.9/arch/arm64（32 位平台为 arm）/boot/dts/sunxi/CHIP.dtsi(CHIP 为研发代号，如sun50iw10p1 等)。Standby 模块在 dtsi 中无用户可用配置。



#### 2.3.2 board.dts 配置说明

board.dts 用于保存每一个板级平台的设备信息（如 demo 板，perf1 板等），里面的配置信息会覆盖上面的 Device Tree 默认配置信息。

*•* standby 参数配置

描述系统资源的相关信息。例如，“vdd-cpu = <0x00000006>;”，其中各个 bit 代表 PMU 的各路供电，则 vdd-cpu 使用 PMU的第二、三路供电。

“osc24m-on = <0x1>;”，表示系统休眠后 osc24m 是否关闭，0x0 表示关闭，0x1 表示不关闭。

```
standby_param: standby_param {
    vdd-cpu = <0x00000006>;
    vdd-sys = <0x00000008>;
    vcc-pll = <0x00000100>;
    osc24m-on = <0x1>;
    ...
};
```

注：由于 r528/v853 部分方案, 没有外挂 PMU，即硬件上不支持 vdd-sys/vdd-cpu 掉电，故R528/v853 board.dts 不需要该信息（standby_param）。



*•* 唤醒源配置

以 RTC 模块为例，RTC 驱动支持通过 “wakeup-source” 配置是否作为唤醒源；在 RTC 模块节点下添加 “wakeup-source” 属性，则可以设置为唤醒源。

```
rtc: rtc@07000000 {
    compatible = "allwinner,sunxi-rtc";
    device_type = "rtc";
    wakeup-source;
    ...
};
```



*•* GPIO 为唤醒源配置

下面以 gpio-key 驱动为例, 演示一下 gpio 作为 wakeup-source 的代码编写:

```
keys {
    compatible = "gpio-keys";
    status = "okay";
    wakeup {
        wakeup-source; (1)
        gpios = <&pio PH 9 6 2 2 1>; (2)
        label = "wakeup";
        linux,code = <KEY_WAKEUP>;
    };
};
1.wakeup-source:device的wakeup source属性.需要设备驱动自己去解析该属性,如果有,则表示设备具有wakeup source功能.
2.gpios:配置gpio的mux,pull,drive,data等属性,如上的配置代表把PH9设置为6号功能(中断功能),下拉,驱动能力为2,data值为1.
```

*•* 假关机参数配置

描述系统关机的方式及假关机时需要用到的系统资源。如：

```
	box_start_os0 {
        compatible = "allwinner,box_start_os";
        start_type = <0x1>;
        irkey_used = <0x0>;
        pmukey_used = <0x0>;
        pmukey_num = <0x0>;
        led_power = <0x0>;
        led_state = <0x0>;
        pinctrl-0 = <&standby_blue>;
        pinctrl-1 = <&standby_red>;
        pinctrl-2 = <&standby_bt>;
        /*pinctrl-3 = <&standby_bt>;*/
    }
    ...
    s_cir0 {
        s_cir0_used = <1>;
        ir_power_key_code0 = <0x40>;
        ir_addr_code0 = <0xfe01>;
        ir_power_key_code1 = <0x1a>;
        ir_addr_code1 = <0xfb04>;
        ir_power_key_code2 = <0xf2>;
        ir_addr_code2 = <0x2992>;
        ir_power_key_code3 = <0x57>;
        ir_addr_code3 = <0x9f00>;
        ir_power_key_code4 = <0xdc>;
        ir_addr_code4 = <0x4cb3>;
        ir_power_key_code5 = <0x18>;
        ir_addr_code5 = <0xff00>;
        ir_power_key_code6 = <0xdc>;
        ir_addr_code6 = <0xdd22>;
        ir_power_key_code7 = <0x0d>;
        ir_addr_code7 = <0xbc00>;
        ir_power_key_code8 = <0x4d>;
        ir_addr_code8 = <0x4040>;
        wakeup-source;
	}
```

“start_type = <0x1>;”，0x0 代表当系统启动时检测到是适配器上电启动则进入假关机（H700 TV）；0x1 不做适配器上电检测判断，按普通的假关机判断流程；

“irkey_used = <0x0>;”，当前代码并未解析该节点，为无用节点，不影响假关机流程；

“pmukey_used = <0x0>;”，当前代码并未解析该节点，为无用节点，不影响假关机流程；

“pmukey_num = <0x0>;”，当前代码并未解析该节点，为无用节点，不影响假关机流程；

“led_power = <0x0>;”，当前代码并未解析该节点，为无用节点，不影响假关机流程；

“led_state = <0x0>;”，当前代码并未解析该节点，为无用节点，不影响假关机流程；

“pinctrl-0 = <&standby_blue>;”，LED 显示需要用到的 pinctrl 配置（其中一种颜色）；

“pinctrl-1 = <&standby_red>;”，LED 显示需要用到的 pinctrl 配置（其中一种颜色）；

“pinctrl-2 = <&standby_bt>;”，外部唤醒源（如蓝牙/wifi/rtc 模块）唤醒系统时拉对应 gpio，高脉冲有效，高脉冲持续时间需要大于 200 ms；

“pinctrl-3 = <&standby_bt>;”，外部唤醒源（如蓝牙/wifi/rtc 模块）唤醒系统时拉对应 gpio，低脉冲有效，低脉冲持续时间需要大于 200 ms；

“ir_power_key_code = <0x40>;”，ir 模块特定数据的码值，根据方案需求进行配置；

“ir_addr_code = <0xfe01>;”，ir 模块特定地址的码值，根据方案需求进行配置。





​																	表 2-2: 平台支持唤醒源列表


| 平台/唤醒源 | 非CPUS域GPIO   | CPUS域GPIO    | POWER-KEY      | RTC            | USB            | 红外           | 蓝牙/WiFi      | MAD    |
| ----------- | -------------- | ------------- | -------------- | -------------- | -------------- | -------------- | -------------- | ------ |
| T509        | 不支持         | super standby | super standby  | super standby  | super standby  | 不支持         | super standby  | 不支持 |
| H616        | normal standby | 不支持        | normal standby | normal standby | normal standby | normal standby | normal standby | 不支持 |
| V853        | normal standby | 不支持        | super standby  | super standby  | super standby  | 不支持         | normal standby | 不支持 |
| T113        | normal standby | 不支持        | 不支持         | normal standby | 不支持         | 不支持         | normal standby | 不支持 |





#### 2.3.3 kernel menuconfig 配置说明

linux-4.9 内核版本：在命令行中进入 linux 目录，执行 make ARCH=arm64 menuconfig(32 位系统为 make ARCH=arm menuconfig) 进入配置主界面 (Linux-5.4 内核版本执行：./build.sh menuconfig)，并按以下步骤作。

*•* 内核 PSCI 选项

```
Kernel Features --->
	[*] Support for the ARM Power State Coordination Interface (PSCI)
```

*•* 内核 CPUIDLE 相关选项（可选）

```
CPU Power Management --->
	CPU Idle --->
        [*] CPU idle PM support
        ARM CPU Idle Drivers --->
			[*] Generic ARM/ARM64 CPU idle Driver
```

*•* 内核 POWER 相关选项

```
Power management options --->
    [*] Suspend to RAM and standby
    [ ] Opportunistic sleep
    [*] User space wakeup sources interface
    (100) Maximum number of user space wakeup sources (0 = no limit)
    -*- Device power management core functionality
    [*] Power Management Debug Support
    [*] Extra PM attributes in sysfs for low-level debugging/testing
```

*•* 内核 FAKE_POWEROFF 相关选项

```
Device Drivers --->
	[*] Real Time Clock --->
		[*] support ir fake poweroff
```





#### 2.3.4 uboot-2018 配置

*•* uboot-2018 FAKE_POWEROFF 相关配置

在平台的 defconfig 中，将 CONFIG_ATF_BOX_STANDBY 配置为 Y

```
CONFIG_ATF_BOX_STANDBY = Y;
```





### 2.4 源码结构介绍

Standby 的源代码位于内核 kernel/power/目录下：

```
kernel/power/
    ├── autosleep.c
    ├── console.c
    ├── hibernate.c
    ├── Kconfig
    ├── main.c
    ├── Makefile
    ├── modules.builtin
    ├── modules.order
    ├── power.h
    ├── poweroff.c
    ├── process.c
    ├── qos.c
    ├── snapshot.c
    ├── suspend.c
    ├── suspend_test.c
    ├── swap.c
    ├── user.c
    ├── wakelock.c
    └── wakeup_reason.c
```



### 2.5 驱动框架介绍

休眠唤醒指系统进入低功耗和退出低功耗模式，一般称之为 Standby。休眠过程由应用发起，经由内核的电源管理框架来进行休眠唤醒管理工作，如果存在 CPUS（一颗集成在 IC 内部的对电源进行管理的 openrisc 核，是 SoC 内置的超低功耗硬件管理模块），最终会传递到到 CPUS。因此休眠唤醒类出现问题的可能为应用层、内核层、CPUS 层，如果不存在 CPUS，则 CPU 进入WFI。休眠唤醒流程图如下，虚线部分为部分内核实现。

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxStandbyDevelopmentGuide_001.png)

​																	图 2-1: standby 驱动总体结构			

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxStandbyDevelopmentGuide_002.png)

​																	图 2-2: linux standby 流程