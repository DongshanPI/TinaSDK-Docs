# Linux_E907 开发指南

## 1 编写目的

介绍v85X 上E907 的启动环境和AMP 的环境搭建。

## 2 使用范围

全志V85X 系列芯片

## 3 环境

A7 SDK：Tina
E907 SDK：melis

## 4 SDK 快捷命令说明

这里主要介绍几个下文会用到的命令，并不会介绍全部命令，如果想了解全部命令，可以在lunch
方案后使用hmm打印出所有tina提供的快捷命令。
1. ckernel, m kernel_menuconfig, mkernel：分别对应进入到内核目录，配置内核，单独编译内核
2. cboot0, mboot0：进入boot0 目录，单独编译boot0
3. cmelis, mmelis, mmelis menuconfig：分别对应进入melis 根目录，编译melis，配置melis
4. make：编译整个tina 除了melis 外的所有东西，如boot0，uboot，内核，跟文件系统等
5. cconfigs：进入板级配置目录，这里主要存放板级的设备树，分区等配置文件
6. p：打包命令，将编译后的东西打包成固件

## 5 E907 启动环境

### 5.1 预先工作

选择方案

```
cd tina
source build/envsetup.sh
lunch
选择对应的V85x方案
```

### 5.2 配置boot0 启动e907

e907 在boot0 阶段启动，需要对boot0 进行一些配置

```
cboot0
vim board/sun8iw21p1/common.mk
# 如下图取消注释
保存退出
mboot0 #编译
```

![image-20221121115033101](${media}/image-20221121115033101.png)

<center>图5-1: 配置1</center>

#### 5.2.1 关闭RISCV 的IOMMU

本步骤只有需要在boot0 阶段启动E907 的需要配置。打开设备树，注释掉下面2 条属性，因为
e907 在boot0 阶段就启动了，不能打开其IOMMU。

```
cconfigs
vim ../board.dts
```

![image-20221121115124735](${media}/image-20221121115124735.png)

<center>图5-2: 关闭IOMMU</center>

### 5.3 配置打包e907 固件

```
cconfigs
cd ../../default/
vim boot_package_nor.cfg # 取消melis-elf选项的注释,如下图
vim boot_package.cfg # 取消melis-elf选项的注释，如下图
保存退出
```

![image-20221121115200889](${media}/image-20221121115200889.png)

<center>图5-3: 打包配置</center>

### 5.4 Linux 配置

```
ckernel
m kernel_menuconfig
# 如下图选中2个驱动
mkernel -j
```

![image-20221121115234331](${media}/image-20221121115234331.png)

<center>图5-4: 补丁下载</center>

```
mmelis menuconfig # 如下图选中standby支持
```

![image-20221121115354868](${media}/image-20221121115354868.png)

<center>图5-5: e907-standby 配置</center>

### 5.5 编译打包

至此关于E907 启动的配置完成，进行编译烧录即可

```
make -j16 # 编译tina
mmelis # 编译melis
p
烧录
```