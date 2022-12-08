# Linux_启动优化_开发指南

## 1 概述

### 1.1 编写目的

介绍TinaLinux下启动速度优化使用方法。

### 1.2 适用范围

硬件平台：全志R/V/F/MR/H系列芯片。

软件平台: Tina V3.5及其后续版本。

### 1.3 相关人员

适用于TinaLinux平台的客户及相关技术人员。


## 2 启动速度优化简介

启动速度是嵌入式产品一个重要的性能指标，更快的启动速度会让客户有更好的使用体验，在某

些方面还会节省能耗，因为可以直接关机而不需要休眠。

启动速度优化可提升产品的竞争力。对于某些系统来说，启动速度是硬性要求。

### 2.1 启动流程

TinaLinux系统当前的启动流程如下：

```
brom --> boot0 --> (monitor/secure os) --> uboot --> rootfs --> app
```

brom固化在IC内部，芯片出厂后就无法更改。

后续将从boot0开始分阶段介绍启动优化的方法。

对于某些方案，会存在monitor或secure os，这两者耗时很短，本文略过。

下文涉及到一些配置文件，提前在此说明。

env配置文件路径：

```
tina/device/config/chips/<chip>/configs/<board>/env.cfg #优先级高
tina/device/config/chips/<chip>/configs/<board>/linux/env-<kernel-version>.cfg #优先级中
tina/device/config/chips/<chip>/configs/default/env.cfg #优先级低
```

sys_config.fex路径：

```
tina/device/config/chips/<chip>/configs/<board>/sys_config.fex
```

uboot-board.dts路径：

```
tina/device/config/chips/<chip>/configs/<board>/uboot-board.dts
```

