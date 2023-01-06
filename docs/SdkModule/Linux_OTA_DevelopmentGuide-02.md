## 2 ota-burnboot 介绍

### 2.1 文档说明

此文档主要介绍如何在OTA 时升级boot0/uboot。

升级工具包含两个方面内容：

OTA 命令升级boot0 和uboot。

OTA 升级boot0 和uboot 的C/C++ APIs。



### 2.2 概念说明

<center>表2-1: ota-burnboot 相关概念说明表</center>

| 概念             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| boot0            | 较为简单, 主要作用是初始化dram 并加载运行uboot。一般不需修改。 |
| uboot            | 功能较丰富, 支持烧写, 启动内核, 烧key 及其他一些定制化的功能。 |
| sys_config       | 全志特有的配置文件, 对于使用linxu3.4/uboot2011 的平台, 在打包之<br/>后sys_config 会跟uboot 拼接到一起。对于使用<br/>linux3.10/uboot2014 及更高版本的平台,sys_config 会在打包阶段,<br/>跟设备树的配置合并, 生成最终的dtb。linux5.4 开始不再合并到<br/>dtb。 |
| dtb              | 设备树, 由dts 配置和sys_config 配置综合得到。                |
| u-boot.fex       | 使用linxu3.4/uboot2011 的平台最终用到的uboot, 其实是<br/>uboot+sys_config。 |
| boot_package.fex | 使用linux3.10/uboot2014 及更高版本的平台最终用到的uboot, 其<br/>实包含的文件由配置文件boot_package.cfg 决定, 一般至少包含了<br/>uboot 和dtb, 安全方案会包含一些安全所需文件文件, 可能还有<br/>bootlogo 等文件。 |
| toc0.fex         | 安全方案使用的boot0。                                        |
| toc1.fex         | 安全方案使用的uboot, 类似boot_package.fex 说明, 其中实际也包<br/>含了dts 等多个文件。 |

即, 本文介绍的升级uboot, 其实是升级uboot+dtb 这样的一个整体文件。后文不再区分更新uboot, 更新sys_config, 更新dtb。这几个打包完毕是合成一个文件的, 

暂不支持单独更新其中一个, 需整体更新。

### 2.3 用于更新的bin 文件

获取用于OTA 的boot0 与uboot 的bin 文件, 用于加入OTA 包中。

#### 2.3.1 编译boot0 uboot

如果原本的固件生成流程已经包含编译uboot, 则正常编译固件即可。

否则可按照如下步骤编译生成uboot

```
$ source build/envsetup.sh
=> 设置环境变量。
$ lunch
=> 选择方案。
$ muboot
=> 编译uboot。
$ mboot0
=> 编译boot0 (注意，此命令在大多数平台无效，因为boot0不开源，SDK中提供了编译好的bin文件)。
```

编译后会自行拷贝bin 文件到该平台的目录下, 即：

对于tina3.5.0 及之前版本，路径为：

```
target/allwinner/xxx-common/bin
```

对于tina3.5.1 及之后版本，路径为：

```
device/config/chips/${CHIP}/bin
```

编译出的boot0/uboot 还不能直接用于OTA, 请继续编译和打包固件, 如执行：

```
$ make -j <N>
=> 编译命令,若只修改boot0/uboot/sys_config 无需重新编译,可跳过。
=> 若修改了dts 则需要执行,重新编译。
$ pack [-d]。
=> 非安全方案的打包命令。
$ pack -s [-d]
=> 安全方案的打包命令。
```

#### 2.3.2 关于更新boot0

大多数平台，代码环境中并不包含boot0 相关代码, 因此无法编译boot0。

一般情况下并不需要修改boot0, 而是直接使用提供的boot0 的bin 文件即可。少部分平台提供了可编译的boot0 代码，可使用mboot0 编译。

#### 2.3.3 Bin 文件路径

##### 2.3.3.1 使用uboot2011 的非安全方案

以R16 的astar-parrot 方案为例。

根据对应存储介质选择bin。

boot0：

```
out/astar-parrot/image/boot0_nand.fex ：nand 方案使用的boot0。
out/astar-parrot/image/boot0_sdcard.fex ：mmc 方案使用的boot0。
out/astar-parrot/image/boot0_spinor.fex ：nor 方案使用的boot0。
```

uboot：

```
out/astar-parrot/image/u-boot.fex ：nand/mmc 方案使用的uboot。
out/astar-parrot/image/u-boot-spinor.fex ：nor 方案使用的uboot。
```

##### 2.3.3.2 使用uboot2014 及更高版本的非安全方案

以R6 的sitar-evb 方案为例。

根据对应存储介质选择bin。

boot0：

```
out/sitar-evb/image/boot0_nand.fex ：nand 方案使用的boot0。
out/sitar-evb/image/boot0_sdcard.fex ：mmc 方案使用的boot0。
out/sitar-evb/image/boot0_spinor.fex ：nor 方案使用的boot0。
```

uboot：

```
out/sitar-evb/image/boot_package.fex ：nand/mmc 方案使用的uboot。
out/sitar-evb/image/boot_package_nor.fex ：nor 方案使用的uboot。
```

##### 2.3.3.3 安全方案

以R18 的tulip-noma 方案为例。

boot0：

```
out/tulip-noma/image/toc0.fex ：安全方案使用的boot0。
```

uboot：

