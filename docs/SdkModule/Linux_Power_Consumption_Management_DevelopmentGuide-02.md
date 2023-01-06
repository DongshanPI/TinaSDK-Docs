## 2 Tina 功耗管理框架概述

### 2.1 功能介绍

tina 功耗管理系统主要由休眠唤醒（standby、autosleep、runtime pm）， 调频调压（cpufreq、devfreq、dvfs ），开关核（cpu hotplug），cpuidle 等子系

统组成。主要用于对系统功耗进行管理和控制，平衡设备功耗和性能。

一般我们可将其分为两类，即静态功耗管理和动态功耗管理。

![Tina_Linux_Power_Management_Development_Guide-image-20230104145122259](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Tina_Linux_Power_Management_Development_Guide-image-20230104145122259.png)

<center>图2-1: 功耗管理系统分类</center>

一般地，可以动态调整或实时改变系统状态而达到节能目的技术，称为动态功耗管理，例如调频调压，idle, hotplug, runtime-pm 等；

相对地，我们把单纯地将系统置为某一种状态，而不实时调整的低功耗技术，称为静态功耗管理，例如休眠唤醒相关技术等。

由于在tina 系统中，动态功耗技术一般来说默认配置好了，基本不需要客户修改, 另外如调频，温控等模块会在Linux 模块开发指南目录下，由模块相关的文档说

明。因此本文主要介绍静态功耗管理技术，即休眠唤醒框架。

### 2.2 相关术语

<center>表2-1: 术语表</center>

| 术语                             | 解释                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| CPUS                             | 全志平台上，专用于低功耗管理的协处理器单元。                 |
| CPUX                             | 主处理器单元，主要为客户应用提供算力的ARM/RISC-V 核心。      |
| WFI                              | ARM 体系中一种指令，可将CPUX 置于低功耗状态，直到有中断发生<br/>而退出该状态。详细请参考ARM 手册，例如<br/>《DDI0487A_d_armv8_arm.pdf》。 |
| NormalStandby、<br/>SuperStandby | Allwinner 内部术语，系统进入一种低功耗状态，暂停运行，以获取更低<br/>的功耗表现，区别是CPUX 是否掉电。前者CPUX 不掉电，系统唤醒直<br/>接借助于CPUX 的WFI 指令完成。后者CPUX 掉电，系统唤醒需借助<br/>其他硬件模块实现，如CPUS。 |
| Arm Trusted<br/>Firmware         | ARMv8-A 安全世界软件的一种实现，包含标准接口：PSCI、TBRR、<br/>SMCCC 等。在本文中，将其软硬件实现，统称为ATF。 |
| OP-TEE                           | 一种安全操作系统方案，具有单独的SDK 环境，以二进制文件的形式集<br/>成在tina 中，在本文中，统称为OP-TEE。 |
| SCP、ARISC                       | 即CPUS 的SDK 环境。最初CPUS 固件以闭源方式集成在tina 环境<br/>中，文件名为scp.bin，故称SCP。现已在tina 中提供开源代码包，目<br/>录名为arisc，故又称为ARISC。 |
| BMU                              | 电池管理芯片，提供电池升压，充电管理等功能，同时可外接电源键，用<br/>于开机，休眠，唤醒等。 |
| PMU                              | 电源管理芯片，有多个可调的DC-DC, LDO 通道，提供电源管理功能，<br/>同时可外接电源键，用于开机，休眠，唤醒等。 |

### 2.3 唤醒源支持列表

|      | PowerKey | LRADC | RTC  | WIFI(GPIO) | BT   | UART | USB  | 插拔MAD |
| ---- | -------- | ----- | ---- | ---------- | ---- | ---- | ---- | ------- |
| R328 | N        | Y     | N    | Y          | –    | –    | –    | Y       |
| R329 | Yˆ1      | Y     | Y    | Y          | –    | Yˆ2  | –    | Y       |
| D1-H | N        | Y     | Y    | Y          | –    | –    | –    | –       |
| V853 | Y        | –     | Y    | Y          | –    | –    | Y    | –       |

注：Y: 支持；N: 不支持；--: 未明确

ˆ1: 仅带PMU 的方案支持；

ˆ2: 仅suart 支持；