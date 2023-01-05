## 2 Tina系统裁剪简介

Tina固件中通常包含boot0、uboot、kernel、rootfs等镜像。基于经验，各个镜像尺寸的量级如下表所示：

**表2-1:各镜像尺寸的量级**

| 镜像   | 大小         |
| :----- | :----------- |
| boot0  | < 100K       |
| uboot  | < 1M         |
| kernel | >= 3M，< 15M |
| rootfs | >= 4M        |

可以看到boot0、uboot、kernel、rootfs的尺寸是依次增大的。对于大尺寸的裁剪效果往往比小尺寸的裁剪效果明显，比如rootfs裁剪1M可能很容易，对于

uboot来说，则非常困难。

因此，后续主要介绍kernel以及rootfs的裁剪。

### 2.1 boot0裁剪

由于boot0很小，通常来说boot0代码也不开源，因此略过。

### 2.2 uboot裁剪

目前tina环境中有不同版本的uboot，分别存在两个不同的文件路径中，以实际sdk的目录为准，cboot可以进入到相应的uoot目录，这两个路径分别为

```
lichee/brandy/u-boot或者lichee/brandy-2.0/u-boot
```

主要有下面两种裁剪思路：

- 修改uboot配置文件，删减不需要的配置。uboot配置文件通常位于源码下include/configs/${CHIP}.h或者configs/${CHIP}_*_defconfig。
- 删除不需要的uboot命令。


### 2.3 内核裁剪

通常关于Linux内核裁剪主要有如下方法：

- 删除不使用的功能。如符号表、打印、调试等功能。
- 删除不使用的驱动。
- 修改内核源代码。
- 内核压缩。

#### 2.3.1 删除不使用的功能

下表中列出了一些内核选项，包含选项的描述，默认值以及推荐值（减小内核镜像尺寸）。

**表2-2:内核选项及描述**

| CONFIG option        | Description                               | Def  | Small |
| :------------------- | :---------------------------------------- | :--- | :---- |
| CORE_SMALL           | tune some kernel data sizes               | N    | Y     |
| NET_SMALL            | tune some net-related data sizes          | N    | Y     |
| KMALLOC_ACCOUNTING   | turn on kmalloc accounting                | N    | Y *   |
| AUDIT_BOOTMEM        | print out all bootmem allocations         | N    | Y *   |
| DEPRECATE_INLINES    | cause compiler to emit info about inlines | N    | Y *   |
| PRINTK               | printk code and message data              | Y    | N     |
| BUG                  | allow elimination of BUG code             | Y    | N     |
| ELF_CORE             | allow disabling of ELF core dumps         | Y    | N     |
| PROC_KCORE           | allow disabling of /proc/kcore            | Y    | N     |
| AIO allow            | disabling of async IO syscalls            | Y    | N     |
| XATTR allow          | disabling of xattr syscalls               | Y    | N     |
| FILE_LOCKING         | allow disabling of file locking syscalls  | Y    | N     |
| DIRECTIO             | allow disabling of direct IO support      | Y    | N     |
| MAX_SWAPFILES_SHIFT  | number of swapfiles                       | 5    | 0     |
| NR_LDISCS            | number of tty line disciplines            | 16   | 2     |
| MAX_USER_RT_PRIO     | number of RT priority levels              | 100  | 5     |
| KALLSYMS             | load all symbols for debugging/kksymoops  | Y    | N     |
| SHMEM                | allow disabling of shmem filesystem       | Y    | N +   |
| SWAP                 | support for a swap segment                | Y    | N     |
| SYSV_IPC             | support for System V IPC                  | Y    | N +   |
| POSIX_MQUEUE         | POSIX message queue support               | Y    | N +   |
| SYSCTL               | allow disabling of sysctl support         | Y    | N +   |
| LOG_BUF_SHIFT        | control size of kernel printk buffer      | 14   | 11    |
| CC_OPTIMIZE_FOR_SIZE | Use gcc -os to optimize for size          | Y    | Y     |
| MODULES              | allow support for kernel loadable modules | Y    | N +   |
| KMOD                 | automatic kernel module loading           | Y    | N     |
| PCI                  | allow support for PCI bus and devices     | Y    | Y -   |
| XIP_KERNEL           | allow support for kernel Execute-in-Place | N    | N     |
| BLK_DEV_LOOP         | support for loopback block device         | Y    | Y -   |
| BLK_DEV_RAM          | block devices for RAM filesystems         | Y    | Y -   |
| IOSCHED_AS           | Include Anticipatory IO scheduler         | Y    | Y     |
| IOSCHED_DEADLINE     | Include Deadline IO scheduler             | Y    | N +   |
| IOSCHED_CFQ          | Include CFQ IO scheduler                  | Y    | N +   |
| IP_PNP               | support for IP autoconfiguration          | Y    | N +   |
| IP_PNP_DHCP          | support for IP autoconfiguration via DHCP | Y    | N +   |
| IDE                  | support for IDE devices                   | Y    | N +   |
| SCSI                 | support for SCSI devices                  | Y    | N +   |


