## 3 配置

​	本章节主要介绍 MPP sample 的配置方法。目前 Tina 和 Melis 上存在差异。

### 3.1 Tina 上 MPP sample 的配置方法 

Tina V853、V833 和 V536 方案上，MPP sample 支持从 menuconfig 中配置。选中 [*] select mpp sample 之后，当前支持测试的 MPP sample 就会显示出来，默认都是未选中的状态。此时， 可以勾选想要测试的 sample，然后保存配置，重新编译 MPP 即可。

```
$ make menuconfig
	Allwinner --->
		eyesee-mpp --->
			[*] select mpp sample
```

Tina 其他方案（如：V316 等）上，目前 MPP sample 不支持从 menuconfig 中配置。若想开启 或关闭 sample，只能修改 tina.mk 文件，然后重新编译 MPP。

1. 各 sample 对组件的依赖关系详情，请参考文件 tina\package\allwinner\eyesee-mpp\middleware \Config.in。
2. 因为每个 MPP sample 编译时，依赖的 MPP 组件不一样，所以，只有当支持的 MPP 组 件打开后，相关的 MPP sample 才会显示出来。否则，不可见。 
3. 由于在编译 MPP 基础库libaw_mpp.a 或 libmedia_mpp.so 时，mpi region 是默认开启的， 且 mpi region 依赖 mpi_vi 和 mpi_venc，所以 mpi_vi 和 mpi_venc 是运行 MPP sample 的基础组件，需要保持常开。

### 3.2 Melis 上 MPP sample 的配置方法

 Melis 上各方案，目前 MPP sample 不支持从 menuconfig 中配置。若想开启或关闭 sample，只 能修改 Makefile 文件，然后重新编译 MPP。

## 4 编译

本章节主要介绍 MPP sample 的编译方法。 

以下以 tina 为例。 

编译命令

```
cleanmpp && mkmpp
```

编译后 MPP sample 测试程序和配置文件存放位置

```
有两个位置：
（1）每个 MPP sample 的源码目录下
tina\external\eyesee-mpp\middleware\sun8iw21\sample\sample_xxx\
（2）统一存放到bin目录下
tina\external\eyesee-mpp\middleware\sun8iw21\sample\bin\
```

PS：由于 MPP sample 测试程序个数较多，且每个测试程序大小约 10M 左右，故不方便打包 到文件系统中，测试时需要接 SD 卡，同时把测试程序和配置文件放到 SD 卡中进行测试。

## 5 测试

本章节主要介绍 MPP sample 的测试方法。

###  5.1 在板端用串口测试

1.将测试程序和配置拷贝到 SD 卡，然后将 SD 卡接到板端。 

2.将 SD 卡 mount 到/mnt/extsd 目录，检查测试程序和配置是否在路径/mnt/extsd 下，然后准备测试。

```
mkdir -p /mnt/extsd
mount -t vfat /dev/mmcblk0p1 /mnt/extsd
```

### 5.2 在 PC 端用 adb shell 测试

前提，需要配置 sdk 支持 adb（一般默认都支持）。

1. 在 PC 端启动 Windows PowerShell 终端，然后切换到测试程序和配置所在路径。 
2. 在 Windows PowerShell 终端，使用 adb push 命令将测试程序和配置推送到板端。

```
> adb push .\sample_virvi2venc\sample_virvi2venc /mnt/extsd/
.\sample_virvi2venc\sample_virvi2venc: 1 file pushed. 3.4 MB/s (8699676 bytes in 2.442s)
>
> adb push .\sample_virvi2venc\sample_virvi2venc.conf /mnt/extsd/
.\sample_virvi2venc\sample_virvi2venc.conf: 1 file pushed. 0.1 MB/s (693 bytes in 0.005s)
```

3. 在 Windows PowerShell 终端，使用 adb shell 命令登录到板端，然后准备测试。

### 5.3 测试指令

 以下分别以 sample_driverVipp 和 sample_virvi2venc 为例介绍没有配置文件和有配置文件两 种情况下，MPP sample 的测试指令。 【Tina】

```
# cd /mnt/extsd/
# ./sample_driverVipp # 需要逐个指定参数。
或者一次性指定所有测试参数
# ./sample_driverVipp 0 1920 1080 8 60 0 10 60 /mnt/extsd/
```

【Melis】

```
Melis上该sample的位置：
melis\source\ekernel\subsys\avframework\v4l2\drivers\media\platform\sunxi-vin\
sample_driverVipp
测试方法：
sample_driverVipp 0 1920 1080 8 60 0 10 60 /mnt/E/
```

【Tina】

```
# cd /mnt/extsd/
# ./sample_virvi2venc -path ./sample_virvi2venc.conf
```

【Melis】 

```
# sample_virvi2venc -path ./mnt/E/sample_virvi2venc.conf
```

