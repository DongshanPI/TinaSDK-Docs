# 2 分区管理

## 2.1 分区配置文件

在全志平台中，通过sys_partition.fex 文件配置分区。在Tina 中，可以在lunch 选择方案后，通过命令cconfigs 快速跳转到分区配置目录，通常情况下，其路径如下。

```
tina/device/config/chips/<芯片编号>/configs/<方案名>/linux/sys_partition.fex
tina/device/config/chips/<芯片编号>/configs/<方案名>/linux/sys_partition_nor.fex
#以上路径不存在，则使用
tina/device/config/chips/<芯片编号>/configs/<方案名>/sys_partition.fex

```

**说明**
**sys_partition_nor.fex 适用于nor。**
**sys_partition.fex 适用于rawnand/spinand/mmc。**

## 2.2 分区配置格式

以rootfs 分区为例：

```
[partition]
name = rootfs
size = 20480
downloadfile = "rootfs.fex"
user_type = 0x8000
```

每个分区以[partition] 标识，分区属性及其意义如下表。

| 属性         | 含义                 | 必选 | 备注                       |
| ------------ | -------------------- | ---- | -------------------------- |
| name         | 分区名               | Y    |                            |
| size         | 分区大小             | Y    | 单位：扇区（512B），注1    |
| downlodefile | 分区烧入的镜像文件   | N    | 注2                        |
| verify       | 量产后校验标识       | N    | (默认)1：使能；0:禁用，注3 |
| user_type    | 分区属性             | N    | 注 4                       |
| keydata      | 量产时是否擦除本分区 | N    | 0x8000：使用;其他无效      |

说明：

1. 最后一个分区(UDISK)，不设置size，表示分配所有剩余空间。
2. downloadfile 支持绝对路径和相对路径，相对于tina/out/<方案名>/image。
3. verify 决定是否校验downloadfile 中指定的镜像，若为ext4 稀疏镜像，务必禁用。
4. 历史遗留，目前只对UBI 方案有效。bit0为1 时，表示创建静态卷，反之为动态卷。

创建downloadfile 的资源镜像包看章节分区资源镜像文件。
[partition] 标识用户空间的逻辑分区，在UBI 方案中，表现为UBI 卷。此外，在sys_partiton.fex中存在特殊的配置MBR，用于配置MBR 空间大小，此配置在UBI 方案中无效。例如：

```
[mbr]
size = 2048
```

MBR 分区以Kbyte 为单位，对用户不可见，属于隐藏空间，其大小也必须满足对齐原则。

**警告**
**一般情况下，不建议用户修改mbr 分区的大小。**

## 2.3 常见分区及其用途

| 分区名      | 用途                   | 大小                     | 备注                             |
| ----------- | ---------------------- | ------------------------ | -------------------------------- |
| boot        | 内核镜像分区           | 比实际镜像等大或稍大即可 |                                  |
| rootfs      | 根文件系统镜像         | 比实际镜像等大或稍大即可 |                                  |
| extend      | 扩展系统镜像           | 参考OTA 文档             | 参考OTA 文档仅小容量OTA 方案使用 |
| recovery    | 恢复系统镜像           | 参考OTA 文档             | 仅限大容量OTA 方案使用           |
| private     | 存储SN、MAC 等数据     | 使用默认大小即可         | 量产时默认不丢失                 |
| misc        | 存储系统状态、刷机状态 | 使用默认大小即可         |                                  |
| env         | 存放Uboot 使用的数据   | 使用默认大小即可         |                                  |
| pstore      | 内核奔溃日志转存分区   | 使用默认大小即可         |                                  |
| rootfs_data | 根目录覆盖分区         | 根据需求配置             | 注1                              |
| UDISK       | 用户数据分区           | 不需要配置大小           | 注2                              |



说明：

1. rootfs_data 分区通过overlayfs 覆盖根文件系统，以支持squashfs 根文件系统的可写，此时对根文件系统写入的数
   据实际是保存到rootfs_data 分区，因此rootfs_data 分区的容量标识着根文件系统最大可写数据量。
2. UDISK 作为最后一个分区，不需要设置size，表示分配剩余所有空间给UDISK。

## 2.4 分区大小与对齐

分区大小的对齐要求与不同介质(nor/nand/mmc)、不同存储方案相关。不按对齐要求配置，可能出现文件系统异常，分区边界数据丢失等现象。对齐规则如下表。

