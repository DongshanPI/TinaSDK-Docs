# MPP_Sample_使用说明
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
