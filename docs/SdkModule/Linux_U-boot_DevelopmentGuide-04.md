## 5 U-Boot 常用命令介绍

### 5.1 env 命令说明

通过env命令可以对{LICHEE_CHIP_CONFIG_DIR}/configs/default/env.cfg中的环境变量进行查看及更改。在小机启动过程中按任意键进入 U-Boot shell 命令状

态，输入命令"env"即可查看命令帮助信息。

具体示例如下：

1. 输入命令"env print"，可查看当前所有的环境变量信息，如下：

```
=> pri
ab_partition_list=bootloader,env,boot,vendor_boot,dtbo,vbmeta,vbmeta_system,vbmeta_vendor
android_trust_chain=true
boot_fastboot=fastboot
boot_normal=sunxi_flash read 45000000 boot;bootm 45000000
boot_recovery=sunxi_flash read 45000000 recovery;bootm 45000000
bootcmd=run setargs_mmc boot_normal
bootdelay=0
bootreason=charger
bt_mac=20:A1:11:12:13:44
cma=8M
console=ttyAS0,115200
earlyprintk=sunxi-uart,0x05000000
fdtcontroladdr=7bed0e60
fileaddr=40000000
filesize=15cf6
force_normal_boot=1
init=/init
initcall_debug=0
keybox_list=widevine,ec_key,ec_cert1,ec_cert2,ec_cert3,rsa_key,rsa_cert1,rsa_cert2,rsa_cert3
loglevel=8
mac=10:14:15:15:9A:CA
mmc_root=/dev/mmcblk0p4
nand_root=/dev/nand0p4
partitions=bootloader_a@mmcblk0p1:bootloader_b@mmcblk0p2:env_a@mmcblk0p3:env_b@mmcblk0p4:
boot_a@mmcblk0p5:boot_b@mmcblk0p6:vendor_boot_a@mmcblk0p7:vendor_boot_b@mmcblk0p8:
super@mmcblk0p9:misc@mmcblk0p10:vbmeta_a@mmcblk0p11:vbmeta_b@mmcblk0p12:
vbmeta_system_a@mmcblk0p13:vbmeta_system_b@mmcblk0p14:vbmeta_vendor_a@mmcblk0p15:
vbmeta_vendor_b@mmcblk0p16:frp@mmcblk0p17:empty@mmcblk0p18:metadata@mmcblk0p19:
private@mmcblk0p20:dtbo_a@mmcblk0p21:dtbo_b@mmcblk0p22:media_data@mmcblk0p23:
UDISK@mmcblk0p24
rotpk_status=0
setargs_mmc=setenv bootargs earlyprintk=${earlyprintk} clk_ignore_unused initcall_debug=${
initcall_debug} console=${console} loglevel=${loglevel} root=${mmc_root} init=${init}
cma=${cma} snum=${snum} mac_addr=${mac} wifi_mac=${wifi_mac} bt_mac=${bt_mac}
specialstr=${specialstr} gpt=1 androidboot.force_normal_boot=${force_normal_boot}
androidboot.slot_suffix=${slot_suffix}
setargs_nand=setenv bootargs earlyprintk=${earlyprintk} clk_ignore_unused initcall_debug=${
initcall_debug} console=${console} loglevel=${loglevel} root=${nand_root} init=${init}
cma=${cma} snum=${snum} mac_addr=${mac} wifi_mac=${wifi_mac} bt_mac=${bt_mac}
specialstr=${specialstr} gpt=1 androidboot.force_normal_boot=${force_normal_boot}
androidboot.slot_suffix=${slot_suffix}
slot_suffix=_a
snum=A100B3N041
wifi_mac=10:A1:11:12:13:44
Environment size: 2078/131068 bytes
=>
```

2. 输入命令"env set bootdelay 3"，可更改环境变量bootdelay（即 boot 启动时 log 中的倒计时延迟时间）值的大小。

3. 输入命令"env save"，即可将上述更改进行保存，保存后重新上电，或输入命令"reset"，即可看到上述更改bootdelay的延时时间被更改生效。

4. 其他env命令请查看env帮助信息。





### 5.2 sunxi_flash read 命令说明

#### 5.2.1 使用方法

用以下命令将 flash 指定地址中数据读到 DRAM 的指定地址处：

```
sunxi_flash read dram_addr flash_addr
```



#### 5.2.2 使用示例

