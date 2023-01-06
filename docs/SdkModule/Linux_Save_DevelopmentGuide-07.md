# 7 关键数据保护

设备上保存的一些关键的数据，例如mac，SN 号等，一般要求在重新刷机时不丢失。以下介绍刷机数据不丢失的解决方案。

## 7.1 逻辑分区保护方案

### 7.1.1 分区设置

此处的逻辑分区，是指在分区表(sys_partition.fex/sys_partition_nor.fex) 中定义的分区。名字为private 的分区会特殊处理，默认刷机数据不丢失。
其他名字的分区，如果指定keydata=0x8000 属性，则刷机数据不丢失。对于private 分区或设置了keydata=0x8000 属性的分区，请勿设置downloadfile。

### 7.1.2 实现原理

对private 分区(配置了keydata=0x8000 属性同理) 保护的方式是，擦除之前先申请一片内存，然后根据flash 中的旧分区表，读出private 分区内容。
接着进行擦除，然后再按照新的分区表，将private 分区内容写回flash 上新分区所在位置。

### 7.1.3 常见用法

1. 使用全志的DragonSN 工具，选择私有key 模式，将key 写入private 分区。写入后private分区默认会是一个vfat 文件系统，启动后挂载/dev/by-name/private，即可读出key。DragonSN 的具体用法请参考工具自带文档。
2. 不使用DragonSN 工具， 由应用自行负责写入数据和读出数据， 直接在用户空间操作/dev/by-name/private 节点即可。量产时可自行开发PC 端工具，通过adb 命令来完成key 的写入。

### 7.1.4 ubi 方案特殊说明

#### 7.1.4.1 模拟块设备

对于nand nftl 方案，emmc 方案，nor 方案，逻辑分区是对应到一个块设备，即对于private分区，可以直接读写/dev/by-name/private 节点，也可以借助DragonSN 工具制作成一个vfat文件系统，再挂载使用，挂载后文件系统是可读写的。
但对于nand ubi 方案，逻辑分区是对应到ubi 卷，由于ubi 的特性，无法再直接写数据到/dev/by-name/private 节点，需要通过ubiupdatevol 工具来更新卷，或者自行在应用中按照ubi 卷更新步骤操作。
当基于ubi 卷构建vfat 文件系统时，需要先基于ubi 卷模拟块设备，且挂载上的vfat 文件系统是只读的。操作示例如下。

```
#查看private分区对应的ubi节点
root@TinaLinux:/# ll /dev/by-name/private
lrwxrwxrwx 1 root root 11 Jan 1 00:00 /dev/by-name/private -> /dev/
ubi0_4
#创建模拟的块设备
root@TinaLinux:/# ubiblock -c /dev/ubi0_4
block ubiblock0_4: created from ubi0:4(private)
#挂载
root@TinaLinux:/# mkdir /tmp/private
root@TinaLinux:/# mount -t vfat /dev/ubiblock0_4 /tmp/private/
#可以读取内容
root@TinaLinux:/# ls /tmp/private/
ULI magic.bin
#查看挂载情况，为ro
root@TinaLinux:/# mount | grep private
/dev/ubiblock0_4 on /tmp/private type vfat (ro,relatime,fmask=0022,dmask=0022,codepage=437,
iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
```

#### 7.1.4.2 在设备端制作vfat 镜像

由于ubifs 需占用17 个LEB，比较占空间，对于只在工厂一次性写入信息，后续只读的场景，一种可考虑的方案是使用vfat 文件系统。
首先选上kernel 的loopback 支持：

```
make kernel_menuconfig 选上Device Drivers --> [*] Block Devices --> <*>Loopback device
support
```

分区表中建立分区，假设为test 分区随后可以在小机端准备一个vfat 镜像：

```
dd if=/dev/zero of=/mnt/UDISK/test.img bs=1M count=1
mkfs.vfat /mnt/UDISK/test.img
mkdir -p /tmp/test
mount /mnt/UDISK/test.img /tmp/test
此时可向/tmp/test 写入文件
umount /tmp/test
```

将镜像写入卷中：

```
ubiupdatevol /dev/by-name/test /mnt/UDISK/test
```

后续按上文介绍的方法，使用模拟块设备挂载，注意使用模拟块设备挂载后就是只读的了。

### 7.1.5 可能造成数据丢失的情况

出现以下情况，会导致private 分区数据丢失。

1. 配置了强制擦除，例如sys_config.fex 中配置了eraseflag = 0x11。
2. 无法读取flash 上的分区表或private 分区。这个可能的原因包括flash 上的数据被破坏了
   等。
3. 新的分区表不包含private 分区。
4. 在烧录过程中掉电。如上所述，烧录时是读出-> 擦除-> 写回，在擦除之后，写回之前掉电，则数据丢失。
   检测到private 分区，开始执行保护private 分区的代码，但执行过程中出错，如malloc 失败,private 无法读取等，则会导致烧录失败。出现malloc 失败问题一般是因为板子上烧录了Android固件。因为安卓的private 分区比较大，而tina 的uboot 分配给malloc 的空间比较小。
   这个时候，需要打包一份不保护private 分区的tina 固件先进行一次烧录，即可解决问题。具体的：将sys_config.fex 的eraseflag 改为0x11，强制擦除。或者临时移除sys_partition.fex中的private 分区。