| 介质           | 对齐大小           | 备注                           |
| -------------- | ------------------ | ------------------------------ |
| nor            | 64K                | 对齐物理擦除块大小，注1        |
| (nftl) spinand | 驱动超级块大小     | 注2                            |
| (ubi) spinand  | 2 × 物理块- 2 × 页 | 注3                            |
| rawnand        | 驱动超级块大小     | 与物料相关，16M 对齐可基本兼容 |
| emmc           | 16M                | 与物料相关，16M 对齐可基本兼容 |



说明

1. nor 的擦除块常见为64K，即在sys_partition_nor.fex 中分区size 进行128 对齐。在id 表配置为4K 擦除且使能内核CONFIG_MTD_SPI_NOR_USE_4K_SECTORS 时，也可使用4K 对齐。推荐使用默认64K 对齐。
2. 在常见的128M Spi Nand 中，为256K 对齐，即在sys_partition.fex 中分区size 进行512 对齐。
3. 在常见的128M Spi Nand 中，需要和逻辑擦除块（super block）对齐，1 个super block 包含两个物理擦除块，常见的物理擦除块128K，1 个逻辑的超级块为256K，但是需要使用每个物理块的第一个page（2K）来作为ubi 所需的信息头部，所以实际的为（256k-2*2k），为252K 对齐，即在sys_partition.fex 中分区size 进行504 对齐。

**警告**
**如果分区不对齐，可能会出现以下情况。**
**• nor/rawnand/spinand 可能会导致数据丢失。**
**• mmc 不会造成数据丢失，但可能导致性能损失。**

如果分区使用ubifs 文件系统，分区最小为5M ，否则大概率提示空间不够。
如果分区使用ext4 文件系统，分区最小为3M ，否则无法形成日志，会有掉电变砖风险。

技巧

1. 在ext4 与日志章节有描述判断创建的ext4 文件系统是否支持日志的方法。
2. 在分区资源镜像文件章节指导如何创建带文件系统的资源镜像、分区大小、文件系统大小、文件大小更多内容，请参考总容量说明。

## 2.5 分区与文件系统

常见的分区与文件系统对应关系如下表。

| 分区名      | 默认文件系统     | 文件系统特性 | 备注                         |
| ----------- | ---------------- | ------------ | ---------------------------- |
| rootfs      | squashfs         | 压缩、只读   | 为了安全，根文件系统建议只读 |
| rootfs_data | jffs2/ext4/ubifs | 可写         | 注1                          |
| UIDSK       | jffs2/ext4/ubifs | 可写         | 注1                          |
| boot        | vfat             |              | 裸数据分区，部分方案为vfat   |
| private     | vfat             |              | 注2                          |
| misc        | none             |              | 裸数据分区                   |
| env         | none             |              | 裸数据分区                   |
| pstore      | pstore           |              | 转存奔溃日志                 |

说明

1. 可写的分区，nor 为jffs2；UBI 方案为ubifs；其他为ext4。
2. private 默认为裸数据，使用dragonSN 工具烧录后会成为vfat 文件系统。

只读文件系统推荐使用squashfs。可写文件系统，nor 推荐jffs2，UBI 方案推荐ubifs，其他推荐ext4。更多文件系统信息，请参考文件系统支持情况。

## 2.6 分区资源镜像文件

在sys_partition.fex中通过downloadfile 指定需要烧录到分区的资源镜像文件。大多数情况下，资源镜像文件都构建在文件系统上，通过某些命令实现把系统需要的文件，例如音频文件、视频文件等资源，打包成一个带文件系统的镜像包，并在烧录时把镜像包烧写入存储
介质。
创建不同文件系统镜像的命令不一样，常见有以下几种：



| 文件系统 | 创建镜像命令 |
| -------- | ------------ |
| vfat     | mkfs.vfat    |
| ext4     | make_ext4fs  |
| ubifs    | mkfs.ubifs   |
| squashfs | mksquashfs4  |

为了最大程度利用空间，一般会使文件系统等于物理分区大小，即创建文件系统时使用分区表划定的分区大小来创建。

如果不希望硬编码大小，则可在打包时从分区表获得大小，再传给文件系统创建工具，具体的实现可以参考tina/scripts/pack_img.sh 中的make_data_res() 和make_user_res() 等函数。

### 2.6.1 创建squashfs 镜像

生成squashfs 的命令，可参考编译过程的log 得到，或者在网上搜索squashfs 生成方式。
例如在scripts/pack_img.sh 中定义一个函数

