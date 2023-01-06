## 4 U-Boot 功能及其配置方法/文件介绍

### 4.1 U-Boot 功能介绍

在嵌入式操作系统中，BootLoader/U-Boot 是在操作系统内核运行之前运行。可以初始化硬件设备、建立内存空间映射图，从而将系统的软硬件环境带到一个合适状态，以便为最终调用操作系统内核准备好正确的环境。在 sunxi 平台中，除了必须的引导系统启动功能外，BOOT 系统还提供烧写、升级等其它功能。

U-Boot 主要功能可以分为以下几类

1. 引导内核

   能从存储介质（nand/mmc/spinor）上加载内核镜像到 DRAM 指定位置并运行。

2. 量产 & 升级

   包括卡量产，USB 量产，私有数据烧录，固件升级

3. 开机提示信息

   开机能显示启动 logo 图片（BMP 格式)

4. Fastboot 功能

   实现 fastboot 的标准命令，能使用 fastboot 刷机





### 4.2 U-Boot 功能配置方法介绍

U-Boot 中的各项功能可以通过 defconfig 或配置菜单 menuconfig 进行开启或关闭, 具体配置

方法如下:

#### 4.2.1 通过 defconfig 方式配置

1. vim /longan/brandy/brandy-2.0/u-boot-2018/configs/{LICHEE_CHIP}_defconfig

2. 开{LICHEE_CHIP}_defconfig或{LICHEE_CHIP}_nor_defconfig后，在相应的宏定义前去掉或添加"#"即可将相应功能开启或关闭。如下图，只要将CONFIG_SUNXI_NAND前的#去掉即可支持 NAND 相关功能，其他宏定义的开启关闭也类似。修改后需要运行make xxx_defconfig使修改后的配置生效。

   ![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxU-bootDevelopmentGuide_001.png)

​																			图 4-1: defconfig 配置图

#### 4.2.2 通过 menuconfig 方式配置

通过 menuconfig 方式配置的方法步骤如下：

1. cd brandy/brandy-2.0/u-boot-2018/

2. 执行make menuconfig命令，会弹出 menuconfig 配置菜单窗口，如下图所示。此时即可对各模块功能进行配置，配置方法 menuconfig 配置菜单窗口中有说明。

3. 修改后配置已经生效，直接 make 即可生成对应 bin。如果重新运行make xxx_defconfig，通过menuconfig 方式修改的配置会在运行make xxx_defconfig后被xxx_defconfig中的配置覆盖。

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxU-bootDevelopmentGuide_002.png)

​																		图 4-2: menuconfig 配置菜单图



### 4.3 U-Boot 配置参数文件介绍

U-Boot 自 linux-5.4 以后不再使用 sysconfig 和内核 dts 作为配置文件，而是使用 U-Boot 自带的 dts 来配置参数。kernel-dts 与 U-Boot-dts 完全独立。



#### 4.3.1 U-Boot-dts 路径

U-Boot-dts 路径为：vim longan/brandy/brandy-2.0/u-boot-2018/arch/arm/dts



#### 4.3.2 U-Boot-dts，defconfig 配置

| 配置项                             | 配置项含义                             |
| ---------------------------------- | -------------------------------------- |
| CONFIG_OF_SEPARATE                 | 构建 U-Boot 设备树成为 U-Boot 的一部分 |
| CONFIG_OF_BOARD                    | 关闭使用外部 dts                       |
| CONFIG_DEFAULT_DEVICE_TREE         | 选择构建的 dts 文件文件名              |
| CONFIG_SUNXI_NECESSARY_REPLACE_FDT | 开启选项, 实现内部 dts 换成外部 dts    |



| 配置项                             | 选项                       |
| ---------------------------------- | -------------------------- |
| CONFIG_OF_SEPARATE                 | y                          |
| CONFIG_OF_BOARD                    | n                          |
| CONFIG_DEFAULT_DEVICE_TREE         | “{LICHEE_CHIP}-soc-system” |
| CONFIG_SUNXI_NECESSARY_REPLACE_FDT | y                          |



#### 4.3.3 U-Boot-dts 注意事项

##### 4.3.3.1 编译注意事项

1.dts 分为板级 dts，和系统 dts。

   系统 dts 由 CONFIG_DEFAULT_DEVICE_TREE 决定，可以在 $(CONFIG_SYS_CONFIG_NAME)_defconfig找到该宏的定义。

   系统 dts 最终会 include 板级 dts，文件路径 {LICHEE_BOARD_CONFIG_DIR}，文件名:uboot-board.dts。



2. 我们可以通过编译时的打印判断启动的 dts

```
OBJCOPY examples/standalone/hello_world.srec
OBJCOPY examples/standalone/hello_world.bin
LD u-boot
OBJCOPY u-boot.srec
OBJCOPY u-boot-nodtb.bin
‘{LICHEE_BOARD_CONFIG_DIR}/uboot-board.dts’ -> ‘~/longan/brandy/brandy-2.0/u-boot-2018/
arch/{LICHEE_ARCH}/dts/.board-uboot.dts’
DTC arch/{LICHEE_ARCH}/dts/{LICHEE_CHIP}-soc-system.dtb
SYM u-boot.sym
SHIPPED dts/dt.dtb
FDTGREP dts/dt-spl.dtb
COPY u-boot.dtb
CAT u-boot-dtb.bin
COPY u-boot.bin
‘u-boot.bin’ -> ‘{LICHEE_CHIP}.bin’ ‘u-boot-g{LICHEE_CHIP}.bin’ -> ‘{LICHEE_BRANDY_OUT_DIR}/bin/u-boot-g{LICHEE_CHIP}.bin’ ‘u-boot-g{LICHEE_CHIP}.bin’ -> ‘{LICHEE_PLAT_OUT}/u-boot-g{LICHEE_CHIP}.bin’
CFGCHK u-boot.cfg
```



##### 4.3.3.2 语法注意事项

当系统 dts 与板级 dts 存在同路径下同名节点时，板级 dts 将会覆盖系统 dts。



##### 4.3.3.3 运行时注意事项

1. 为了在启动内核前更新参数到内核 dts 和可以在 U-Boot 控制台查看修改 dts。按阶段划分可以分为使用内部 dts 阶段和使用内核 dts 阶段，如下图所示。

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxU-bootDevelopmentGuide_003.png)

​																	图 4-3: dts 变化图

2. 可以通过命令set_working_fdt来切换当前生效的 fdt。

```
[04.562]update bootcmd
[04.576]change working_fdt 0x7bebee58 to 0x7be8ee58
[04.587]update dts
Hit any key to stop autoboot: 0
=> set
	set_working_fdt setenv setexpr
=> set_working_fdt 0x7bebee58
change working_fdt 0x7be8ee58 to 0x7bebee58
=>
```

