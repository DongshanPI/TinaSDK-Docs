# 5 UBI VS. NFTL

对nand 存储介质，全志有两套解决方案，分别是NFTL spi/raw nand 和UBI spi nand。UBI 存储方案常用于小容量spinand，其实现原理跟NFTL 存储方案完全不同。

## 5.1 NFTL Nand

这是全志实现的不开源的Nand 驱动，NFTL 全称为NAND Flash Translation Layer，其可实现屏蔽Nand 的特性，对上呈现为块设备。
我们常见的mmc 设备也是块设备，可以简单理解为，MMC = Nand Flash + 控制器+NFTL。所以通过全志的NFTL nand 驱动后，我们可以像mmc 设备一样，以块设备操作Nand。
例如，上层可以直接裸读写块设备，常见的ota 更新也是基于这样的实现：

```
dd if=boot.fex of=/dev/by-name/boot
```

**技巧**

**详细的OTA 方法，请参见OTA 相关文档。**

全志的NFTL Nand 驱动中，预留一部分空间做算法和关键数据保存。预留空间大致为1/10 ~1/8 的可用空间。这里的可用空间是指剔除出厂坏块之外的空间。由于

每一颗Flash 的出厂坏块数量不尽相同，因此最终呈现给用户的可用空间不尽相同。用户也不需要担心使用过程中出现的坏块导致用户可用空间变小，在算法实现

中，使用坏块会体现在预留空间而不是用户空间。

此外，Nand 的磨损平衡、坏块管理等特性全由NFTL 驱动实现。换句话说，驱动保证用户数据的稳定，用户可将其按块设备使用。

## 5.2 UBI (spi) Nand

当前UBI 方案仅适用于小容量spinand。
UBI 方案是社区普遍使用的Flash 存储方案，其构建在mtd 设备之上，由UBI 子系统屏蔽Nand 特性，对接UBIFS 文件系统。其层次结构由上往下大致如下：

| 层次       | 层级       | 功能                        |
| ---------- | ---------- | --------------------------- |
| 0 (最上层) | UBIFS      | ubifs 文件系统              |
| 1 UBI      | 子系统     | Nand 特性管理               |
| 2          | MTD 子系统 | 封装Flash，向上提供统一接口 |
| 3 (最下层) | Flash      | 具体的Flash 驱动            |

我们把MTD 分区称为物理分区，把UBI 卷(分区) 成为逻辑分区，因为
• MTD 分区是按Flash 的物理地址区间划分分区
• UBI 卷(分区) 是动态映射的区间
全志的UBI 方案中，创建了这些MTD 物理分区：

| 分区名         | 大小         | 作用                         |
| -------------- | ------------ | ---------------------------- |
| boot0          | 4/8 个物理块 | 存放boot0                    |
| uboot          | 4/20M        | 存放uboot                    |
| secure storage | 8 个物理块   | 存放关键数据                 |
| pstore         | 512K         | 奔溃日志转存                 |
| sys            | 剩余空间     | 提供给ubi 子系统划分逻辑分区 |

驱动会在sys 的MTD 物理分区上根据sys_partition.fex构建UBI 逻辑分区(卷)。

UBI 设备向上呈现为字符设备，无法直接使用诸如ext4 这样基于块设备的文件系统。但UBI 子系统支持模拟只读的块设备，即把UBI 逻辑卷模拟成只读的块设备。

基于此，可以做到根文件系统依然使用squashfs 这样的块文件系统。

社区为UBI 设备专门设计了ubifs 文件系统。经过验证，其配合UBI 子系统可保证数据掉电安全以及提供通用文件系统所有功能，甚至还提供文件系统压缩功能。

除此之外，UBI 设备更新（OTA 更新）也不能直接裸写设备，需要通过ubiupdateval 命令更新。

技巧：详细的OTA 方法，请参见OTA 相关文档。

## 5.3 ubi 相关工具

### 5.3.1 ubinfo

输出指定ubi 设备信息。
例子:

```
ubinfo /dev/by-name/rootfs #查看rootfs卷的信息
ubinfo -a #查看所有卷的信息
```

可参考总容量说明

### 5.3.2 ubiupdatevol

更新指定卷上的数据。
例子:

```
ubiupdatevol -t /dev/by-name/boot #清除boot卷的数据
ubiupdatevol /dev/by-name/boot /tmp/boot.img #将/tmp/boot.img写到boot卷，卷上原有数据会完全丢失
```

可参考总容量说明

可参考模拟块设备

### 5.3.3 ubiblock

基于一个ubi 卷，生成模拟的只读块设备
例子:

```
ubiblock -c /dev/by-name/test #将test卷生成一个块设备节点
```

可参考模拟块设备

### 5.3.4 其他

在tina 方案上，烧录固件时已经完成mtd 和ubi 卷的创建，启动时自动attach 并挂载对应的分区，无需再自行处理。因此以下命令一般不会用到。

ubiformat: 将裸mtd 格式化成ubi

ubiattach: 将mtd 关联到ubi

ubidetach: 将ubi 与mtd 解除关联ubimkvol: 创建ubi 卷

ubirmvol: 移除ubi 卷