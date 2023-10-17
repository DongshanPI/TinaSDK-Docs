## 9 FAQ

本章节主要汇总 MPP sample 使用过程中可能遇到的一些问题及对应的解决方法。

### 9.1 某些方案上开机后执行某些 MPP sample 会运行失败

在某些客户分支方案上运行 MPP sample 之前，需要先 kill 掉客户的应用程序（比如：/usr/bin /sdvcam），否则，可能出现与 MPP sample 测试的资源冲突而导致其运行失败。

### 9.2 MPP sample 测试时 SD 卡识别异常

情况一： 

在测试一些录流的 MPP sample 之前，由于系统存储空间不足，需要准备 SD 卡。但是，有时会 碰到识别到的 SD 卡空间太小，格式化 SD 卡也没用。此时，需要在 Linux 环境下用 dd 命令删 除前面的分区。 

情况二： 

某些客户方案上，SD 卡默认没有 mount。mount 的方法：

```
# mkdir /mnt/extsd/
# mount -t vfat /dev/mmcblk0 /mnt/extsd/
或
# mount -t vfat /dev/mmcblk0p1 /mnt/extsd/
```

### 9.3 视频编解码常用功能检查方法 

#### 9.3.1 常用检测工具介绍 

##### 9.3.1.1 码流播放软件

测试时，检查编码后的文件是否正常，一般需要使用 VLC 等播放软件看下是否有花屏、马赛克、 卡顿等问题。

![image-20221120185053786](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185053786.png)

<center>图 9-1: VLC 播放码流</center>



##### 9.3.1.2 码流分析软件

测试时，检查编码参数是否符合预期，一般需要使用 MediaInfo 软件。

![image-20221120185136099](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185136099.png)

<center>图 9-2: MediaInfo 分析码流</center>



提取 MediaInfo 分析码流的信息如下：

```
概览
完整名称 : Y:\tools\ffmpeg\1-编码格式\1-1080p-H264.mp4
格式 : MPEG-4
格式配置 (Profile) : Base Media
编解码器 ID : isom (isom/iso2/mp41)
文件大小 : 15.5 MiB
时长 : 57 秒 648 毫秒
总体码率 : 2 252 kb/s
视频
ID : 1
格式 : AVC
格式/信息 : Advanced Video Codec
格式配置 (Profile) : High@L5.1
格式设置 : CABAC / 1 Ref Frames
格式设置, CABAC : 是
格式设置, 参考帧 : 1 帧
格式设置, GOP : M=1, N=50
编解码器 ID : avc1
编解码器 ID/信息 : Advanced Video Coding
时长 : 57 秒 648 毫秒
源, 时长 : 57 秒 599 毫秒
码率 : 2 106 kb/s
宽度 : 1 920 像素
高度 : 1 080 像素
画面比例 : 16:9
帧率 : 20.018 FPS
色彩空间 : YUV
色度抽样 : 4:2:0
位深 : 8 位
扫描类型 : 逐行扫描 (连续)
数据密度 [码率/(像素*帧率)] : 0.051
流大小 : 14.5 MiB (94%)
源, 流大小 : 14.5 MiB (94%)
语言 : 英语 (English)
色彩范围 : Limited
色彩原色 : BT.709
传输特性 : BT.709
矩阵系数 : BT.709
mdhd_Duration : 57648
编码配置区块 (box) : avcC
```

##### 9.3.1.3 YUV 文件分析软件

分析 YUV 文件一般使用 YUView 软件。 

为使工具分析能力更强，需要配置 ffmpeg 动态库。 

YUView 工具配置 ffmpeg 动态库的方式：

![image-20221120185203405](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185203405.png)

<center>图 9-3: YUView 软件配置 ffmpeg 动态库</center>



##### 9.3.1.4 H264 码流分析工具

常用的 H264 码流分析工具：Elescard StreamEye。 该工具可解析帧类型信息、流的信息等，支持逐帧播放和分析编码层内容等。

#### 9.3.2 典型的视频编解码功能检测示例

