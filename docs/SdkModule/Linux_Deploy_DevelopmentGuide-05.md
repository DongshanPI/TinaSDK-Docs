## 7 分区表

请参考，TinaLinux存储管理开发指南。


## 8 env

### 8.1 配置文件路径.

env.cfg 根据使用内核版本（linux3.4 或 linux3.10 或 linux4.4）分为 env-3.4.cfg，env-3.10.cfg，env-4.4.cfg，用于环境变量。

Tina下的配置文件可能有几个路径。

Tina 3.5.0及之前版本：

Tina默认配置文件路径：

```
target/allwinner/generic/configs
```

芯片默认配置文件路径：

```
target/allwinner/xxx-common/configs
```

具体方案配置文件路径：

```
target/allwinner/xxx-xxx/configs
```

Tina 3.5.1及之后版本：

芯片默认配置文件路径：

```
device/config/chips/${chip}/configs/default
```

具体方案配置文件路径：

```
device/config/chips/${chip}/configs/${borad}/linux
```

优先级依次递增，即优先使用具体方案下的配置文件，没有方案配置，则使用芯片默认配置文件，没有方案配置和芯片配置，才使用tina默认配置文件。

### 8.2 常用配置项说明


| 配置项        | 含义                                                         |
| :------------ | :----------------------------------------------------------- |
| bootdelay     | 串口选择是否进入uboot命令行的等待时间，单位秒。例如：为 0 时自动加载内核，为 3 则等待 3 秒，期间按任何按键都可进入uboot命令行。 |
| bootcmd       | 默认为run setargs_nand boot_normal，但uboot会根据实际介质正确修改setargs_nand，称为＂update_bootcmd＂。另外，在烧录固件时bootcmd会被修改成runsunxi_sprite_test，即此时不会去加载内核，而是去执行烧录固件命令。 |
| setargs_xxx   | 会去设置bootargs、console、root、init、loglevel、partitions，这些都是内核需要用到的环境变量。其中partitions会根据分区进行自适应。 |
| boot_normal   | 正常启动加载内核。                                           |
| boot_recovery | 正常启动加载恢复系统。                                       |
| boot_fastboot | 正常启动加载fastboot。                                       |
| console       | 设置内核的串口。                                             |
| loglevel      | 设置内核的log级别。                                          |
| verify        | 设置是否校验内核，默认会进行校验，设置为=no则不校验。        |
| cmd           | 设置cmd内存大小。                                            |

### 8.3 uboot中的修改方式.

进入uboot命令行执行env相关命令可以查看，修改，保存env环境变量。常用的命令如：

```
env print --打印所有环境变量。
env set bootdelay 1 --设置bootdelay为 1 。
env save --保存环境变量，env set之后需要执行env save才能真正写入flash保存。
```

### 8.4 用户空间的修改方式.

Tina中提供了uboot-envtools软件包，选中即可：

```
make menuconfig ---> Utilities ---> <*>uboot-envtools
```

可在用户空间，调用fw_setenv和fw_printenv，对env进行读写。

fw_printenv使用方法：

```
Usage: fw_printenv [OPTIONS]... [VARIABLE]...
Print variables from U-Boot environment
-h, --help print this help.
```

```
-v, --version display version
-c, --config configuration file, default:/etc/fw_env.config
-n, --noheader do not repeat variable name in output
-l, --lock lock node, default:/var/lock
```

fw_setenv使用方法：

```
fw_setenv: option requires an argument -- 'h'
Usage: fw_setenv [OPTIONS]... [VARIABLE]...
Modify variables in U-Boot environment
-h, --help print this help.
-v, --version display version
-c, --config configuration file, default:/etc/fw_env.config
-l, --lock lock node, default:/var/lock
-s, --script batch mode to minimize writes
Examples:
fw_setenv foo bar set variable foo equal bar
fw_setenv foo clear variable foo
fw_setenv --script file run batch script
Script Syntax:
key [space] value
lines starting with '#' are treated as comment
A variable without value will be deleted. Any number of spaces are
allowed between key and value. Space inside of the value is treated
as part of the value itself.
Script Example:
netdev eth0
kernel_addr 400000
foo empty empty empty empty empty empty
bar
```

## 9 nor/nand介质配置

由于spinor一般容量远小于其他介质，为此分离出了独立的分区表sys_partition_nor.fex。打包时需要特殊处理，跟其他介质不兼容。即，可以用一个固件，兼容nand和emmc，但不能兼容spinor。

一般需要跟spinor互换的，是spinand。

配置的目的是：

- 设置sys_config.fex，方便打包时进行判断
- 选上适配的驱动，即nor要选nor的驱动，nand要选nand的驱动
- 选上UDISK使用的文件系统支持，nor默认使用jffs2，nand默认使用ext4
- 选上对应的文件系统工具，nor需要mtd-utils，nand需要e2fsprogs

具体配置方法如下。

### 9.1 spinand切换为spinor.

#### 9.1.1 sys_config.

设置介质为nor：

```
[target]
storage_type = 3
```

配置所用nor的大小，如16M：

```
[norflash]
size = 16
```

#### 9.1.2 内核配置

```
make kernel_menuconfig --->
Device Drivers --->
< >Block devices (取消选中)
Device Drivers --->
<*>Memory Technology Device (MTD) support
<*>OpenFirmware partitioning information support
<*>SUNXI partitioning support
<*> Caching block device access to MTD devices
<*> SPI-NOR device support (对于linux4.9，先选这个，下面的选项才出现)
Self-contained MTD device drivers --->
<*> Support most SPI Flash chips (AT26DF, M25P, W25X, ...)
File systems --->
< > The Extended 4 (ext4) filesystem（取消选中）
File systems --->
[*] Miscellaneous filesystems --->
<*> Journalling Flash File System v2 (JFFS2) support（选中）
[*] Enable the block layer --->
[ ] Support for large (2TB+) block devices and files（取消选中）
```

#### 9.1.3 menuconfig配置

```
make menuconfig --->
Utilities --->
<*> mtd-utils (选择) --->
<*> mtd-utils-mkfs.jffs2
make menuconfig --->
Utilities --->
Filesystem --->
< > e2fsprogs(取消选择)
```

### 9.2 spinor切换为spinand.

#### 9.2.1 sys_config.

设置介质为spinand：

```
[target]
storage_type = 5
```

#### 9.2.2 内核配置

```
make kernel_menuconfig --->
Device Drivers --->
[*]Block devices --->
<*> sunxi nand flash driver
make kernel_menuconfig --->
Device Drivers --->
< >Memory Technology Device (MTD) support（取消选择）
make kernel_menuconfig --->
[*] Enable the block layer --->
[*] Support for large (2TB+) block devices and files
make kernel_menuconfig --->
File systems --->
<*> The Extended 4 (ext4) filesystem
```

#### 9.2.3 menuconfig配置

```
make menuconfig --->
Utilities --->
< > mtd-utils (取消选择)
make menuconfig --->
Utilities --->
Filesystem --->
<*> e2fsprogs
```

