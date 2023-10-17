## 5 AXP-Key

使用AXP-Key 的话需要电路板上带上我司提供power-key 功能的PMU，这是由PMU 识别该按键，并在驱动中上报相关事件，power-key 驱动源码在电源驱动源码

同目录下。用户不需要做任何更改，这部分是可以直接使用的。

这里使用R329 来作为例子，其对应的设备节点为：

![Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228150541561](https://photos.100ask.net/Tina-Sdk/Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228150541561.png)

<center>图5-1: AXP 按键节点图</center>

上报的键值为KEY_POWER 116

### 5.1 4.9 内核

1. 修改设备树文件这里以R329-evb1 为例，设备树文件为board.dts，路径：

```
device/config/chips/r329/configs/evb1/board.dts
```

详细配置为：

```
&twi2{
    no_suspend = <1>;
    status = "okay";
    pmu0: pmu@34 {
        compatible = "x-powers,axp2585";
............省略其他无关配置
        powerkey0: powerkey@0 {
            status = "okay";
            compatible = "x-powers,axp2585-pek";
            pmu_powkey_off_time = <6000>;
            pmu_powkey_off_func = <0>;
            pmu_powkey_off_en = <1>;
            pmu_powkey_long_time = <1500>;
            pmu_powkey_on_time = <1000>;
            wakeup_rising;
            /* wakeup_falling; */
        };
    };
};
```

2. 确认驱动是否被选中

tina 目录下执行make kernel_menuconfig，确认以下配置：

```
Device Drivers
    └─>Input device support
        └─>Miscellaneous devices
            └─>X-Powers AXP2101 power button driver
```

![Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228151211736](https://photos.100ask.net/Tina-Sdk/Tina_Linux_Key_Quick_Configuration_User_Guide-image-20221228151211736.png)

<center>图5-2: AXPKEY 配置图</center>

**警告**
**不同方案使用的PMU 可能会不一样，因此AXP-KEY 的配置也会不一致，详细查看各个方案使用的PMU 型号。**



这里另外，PMU 还提供了AXP GPIO, 根据不同型号的PMU, GPIO 数量可能不一样，但gpio number 均以1024 开始，例如AXP GPIO0 的gpio number 为1024, 

AXP GPIO1 的gpio number 为1025。它们均能当做普通gpio 使用，因此按照GPIO-Key 章节的描述去操作它们即可需要注意的是，sys_config 中axp gpio 的命名

与普通gpio 有些区别，例如AXP gpio0：

```
port:power0<1><0><default><0>
```

### 5.2 5.4 内核

**说明**
**本章节暂不介绍Linux-5.4 内核。**