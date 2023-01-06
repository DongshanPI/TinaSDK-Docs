# 3 系统挂载

Tina 目前支持两种启动方式，分别是busybox 和procd，不同启动方式，其自动挂载的配置不同。
此处的自动挂载指开机冷挂载以及热插拔挂载，其中冷挂载指启动时挂载，热挂载指TF/U 盘等插拔设备时的挂载。



## 3.1 块设备节点

Tina 中设备节点都在/dev 目录下，对于不同存储介质，生成的设备节点会不一样。

| 存储 | 介质设备                  | 节点备注   |
| ---- | ------------------------- | ---------- |
| nand | /dev/nand{a,b,c…}         | MBR 分区表 |
| nand | /dev/nand0p{1,2,3…}       | GPT 分区表 |
| mmc  | /dev/mmcblk0p{1,2,3…}     |            |
| nor  | /dev/mtdblock{0,1,2…}     |            |
| TF   | 卡/dev/mmcblk{0,1}p{1,2…} | 注1        |
| U 盘 | /dev/sda{1,2…}            |            |
| SATA | 硬盘/dev/sda{1,2…}        |            |

说明

1. 若使用mmc 做内部存储介质，由于mmc 占用了mmcblk0的设备名，此时TF 卡的设备名序号递增为mmcblk1，否则生
   成mmcblk0的设备名。因此配置fstab 时尤其注意TF 设备名是否正确。
   对sys_partition.fex 中设置的内部存储介质的设备节点，会自动动态在/dev/by-name 中创建软链
   接。例如：

```
root@TinaLinux:/# ll /dev/by-name/
drwxr-xr-x 2 root root 220 Mar 1 15:05 .
drwxr-xr-x 7 root root 3060 Mar 1 15:05 ..
lrwxrwxrwx 1 root root 12 Mar 1 15:05 UDISK -> /dev/nand0p9
lrwxrwxrwx 1 root root 12 Mar 1 15:05 boot -> /dev/nand0p2
lrwxrwxrwx 1 root root 12 Mar 1 15:05 env -> /dev/nand0p1
lrwxrwxrwx 1 root root 12 Mar 1 15:05 misc -> /dev/nand0p6
lrwxrwxrwx 1 root root 12 Mar 1 15:05 private -> /dev/nand0p8
lrwxrwxrwx 1 root root 12 Mar 1 15:05 pstore -> /dev/nand0p7
lrwxrwxrwx 1 root root 12 Mar 1 15:05 recovery -> /dev/nand0p5
lrwxrwxrwx 1 root root 12 Mar 1 15:05 rootfs -> /dev/nand0p3
lrwxrwxrwx 1 root root 12 Mar 1 15:05 rootfs_data -> /dev/nand0p4
```

因此，在fstab 也可以使用/dev/by-name/XXXX的形式匹配设备。块设备如果有分区，会形成分区设备节点，以mmc、U 盘为例介绍设备节点名与分区的关系：

| 设备节点名     | 含义                           |
| -------------- | ------------------------------ |
| /dev/mmcblk0   | 表示整个mmc 空间，包含所有分区 |
| /dev/mmcblk0p1 | 表示mmc 中的第1 个分区         |
| /dev/mmcblk0p2 | 表示mmc 中的第2 个分区         |
| /dev/sda       | 表示整个U 盘，包含所有分区     |
| /dev/sda1      | 表示U 盘内的第1 个分区         |
| /dev/sda2      | 表示U 盘内的第2 个分区         |

热插拔块设备分区有以下特殊情况。

1. 块设备没有分区
   有一些特殊的TF 卡/U 盘没有分区，而是直接使用整个存储，表现为只有/dev/mmcblk1 和
   /dev/sda ，而没有分区节点/dev/mmcblk1p1 和/dev/sda1 。此时需要直接挂载整个存储
   设备，Tina 大部分方案都支持这种特殊情况。
2. 块设备有多个分区
   有一些特殊的TF 卡/U 盘被分为多个分区，表现为存在多个/dev/mmcblk1p{1,2…} 和
   /dev/sda{1,2…}。默认情况下，Tina 的fstab 配置为只支持挂载热插拔存储设备的第一个
   分区到/mnt/SDCARD 或者/mnt/exUDISK。

