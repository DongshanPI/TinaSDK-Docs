## 2 模块介绍

### 2.1 模块功能介绍

Linux 中SPINOR 体系结构如下图所示：

![image-20221216110030034](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216110030034.png)

SPI NOR Framework：这层主要是处理不同厂家的NOR 物理特色差异，初始化SPINOR的工作状态，如工作线宽（1 线、2 线、4 线、8 线）、有效地址位（16M 以上的NOR 需要使用4 地址模式），为上层MTD 提供读写擦接口。

```
对应代码目录：drivers/mtd/spi-nor/spi-nor.c
M25P80（generic SPI NOR controller driver）：这层主要对SPI NOR Framework
层传下来的数据封装成msg，传递给SPI framework 层。
对应代码目录：drivers/mtd/devices/m25p80.c
SPI Framework：这层主要是将msg 加入ctl 的工作队列中，启动内核线程队列，处理队列
中的msg。
对应代码目录：drivers/spi/spi.c
SPI controller driver：这层初始化SPI 控制器频率、时钟模式、cs 有效电平、大小端等
配置，同时处理上层传下来的msg，通过CPU/DMA 方式传输数据到FIFO，再传输给外设
SPINOR。
对应代码目录：drivers/spi/spi-sunxi.c
```

### 2.2 相关术语介绍

| 术语      | 解释说明                                                     |
| --------- | ------------------------------------------------------------ |
| Sunxi     | 指Allwinner 的一系列SOC 硬件平台                             |
| SPI       | Serial Peripheral Interface，同步串行外设接口                |
| NOR Flash | NOR Flash 是一种非易失闪存技术，是Intel 在1988 年创建        |
| MTD       | MTD(memory technology device 内存技术设备) 是用于访问memory 设备（ROM、flash）的Linux 的子系统 |

### 2.3 模块配置介绍

#### 2.3.1 longan 的配置和打包

```
./build.sh config
All available platform:
	0. android
	1. linux
Choice [linux]: 1
... //配置根据需求选择
All available flash: //flash类型，只区分nor和非nor方案，Android方案无此选项，默认非nor
	0. default
	1. nor
Choice [default]: 1
```

1. 打包普通固件

```
#./build.sh clean
#./build.sh
#./build.sh pack
```

2. 打包卡打印固件

```
#./build.sh clean
#./build.sh
#./build.sh pack_debug
```

在配置的过程中会把平台目录下的BoardConfig.mk 的信息拷贝到.buildconfig 中。

#### 2.3.2 sys_config 配置

SPINOR 的boot0 启动阶段，部分参数是从boot0 头部获取的，而这些参数是我们在打包固件时，通过工具update_boot0 将sys_config.fex 中[spinor_para]，更新到boot0 头部的，sys_config.fex 的[spinor_para] 配置参数如下：

```
[spinor_para]
;readcmd =0x6b
;read_mode =4
;write_mode =4
;flash_size =16
;delay_cycle =1
;frequency =100000000
;erase_size =64
;lock_flag =0
;sample_delay =0
;sample_mode =2
spi_sclk = port:PC00<4><0><2><default>
spi_cs = port:PC01<4><1><2><default>
spi0_mosi = port:PC02<4><0><2><default>
spi0_miso = port:PC03<4><0><2><default>
spi0_wp = port:PC04<4><0><2><default>
spi0_hold = port:PC05<4><0><2><default>
```

其中：

```
readcmd：boot0 用于读取数据的命令，不填默认用uboot 传递过来的readcmd
read_mode、write_mode：boot0 的工作线宽（1、2、4），不填默认更加readcmd 决
定线宽
flash_size：flash 的大小
delay_cycle：boot0 的采样延时配置，大于60MHZ 配置为1，小于24MHZ 配置为2，
大于24MHZ 小于60HZ 配置为3
frequency：boot0 的SPI 工作频率，不填使用默认值50M
erase_size：boot0 的擦除单位
lock_flag：锁功能是否打开
sample_delay：boot0 的细调采样的采样延时，uboot、kernel 也会用到，默认不填等于
0xaaaaffff
sample_mode：boot0 的细调采样的采样模式，uboot、kernel 也会用到，默认不填等于
0xaaaaffff
spi_sclk、spi_cs、spi0_mosi、spi0_miso、spi0_wp 和spi0_hold 用于配置相应的GPIO。
```

#### 2.3.3 UBOOT 配置

##### 2.3.3.1 编译和配置