其中：

- "Y *"-表示开发的时候设置成Y，发布的时候可以设置成N。
- "N +"-表示基于应用需要来判断是否设置成N。
- "Y -"-表示可能需要，可以设置N尝试一下。

在Tina中，集成了CONFIG_REDUCE_KERNEL_SIZE宏。一旦使能该宏后，将会采用部分上面的裁剪措施来减小kernel镜像尺寸，主要思路是关闭与log/debug等相关的配置，然后对kernel进行xz压缩，可参考tina/scripts/reduce-kernel-size.sh。

执行make menuconfig，开启如下选项：

```
Tina Configuration
    Target Images --->
        [*] downsize the kernel size (EXPERIMENTAL)
```


> 说明：此功能当前是 EXPERIMENTAL 的。建议直接执行 make kernel_menuconfig ，然后按照上述表格来配置。

#### 2.3.2 删除不使用的驱动

方案明确之后，所需的内核驱动也明确了。可以执行make kernel_menuconfig，将没有用到的驱动关闭。


#### 2.3.3 修改内核源代码

内核源码庞大，直接修改往往难度很大，可借助相关工具来评估模块以及符号的大小，然后进行针对性的裁剪。

##### 2.3.3.1 size工具.

size命令可查看内核镜像的text、data、bss等段的大小。如执行"size vmlinux"，将会得到：

```
text data bss dec hex filename
5818117 1378944 168972 7366033 706591 vmlinux
```

##### 2.3.3.2 ksize.py脚本

在tina/lichee/linux-4.9/scripts目录下有一个ksize脚本，可以对内核目录下的built-in.o进行解析，并将解析的内容按照尺寸进行排序，显示出来。执行结果如下所示：

