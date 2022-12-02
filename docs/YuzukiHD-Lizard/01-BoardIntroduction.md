# 东山哪吒STU硬件简述

* 此开发板的任何问题都可以在我们的论坛交流讨论 https://forums.100ask.net/c/aw/15 

## 硬件简述

东山哪吒STU开发板是一款针对于教育学习专门设计的一系列开发板，分别有

 * 最小主板：只保留一些学习调试最基本接口，做到最具性价比，**仅售149**。
 * 全阵脚引出的DIY底板：主要是供DIY极客爱好者使用，可以自行DIY设计。 **仅售29**
 * 专门的配套项目底板：针对于芯片的使用场景设计出专门的项目底板，结合课程学习使用。 **仅售149**
 * 配套的邮票孔封装D1s核心板：针对于企业级客户或者做产品的客户使用。

### 最小主板
如下图板载资源所示最小主板有:

 * 正面：TYPE-C TTL供电与调试接口，直接连接电脑USB接口即可实现 串口调试与供电二合一，无需额外的连接线。
 * 正面：RJ45千兆以太网接口，主要用于网络启动系统下载内核等操作，方便调试开发。
 * 正面：TYPE-C的USB OTG接口，用于烧写系统与作为OTG主从设备使用。
 * 正面：引出 HDMI接口，可用于连接显示器等设备。
 * 背面：TF卡接口，可用于调试与连接TF卡启动系统。
 * 背面：256MB  SPI NAND FLASH 芯片。

![DongshanNezhaSTU-TOP_001](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/DongshanNezhaSTU/DongshanNezhaSTU-TOP_001.png)


### DIY全针脚底板
全针脚DIY底板，将最小主板的所有未使用引脚全都引出到底板排针上，并提供全部硬件设计资料，可以自行使用 嘉立创 设计生产，也可以直接从我们这里购买。
主要适用于喜欢DIY的同学。

下图是 **最小主板** 与 **DIY全针脚底板** 连接示意图。

![DongshanNezhaSTU-DIY_003](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/DongshanNezhaSTU/DongshanNezhaSTU-DIY_003.jpg)

### 全功能项目底板

全功能项目底板是用于扩展 哪吒STU最小板功能而设计，拥有更丰富的功能，主要用于项目学习，网络，蓝牙，音频，显示，红外， 以及传感器模块等设备。

全功能底板的板载功能有

* XR829 WIFI蓝牙模组芯片，Bluetooth支持标准蓝牙与 低功耗蓝牙，Wifi 支持2.4G hz 无线网络通信。
* MIPI DSI屏幕显示接口：支持最高 1920x 1200分辨率，接口兼容 全志哪吒 公板，后续会有配套屏幕模块。
* IR红外接收接口：支持红外信号接收。
* 3.5MM Audio OUT：支持常见 手机的四段式 3.5MM耳机，可用于播放音乐并录制声音。
* MIC1 MIC2:使用硅敏麦克风，用于专业拾音。
* SPEAKER：专门的功放接口，用于扬声器播放声音，接口是 1.25 mmx2 PH.
* USB TYPE-A HOST接口：用于连接 标准的 USB设备，比如 U盘 支持UVC的摄像头 等等设备。
* PCI-E接口：支持4G模块连接，可以进行PPPOE通信，开发板独立上网。
* 排针：将多余IO全部引出，用于扩展传感器模块等，电源兼容树莓派 接口。

![DongshanNezhaSTU-FullProject-Board_001](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/DongshanNezhaSTU/DongshanNezhaSTU-FullProject-Board_001.png)