```
sunxi_flash read 0x45000000 env—将env分区数据读到DRAM的0x45000000地址处
sunxi_flash read 45000000 boot;bootm 45000000—将flash中boot分区数据读到DRAM的0x45000000地		址,并 从0x45000000处启动。
```



### 5.3 fastboot 命令说明

​	fastboot 是 Android 平台上一个通用的刷机工具，也是一个很好的开发调试工具，以下介绍 fastboot 的基本使用方法。



#### 5.3.1 使用前提

fastboot PC 端工具可以从 Google Android SDK(Android-sdk-windows/tools) 中获得，也可以在 Android 源代码编译过后的生成文件获得 (out/host/linux-x86/bin)。 

在 Linux 系统中，使用 fastboot 不需要安装驱动。但在 Windows 系统中，使用 fastboot 前需安装 fastboot 相关驱动。adb 的驱动在 fastboot 模式下也可以安装成功，但是无法使用，请使用我们提供的驱动，并手动安装。



#### 5.3.2 使用步骤

1. 小机上电启动，按任意键进入 U-Boot 命令状态；

2. 串口端输入"fastboot"命令；

3. 打开 PC 端 fastboot 工具，并输入"fastboot devices"命令，看是否有 fastboot 设备显示；

4. 在正确获取 fastboot 设备的前提下，输入命令"fastboot flash env /path/to/env.fex"，将env.fex写到env分区（/path/to/目录下的env.fex中bootdelay值应该与 flash 中原有env中bootdelay值不同，这样可根据bootdelay值不同来确定 fastboot 烧写是否成功）, 同下载env.fex分区一样, 输入命令“fastboot flash boot /path/to/boot.img”将内核下载到内存中；

5. 输入"fastboot reboot"命令重启，查看启动倒计时即bootdelay的值是否改变；





#### 5.3.3 fastboot 基本命令使用示例

1. fastboot 几个基本命令示例如下:

   fastboot devices ：显示 fastboot 的设备。

   fastboot erase ：擦除分区，例如fastboot erase boot，擦除boot分区。

   fastboot flash：旧分区（待写分区），例如fastboot flash boot/path/to/boot.img，将boot.img写到boot分区。

2. 注意事项:

   fastboot 中使用的分区和sys_partition.fex中分区一致，具体的分区信息可以从小机上电启动进入 U-Boot shell 命令状态，输入命令"part list sunxi_flash 0"中获取，分区信息如下：

```
=> part list sunxi_flash 0

Partition Map for UNKNOWN device 0 -- Partition Type: EFI

Part         Start LBA          End LBA          Name
		     Attributes
	         Type GUID
		Partition GUID
1 		0x00008000 		0x00017fff 		"bootloader"
		attrs: 0x8000000000000000
		type: ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
		guid: a0085546-4166-744a-a353-fca9272b8e45
2 		0x00018000		0x0001ffff 		"env"
        attrs: 0x8000000000000000
        type: ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid: a0085546-4166-744a-a353-fca9272b8e46
3 		0x00020000 		0x0002ffff 		"boot"
        attrs: 0x8000000000000000
        type: ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid: a0085546-4166-744a-a353-fca9272b8e47
4 		0x00030000 		0x0032ffff 		"super"
		attrs: 0x8000000000000000
        type: ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid: a0085546-4166-744a-a353-fca9272b8e48
5 		0x00330000 		0x00337fff 		"misc"
        attrs: 0x8000000000000000
        type: ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid: a0085546-4166-744a-a353-fca9272b8e49
6 		0x00338000 		0x00347fff 		"recovery"
        attrs: 0x8000000000000000
        type: ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid: a0085546-4166-744a-a353-fca9272b8e4a
```



### 5.4 fat 命令说明

fat命令可以对 FAT 文件系统的相关存储设备进行查询及文件读写操作，在打包固件的时候, 我们会制作启动资源分区镜像, 把指定的目录下的文件按照文件系统的格式排布，文件中包括了原来目录中的所有文件，并完全按照目录结构排列。当把这个镜像文件烧写到存储设备上的某一个分区的时候，可以看到这个分区和原有目录的内容一样。使用fat可以方便地以文件和目录的方式对小机 flash 进行数据访问，如显示 logo。这些指令基本上要和 U 盘或者 SD 卡同时使用，主要用于读取这些移动存储器上的 FAT 分区。其相关操作命令如下：

1. fatls : 列出相应设备目录上的所有文件，示例如下图：

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxU-bootDevelopmentGuide_004.png)

​																	图 5-1: fatls 命令执行示例图

说明

