## 2 模块描述

### 2.1 模块功能

SID 提供的功能可以分为四大部分：ChipID、SoC Version、Efuse 功能、一些状态位。

#### 2.1.1 Chip ID 功能

对于全志的SoC 来说，ChipID 用于该SoC 的唯一标识，如A83 的ChipID 标识其在所有A83 中的唯一（目前仅保证同一型号SoC 中的ChipID 唯一）。ChipID 由4 个word（16 个byte）组成，共128bit，通常放在Efuse（见2.1.3 节）的起始4 个word。具体ChipID 的bit 含义，请参考生产制造部为每颗SoC 定义的《ChipID 烧码规则》。

#### 2.1.2 SoC Version 功能

严格讲SoC Version 包含两部分信息：
1.Bonding ID，表示不同封装。

2. Version，表示改版编号。

说明：这两个信息所在的寄存器不一定都在SID 模块内部，且各平台位置不一，但软件上为了统一管理，都归属为SID 模块。

BSP 会返回这两个信息的组合值，由应用去判断和做出相应的处理。

#### 2.1.3 Efuse 功能

对软件来说，Efuse 中提供了一个可编程的永久存储空间，特点是每一位只能写一次（从0到1）。
Efuse 接口方式，Efuse 容量大于512bit 采用SRAM 方式。带有SRAM 的硬件结构示意图如下：

![image-20221219105300040](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_SID_DevGuide_image-20221219105300040.png)

#### 2.1.4 一些状态位

Secure Enable标明当前系统的Security 属性是否打开，即是否运行了SecureBoot 和SecureOS。

芯片SecureEnable 状态位保存在SID 模块的0xa0 寄存器。

### 2.2 模块位置

SID 是一个比较独立的模块，在Linux 内核中没有依赖其他子系统，在Sunxi 平台默认是ko 方式，存放在drivers/soc/sunxi 目录中。

SID 为其他模块提供API 的调用方式。关系如下图：

![image-20221219105416344](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_SID_DevGuide_image-20221219105416344.png)

1）TV、Thermal、GMAC 的校准参数保存在SID 中；
2）Nand、SMP、VE 需要读取SoC Version；
3）CE 和HDMI 会用到SID 中的一些Key；
4）Sysinfo 比较特殊，为了方便用户空间获取、调试SID 信息，专门设计的一个字符型设备驱动。

### 2.3 模块device tree 配置说明(适用Linux-5.4)

SID 模块在Device tree 中通常会用到两个模块的配置信息：sunxi-sid 以sun50iw10p1为例，需要在sun50iw10p1.dtsi 中添加节点：

```
sid@3006000 {
compatible = "allwinner,sun50iw10p1-sid", "allwinner,sunxi-sid";
reg = <0x0 0x03006000 0 0x1000>;
#address-cells = <1>;
#size-cells = <1>;
/* some guys has nothing to do with nvmem */
secure_status {
reg = <0x0 0>;
offset = <0xa0>;
size = <0x4>;
};
chipid {
reg = <0x0 0>;
offset = <0x200>;
size = <0x10>;
};
rotpk {
reg = <0x0 0>;
offset = <0x270>;
size = <0x20>;
};
};
```

在sid 下增加子节点secure_status, chipid, rotpk。就可以用key_info 来访问。

```
console:/ # echo chipid > /sys/class/sunxi_info/key_info ; cat /sys/class/sunxi_info/
key_info
console:/ # 00000400
```

### 2.4 模块源码结构

SID 驱动的源代码目录下：

```
linux-4.9，linux-5.4
./drivers/soc/sunxi/
└── sunxi-sid.c // 实现了SID对外的所有API接口
```

对外提供的接口头文件：./include/linux/sunxi-sid.h

### 2.5 内核配置

此配置项一般默认开，不需要重新配置

在longan 环境中在根目录执行./build.sh menconfig进入配置主界面，配置路径如下：

```
System Type
└─>ARM system type
└─>Allwinner Ltd. SUNXI family
版
```

配置界面图示：

![image-20221219105647967](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_SID_DevGuide_image-20221219105647967.png)

SID 驱动本身没有注册为单独的模块，需要通过注册sysinfo 字符驱动（实现代码见drivers/char/sunxi-sysinfo/）来提供sysfs 节点。

在longan 环境中在根目录执行./build.sh menconfig进入配置主界面，配置路径如下

```
Device Drivers
    └─>Character devices
    	└─>sunxi system info driver
```

配置界面图示：

![image-20221219105712117](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_SID_DevGuide_image-20221219105712117.png)