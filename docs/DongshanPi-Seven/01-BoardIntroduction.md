# 东山Pi柒号-开发板
> 东山PI柒号开发板是基于 STM32MP157DAC 设计的一款最小开发板。其中核心板使用的是米尔的DDR512MB  EMMC4GB配置，最小底板引出了一些常用接口，比如typec 调试供电，typec OTG系统烧写接口，Tf卡接口 TypeA 立式usb接口，千兆以太网接口，同时还专门引出了 RGB888 以及MIPI DSI显示接口，最后，依旧在最小板上保留了M4调试接口。

  使用这款板子您可以用来入门学习 `嵌入式Linux开发`,学习 `双核异域架构`,学习`TF-A`ARM安全固件,学习 `OPTEE-OS`,同时也可以用来学习了解 `rt-smart` `鸿蒙系统`,搭配底板您可以用来学习 完善的 `嵌入式驱动开发课程` ,详细的`Linux应用开发` ,还有后续的项目实战课程等。

  我们的配套资源主要分为两大部分，一部分是针对于只想学习嵌入式Linux应用开发快速做出一个项目，另一部分是想深入学习嵌入式Linux底层设备驱动开发，以及系统构建等。
## 硬件功能描述
> 如下图所示， DongshanPISeven 开发板功能位置示例图。

![DongshanPi-Seven01_BoardIntroduction-01](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/DongshanPI-Seven/DongshanPi-Seven01_BoardIntroduction-01.png)


* 开发板规格: 长宽约100mm x 70mm，为了方便连接使用了SODIMM接口。
* 核心板规格：核心板所有的芯片都在屏蔽罩下面，可以取下屏蔽罩看到。
    * SOC主控: STM32MP157DAC （双核CorteX A7 800Mhz  + 209Mhz M4 + 3D GPU ）
    * DDR：内置512MB DDR3
    * 存储: EMMC 4GB FLASH
    * 网络：千兆PHY芯片。
* 底板规格：
    * TypeA 立式USB 2.0 HOST接口。
    * RJ45千兆以太网接口。
    * Type-C OTG烧录接口。
    * Type-C 电源&TTL调试接口。
    * Reset Key  x1
    * Wakeup Key  x1
    * User Key x1
    * BootMode 启动方式切换开关。
    * Power Red Led x1
    * User Led x2 
    * FPC-50Pin RGB888显示。
    * FPC-15Pin MIPI DSI显示。
    * RTC 1.25mm 接口。
    * M4 Core Debug 接口。

## 硬件资料
* 原理图文件：https://cowtransfer.com/s/33c3b1ca51934e

## 扩展引脚定义

## 配套项目底板
![DongshanPi-Seven01_BoardAndBase-Introduction-01](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/DongshanPI-Seven/DongshanPi-Seven01_BoardAndBase-Introduction-01.png)