**补充说明，fatls mmc 2:2 中的第一个 2 表示的是 emmc 设备，2 表示其分区号，其说明如下图：**

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxU-bootDevelopmentGuide_005.png)

​																	图 5-2: fatls 命令参数说明图



2. fatinfo: 打印出相应设备目录的文件系统信息，示例如下图：

   ![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxU-bootDevelopmentGuide_006.png)

​																	图 5-3: fatinfo 命令执行示例图



3. fatload: 从 FAT 文件系统中读取二进制文件到 RAM 存储中，示例如下：

```
sunxi#usb start
(Re)start USB...
USB0: start sunxi ehci1...
config usb pin success
config usb clk ok
sunxi ehci1 init ok...
USB EHCI 1.00
scanning bus 0 for devices... 3 USB Device(s) found
scanning usb for storage devices... 1 Storage Device(s) found
sunxi#fatls usb 0:1 /
16024600 sandisksecureaccessv3_win.exe
sandisk secureaccess/
lost.dir/
Android/
test/
video test/
amapauto/
0 vid_20161017_160818.ts
phoenixsuit/
system volume information/
0 vid_20161017_160919.ts
video/
156672 wifi pro_com su.exe
495 sys.ini
1035 pr_80211g_all.ini
config/
158208 wifi pro_new.exe
158208 wifi pro.exe
0 vid_20161017_164822.ts
0 vid_20161017_164906.ts
sunxi-tvd/
71149 sys_config.fex
vga/
397836884 system.img
14180352 boot.img
13 file(s), 13 dir(s)
sunxi#fatload usb 0:1 0x42000000 boot.img
reading boot.img
14180352 bytes read in 1149 ms (11.8 MiB/s)
sunxi#mmc dev 2
mmc2(part 0) is current device
sunxi#mmc write 0x42000000 0x15000 5000
MMC write: dev # 2, block # 86016, count 20480 ... 20480 blocks written: OK
```

说明：以上操作即将 U 盘的boot.img写到对应的 mmc 分区地址处。



4. fatwrite: 从内存中将对应的文件写到设备文件系统中。



### 5.5 md 命令说明

md命令可以对指定内存的数据进行查看，方便了解内存的数据情况及调试工作。其使用方法如下：

```
md 0xF0000000： 即用md命令查看内存DRAM 0xF0000000处内容
```



### 5.6 FDT 命令说明

FDT：flattened device tree 的缩写在 U-Boot 控制台停下后，输入fdt，可以查看fdt命令帮助。

```
sunxi#fdt
fdt - flattened device tree utility commands
Usage:
fdt addr [-c] <addr> [<length>] - Set the [control] fdt location to <addr>
fdt move <fdt> <newaddr> <length> - Copy the fdt to <addr> and make it active
fdt resize - Resize fdt to size + padding to 4k addr
fdt print <path> [<prop>] - Recursive print starting at <path>
fdt list <path> [<prop>] - Print one level starting at <path>
fdt get value <var> <path> <prop> - Get <property> and store in <var>
fdt get name <var> <path> <index> - Get name of node <index> and store in <var>
fdt get addr <var> <path> <prop> - Get start address of <property> and store in <var>
fdt get size <var> <path> [<prop>] - Get size of [<property>] or num nodes and store in <var>
fdt set <path> <prop> [<val>] - Set <property> [to <val>]
fdt mknode <path> <node> - Create a new node after <path>
fdt rm <path> [<prop>] - Delete the node or <property>
fdt header
fdt bootcpu <id> - Set boot cpuid
fdt memory <addr> <size> - Add/Update memory node
fdt rsvmem print - Show current mem reserves
fdt rsvmem add <addr> <size> - Add a mem reserve
fdt rsvmem delete <index> - Delete a mem reserves
fdt chosen [<start> <end>] - Add/update the /chosen branch in the tree
<start>/<end> - initrd start/end addr
NOTE: Dereference aliases by omiting the leading '/', e.g. fdt print ethernet0。
sunxi#
```

说明

**其中常用的命令就是fdt list 和 fdt set,fdt list 用来查询节点配置,fdt set 用来修改节点配置。**



#### 5.6.1 查询配置

首先确定要查询的字段在 device tree 的路径，如果不知道路径，则需要用fdt命令按以下步骤进

行查询。1. 在根目录下查找。

