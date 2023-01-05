## 8 Tina uboot定制开发

### 8.1 概述

本章节简单介绍uboot基本配置、功能裁剪、编译打包、常用命令的使用，帮助客户了解Tina
平台uboot框架，为boot定制开发提供基础。

目前Tina SDK共有三版uboot，分别是uboot-2011、uboot-2014、uboot-2018，分别在不
同硬件平台上使用，客户拿到SDK需要根据开发的硬件平台核对版本信息。

### 8.2 代码路径

```
TinaSDK/
├── brandy
│   ├── ...
│   ├── u-boot-2011
│   └── u-boot-2014
├── brandy-2.0
│   ├── ...
│   └── u-boot-2018
```

### 8.3 uboot功能

TinaSDK中，bootloader/uboot在内核运行之前运行，可以初始化硬件设备、建立内存空间映
射图，从而将系统的软硬件环境带到一个合适状态，为最终调用linux内核准备好正确的环境。
在Tina系统平台中，除了必须的引导系统启动功能外，uboot还提供烧写、升级等其它功能。

- 引导内核能从存储介质（nand/mmc/spinor）上加载内核镜像到DRAM指定位置并运行。
- 量产&升级包括卡量产，USB量产，私有数据烧录，固件升级。
- 电源管理包括进入充电模式时的控制逻辑和充电时的显示画面。
- 开机提示信息开机能显示启动logo图片(BMP格式)。
- Fastboot功能实现fastboot的标准命令，能使用fastboot刷机。


### 8.4 uboot配置

以uboot-2018为例，各项功能可以通过defconfig或配置菜单menuconfig进行开启或关闭，
具体配置方法如下：

#### 8.4.1 defconfig方式

##### 8.4.1.1 defconfig配置步骤

1. vim /TinaSDK/lichee/brandy2.0/u-boot-2018/configs/sun8iw18p1_defconfig (若是spinor方案则打开sun8iw18p1_nor_defconfig)
2. 打开sun8iw18p1_defconfig或sun8iw18p1_nor_defconfig后，在相应的宏定义前去掉或添加"#"即可将相应功能开启或关闭。


![图8-1: defconfig配置图](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_System_software_development_Guide-8-1.jpg)

如上图，只要将CONFIG_SUNXI_NAND前的#去掉即可支持NAND相关功能，其他宏定义的开启关闭也类似。

##### 8.4.1.2 defconfig配置宏介绍.

如下图是sun8iw18p1_defconfig/sun8iw18p1_nor_defconfig中的基本宏定义的介绍：


![图8-2: defconfig基本宏定义介绍图](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_System_software_development_Guide-8-2.jpg)

#### 8.4.2 menuconfig方式

通过menuconfig方式配置的方法步骤如下：

```
cd /TinaSDK/lichee/brandy2.0/u-boot-2018/
make ARCH=arm menuconfig或make ARCH=arm64 menuconfig

注意：arm针对 32 位平台，arm64针对 64 位平台。
```

执行上述命令会弹出menuconfig配置菜单，如下图所示，此时即可对各模块功能进行配置，配
置方法menuconfig配置菜单窗口中有说明。


![图8-3: menuconfig配置菜单图](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_System_software_development_Guide-8-3.jpg)

### 8.5 uboot编译

#### 8.5.1 方法一

在tina目录下即可编译uboot。

```
source build/envsetup.sh(见详注1)
lunch (见详注2)
muboot(见详注3)
详注：
1 加载环境变量及tina提供的命令。
2 输入编号，选择方案。
3 编译uboot，编译完成后自动更新uboot binary到TinaSDK/target/allwinner/$(BOARD)-common/bin/。
```

#### 8.5.2 方法二

```
source build/envsetup.sh(见详注1)
lunch (见详注2)
cboot(见详注3)
make XXX_config(见详注4)
make -j

详注：
3 跳转到Uboot源码目录。
4 选择方案配置，如果是使用norflash，运行make XXX_nor_config。
5 执行编译uboot的动作。
```

### 8.6 uboot的配置

#### 8.6.1 sys_config配置.

sys_config.fex是对不同模块参数进行配置的重要文件，对各模块重要参数的更改及更新提供了
极大的方便。其文档存放路径：

```
TinaSDK/target/allwinner/$(BOARD)/configs/sys_config.fex
TinaSDK/device/config/chips/$(CHIP)/configs/$(BOARD)/sys_config.fex
```

##### 8.6.1.1 sys_config.fex结构介绍

sys_config.fes主要由主键和子键构成，主键是某项功能或模块的主标识，由[]括起，子键是对
该功能或模块中各个参数的配置项，如下图所示，dram_para是主键，dram_clk、dram_type
和dram_zp是子键。

![图8-4: sysconfig.fex基本结构图](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_System_software_development_Guide-8-4.jpg)


##### 8.6.1.2 sys_config.fex配置实例

[platform]：平台相关配置项。

![图8-5: platform配置图](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_System_software_development_Guide-8-5.jpg)

例如，debug_mode =1表示开启uboot的调试模式，开启后会在log中打印出对应的调试信
息。next_work=2表示烧录完成后系统的下一步执行动作(0x1表示正常启动、0x2表示重启、
0x3表示关)，其他配置可以查看[platform]前的提示说明。

[target]：目标平台相关功能配置项

![图8-6: target配置图](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_System_software_development_Guide-8-6.jpg)

上图中的可以通过配置boot_clock配置cpu的频率大小。

[uart_para]：串口配置项，uart_para配置项是uboot串口打印调试时用到的重要配置