```shell
Linux Kernel total | text data bss
--------------------------------------------------------------------------------
vmlinux 7366033 | 5818117 1378944 168972
--------------------------------------------------------------------------------
drivers/built-in.o 2244823 | 2080782 123885 40156
net/built-in.o 1682005 | 1630911 32590 18504
fs/built-in.o 975830 | 950780 5442 19608
kernel/built-in.o 678363 | 593347 41064 43952
mm/built-in.o 302442 | 272965 7309 22168
sound/built-in.o 237890 | 227338 6836 3716
security/built-in.o 170272 | 145055 13989 11228
block/built-in.o 149110 | 145408 2458 1244
crypto/built-in.o 145972 | 131610 7258 7104
lib/built-in.o 141721 | 141093 559 69
init/built-in.o 33551 | 18558 14909 84
ipc/built-in.o 29998 | 29218 772 8
usr/built-in.o 138 | 138 0 0
--------------------------------------------------------------------------------
sum 6792115 | 6367203 257071 167841
delta 573918 | -549086 1121873 1131
drivers total | text data bss
--------------------------------------------------------------------------------
drivers/built-in.o 2244823 | 2080782 123885 40156
--------------------------------------------------------------------------------
drivers/usb/built-in.o 448756 | 409279 27813 11664
drivers/block/built-in.o 357202 | 324752 21582 10868
drivers/tty/built-in.o 174213 | 155371 13938 4904
drivers/base/built-in.o 157961 | 153861 3460 640
drivers/mmc/built-in.o 133678 | 131782 1756 140
drivers/scsi/built-in.o 105021 | 95105 9348 568
drivers/md/built-in.o 100909 | 98382 1291 1236
drivers/mtd/built-in.o 96023 | 92244 1467 2312
drivers/hid/built-in.o 86072 | 81552 4160 360
drivers/clk/built-in.o 69737 | 58289 10856 592
drivers/cpufreq/built-in.o 51525 | 44400 1793 5332
drivers/pinctrl/built-in.o 50463 | 46458 3921 84
drivers/input/built-in.o 45250 | 44046 1156 48
drivers/i2c/built-in.o 43511 | 42791 656 64
drivers/spi/built-in.o 39888 | 38323 1557 8
drivers/thermal/built-in.o 38654 | 36673 1893 88
drivers/regulator/built-in.o 36217 | 35257 820 140
drivers/of/built-in.o 35994 | 35095 407 492
drivers/gpio/built-in.o 29432 | 29167 224 41
drivers/leds/built-in.o 25548 | 25076 464 8
drivers/rtc/built-in.o 25072 | 24428 460 184
drivers/tee/built-in.o 24823 | 24662 113 48
drivers/char/built-in.o 23718 | 21642 1224 852
drivers/soc/built-in.o 20916 | 13980 6804 132
drivers/bluetooth/built-in.o 20223 | 19319 100 804
drivers/dma/built-in.o 17892 | 17458 334 100
drivers/irqchip/built-in.o 14767 | 12371 2308 88
drivers/pwm/built-in.o 14636 | 14036 472 128
drivers/dma-buf/built-in.o 13975 | 13904 23 48
drivers/cpuidle/built-in.o 12613 | 10848 1749 16
drivers/watchdog/built-in.o 9986 | 9660 281 45
drivers/power/built-in.o 9836 | 8296 1224 316
drivers/clocksource/built-in.o 9608 | 8708 796 104
drivers/misc/built-in.o 8471 | 8105 340 26
drivers/bus/built-in.o 6357 | 5691 618 48
drivers/hwmon/built-in.o 5230 | 5054 144 32
drivers/hwspinlock/built-in.o 4792 | 4664 128 0
drivers/firmware/built-in.o 4453 | 4384 9 60
drivers/reset/built-in.o 3818 | 3686 132 0
drivers/net/built-in.o 1803 | 1755 48 0
drivers/mfd/built-in.o 1623 | 1511 108 4
drivers/video/built-in.o 379 | 379 0 0
--------------------------------------------------------------------------------
sum 2381045 | 2212444 125977 42624
delta -136222 | -131662 -2092 -2468
net total | text data bss
--------------------------------------------------------------------------------
net/built-in.o 1682005 | 1630911 32590 18504
--------------------------------------------------------------------------------
net/ipv4/built-in.o 428233 | 401161 14719 12353
net/mac80211/built-in.o 302085 | 301822 259 4
net/core/built-in.o 267334 | 256693 8573 2068
net/bluetooth/built-in.o 227913 | 226708 1033 172
net/wireless/built-in.o 160236 | 158651 557 1028
net/xfrm/built-in.o 74537 | 72737 1384 416
net/bridge/built-in.o 59936 | 58812 1112 12
net/sched/built-in.o 29706 | 28344 1346 16
net/packet/built-in.o 26453 | 26172 281 0
net/netlink/built-in.o 26105 | 25498 455 152
net/unix/built-in.o 24671 | 22266 340 2065
net/key/built-in.o 21095 | 20783 308 4
net/*.o 16279 | 15811 412 56
net/8021q/built-in.o 15245 | 14981 264 0
net/ipv6/built-in.o 9445 | 8295 1136 14
net/rfkill/built-in.o 7142 | 6710 408 24
net/ethernet/built-in.o 2431 | 2391 40 0
net/llc/built-in.o 2068 | 1980 72 16
net/802/built-in.o 1944 | 1792 140 12
--------------------------------------------------------------------------------
sum 1702858 | 1651607 32839 18412
delta -20853 | -20696 -249 92
fs total | text data bss
--------------------------------------------------------------------------------
fs/built-in.o 975830 | 950780 5442 19608
--------------------------------------------------------------------------------
fs/*.o 351665 | 339511 1774 10380
fs/ext4/built-in.o 295807 | 294110 1125 572
fs/jffs2/built-in.o 99642 | 99446 124 72
fs/proc/built-in.o 77696 | 73111 377 4208
fs/fat/built-in.o 49264 | 49088 144 32
fs/jbd2/built-in.o 47379 | 47254 65 60
fs/overlayfs/built-in.o 24939 | 24842 93 4
fs/squashfs/built-in.o 23156 | 23092 60 4
fs/kernfs/built-in.o 21086 | 16863 111 4112
fs/configfs/built-in.o 18193 | 17916 261 16
fs/debugfs/built-in.o 16120 | 16056 52 12
fs/pstore/built-in.o 13904 | 13531 325 48
fs/crypto/built-in.o 13083 | 12799 264 20
fs/notify/built-in.o 12525 | 12186 227 112
fs/nls/built-in.o 11024 | 10904 116 4
fs/sysfs/built-in.o 7041 | 6990 39 12
fs/devpts/built-in.o 3359 | 2986 365 8
fs/ramfs/built-in.o 1820 | 1776 40 4
--------------------------------------------------------------------------------
sum 1087703 | 1062461 5562 19680
delta -111873 | -111681 -120 -72
kernel total | text data bss
--------------------------------------------------------------------------------
kernel/built-in.o 678363 | 593347 41064 43952
--------------------------------------------------------------------------------
kernel/*.o 376931 | 346029 17531 13371
kernel/sched/built-in.o 127386 | 119841 6265 1280
kernel/time/built-in.o 101386 | 89633 7465 4288
kernel/printk/built-in.o 55033 | 18477 8444 28112
kernel/irq/built-in.o 47323 | 44325 906 2092
kernel/rcu/built-in.o 29815 | 27655 2131 29
kernel/locking/built-in.o 25617 | 25592 21 4
kernel/power/built-in.o 16652 | 15256 848 548
kernel/bpf/built-in.o 8348 | 7988 68 292
--------------------------------------------------------------------------------
sum 788491 | 694796 43679 50016
delta -110128 | -101449 -2615 -6064
sound total | text data bss
--------------------------------------------------------------------------------
sound/built-in.o 237890 | 227338 6836 3716
--------------------------------------------------------------------------------
sound/soc/built-in.o 125556 | 119220 5416 920
sound/core/built-in.o 108923 | 104799 1400 2724
sound/*.o 6612 | 6440 28 144
--------------------------------------------------------------------------------
sum 241091 | 230459 6844 3788

delta -3201 | -3121 -8 -72
security total | text data bss
--------------------------------------------------------------------------------
security/built-in.o 170272 | 145055 13989 11228
--------------------------------------------------------------------------------
security/selinux/built-in.o 142606 | 119156 12250 11200
security/keys/built-in.o 31690 | 30618 788 284
security/*.o 25826 | 24091 1719 16
security/integrity/built-in.o 1838 | 1806 20 12
--------------------------------------------------------------------------------
sum 201960 | 175671 14777 11512
delta -31688 | -30616 -788 -284
block total | text data bss
--------------------------------------------------------------------------------
block/built-in.o 149110 | 145408 2458 1244
--------------------------------------------------------------------------------
block/*.o 145968 | 142021 2699 1248
block/partitions/built-in.o 7563 | 7543 16 4
--------------------------------------------------------------------------------
sum 153531 | 149564 2715 1252
delta -4421 | -4156 -257 -8
lib total | text data bss
--------------------------------------------------------------------------------
lib/built-in.o 141721 | 141093 559 69
--------------------------------------------------------------------------------
lib/*.o 216133 | 214935 1092 106
lib/zlib_inflate/built-in.o 11187 | 11187 0 0
lib/xz/built-in.o 8215 | 8179 36 0
lib/lzo/built-in.o 2551 | 2551 0 0
lib/lz4/built-in.o 1188 | 1188 0 0
--------------------------------------------------------------------------------
sum 239274 | 238040 1128 106
delta -97553 | -96947 -569 -37
```