```
sunxi#fdt list /
/ {
    model = "{LICHEE_CHIP}";
    compatible = "arm,{LICHEE_CHIP}", "arm,{LICHEE_CHIP}";
    interrupt-parent = <0x00000001>;
    #address-cells = <0x00000002>;
    #size-cells = <0x00000002>;
    ......................
    cpuscfg {
    };
    ion {
    };
    dram {
    };
    memory@40000000 {
    };
    interrupt-controller@1c81000 {
    };
    sunxi-chipid@1c14200 {
    };
    timer {
    };
    pmu {
    };
    dvfs_table {
    };
    dramfreq {
    };
    gpu@0x01c40000 {
    };
    wlan {
    };
    bt {
    };
    btlpm {
    };
};
```

如果找到需要的配置，比如wlan的配置，运行如下命令即可。

```
sunxi#fdt list /wlan //注意路径中的 /
wlan {
    compatible = "allwinner,sunxi-wlan";
    clocks = <0x00000096>;
    wlan_power = "vcc-wifi";
    wlan_io_regulator = "vcc-wifi-io";
    wlan_busnum = <0x00000001>;
    status = "okay";
    device_type = "wlan";
    wlan_regon = <0x00000077 0x0000000b 0x00000002 0x00000001 0xffffffff 0xffffffff 0
    x00000000>;
    wlan_hostwake = <0x00000077 0x0000000b 0x00000003 0x00000006 0xffffffff 0xffffffff
    0x00000000>;
};
```



2. 在 soc目录下找。如果在第一步中没有发现要找的配置，比如nand0的配置，则该配置可能在soc目录下。

```
sunxi#fdt list /soc
soc@01c00000 {
    compatible = "simple-bus";
    #address-cells = <0x00000002>;
    #size-cells = <0x00000002>;
    ranges;
    device_type = "soc";
    ......................
    hdmi@01ee0000 {
    };
    tr@01000000 {
    };
    pwm@01c21400 {
    };
    nand0@01c03000 {
    };
    thermal_sensor {
    };
    cpu_budget_cool {
    };
    .......................
};
```

然后用如下命令显示即可:

```
sunxi#fdt list /soc/nand0
nand0@01c03000 {
    compatible = "allwinner,sun50i-nand";
    device_type = "nand0";
    reg = <0x00000000 0x01c03000 0x00000000 0x00001000>;
    interrupts = <0x00000000 0x00000046 0x00000004>;
    clocks = <0x00000004 0x0000007e>;
    pinctrl-names = "default", "sleep";
    pinctrl-1 = <0x00000081>;
    nand0_regulator1 = "vcc-nand";
    nand0_regulator2 = "none";
    nand0_cache_level = <0x55aaaa55>;
    nand0_flush_cache_num = <0x55aaaa55>;
    nand0_capacity_level = <0x55aaaa55>;
    nand0_id_number_ctl = <0x55aaaa55>;
    nand0_print_level = <0x55aaaa55>;
    nand0_p0 = <0x55aaaa55>;
    nand0_p1 = <0x55aaaa55>;
    nand0_p2 = <0x55aaaa55>;
    nand0_p3 = <0x55aaaa55>;
    status = "disabled";
    nand0_support_2ch = <0x00000000>;
    pinctrl-0 = <0x000000a9 0x000000aa>;
};
```

3. 使用路径别名查找。别名是 device tree 中完整路径的一个简写，有一个专门的节点 ( /aliases) 来表示别名的相关信息，用如下命令可以查看系统中别名的配置情况：

```
sunxi#fdt list /aliases
aliases {
    serial0 = "/soc@01c00000/uart@01c28000";
    ..............
    mmc0 = "/soc@01c00000/sdmmc@01c0f000";
    mmc2 = "/soc@01c00000/sdmmc@01C11000";
    nand0 = "/soc@01c00000/nand0@01c03000";
    disp = "/soc@01c00000/disp@01000000";
    lcd0 = "/soc@01c00000/lcd0@01c0c000";
    hdmi = "/soc@01c00000/hdmi@01ee0000";
    pwm = "/soc@01c00000/pwm@01c21400";
    boot_disp = "/soc@01c00000/boot_disp";
};
sunxi#
```

由于配置了nand0节点的路径别名，因此可以用如下命令来显示nand0的配置信息。

```
sunxi#fdt list nand0
nand0@01c03000 {
    compatible = "allwinner,sun50i-nand";
    device_type = "nand0";
    reg = <0x00000000 0x01c03000 0x00000000 0x00001000>;
    ..................
    pinctrl-names = "default", "sleep";
    pinctrl-1 = <0x00000081>;
};
```

注：在fdt的所有命令中，alias 可以用作path参数。