```
out/tulip-noma/image/toc1.fex ：安全方案使用的uboot。
```

### 2.4 OTA 升级命令

#### 2.4.1 支持OTA 升级命令

升级boot0 与uboot 分别使用ota-burnboot0 与ota-burnuboot 命令。

两个命令都是OTA 升级boot0 和uboot 的C/C++ APIs 的封装。

要支持本功能, 需要选中ota-burnboot 的包, 即：

```
Make menuconfig --> Allwinner ---> <*>ota-burnboot
```

#### 2.4.2 ota-burnboot0

##### 2.4.2.1 命令说明

```
$ Usage: ota-burnboot0 <boot0-image>
```

升级boot0, 其中boot0-image 是镜像的路径。

请注意, 安全和非安全方案所使用的boot0-image 是不同的, 具体见“用于更新的bin 文件” 章节。

##### 2.4.2.2 使用示例

```
root@TinaLinux:/# ota-burnboot0 /tmp/boot0_nand.fex
Burn Boot0 Success
```

#### 2.4.3 ota-burnuboot

##### 2.4.3.1 命令说明

```
$ Usage: ota-burnuboot <uboot-image>
```

升级uboot, 其中uboot-image 是镜像的路径。请注意, 安全和非安全方案, 不同的uboot 版本,所使用的uboot-image 是不同的，具体见第二章。

##### 2.4.3.2 使用示例

```
root@TinaLinux:/# ota-burnuboot /tmp/u-boot.fex
Burn Uboot Success
```

### 2.5 OTA 升级C/C++ APIs

包含头文件OTA_BurnBoot.h，使用库libota-burnboot.so

#### 2.5.1 int OTA_burnboot0(const char *img_path)

<center>表2-2: OTA_burnboot0 函数说明表</center>

| 函数原型 | int OTA_burnboot0(const char *img_path); |
| -------- | ---------------------------------------- |
| 参数说明 | img_path：boot0 镜像路径                 |
| 返回说明 | 0：成功; 非零：失败                      |
| 功能描述 | 烧写boot0                                |

#### 2.5.2 int OTA_burnuboot(const char *img_path)

<center>表2-3: OTA_burnuboot 函数说明表</center>

| 函数原型 | int OTA_burnuboot(const char *img_path); |
| -------- | ---------------------------------------- |
| 参数说明 | img_path:uboot 镜像路径                  |
| 返回说明 | 0：成功; 非零：失败                      |
| 功能描述 | 烧写uboot                                |

### 2.6 底层实现

#### 2.6.1 如何保证安全更新boot0/uboot

前提条件是,flash 中存有不止一份boot0/uboot。在这个基础上, 启动流程需支持校验并选择完整的boot0/uboot 进行启动, 更新流程需保证任意时刻掉电,flash 上

总存在至少一份可用的boot0/uboot。

#### 2.6.2 Nand Flash NFTL 方案实现

在nand nftl 方案中,boot0 和uboot 是由nand 驱动管理, 保存在物理地址中, 逻辑分区不可见。

Nand 驱动会保存多份boot0 和uboot, 启动时, 从第一份开始依次尝试, 直到找到一份完整的boot0/uboot 进行使用。

更新boot0/uboot 时, 上层调用nand 驱动提供的接口, 驱动中会从第一份开始依次更新, 多份全部更新完毕后返回。因此可保证在OTA 过程中任意时刻掉电,flash 

中均有至少一份完整的boot0/uboot 可用。再次启动后, 只需重新调用更新接口进行更新, 直到调用成功返回即可。

目前nand 中的多份boot0/uboot 是由nand 驱动管理的, 只能整体更新, 暂不支持单独更新其中的一份。

#### 2.6.3 Nand Flash UBI 方案实现

在nand ubi 方案中, boot0 一般存放于mtd0 中，uboot 存放于mtd1 中。

与nftl 方案一样，底层实际是保存多份boot0 和uboot。启动时, 从第一份开始依次尝试, 直到找到一份完整的boot0/uboot 进行使用。对上提供多份统一的更新接

口，软件包会通过对mtd的iotcl 接口发起更新。

注：用户空间直接读写/dev/mtdx 节点，需要内核使能CONFIG_MTD_CHAR=y。

#### 2.6.4 MMC Flash 实现

在mmc 方案中, boot0 和uboot 各有两份, 存在mmc 上的指定偏移处, 逻辑分区不可见。需要读写可直接操作/dev/mmcblk0 节点的指定偏移。

具体位置:

```
1 sector = 512 bytes = 0.5k。
boot0/toc0 保存了两份，offset1: 16 sector, offset2: 256 sector。
uboot/toc1 保存了两份，offset1: 32800 sector, offset2: 24576 sector。
```

启动时会先读取offset1，如果完整性校验失败，则读取offset2。

更新时, 默认只更新offset1, 而offset2 是保持在出厂状态的。只要offset1 正常更新了, 则启动时会优先使用。如果在更新offset1 的过程中掉电导致数据损坏, 则自

动使用offset2 进行启动。

如需定制策略，例如改成每次offset1 和offset2 均更新，可自行修改ota-burnboot 代码。

#### 2.6.5 NOR Flash 实现

nor 方案中, 只保存一份boot0 和uboot, 更新过程中掉电可能导致无法启动, 只能进行刷机。故目前未实现ota 更新, 需后续扩展。