```
function make_user_squash()
{
local SOURCE_DATE_EPOCH=$(${PACK_TOPDIR}/scripts/get_source_date_epoch.sh)
# 这一行指定要打包到文件系统的数据
local USER_PART_FILE_PATH=${PACK_TOPDIR}/target/allwinner/方案名字/user_sq
local USER_PART_SQUASHFS=${PACK_TOPDIR}/out/${PACK_BOARD}/image/user_sq.squashfs
local USER_PART_DOWNLOAD_FILE=${PACK_TOPDIR}/out/${PACK_BOARD}/image/user_sq.fex
cd ${ROOT_DIR}/image
[ -e $USER_PART_FILE_PATH ] && {
#这里用了gzip，需要更高压缩率可改成xz
${PACK_TOPDIR}/out/host/bin/mksquashfs4 $USER_PART_FILE_PATH $USER_PART_SQUASHFS \
-noappend -root-owned -comp gzip -b 256k \
-processors 1 -fixed-time $SOURCE_DATE_EPOCH
dd if=${USER_PART_SQUASHFS} of=${USER_PART_DOWNLOAD_FILE} bs=128k conv=sync
}
cd -
}
```

找个地方调用下即可。
这里不用传入分区表的原因是，制作squashfs 不需要指定文件系统大小，只读的文件系统大小完全取决于文件内容。

### 2.6.2 创建vfat 镜像

```
mkfs.vfat <输出镜像> -C <文件系统大小>
mcopy -s -v -i <输出镜像> <资源文件所在文件夹>/* ::
```

可参考pack_img.sh（在其中搜索mkfs.vfat 找到相关代码）。

### 2.6.3 创建ext4 镜像

使用tina/out/host/bin/make_ext4fs 创建ext4 镜像，推荐的使用方法如下:

```
make_ext4fs -l <文件系统大小> -b <块大小> -m 0 -j <日志块个数> <输出的镜像保存路径> <资源文件所在文件
夹>
```

其中，
• -m 0: 表示不需要要为root 保留空间。
• -j < 日志块个数>: 日志总大小为块大小* 日志块个数。

例如：

```
make_ext4fs -l 20m -b 1024 -m 0 -j 1024 ${ROOT_DIR}/img/data.fex ${FILE_PATH}
```



如果空间不够大，会显示类似如下的错误日志：

```
$ ./bin/make_ext4fs -l 10m -b 1024 -m 0 -j 1024 data.fex ./bin
Creating filesystem with parameters:
Size: 10485760
Block size: 1024
Blocks per group: 8192
Inodes per group: 1280
Inode size: 256
Journal blocks: 1024
Label:
Blocks: 10240
Block groups: 2
Reserved blocks: 0
Reserved block group size: 63
error: ext4_allocate_best_fit_partial: failed to allocate 7483 blocks, out of space?
```



上述错误中，资源文件达到100+M，但是创建的镜像-l 指定的大小只有10M，导致空间不够而报错。只需要扩大镜像大小即可。
如需使用分区大小作为文件系统大小，可参考pack_img.sh（在其中搜索make_ext4fs 找到相关代码）。

技巧
镜像大小可以根据分区大小设置，也可以根据资源大小设置，后通过稀疏和resize 处理，即可保证最短烧录时间和动态匹配分区大小。见稀疏ext4 镜像和动态resize 章节。



#### 2.6.3.1 稀疏ext4 镜像

如果资源文件只有10M，但创建了100M 的镜像文件，导致烧录100M 的文件拖慢了烧录速度。此时可以采用稀疏ext4 镜像。

```
tina/out/host/bin/img2simg <原镜像> <输出镜像>
```



稀疏镜像的原理，类似与把文件系统没用到的无效数据全删掉，把文件系统压缩。可参考pack_img.sh 中的函数sparse_ext4() 的实现与运用。

#### 2.6.3.2 动态resize

如果担心创建镜像时指定的大小与实际的分区大小不匹配，可以在设备启动后执行resize2fs 动态调整文件系统的大小。
例如:

```
resize2fs /dev/by-name/UDISK
```

命令后不指定大小，则默认为分区大小。通过这方法可以让打包镜像创建的文件系统大小匹配分区大小。
此命令可直接写入启动脚本，在挂载前执行。每次启动都执行一遍不会有不良影响。

### 2.6.4 创建ubifs 镜像

使用tina/out/host/bin/make.ubifs 创建ubifs 镜像，推荐的使用方法如下:

