# 使用buildroot-SDK编译构建系统

## 简介

* 此套构建系统基于全志RISCV-64 Linux D1-H  芯片，适配了buildroot 2022lts主线版本，兼容了百问网的项目课程以及相关组件，真正做到了低耦合，高可用，使用不同的buildroot external tree规格，讲不同的项目 不同的组件分别管理，来实现更容易上手 也更容易学习理解。

## 获取sdk源码

* 默认源码都存放在github仓库内，请使用如下命令获取


```bash
book@virtual-machine:~$ git clone  https://github.com/DongshanPI/buildroot_dongshannezhastu
book@virtual-machine:~$ cd buildroot_dshannezhastu
book@virtual-machine:~/buildroot_dongshannezhastu$ git submodule update --init --recursive
book@virtual-machine:~/buildroot_dongshannezhastu$ git submodule update --recursive --remote
```



*  对于国内无法访问github的同学，可以使用国内备用gitee站点， 如下命令。

```bash
book@virtual-machine:~$ git clone  https://gitee.com/weidognshan/buildroot_dongshannezhastu
book@virtual-machine:~$ cd buildroot_dshannezhastu
book@virtual-machine:~/buildroot_dongshannezhastu$ git submodule update --init --recursive
book@virtual-machine:~/buildroot_dongshannezhastu$ git submodule update --recursive --remote
```

## 安装必要依赖包

### ubuntu-18.04

运行环境配置： 此系统基于ubuntu18.04进行验证，在之前的基础之上还需要安装以下必要依赖

```bash
 sudo apt-get install -y  libncurses5-dev   u-boot-tools
```

安装完成后，执行如下命令进行开始编译操作。


## 最小系统编译烧写

### 编译sdcard 最小系统镜像

```bash
book@virtual-machine:~/buildroot_dongshannezhastu$ cd buildroot-awol/
book@virtual-machine:~/buildroot_dongshannezhastu/buildroot-awol$ make  BR2_EXTERNAL="../br2lvgl  ../br2qt5 ../br2nezhastu"  dongshannezhastu_sdcard_core_defconfig

book@virtual-machine:~/buildroot_dongshannezhastu/buildroot-awol$ make 
```

### 烧写sdcard 最小系统镜像

编译完成后会在 output/images目录下输出 dongshannezhastu-sdcard.img 文件，将文件拷贝到Windows系统下使用 wind32diskimage烧写，或者使用dd if 烧录到tf卡内，
之后插到开发板上，即可启动。 请参考左侧 [快速启动](https://dongshanpi.com/DongshanNezhaSTU/03-QuickStart/#tf) 页面



### 编译spi nand最小系统镜像
```bash
book@virtual-machine:~/buildroot_dongshannezhastu$ cd buildroot-awol/
book@virtual-machine:~/buildroot_dongshannezhastu/buildroot-awol$ make  BR2_EXTERNAL="../br2lvgl  ../br2qt5 ../br2nezhastu"  dongshannezhastu_spinand_core_defconfig

book@virtual-machine:~/buildroot_dongshannezhastu/buildroot-awol$ make 
```


### 烧写spi nand最小系统镜像

编译完成后会在 output/images目录下输出 d1-h-nezhastu_uart0.img 文件，将文件拷贝到Windows系统下使用 使用 全志官方的  AllwinnertechPhoeniSuit 进行烧写。
详细烧写步骤请，请参考左侧 [快速启动](https://dongshanpi.com/DongshanNezhaSTU/03-QuickStart/#spi-nand) 页面。