## 3.2 挂载点

### 3.2.1 默认挂载设备目录

Tina 中对常见的分区和热插拔块设备，有默认的挂载点。



| 存储介质     | 挂载节点                 | 设备节点           | 备注 |
| ------------ | ------------------------ | ------------------ | ---- |
| nor/nand/mmc | /mnt/UDISK               | /dev/by-name/UDISK | 注1  |
| TF 卡        | /mnt/SDCARD 或/mnt/extsd | /dev/mmcblk{0,1}p1 | 注2  |
| U 盘         | /mnt/exUDISK             | /dev/sda1          | 注3  |
| SATA 磁盘    | /mnt/exUDISK             | /dev/sda1          | 注3  |

说明

1. /dev/by-name/UDISK 为sys_partition.fex 的UDISK 分区的软连接。
2. 当无分区时，默认挂载整个TF 卡; 当有1 个或多个分区时，只挂载第一分区。
3. 当无分区时，默认挂载整个设备(/dev/sda)，当有1 个或多个分区时，只挂载第一个分区。

### 3.2.2 新建挂载点

挂载文件系统需要有挂载点。
如果挂载点所在目录可写，则在挂载之前先创建目录即可。

```
mkdir -p xxx
```

若挂载点所在目录为只读，则需要在制作文件系统时提前创建好。
如创建非空目录，则在对应方案的base-files 目录创建。

```
procd-init: target/allwinner/方案/base-files
busybox-init: target/allwinner/方案/busybox-init-base-files
```

如创建空目录，由于git 不管理空目录，因此需在Makefile 中动态创建，可仿照现有Makefile中创建UDISK 目录的写法。

```
procd-init: package/base-files/Makefile
busybox-init: package/busybox-init-base-files/Makefile
```

## 3.3 procd 启动下的挂载

procd 启动时，自动挂载由procd、fstools、fstab 配合完成。如果需要修改冷/热挂载规则，只需要修改fstab 配置文件即可。
SDK 中，配置文件位于：

```
tina/target/allwinner/<方案名>/base-files/etc
```

若只是调试或临时修改挂载规则，只需要修改小机端的配置文件：

```
/etc/config/fstab
```

### 3.3.1 fstab 编写格式

fstab 由多个config 组成，每个config 的基本格式示例如下：

```
config ‘xxxx’
option xxxx ‘xx’
option xxxx ‘xx’
option xxxx ‘xx’
```

config 有3 种类型，分别是mount|global|swap 。Tina SDK 中没使用swap，在本文中不做介绍。

### 3.3.2 global 类型config

global 类型的config 是全局配置，示例如下。

```
config 'global'
option anon_swap '0'
option anon_mount '0'
option auto_swap '1'
option auto_mount '1'
option delay_root '5'
option check_fs '1'
```

配置项的意义如下表：

| 配置名称   | 可选值 | 意义                                 |
| ---------- | ------ | ------------------------------------ |
| anon_mount | 0/1    | 注1                                  |
| anon_swap  | 0/1    | swap 使用，此处省略                  |
| auto_mount | 0/1    | 只适用于设置热插拔是否自动挂载块设备 |
| auto_swap  | 0/1    | swap 使用，此处忽略                  |
| check_fs   | 0/1    | 建议配置为1，注2                     |
| delay_root | 1,2,3… | 注3                                  |

说明

1. anon_mount: 当fstab 中无匹配要挂载设备的uuid/label/device 属性的配置节时, 是否采用默认挂载为
   /mnt/“$device-name”。
2. check_fs: 是否在挂载前用/usr/sbin/e2fsck 检查文件系统一致性(只适用于ext 系统)。
3. delay_root: 对应fstab 中target 为”/” 或”/overlay” 的设备节点不存在时，最长等待delay_root 秒。

### 3.3.3 mount 类型config

mount 类型的config 是具体的设备挂载配置，示例如下