```
mkfs.ubifs -x <压缩方式> -b <超级页大小> -e <逻辑擦除块大小> -c <最大逻辑擦除块个数> -r <资源文件所在
文件夹> -o <输出的镜像保存路径>
压缩方式可选none lzo zlib, 压缩率zlib > lzo > none
对常见的128MB spinand，1 page = 2048 bytes, 1 block = 64 page, 则
超级页大小为2048 * 2 = 4096
逻辑擦除块大小为2048 * 2 * 64 = 262144
最大逻辑擦除块个数，可简单设置为一个较大的值，例如128MB / ( 2048 bytes * 2 * 64) = 512
则最终的命令为:
mkfs.ubifs -x zlib -b 4096 -e 262144 -c 512 -r ${FILE_PATH} -o ${ROOT_DIR}/img/data_ubifs.
fex
```

## 2.7 根文件系统改用ubifs

使用suqashfs + overlayfs(ubifs) 方案实现根目录可写，但是ubifs 会占用大量的空间存放元数据，造成空间浪费。理论上，UBIFS 可直接作为根文件系统，其稳定性和可压缩性足够保证安全和提高空间利用率。

**警告**
**请谨慎使用，UBIFS 作为根文件系统只是理论安全，全志暂无量产方案佐证。**

修改步骤如下：

1. 执行cdevice ，修改跳转目录下的Makefile 在FEATURES 变量中添加ubifs 和nand
2. 执行make menuconfig 使能在Target Image 页面下使能ubifs 在Utilities->mtdutils
   页面中使能mtd-utils-mkfs.ubifs
3. 执行cconfigs ，修改跳转目录下的env-XXX.cfg 把rootfstype 值改为ubifs 在对应
   存储介质的setargs_XXX 的root 值改为root=ubi0_X ，其中X 表示对应的第几个分区；删除
   ubi.block 项。
4. 执行make kernel_menuconfig，取消使能overlayfs
5. 在sys_partition.fex 中删除rootfs_data 分区



## 2.8 总容量说明

在全志的驱动中，会预留一部分空间存储特殊数据，因此提供给用户分区空间不等于实际Flash总容量。

```
分区表可用空间= flash总容量- 保留空间
```

不同存储介质，其保留空间会有差异。

| 存储介质      | 保留空间         | 备注                                          |
| ------------- | ---------------- | --------------------------------------------- |
| nor           | 512K             | 对应bootloader 分区，包含分区表，boot0，uboot |
| (nftl) nand   | 总容量的1/10~1/8 | 注1，注2                                      |
| (UBI) spinand |                  |                                               |
| mmc           | 20M              | 包含boot0，uboot 等                           |

最新spinor 的存储分布, 隐藏空间1MB，其中mbr 占用16KB，mbr 往前共占用1008KB，uboot 往前共占用64KB，其中包括boot0 和uboot 中间4KB 的预留区域，这段区域用于存放flash 的spi 采样点等信息。

```
|boot0|4kb|uboot|mbr |分区表可见的用户分区|
```

说明

1. (nftl) nand 的隐藏空间对用户不可见，包含分区表MBR 分区，boot0，uboot, 磨损算法、坏块保留等。对128M 的
   spinand 来说，用户可用空间一般为108M。

2. 由于出厂坏块的存在，可能会导致每一颗Flash 呈现的用户可用总容量不同，但全志(nftl) nand 保证总容量不会随着使
   用过程出现坏块而导致可用容量减少。

3. UBI 方案中，除了必要的mtd 物理分区之外(boot0/uboot/pstore 等)，其余空间划分到一个mtd 物理分区。在此
   mtd 物理分区中根据sys_partition.fex 的划分构建ubi 卷。UBI 的机制，每个块都需要预留1~2 个页作为EC/VID
   头。因此可用容量会小于mtd 物理分区容量。

  

  对于非ubi 方案，用户空间可通过下面的命令查看用户可用分区大小，大小单位为KB。

```
# cat /proc/partitions
major minor #blocks name
93 0 112384 nand0
93 1 256 nand0p1
93 2 5120 nand0p2
93 3 10240 nand0p3
93 4 10240 nand0p4
93 5 7168 nand0p5
93 6 64 nand0p6
93 7 512 nand0p7
93 8 256 nand0p8
93 9 76463 nand0p9
```

如例子中的结果，nand0 分为多个分区，每个nand0px 对应一个分区表中的分区。对于ubi 方案，整个nand 分为若干mtd。可使用以下命令查看