##### 9.3.2.1 编码格式

使用 MediaInfo 软件检查编码格式。

• 格式: HEVC

##### 9.3.2.2 彩转灰

使用 PC 软件 VLC 播放彩转灰测试生成的视频文件，效果如下：

![image-20221120185237306](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185237306.png)

<center>图 9-4: 彩转灰效果</center>



##### 9.3.2.3 旋转编码

使用 PC 软件 VLC 播放测试生成的视频文件，效果如下：

1. 第一组

   • 不旋转、不翻转的效果

   • 不旋转、翻转的效果

   • 旋转 90 度、不翻转的效果 

   • 旋转 90 度、翻转的效果

   ![image-20221120185305801](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185305801.png)

<center>图 9-5: 旋转-1</center>



2. 第二组 

   • 旋转 180 度、不翻转的效果 

   • 旋转 180 度、翻转的效果 

   • 旋转 270 度、不翻转的效果 

   • 旋转 270 度、翻转的效果

   ![image-20221120185325320](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185325320.png)

   <center>图 9-6: 旋转-2</center>

   



##### 9.3.2.4 P 帧帧内刷新

H264： 

使用 Elescard StreamEye 4.6 工具（只支持 H264），按如下截图配置后，开启 P 帧帧内刷 新后，图像看到一个橙色的竖状矩形条，同时逐帧往后查看时矩形条会从左往右移动；关闭 P 帧 帧内刷新后，则无此矩形条。

![image-20221120185345891](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185345891.png)

<center>图 9-7: H264-开启 P 帧帧内刷新</center>

![image-20221120185407387](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185407387.png)

<center>图 9-8: H264-关闭 P 帧帧内刷新</center>





H265： 

使用 YUView 工具（需要配置 ffmpeg 动态库）可分析 H265 文件，勾选 Pred Mode 后，可显 示一个蓝色的竖状矩形条，同时逐帧往后查看时矩形条会从左往右移动；关闭 P 帧帧内刷新后， 则无此矩形条。

![image-20221120185425260](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185425260.png)

<center>图 9-9: H265-开启 P 帧帧内刷新</center>

![image-20221120185502545](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185502545.png)

<center>图 9-10: H265-关闭 P 帧帧内刷新</center>

##### 9.3.2.5 视频编码 OSD 

使用 PC 软件 VLC 播放测试生成的视频文件，效果如下： 

1.显示 OSD 的效果：

![image-20221120185520358](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185520358.png)

<center>图 9-11: 编码 OSD-1</center>



2.改变 Region 位置后，显示 OSD 的效果：

![image-20221120185539487](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185539487.png)

<center>图 9-12: 编码 OSD-2</center>



3.OSD 坐标和大小示意图

![image-20221120185558887](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185558887.png)

<center>图 9-13: OSD 坐标和大小示意图</center>





### 9.4 音频编解码常用功能检查方法 

#### 9.4.1 常用检测工具介绍

1.普通效果测试 将测试生成的 wav 文件拷贝到 PC 端使用 Windows Media Player 播放。 

2.专业波形测试 专业的音频波形分析软件：Audacity。 比如测试回声消除效果时，可借助该工具分析。 

使用说明： 检测采集通道的数据和回采通道的数据的延迟关系是否正确。 需要特殊的正弦波文件 tone_16k_accurate.wav，tone_8k_accurate.wav。 一般采集通道的数据应该稍微延迟一些。

操作步骤： 

1. 为进行专业波形测试，需修改代码，audio_hw.c 中打开宏 #define AI_HW_AEC_DEBUG_EN。
2. 然后编译 sample_aec，这样底层模块在运行过程中会保存采集通道的数据和回采通道的数 据。 
3. 再用软件 Audacity 进行分析，从波形上看延迟时间是否正常。

![image-20221120185621911](https://photos.100ask.net/Tina-Sdk/MPPSampleInstructionsUse_image-20221120185621911.png)

<center>图 9-14: 通道波形对比图</center>