```
#make clean
#make sun8iw19p1_nor_config ----启动的uboot (#make sun8iw19p1_config----烧写uboot)
#make -j32
```

##### 2.3.3.2 Menuconfig 配置

```
#cd brandy/brandy-2.0/u-boot-2018
#make menuconfig
```

进入Device Drivers

```
Device Drivers ---->
[*]SPI Suppport ---->
[*]Sunxi flash support ---->
```

![image-20221216110954138](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216110954138.png)

进入SPI Support

```
Device Drivers ---->
[*]SPI Suppport ---->
[*]Sunxi SPI driver
```

![image-20221216111017858](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216111017858.png)

进入sunxi_flash_support

```
Device Drivers ---->
[*]Sunxi flash support ---->
[*]Support sunxi spinor devices
```

![image-20221216111039312](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216111039312.png)

#### 2.3.4 KERNEL 配置

##### 2.3.4.1 SPINOR-驱动配置

```
#cd kernel/liunx-4.9
#make ARCH=arm menuconfig
```

进入Device Drivers

```
Device Drivers ---->
<*>Memory Technology Device (MTD) support ---->
[*]SPI support ---->
```

![image-20221216111201111](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216111201111.png)

进入Menory Technology Device(MTD) support

```
Device Drivers ---->
<*>Memory Technology Device (MTD) support ---->
<*>SUNXI partitioning support
<*>Direct char device access to MTD devices
<*>Caching block device access to MTD devices
Self-contained MTD device drivers ---->
SPI-NOR device support ---->
```

![image-20221216111231797](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216111231797.png)

进入Self-contained MTD device drivers（5.4 内核不需要选择此项）

```
Device Drivers ---->
<*>Memory Technology Device (MTD) support ---->
Self-contained MTD device drivers ---->
<*>Support most SPI Flash chips (AT16DF, M25P.....)
```

![image-20221216111301751](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216111301751.png)

##### 2.3.4.2 cmdline 方式选择

```
Boot opttions ---->
```

![image-20221216111354759](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216111354759.png)

进入Boot options

```
Boot opttions ---->
Kernel command line type ---->
```

![image-20221216111427021](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216111427021.png)

进入kernel command line type

```
Boot opttions ---->
Kernel command line type ---->
(X)Use bootloade kernel arguments if available
```

![image-20221216111454068](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216111454068.png)

##### 2.3.4.3 文件系统配置

进入File systems

```
File system ---->
[*]Miscellaneous filesystems ---->
```

![image-20221216111527624](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216111527624.png)

• 进入Miscellaneous filesystems
• Incluede support for ZLIB compressed file systems (NEW)
• Incluede support for LZ4 compressed file systems (NEW)
• Incluede support for LZO compressed file systems (NEW)
• Incluede support for XZ compressed file systems (NEW)

```
File system ---->
[*]Miscellaneous filesystems ---->
[*]Incluede support for XZ compressed file systems (NEW)(压缩方式选择如下)
```

![image-20221216111604498](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216111604498.png)

以上的压缩方式（ZLIB/LZ4/LZO/XZ）具体选择哪一种需要根据longan/build/mkcmd.sh 中如下代码使用的压缩方式而定，如下代码使用的是gzip 压缩方式，则内核File systems 中配置需选择LZO 压缩方式，若使用的是xz, 则需选择XZ 压缩方式。

```
${ROOTFS} ${LICHEE_PLAT_OUT}/rootfs.squashfs -root-owned -no-progress -comp gzip -noappend
```

### 2.4 源码目录介绍

#### 2.4.1 UBOOT 源码目录

```
\u-boot-2018\drivers
├──sunxi_flash ---sunxi_flash的初始化/退出/读/写/擦除等flash接口
├─mmc ---mmc接口代码
├─nand ---nand接口代码
├─spinor ---spi nor接口代码
├─sunxi_flash.c ---sunxi_flash操作接口
└──其他
├── spi --sunxi_spi的接口代码
├─sunxi_spi.c ---具体代码的实现
├──mtd
├─spi
├─sf_probe.c ---nand接口代码
├─spinor ---spi nor接口代码
├─sunxi_flash.c ---sunxi_flash
```

#### 2.4.2 KERNEL 源码目录

```
\longan\kernel\linux-4.9\drivers\
├── mtd
├─spi-nor
├─spi-nor.c ---spi nor驱动代码
└──其他
├── spi --spi的接口代码
└── makefile ---编译文件
```