## 7.2 物理区域保护方案

另一种保护数据不丢失的思路是，在flash 上划定一块物理区域，烧录时默认不擦除。在Tina 上实现的secure storage 区域即具有这种特性。secure storage 区域用于保存key，可代替private 分区，理论上更为安全（被烧录时误擦除和被别的应用误写的可能性较低）。

### 7.2.1 nand nftl 方案实现

nand nftl 方案中，预留了一块物理区域，用于secure storage。这块区域不是逻辑分区，用户空间不可见。烧录时不会擦除这块区域。在用户空间读写secure storage 需要使用ioctl，由内核nand 驱动来协助完成。

### 7.2.2 nand ubi 方案实现

nand ubi 方案中，预留了一块物理区域，用于secure storage，对上表现为一个mtd 分区。用户空间可见。烧录时不会擦除这块区域。在用户空间读写secure storage 需要使用ioctl，由内核来协助完成。理论上也可以直接读写mtd 设备节点，但不推荐，使用这种方式应用需要自行处理坏块等问题。

### 7.2.3 emmc 方案实现

预留了一块物理区域，默认是偏移6M-6.25M 的区域，作为secure storage。在用户空间读写，可以直接通过mmcblk0 节点读写指定偏移。

### 7.2.4 nor 方案实现

暂未实现。

### 7.2.5 常见用法

1. 使用全志的DragonSN 工具，选择安全key 模式，将key 写入secure storage 区域。启动后在用户空间调用库读出key。DragonSN 的具体用法请参考工具自带文档。
2. 不使用DragonSN 工具，由应用自行在用户空间调用库写入数据和读出数据。量产时可自行开发PC 端工具，通过adb 命令来完成key 的写入。

### 7.2.6 secure storage 格式

secure storage 有预设的格式，简单总结如下
• secure storage 总大小为256KB，因为有备份，所以实际能用128KB。
• 128KB 分为32 个item, 即每个item 是4KB。
• item0 用作secure_sotrage_map，所以用户能用的实际为31 个item。
• 每个item 内部为键值对的格式，包含CRC 校验，其中用户可存储的key 长度最多为3KB。
• secure_storage_map 中是以“name:length” 的格式保存所有item 的信息，所以所有item的“name:length” 字符串长度总和不能超过secure_storage_map 中data 的长度。一般而言，31 个3KB 的key 可以满足需求，如果无法满足，例如需要更多数量的key，则一种解决方式是上层应用自行将多个key 拼接起来，只要总大小不超过3KB 即可当成一个key 写入
secure storage。读出时应用自行反向解析出目标key 即可。

### 7.2.7 在uboot 中读写

基于以上介绍的格式，uboot 中封装了pst 命令。

```
pst - read data from secure storageerase flag in secure storage
Usage:
pst pst read|erase [name]
pst read, then dump all data
pst read name, then dump the dedicate data
pst write name, string, write the name = string
pst erase, then erase all secure storage data
pst erase key_burned_flag, then erase the dedicate data
```

### 7.2.8 在用户空间读写

Tina 提供了读写库。

```
make menuconfig 选中　Allwinner --> <*> libsec_key --> [*] Enable secure storage key support
```

如果选上demo，则会编译出demo 程序sec_key_test。

```
root@TinaLinux:/# sec_key_test -h
-w <name> <data>: write key to sst and private
-r <name> <data>: read key from sst or private
-t 0: write some key to secure storage
-t 1: repetitve read + verify some key from secure storage
-t 2: repetitve write + verify some key from secure storage
-t 3: verify some key from secure storage
-t 4: write some key to private
-t 5: repetitve read + verify some key from private
-t 6: repetitve write + verify some key from private
-t 7: verify some key from private
-l : list
-d : printf hex
-h : help
```

更详细的使用方式请参考：

```
tina/package/allwinner/libkey/readme.txt
```

## 7.3 secure storage 区域与private 分区比较

**刷机不丢失的特性：**
• private 分区每次刷机，是先读出到dram，擦除flash，再写入flash。刷机中途掉电可能丢失
• secure storage 则是刷机过程完全不会擦除。理论上secure storage 的数据更安全。

**用户空间误操作的可能：**
• private 分区可以用常规分区更新命令清除掉。
• secure storage 需要通过ioctl 专用接口访问。

**数据格式：**
• private 分区可以裸分区读写，也可以格式化成文件系统。
• secure storage 已限制为键值对，key 的长度也有限制。

**存放位置：**
• private 分区大小由分区表配置，是一个可见的普通分区。
• secure storage 是flash 上的保留区域，大小固定，分区表中不可见。

• private 分区没有备份，请避免写入时掉电。或者自行在private 分区中构建备份。
• secure storage 默认是双备份。

**在uboot 访问：**
• private 分区，uboot 通过通用的读写flash 接口或文件系统接口访问，取决于数据格式。
• sectre storage 区域，有固定格式，通过uboot 提供的secure storage 专用接口访问。

**校验：**
• private 分区如果是裸数据，是否校验由应用自行处理。如果是文件系统，则由文件系统特性决定。
• secure storage 格式中带了crc 校验。