![图8-7: uart_para配置图](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_System_software_development_Guide-8-7.jpg)


上图中的uart_debug_port=0表示使用的是uart0，uart_debug_tx/uart_debug_rx配置的gpio口（PA04/PA05）需要根据对应的GPIO DATASHEET进行配置。

##### 8.6.1.3 sys_config.fex解析流程

在uboot2014/2018中sys_config.fex最终会被转化为dtb（device tree binary，linux内核配置方式），dtb最终会被打包烧录至flash中，启动过程中会将该文件加载至内存，之前在sys_config.fex中配置的参数已转化为dtb节点，最终会调用fdt_getprop_32()函数对dtb中的节点进行解析。

#### 8.6.2 环境变量配置.

uboot的环境变量就是一个个的键值对，操作接口为：getenv()，setenv()，saveenv()。环境变量的形式：


```
boot_normal=sunxi_flash read 40007800 boot;boota 4000780\
boot_recovery=sunxi_flash read 40007800 recovery;boota 40007800\
boot_fastboot= fastboot
```

##### 8.6.2.1 环境变量作用.

可以把一些参数信息或者命令序列定义在该环境变量中。在环境变量中定义UBOOT命令序列，

可以把UBOOT各个功能模块按顺序组合在一起执行，从而完成某个重要功能。

例如，如果执行了上述提到的 boot_normal 环境变量对应的命令，Uboot 则会先调用
sunxi_flash命令从存储介质的boot分区上加载内核到DRAM的0x40007800位置；然后调
用boota命令完成内核的引导。

uboot启动时调用环境变量方式下如图所示：


![图8-8: uboot启动调用环境变量方式图](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_System_software_development_Guide-8-8.jpg)


##### 8.6.2.2 环境变量配置示例介绍.

TinaSDK中，环境变量配置文件保存在TinaSDK/target/allwinner/$(BOARD)/configs/env.cfg
文件，用户使用的时候，可能会看到env-4.4.cfg、env-4.9.cfg等文件，env-xxx后缀数字表示在不同内核版本上的配置。打开后其内容示例如下，

- bootdelay=0，改环境变量bootdelay（即boot启动时log中的倒计时延迟时间）值的大小，为便于调试，bootdelay的值一般不要等于 0 ，这样在小机上电后按下任意键才能进入uboot shell命令状态。
- boot_normal=sunxi_flash read 40007800 boot;boota 4000780，设置启动内核命令，即将boot分区读到内存0x40007800地址处，然后从内存0x40007800地址处启动内核。
- Setargs_nand=setenv bootargs earlyprintk=${earlyprink}....... ，设置内核相关环境变量，该变量在启动至内核的log中会打印处理，即cmdline如下图：


![图8-9: kernel cmdline图](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_System_software_development_Guide-8-9.jpg)


- loglevel=8，设置内核log打印等级。


#### 8.6.3 sys_partition.fex分区配置

分区配置文件是一个规划磁盘分区的文件，烧录过程会按照该分区配置文件将各分区数据烧录至flash中。

TinaSDK中，分区配置文件路径TinaSDK/target/allwinner/$(BOARD)/configs/sys_partition.fex。有些方案可以看到sys_partition.fex、sys_partition_nor.fex两个分区配置文件，若是打包Tina非nor固件，则使用的是sys_partition_linux.fex配置文件，若是打包nor固件，则使用的是sys_partition_nor.fex。

##### 8.6.3.1 sys_partition.fex分区配置介绍

一个分区的属性，包含名称、分区大小、下载文件与用户属性。以下是文件中所描述的一个分区的属性：

- name，分区名称由用户自定义。当用户在定义一个分区的时候，可以把这里改成自己希望的字符串，但是长度不能超过 16 个字节。
- size，定义该分区的大小，以扇区的单位(1扇区=512bytes，如上图给env 分区分配了32768 个扇区，即32768*512/1024/1024 = 16M)，注意，为了字节对齐，这里分配的扇区大小应当能整除 128 。
- downloadfile，下载文件的路径和名称。可以使用相对路径，相对是指相对于image.cfg文件所在分区。也可以使用绝对路径。
- user_type，提供给操作系统使用的属性。目前，每个操作系统在读取分区的时候，会根据用户属性来判断当前分区是不是属于自己的然后才进行操作。这样设计的目的是为了避免在多系统同时存在的时候，A操作系统把B操作系统的系统分区进行了不应该的读写操作，导致B操作系统无法正常工作。


更具体的说明，可参考《TinaLinux存储管理开发指南》。


## 9 Tina kernel定制开发

### 9.1 概述

本章节简单介绍kernel基本配置、功能裁剪、常用命令的使用，帮助客户了解Tina平台linux
内核，为内核定制开发提供基础。

目前Tina SDK共有 4 版linux kernel，分别是linux-3.4、linux-3.10、linux-4.4、linux-
4.9，分别在不同硬件平台上使用，客户拿到SDK需要根据开发的硬件平台核对内核信息。

### 9.2 代码路径

```
TinaSDK/
    ├── ...
    ├── linux-3.10
    ├── linux-3.4
    ├── linux-4.4
    └── linux-4.9
```

### 9.3 模块开发文档.

详阅BSP开发文档，文档目录包括常用内核模块使用与开发说明。

### 9.4 内核配置

客户在定制化产品时，通常需要更改linux内核配置，在TinaSDK中，打开内核配置的方式如
下，

```
croot
make kernel_menuconfig
```

执行完后，shell控制台会跳出配置菜单。如下图所示，


![图9-1: TinaLinux内核配置菜单](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_System_software_development_Guide-9-1.jpg)