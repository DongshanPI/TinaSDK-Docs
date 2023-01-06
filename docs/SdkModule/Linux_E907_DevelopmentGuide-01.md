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