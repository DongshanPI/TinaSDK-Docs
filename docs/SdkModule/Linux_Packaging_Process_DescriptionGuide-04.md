# 5 获得打包后的分区镜像文件



由于方案的差异化不同，一些方案需要获取到每个分区的镜像文件而非全志的整个固件烧录包。这些分区镜像可能主要用来做OTA 升级，或者创建自己的烧录固件。
出于这方面考虑，Tina SDK 提供一种可以将分区镜像文件整理输出，同时输出后的可以脱离Tina SDK 再次进行打包的方法。方便可移植到贵方SDK 中进行二次修改和再次打包。

## 5.1 开启[pack out of tina] 功能

上述功能需要开启[pack out of tina] 功能

```
make meunconfig
Target Image -->
[*] support pack out of tina
```

开启[pack out of tina] 功能后，直接pack 即可看到打印提示多生成了一个文件夹：

```
out/xx方案/aw_pack_src
```

## 5.2 aw_pack_src 目录介绍

```
./aw_pack_src
|--aw_pack.sh #执行此脚本即可在aw_pack_src/out/目录生成固件
|--config #打包配置文件
|--image #各种镜像文件，可替换，但不能改文件名
| |--boot0_nand.fex #nand介质boot0镜像
| |--boot0_sdcard.fex #SD卡boot0镜像
| |--boot0_spinor.fex #nor介质boot0镜像
| |--boot0_spinor.fex #nor介质boot0镜像
| |--boot_package.fex #nand和SD卡uboot镜像
| |--boot_package_nor.fex #nor介质uboot镜像
| |--env.fex #env环境变量镜像
| |--boot.fex #内核镜像
| |--rootfs.fex #rootfs镜像
|--other #打包所需的其他文件
|--out #固件生成目录
|--tmp #打包使用的临时目录
|--rootfs #存放rootfs的tar.gz打包,给二次修改使用
|--tools #工具
|--lib_aw #拷贝全志方案的库文件，如多媒体组件eyesempp等,给应用app编译链接使用(没有选择这些库，则可
能是空文件)
|--README #关于板级方案的一些说明，例如分区布局等等(无说明则没有这个文件夹)。
```

1. aw_pack_src 目录以及里面的文件，您可以移植到贵方SDK 上。当您重新执行aw_pack.sh的时候，即会根据这个目录里面的分区文件和分区信息重新打包成生成全志的固件包。基于此，您可以手动二次修改分区镜像文件后再重新打包。
2. 您也可以将分区文件单独拿出来，去做自己的OTA 升级包，或者做自己的flash 固件。
   **说明**
   **Tina SDK 原本设定好的分区大小和分区名字在aw_pack_src 是无法改变的。**