```
fdt list <path> [<prop>] - Print one level starting at <path>
fdt set <path> <prop> [<val>] - Set <property> [to <val>]
```



#### 5.6.2 修改配置

##### 5.6.2.1 修改整数配置

命令格式：fdt set path prop <xxx> 示例：fdt set /wlan wlan_busnum <0x2>

```
sunxi#fdt list /wlan
wlan {
    compatible = "allwinner,sunxi-wlan";
    clocks = <0x00000096>;
    wlan_power = "vcc-wifi";
    wlan_io_regulator = "vcc-wifi-io";
    wlan_busnum = <0x00000001>;
    status = "disable";
    device_type = "wlan";
};
sunxi#fdt set /wlan wlan_busnum <0x2>
sunxi#fdt list /wlan
wlan {
    compatible = "allwinner,sunxi-wlan";
    clocks = <0x00000096>;
    wlan_power = "vcc-wifi";
    wlan_io_regulator = "vcc-wifi-io";
    wlan_busnum = <0x00000002>; //修改后
    status = "disable";
    device_type = "wlan";
};
```

注：修改整数时，根据需要也可配置为数组形式，需要用空格来分隔。命令格式：fdt set path prop <0x1 0x2 0x3>



##### 5.6.2.2 修改字符串配置

命令格式：fdt set path prop "xxxxx" 示例：fdt set /wlan status "disable"

```
sunxi#fdt list /wlan
wlan {
    compatible = "allwinner,sunxi-wlan";
    clocks = <0x00000096>;
    wlan_power = "vcc-wifi";
    wlan_io_regulator = "vcc-wifi-io";
    wlan_busnum = <0x00000001>;
    status = "okay";
    device_type = "wlan";
};
sunxi#fdt set /wlan status "disable"
sunxi#fdt list /wlan
wlan {
    compatible = "allwinner,sunxi-wlan";
    clocks = <0x00000096>;
    wlan_power = "vcc-wifi";
    wlan_io_regulator = "vcc-wifi-io";
    wlan_busnum = <0x00000001>;
    status = "disable"; //修改后
    device_type = "wlan";
};
sunxi#
```

注：修改字符串时，根据需要也可配置为数组形式，需要用空格来分隔。命令格式：fdt set path prop "string1" "string2"



#### 5.6.3 GPIO 或者 PIN 配置特殊说明

接口对应的数字编号说明如下：

```
#define PA 0
#define PB 1
#define PC 2
#define PD 3
#define PE 4
#define PF 5
#define PG 6
#define PH 7
#define PI 8
#define PJ 9
#define PK 10
#define PL 11
#define PM 12
#define PN 13
#define PO 14
#define PP 15
#define default 0xffffffff
```

Sysconfig 中描述 gpio 的形式如下：Port:端口+组内序号<功能分配><内部电阻状态><驱动能力><输出电平状态>



##### 5.6.3.1 Pin 配置说明

Pinctrl 节点分为 cpux 和 cpus，对应的节点路径如下：Cpux : /soc/pinctrl@01c20800 Cpus:/soc/pinctrl@01f02c00



##### 5.6.3.2 查看 PIN 配置

PIN 配置属性字段说明:

| 属性字段           | 含义                                        |
| ------------------ | ------------------------------------------- |
| allwinner,function | 对应于 sysconfig 中的主键名                 |
| allwinner,pins     | 对应于 sysconfig 中每个 gpio 配置中的端口名 |
| allwinner,pname    | 对应于 sysconfig 中主键下面子键名字         |
| allwinner,muxsel   | 功能分配                                    |
| allwinner,pull     | 内部电阻状态                                |
| allwinner,drive    | 驱动能力                                    |
| allwinner,data     | 输出电平状态                                |



说明

**其中0xffffffff表示使用默认值。**



按以下方法查看cpux的 PIN 配置。

```
sunxi#fdt list /soc/pinctrl@01c20800/lcd0
lcd0@0 {
    linux,phandle = <0x000000ab>;
    phandle = <0x000000ab>;
    allwinner,pins = "PD12", "PD13", "PD14", "PD15", "PD16", "PD17", "PD18", "PD19", "PD20", "PD21";
    allwinner,function = "lcd0";
    allwinner,pname = "lcdd0", "lcdd1", "lcdd2", "lcdd3", "lcdd4", "lcdd5", "lcdd6", "lcdd7", "lcdd8", "lcdd9";
    allwinner,muxsel = <0x00000003>;
    allwinner,pull = <0x00000000>;
    allwinner,drive = <0xffffffff>;
    allwinner,data = <0xffffffff>;
};
sunxi#
```

