## 3 编译方法介绍

### 3.1 准备编译工具链

准备编译工具链接执行步骤如下：

```
1）cd longan/brandy/brandy-2.0/\
2）./build.sh -t
```



### 3.2 快速编译 boot0 及 U-Boot

在longan/brandy/brandy-2.0/目录下，执行 ./build.sh -p 平台名称，可以快速完成整个 boot 编译动作。这个平台名称是指，LICHEE_CHIP。

```
./build.sh -p {LICHEE_CHIP}            //快速编译spl/U-Boot
./build.sh -o spl-pub -p {LICHEE_CHIP} //快速编译spl-pub
./build.sh -o uboot -p {LICHEE_CHIP}   //快速编译U-Boot
```



### 3.3 编译 U-Boot

cd longan/brandy/brandy-2.0/u-boot-2018/进入 u-boot-2018 目录。以{LICHEE_CHIP}为例，依次执行如下操作即可。

```
1）make {LICHEE_CHIP}_defconfig
2）make -j
```



### 3.4 编译 boot0/fes/sboot

cd longan/brandy/brandy-2.0/spl-pub进入spl-pub目录，需设置平台和要编译的模块参数。以{LICHEE_CHIP}为例，编译 nand/emmc 的方法如下：

1. 编译boot0

```
make distclean
make p={LICHEE_CHIP} m=nand
make boot0

make distclean
make p={LICHEE_CHIP} m=emmc
make boot0
```



2. 编译fes

```
make distclean
make p={LICHEE_CHIP} m=fes
make fes
```



3. 编译sboot

```
make distclean
make p={LICHEE_CHIP} m=sboot
make sboot
```