```
config 'mount'
option target '/mnt/UDISK'
option device '/dev/by-name/UDISK'
option options 'rw,sync'
option enabled '1'
```

配置项的意义如下表：

| 配置名称 | 意义      | 备注                                 |
| -------- | --------- | ------------------------------------ |
| target   | 挂载点    | 必须是绝对路径，必须有效             |
| device   | 设备名    | 通过设备名指定待挂载的设备，注1      |
| uuid     | 设备UUID  | 通过fs 的UUID 指定待挂载的设备，注1  |
| label    | 设备label | 通过fs 的label 指定待挂载的设备，注1 |
| enabled  | 是否使能  | 该节点是否有效(0/1)                  |
| options  | 挂载属性  | 例如只读挂载等，注2                  |



1. device/uuid/label 是匹配挂载的设备，三者中至少要有一个有效。

2. 默认挂载支持的属性如下表：


| 配置名称              | 意义                                            | 缺省值    |
| --------------------- | ----------------------------------------------- | --------- |
| ro / rw               | 只读/ 可读写                                    | rw        |
| nosuid / suid         | 忽略suid/sgid 的文件属性                        | suid      |
| nodev / dev           | 不允许/允许访问设备文件                         | dev       |
| noexec / exec         | 不允许/允许执行程序                             | exec      |
| sync / async          | 同步/异步写入                                   | async     |
| mand / nomand         | 允许/不允许强制锁                               | nomand    |
| irsync                | 同步更新文件夹                                  | 无效      |
| noatime / atime       | 不更新/更新访问时间(atime)                      | atime     |
| nodiratime / diratime | 不更新/更新目录访问时间(atime)                  | diratime  |
| relatime / norelatime | 允许/不允许根据ctime/mtime 更新actime n         | orelatime |
| strictatime           | 禁止根据内核行为来更新atime, 但允许用户空间修改 | 无效      |



## 3.4 busybox 启动下的挂载

busybox 启动时，通过pseudo_init 和rcS 完成大部分默认的挂载工作。

| 存储节点                 | 挂载路径                 | 用途                        |
| ------------------------ | ------------------------ | --------------------------- |
| /dev/by-name/UDISK       | /mnt/UDISK               | 用户数据                    |
| /dev/by-name/rootfs_data | /overlay                 | 作为overlay 使得rootfs 可写 |
| /dev/mmcblk{0,1}p1       | /mnt/SDCARD 或/mnt/extsd | TF 卡                       |
| /dev/sda1                | /mnt/exUDISK             | U 盘                        |

busybox 启动使用默认挂载配置即可，如果需要修改，需要自行修改脚本。

```
tina/package/busybox-init-base-files/busybox-init-base-files/usr/bin/hotplug.sh
```



## 3.5 挂载文件系统

在分区表中增加的分区默认是空分区，如需挂载成文件系统使用，则首先需要在分区中写入一个文件系统。
方式一，在PC 端预先生成好一个文件系统，并在分区表中指定为download_file，则启动后可直接挂载。例如rootfs 分区就是在PC 端制作好文件系统，烧录时写入rootfs 分区。
方式二，在小机端进行格式化。例如UDISK 分区就是在第一次启动时，由启动脚本进行格式化。客户可自行在某一启动脚本或应用中调用格式化工具（mkfs.xxx）进行格式化。如需参考，可仿照UDISK 分区的格式化：

```
procd-init: package/base-files/files/lib/preinit/79_format_partition
busybox-init: package/busybox-init-base-files/files/pseudo_init
```

### 3.5.1 注意事项

一些格式化工具并未默认选中，需要时请自行在make menuconfig 界面配置。部分文件系统对分区大小有最低要求，如ext4，ubifs，如果在小机端调用格式化分区时报错，可根据报错信息提示增大分区。
对于private 分区默认为空，使用DragonSN 工具写号后，则为vfat 格式的文件系统。对于ubi 方案来说，如果需要使用基于块设备的文件系统，则需要在ubi 之上模拟块设备。在用户空间可调用ubiblock 工具完成，注意这样模拟出的块设备是只读的，如需可写建议直接使用
ubifs。

详见后文ubi 方案特殊说明。