```
# cat /proc/mtd
dev: size erasesize name
mtd0: 00100000 00040000 "boot0"
mtd1: 00300000 00040000 "uboot"
mtd2: 00100000 00040000 "secure_storage"
mtd3: 00080000 00040000 "pstore"
mtd4: 07a80000 00040000 "sys"
```

如例子中的结果，整个nand 分为5 个mtd。
• mtd0 存放boot0, size 1 MB
• mtd1 存放uboot, size 3 MB
• mtd2 存放secure_storage, size 1 MB
• mtd3 存放pstore, size 512 KB
• mtd4 则会进一步分为多个ubi 卷，占用剩余所有空间

以上所有mtd 的size 相加，应该恰好等于flash 总size。分区表中定义的每个逻辑分区，会对应mtd sys 上的ubi 卷。可使用以下命令查看

```
# ubinfo -a
UBI version: 1
Count of UBI devices: 1
UBI control device major/minor: 10:51
Present UBI devices: ubi0
ubi0
Volumes count: 12
Logical eraseblock size: 258048 bytes, 252.0 KiB
Total amount of logical eraseblocks: 489 (126185472 bytes, 120.3 MiB)
Amount of available logical eraseblocks: 0 (0 bytes)
Maximum count of volumes 128
Count of bad physical eraseblocks: 1
Count of reserved physical eraseblocks: 19
Current maximum erase counter value: 3
Minimum input/output unit size: 4096 bytes
Character device major/minor: 247:0
Present volumes: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11
Volume ID: 0 (on ubi0)
Type: static
Alignment: 1
Size: 1 LEBs (258048 bytes, 252.0 KiB)
Data bytes: 258048 bytes (252.0 KiB)
State: OK
Name: mbr
Character device major/minor: 247:1
-----------------------------------
Volume ID: 1 (on ubi0)
Type: dynamic
Alignment: 1
Size: 2 LEBs (516096 bytes, 504.0 KiB)
State: OK
Name: boot-resource
Character device major/minor: 247:2
-----------------------------------
... #此处省略若干卷
-----------------------------------
Volume ID: 11 (on ubi0)
Type: dynamic
Alignment: 1
Size: 242 LEBs (62447616 bytes, 59.6 MiB)
State: OK
Name: UDISK
Character device major/minor: 247:12
```

如例子中的结果
• volume 0 为mbr, 占1 LEBs(252 KB)，对应分区表本身
• volume 1 为boot-resource, 占2 LEBs(504 KB)，对应分区表中第一个分区
• …
• volume 11 为UDISK, 占242 LEBs(59.8 MB)，对应分区表中最后一个分区
常见的关于容量的疑惑与解答。

1. 问：df 查看UDISK 分区大小，明明分区有50M，怎么显示总大小只有40+M？
   答：df 显示的是文件系统的大小，文件系统本身需要额外的空间存储元数据，导致实际可用空间
   会比分区大小略少。
2. 问：df 查看boot 分区大小，为什么显示的大小比实际分区大？
   答：boot 分区是通过镜像烧写的形式格式化的fs，创建镜像时设置的文件系统的大小并不等于分
   区实际大小，导致此时文件系统大小并不能体现实际分区大小。
3. 问：df 查看squashfs 使用率总是100%？
   答：squashfs 是只读压缩文件系统，文件系统大小取决于总文件大小，使用率总是100%，跟分
   区大小无关。

**说明**

1. **文件大小**
   **我们常说的文件大小，指的是文件内容有多少字节。但在一个文件系统中，空间分配以块为单位，必然会造成内部碎片。假**
   **设块为4K，如果文件大小为1K，文件系统依然为其分配4K 的块，就会造成3K 的空间浪费。**
2. **文件系统大小**
   **文件系统大小，指的是文件系统元数据中标识的可用大小。形象来说，是df 命令或者statfs() 函数反馈的大小。文件系**
   **统大小不一定等于分区大小，既可大于分区大小，也可小于分区大小。**
3. **分区大小**
   **在划分分区时规定的大小，往往是sys_partition.fex 中指定的大小。**

## 2.9 特殊隐藏空间

不管是nor，nand 还是mmc，都需要一些隐藏空间存储特殊数据，例如boot0/uboot/dtb/sys_config。用户无法使用这些隐藏空间。
此外，nand 驱动还需要额外的空间以实现磨损平衡、坏块管理算法，因此nand 的隐藏空间更大。