按以下方法查看cpus的 PIN 配置。

```
sunxi# fdt list /soc/pinctrl@01f02c00/s_uart0
s_uart0@0 {
    linux,phandle = <0x000000b4>;
    phandle = <0x000000b4>;
    allwinner,pins = "PL2", "PL3";
    allwinner,function = "s_uart0";
    allwinner,pname = "s_uart0_tx", "s_uart0_rx";
    allwinner,muxsel = <0x00000002>;
    allwinner,pull = <0xffffffff>;
    allwinner,drive = <0xffffffff>;
    allwinner,data = <0xffffffff>;
};
sunxi#
```



##### 5.6.3.3 修改 PIN 配置

使用fdt set命令可以修改 PIN 中相关属性字段

```
sunxi#fdt set /soc/pinctrl@01c20800/lcd0 allwinner,drive <0x1>
sunxi#fdt list /soc/pinctrl@01c20800/lcd0
lcd0@0 {
    linux,phandle = <0x000000ab>;
    phandle = <0x000000ab>;
    allwinner,pins = "PD12", "PD13", "PD14", "PD15", "PD16", "PD17", "PD18", "PD19", "PD20", "PD21";
    allwinner,function = "lcd0";
    allwinner,pname = "lcdd0", "lcdd1", "lcdd2", "lcdd3", "lcdd4", "lcdd5", "lcdd6", "lcdd7", "lcdd8", "lcdd9";
    allwinner,muxsel = <0x00000003>;
    allwinner,pull = <0x00000000>;
    allwinner,drive = <0x00000001>;
    allwinner,data = <0xffffffff>;
};
```

说明

**示例中该处修改会影响allwinner,pins表示的所有端口的驱动能力配置，修改allwinner,muxsel, allwinner,pull,allwinner,data的值也会产生类似效果。**



##### 5.6.3.4 GPIO 配置说明

Device tree 中 GPIO 对应关系，以 usb 中usb_id_gpio为例

```
sunxi#fdt list /soc/usbc0
usbc0@0 {
    test = <0x00000002 0x00000003 0x12345678>;
    device_type = "usbc0";
    compatible = "allwinner,sun50i-otg-manager";
    ........
    usb_serial_unique = <0x00000000>;
    usb_serial_number = "20080411";
    rndis_wceis = <0x00000001>;
    status = "okay";
    usb_id_gpio = <0x00000030 0x00000007 0x00000009 0x00000000 0x00000001 0xffffffff 0xffffffff>;
};
```

对应于 device tree 中 usb_id_gpio = <0x00000030 0x00000007 0x00000009 0x00000000 0x00000001 0xffffffff 0xffffffff>，解释如下：

| 属性数值   | 含义                                           |
| ---------- | ---------------------------------------------- |
| 0x00000030 | device tree 内部一个节点相关信息，这里可以略过 |
| 0x00000007 | 端口 PH, 即 #define PH 7                       |
| 0x00000009 | 组内序号, 即 PH09                              |
| 0x00000000 | 功能分配, 即将 PH09 配为输入                   |
| 0x00000001 | 内部电阻状态, 即配为上拉                       |
| 0xffffffff | 驱动能力, 默认值                               |
| 0xffffffff | 输出电平, 默认值                               |



如果需要修改 usb_id_gpio的配置，可按如下方式（示例修改了驱动能力，输出电平两项）：

```
sunxi#fdt set /soc/usbc0 usb_id_gpio <0x00000030 0x00000007 0x00000009 0x00000000 0x00000001 0x2 0x1>
sunxi#fdt list
usbc0@0 {
    test = <0x00000002 0x00000003 0x12345678>;
    device_type = "usbc0";
    compatible = "allwinner,sun50i-otg-manager";
    ........
    usb_serial_unique = <0x00000000>;
    usb_serial_number = "20080411";
    rndis_wceis = <0x00000001>;
    status = "okay";
    usb_id_gpio = <0x00000030 0x00000007 0x00000009 0x00000000 0x00000001 0x00000002 0x00000001>; //修改ok
};
sunxi#
```



5.7 其他命令说明（boot, reset, efex）

1. boot : 启动内核

2. reset: 复位重启系统

3. efex: 进入烧录状态



说明

**注：其他更多 U-Boot 命令介绍，请进入 U-Boot shell 命令状态后输入"help"进行了解。**