> ! 警告：如 果 存 在 uboot-board.dts ， uboot 会 使 用 uboot-board.dts 中 配 置； 如 果 不 存 在uboot-board.dts ， uboot 会使用 sys_config.fex 中的配置。(AW1886/V853 使用了 uboot-board.dts ）

### 2.2 测量方法

#### 2.2.1 printk time

打开kernel配置，使能如下选项：

```
kernel hacking --->
    [*] Show timing information on printks
```

linux4.9

```
kernel hacking --->
    printk and dmesg options --->
        [*] Show timing information on printks
```

将会在内核的log前加入时间戳。

**注：此方法主要用来测量内核启动过程中各个阶段的耗时。**

#### 2.2.2 initcall_debug

修改env文件，在kernel的cmdline中加入参数，

```
# 增加initcall_debug变量
initcall_debug=1

#将initcall_debug=${initcall_debug}加入setargs_xxx中，如setargs_nand，setargs_mmc，setargs_nor，setatgs_nand_ubi等，
setargs_nand=setenv bootargs console=${console} earlyprintk=${earlyprintk} root=${nand_root} initcall_debug=${initcall_debug} init=${init}
```

开启之后，启动中会打印每个initcall函数调用及其耗时。

**注：此方法主要用来测量内核initcall的耗时。**

一般需同时配置上内核符号表，即kallsyms选项，以打印函数名。

#### 2.2.3 bootgraph.

在内核源码中自带了一个工具(scripts/bootgraph.pl)可用于分析启动时间,需要把log_buff加
大，要不然会丢失最早的启动信息：

make kernel_menuconfig

```
General setup --->
    (17) Kernel log buffer size (16 => 64KB, 17 => 128KB)
Kernel hacking --->
    printk and dmesg options --->
        [*] Show timing information on printks
```

- kernel编译时需要包含CONFIG_PRINTK_TIME选项。
- 在kernel cmdline加上"initcall_debug=1"。
- 在系统启动完毕后执行"dmesg | perl $(Kernel_DIR)/scripts/bootgraph.pl > output.svg"。
- 使用SVG浏览器（比如Inkscape，Gimp，Firefox等）来查看输出文件output.svg。

> 注：此方法主要用来测量内核启动过程中各个阶段的耗时。

#### 2.2.4 bootchart

bootchart是一个用于linux启动过程性能分析的开源软件工具，在系统启动过程自动收集CPU
占用率、进程等信息，并以图形方式显示分析结果，可用作指导优化系统启动过程。

- 修改kernel cmdline。修改env配置文件(路径见上文说明)，将其中的init修改为"init=/
    sbin/bootchartd"。
- 收集信息。bootchartd会从/proc/stat，/proc/diskstat，/proc/[pid]/stat中采集信息，经
    过处理后保存为bootchart.tgz文件。
- 转换图片。在PC上通过pybootchartgui.py工具将bootchart.tgz转换为bootchart.png，
    方便分析。

> 注：此方法主要用来测量挂载文件系统到主应用程序启动过程中的耗时。

#### 2.2.5 gpio +示波器.

在适当的地方加入操作gpio的代码，通过示波器抓取波形得到各阶段耗时。

**注：此方法可用来测量整个启动中各阶段的耗时。**

#### 2.2.6 grabserial.

Grabserial是Tim Bird用python写的一个抓取串口的工具，这个工具能够为收到的每一行信
息添加上时间戳。可从如下路径下载使用：https://github.com/tbird20d/grabserial

介绍文档：http://elinux.org/Grabserial

常见的用法：

```
sudo grabserial -v -S -d /dev/ttyUSB0 -e 30 -t
```

如果要在某个字符串重置时间戳，可以使用-m参数：


```
sudo grabserial -v -S -d /dev/ttyUSB0 -e 30 -t -m "Starting kernel"
```

- -v显示参数等信息。
- -s跳过对串口的检查。
- -d指定串口，如上述为指定/dev/ttyUSB0为操作的串口。
- -e参数指定时间，如上述命令表示抓取30s的串口记录。
- -t表示加上时间戳。
- -m匹配到指定字符串就重置时间戳的时间，也就是从 0 开始。

更多配置可以使用-h参数查看帮助。

**注：此方法可用来测量整个启动中各阶段的耗时。**

### 2.3 优化方法

**注：本节提供一些优化方法以供参考，并非所有都在Tina上集成，主要原因有：**

- 优化没有止境。需要根据目标来选择优化方法，综合考虑优化效果与优化难度。
- 优化需要具有针对性。由于各方案CPU个数及频率、flash类型及大小、kernel/rootfs压缩
    类型与尺寸、所需功能、主应用等的不同，需要针对性的进行优化。

#### 2.3.1 boot0启动优化

boot0运行在SRAM，主要功能是对DRAM进行初始化，并将uboot、monitor、secure-os
等加载至DRAM。

##### 2.3.1.1 非安全启动.

boot0可优化的地方不多，可以做的是：

- 关闭串口输出。
- 减少检测按键和检测串口的等待时间。
- 加载uboot的时候，不要先加载后搬运，直接加载到uboot的运行地址。

对于spinor的方案，还可以直接从boot0启动，只需要在boot0中加载好kernel和dtb，
不需要经过uboot ，然后直接跳转到kernel运行，可节省一定的时间。如果采用boot0启
动OS，则boot0读取数据量较大，其flash驱动也需要进行优化，如提高时钟，开启双线/四
线/DMA/Cache等。


##### 2.3.1.2 安全启动

对于安全方案来说，boot0还会对uboot、monitor、secure-os等进行签名校验，因为在启动
时需要引导SecoreOS,需要做一次环境切换，CPU由安全状态切换到非安全状态运行，所以对
于安全方案来说，不支持直接从boot0启动，然后加载dtb和kernel到内存，然后直接启动内
核，主要的优化手段较少，可以做的是：

- 关闭串口打印。
- 减少检测按键和检测串口的等待时间。
- 加载uboot的时候，不要先加载后搬运，直接加载到uboot的运行地址。

#### 2.3.2 uboot启动优化

uboot主要功能是引导内核、量产升级、电源管理、开机音乐/logo、fastboot刷机等。

##### 2.3.2.1 完全去掉uboot

uboot的包含很多重要功能，通常会保留。某些情况可以去掉，直接从boot0加载内核并启动，
可节省一些时间。

##### 2.3.2.2 避免burnkey的影响

对于启用了burnkey支持，且还没使用DragonSN工具将key烧录进去的板子，每次启动到
uboot都会尝试跟PC端工具交互产生如下log，带来延时。

```
[1.334]usb burn from boot
...
[1.400]usb prepare ok
usb sof ok
[1.662]usb probe ok
[1.664]usb setup ok
...
[4.698]do_burn_from_boot usb : have no handshake
```

如果产品不需要 burnkey，可将 uboot-board.dts 或 sys_config.fex 中的 [target] 下
burn_key设置为 0 。

或者使用DragonSN工具，烧录一次key，并设置烧录标志，以使后续启动可跳过检测。

##### 2.3.2.3 提高CPU以及flash读取频率

可设置uboot-board.dts或sys_config.fex中的[target]下boot_clock来修改uboot运行
时CPU频率( **注：不能超过SPEC最大频率** )。

对于spinor/spinand，使用较高的时钟频率（一般是100M），使用四线模式或者双线模式（看
硬件是否支持），提高加载速度。

##### 2.3.2.4 关闭串口输出.

可将uboot-board.dts或sys_config.fex中的[platform]下debug_mode设置为 0 来关闭
uboot的串口输出。

可将sys_config.fex中的[platform]下debug_mode设置为 0 来关闭boot0串口输出。

配置此项后，如果还有少量输出，有两个可能的原因：

第一是这些输出是在获取debug_mode流程之前产生。

第二是因为源码中直接使用了puts而没有使用printf。

对于这两者情况，需要修改源码来完全关闭串口输出。

##### 2.3.2.5 修改kernel加载位置

如果uboot将内核加载到DRAM的地址与内核中load address不匹配，就需要将内核移动到
正确位置，这样会浪费一定的时间。因此，可以直接修改uboot加载内核为正确的地址。

具体是修改env文件(路径见上文)的boot_normal与boot_recovery变量。

**需要根据不同的内核镜像格式来设置不同的值** 。

假设kernel的load address为0x40008000。

- 如果使用的是uImage，也就是在kernel的镜像前加了 64 字节，所以uboot应该将kernel 加载到0x40008000 - 0x40 = 0x40007fc0。

```
#uImage/raw
boot_normal=sunxi_flash read 40007fc0 ${boot_partition};bootm 40007fc
boot_recovery=sunxi_flash read 40007fc0 recovery;bootm 40007fc
```
- 如果使用的是boot.img，即android的kernel格式，其头部大小为0x800，所以uboot应该将kernel加载到0x40008000 - 0x800 = 40007800。


```
#boot.img/raw
boot_normal=sunxi_flash read 40007800 ${boot_partition};bootm 40007800
boot_recovery=sunxi_flash read 40007800 recovery;bootm 40007800
```

如果uboot加载kernel地址与load address不匹配，uboot过程中串口输出可能会有：

```
Loading Kernel Image ... OK
```

如果是匹配的，uboot过程中串口输出可能会有：

```
XIP Kernel Image ... OK
```

##### 2.3.2.6 修改kernel加载大小

最新代码会根据uImage/boot.img的头部信息，只读取必要的大小，可忽略此优化项。

对于旧代码，uboot在加载内核的时候，有些情况会直接将整个分区读取出来,uboot-2018会自
读取kernel镜像的大小。

就是说假如内核只有2M，而分区分了4M的话，uboot就会读取4M。这种情况下，可以将分
区大小设置得刚好容纳下内核，这样可避免uboot在加载内核的时候浪费时间。

nor方案修改sys_partiton_nor.fex的boot分区大小

nand/emmc可修改sys_partition.fex中boot分区的大小。

uboot具体读出多少，通常会有log信息，可同真正内核镜像的size进行比较。

##### 2.3.2.7 关闭kernel校验

uboot加载了内核以后，默认会对内核进行校验，可以在串口输出中看到：

```
Verifying Checksum ... OK
```

如果不想校验可以去掉，目前的情况是可以减少几十毫秒(不同平台，不同内核大小，时间不同)
的启动时间。

具体修改env配置文件（路径见上文），新增一行"verify=no"。

##### 2.3.2.8 uboot重定位

目前的启动过程中，uboot在执行过程中会进行一次重定位，可以在串口中打印出这个值，然后
修改uboot的加载地址使得boot0将uboot加载进DRAM的时候就直接加载到这个地址。

- 对于uboot-2014版本的位置为tina/lichee/brandy/u-boot*/include/configs/sun*iw*p*.h
    中的

```c
c #define CONFIG_SYS_TEXT_BASE=0x
```

- 对于uboot-2018在对应的configs/sun*iw*p*_defconfig文件中

```c
c CONFIG_SYS_TEXT_BASE=0x
```

但这个方法有个弊端，如果后续修改了uboot的代码，则可能需要重新设置。

目前这个操作耗时很少（某平台测得十几毫秒），不必要的话不建议做这个修改。

##### 2.3.2.9 裁剪uboot.

即使流程没有简化，uboot体积的减小也可减少加载uboot的时间。

依据具体情况，可对uboot不需要的功能的模块进行裁剪，避免了启动中执行不必要的流程，可
减少uboot加载时间。

##### 2.3.2.10开启logo及音乐.

可尝试在uboot中开启开机logo/音乐，尽快播出第一帧/声，提升用户体验。

此操作会延缓到达OS/APP的时间，但如果产品定义/用户体验是以第一帧/声为准的话，则有较
大价值。

#### 2.3.3 kernel启动优化.

通常来说，内核启动耗时较多，需要更深入的优化。

##### 2.3.3.1 kernel压缩方式.

比较不同压缩方式的启动时间和flash占用情况，选择一种符合实际情况的。

此处给出某次测试结果供参考。实际优化的时候，需要重新测试，根据实际情况选择。

|压缩方式 |内核大小/M |加载时间/s |解压时间/s |总时间/s|
|:---|:---|:---|:---|:---|
|LZO |2.4 |0.38 |0.23 |0.61|
|GZIP |1.9 |0.35 |0.44 |0.79|
|XZ |1.5 |0.25 |2.17 |2.42|


##### 2.3.3.2 加载位置

内核镜像可以由kernel自解压，也有uboot进行解压的情况。

对于kernel自解压的情况，如果压缩过的kernel与解压后的kernel地址冲突，则会先把自己
复制到安全的地方，然后再解压，防止自我覆盖。这就需要耗费复制的时间。

比如对于运行地址为0x40008000的内核来说，bootloader可以将其加载到0x41008000，
当然其他位置也可以。

##### 2.3.3.3 内核裁剪

裁剪内核，带来的加速是两个方面的。一是体积变小，加载解压耗时减少；二是内核启动时初始

化内容变少。

裁剪要根据产品的实际情况来，将不需要的功能及模块都去掉。

具体是执行"make kernel_menuconfig"，关闭不需要的选项。可参考《TinaLinux_系统裁剪开发指
南.pdf》。

##### 2.3.3.4 预设置lpj数值

LPJ也就是loops_per_jiffy，每次启动都会计算一次，但如果没有做修改的话，这个值每次启动
算出来都是一样的，可以直接提供数值跳过计算。

如下log所示，有skipped，lpj由timer计算得来，不需要再校准calibrate了。

```
[ 0.019918] Calibrating delay loop (skipped), value calculated using timer frequency..
48.00 BogoMIPS (lpj=240000)
```

如果没有skipped，则可以在cmdline中添加lpj=XXX进行预设。

##### 2.3.3.5 initcall优化

在cmdline中设置initcall_debug=1，即可打印跟踪所有内核初始化过程中调用initcall的顺
序以及耗时。

具体修改env配置文件（路径见上文），新增一行"initcall_debug=1"，并在"setargs_*"后加入"
initcall_debug=${initcall_debug}"，如下所示。

```
setargs_nand=setenv bootargs console=${console} console=tty0 root=${nand_root} init=${init}
loglevel=${loglevel} partitions=${partitions} initcall_debug=${initcall_debug}
```

加入后，内核启动时就会有类似如下的打印，对于耗时较多的initcall，可进行深入优化。


```
[ 0.021772] initcall sunxi_pinctrl_init+0x0/0x44 returned 0 after 9765 usecs
[ 0.067694] initcall param_sysfs_init+0x0/0x198 returned 0 after 29296 usecs
[ 0.070240] initcall genhd_device_init+0x0/0x88 returned 0 after 9765 usecs
[ 0.080405] initcall init_scsi+0x0/0x90 returned 0 after 9765 usecs
[ 0.090384] initcall mmc_init+0x0/0x84 returned 0 after 9765 usecs
```
##### 2.3.3.6 内核initcall module并行

内核initcall有很多级别，其中启动中最耗时的就是各module的initcall，针对多核方案，可
以考虑将module initcall并行执行来节省时间。

目前内核do_initcalls是一个一个按照顺序来执行，可以修改成新建内核线程来执行。

> 注：当前Tina还未加入该优化。

##### 2.3.3.7 减少pty/tty个数

加入initcall打印之后，部分平台发现pty/tty init耗时很多，可减少个数来缩短init时间。

```
initcall pty_init+0x0/0x3c4 returned 0 after 239627 usecs
initcall chr_dev_init+0x0/0xdc returned 0 after 36581 usecs
```
##### 2.3.3.8 内核module.

需要考虑启动速度的界定，对于内核module的优化主要有两点：

- 对于必须要加载的模块，直接编译进内核
- 对于不急需的功能，可以编译成模块。

比如某个应用，会开启主界面联网，启动速度以出现主界面为准，那么可以考虑将disp编入内
核，wifi编译成模块，后续需要时再动态加载。

##### 2.3.3.9 Deferred Initcalls

介绍页面及patch：http://elinux.org/Deferred_Initcalls

打上这个patch之后，可以标记一些initcall为Deferred_Initcall。这些被标记的初始化函
数，在系统启动的时候不会被调用

进入文件系统后，在合适的时间，比如启动主应用之后，再通过文件系统接口，启动这些推迟了
的调用，彻底完成初始化。


#### 2.3.4 rootfs启动优化

rootfs启动优化主要是优化rootfs的挂载到init进程执行。

##### 2.3.4.1 initramfs

initramfs是一个内存文件系统，会占用较多DRAM。

部分产品可能会用到initramfs来过渡到rootfs，其优化思路大体与rootfs类似。可参考本节后
续的优化方案。

##### 2.3.4.2 rootfs类型以及压缩.

存储介质、文件系统类型，压缩方式对rootfs挂载有很大影响。

此处给出某次测试结果供参考。实际优化的时候，需要重新测试，根据实际情况选择。

|类型 |压缩 |介质| 总时间/s|
|:---|:---|:---|:---|
|squashfs |gzip |emmc |0.12|
|squashfs| xz| emmc |0.27|
|squashfs |xz |nand |0.26|
|ext4 |- |emmc |0.12|


##### 2.3.4.3 rootfs裁剪

文件系统越小，加载速度越快。裁剪的主要思路是：删换压，即删除没有用到的，用小的换大

的，选择合适的压缩方式。

##### 2.3.4.4 指定文件系统类型

内核在挂载rootfs时，会有一个try文件系统类型的过程。可以在cmdline直接指定，节省时
间。

具体是在cmdline中添加"rootfstype=<type>"，其中type为文件系统类型，如ext4、squashfs
等。


##### 2.3.4.5 静态创建dev节点.

对于dev下面的节点，事先根据实际情况创建好，而不是在系统启动后动态生成，理论上也可以
节省一定的时间。

##### 2.3.4.6 rootfs拆分

可以将rootfs拆分成两个部分，一个小的文件系统先挂载执行，大的文件系统根据需要动态挂
载。

#### 2.3.5 主应用程序启动优化.

主应用程序主要是由客户开发，因此主导优化的还是客户，这里提一些优化措施：

- 提升运行顺序。将应用程序放在init很前面执行。
- 动态/静态链接。
- 编译选项。
- 暂时不使用的库采用dlopen方式。
- 应用程序拆分。


## 3 Tina启动速度优化

Tina中启动优化主要依靠宏CONFIG_BOOT_TIME_OPTIMIZATION来完成，该宏会进行如
下工作：

- 调整Linux内核镜像的压缩方式，调整rootfs的压缩方式。具体如何调整需要依据具体方案进
    行预先设定。
- 使boot0、uboot、kernel的打印不会输出到控制台。具体是在scripts/pack_img.sh脚本
    中完成。
- uboot加载内核时不进行校验。具体是在scripts/pack_img.sh脚本中完成。
- 设置内核命令行参数rootfstype，某些情况下会加快根文件系统的加载。具体是在scripts/-
    pack_img.sh脚本中完成。
- 部分方案会调整kernel镜像的加载地址。具体是在scripts/pack_img.sh脚本中完成。

> 注：通过该宏预计可达到70%左右的优化效果，如还需优化，可参考第二章的内容。

### 3.1 开启Tina启动速度优化.

在tina根目录下执行make menuconfig使能CONFIG_BOOT_TIME_OPTIMIZATION，
具体如下所示

```
Tina Configuration
    └─> Target Images
        └─>[*] kernel compression mode setting ----
            └─>Compression (Gzip) --->
                └─> ( ) Gzip
                ( ) LZMA
                ( ) XZ
                (X) LZO
        └─>[*] Boot Time Optimization
```


![图3-1: Tina menuconfig](./images/3-1.jpg)


> 注：如果看不到该选项，使用？键搜索，会发现此项有一些依赖选项，使能依赖选项即可看到

Boot Time Optimization

### 3.2 实验结果

在某norflash方案上开启CONFIG_BOOT_TIME_OPTIMIZATION后，启动速度提升效果
如下：

- Linux内核镜像压缩方式从GZIP换成LZO，优化> 0.2s。
- rootfs从squashfs XZ压缩换成squashfs GZIP压缩，优化> 0.15s。
- 屏蔽boot0、uboot、kernel启动阶段控制台打印，优化> 2s。
- 取消内核加载时的校验，优化0.3～0.4s。

> 注：对于不同的方案，由于CPU运算速度、存储器类型、内核压缩及尺寸、根文件系统类型及尺

寸、主应用等的不同，优化结果会有一定差异，请以实际优化结果为准。


## 4 参考资料

- [1] https://elinux.org/Boot_Time
- [2] https://docs.blackfin.uclinux.org/doku.php?id=fast_boot_example
- [3] https://github.com/tbird20d/grabserial
- [4] [http://www.bootchart.org](http://www.bootchart.org)
- [5] A Framework for Optimization of the Boot Time on Embedded Linux Environment
with Raspberry Pi Platform

