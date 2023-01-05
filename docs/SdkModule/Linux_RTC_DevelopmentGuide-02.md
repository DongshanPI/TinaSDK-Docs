## 2 模块介绍

Linux 内核中,RTC 驱动的结构图如下所示, 可以分为三个层次:
![image-20221216114050541](http://photos.100ask.net/tina-docs/Linux_RTC_DevGuide_image-20221216114050541.png)

接口层，负责向用户空间提供操作的结点以及相关接口。
• RTC Core, 为rtc 驱动提供了一套API, 完成设备和驱动的注册等。
• RTC 驱动层，负责具体的RTC 驱动实现，如设置时间、闹钟等设置寄存器的操作。

### 2.2 相关术语介绍

| 术语  | 解释说明                         |
| ----- | -------------------------------- |
| Sunxi | 指Allwinner 的一系列SoC 硬件平台 |
| RTC   | Real Time Clock，实时时钟        |



### 2.3 源码结构介绍

```
linux-4.9
└-- drivers
	└-- rtc
		|-- class.c
		|-- hctosys.c
		|-- interface.c
		|-- rtc-dev.c
		|-- rtc-lib.c
		|-- rtc-proc.c
		|-- rtc-sysfs.c
		|-- systohc.c
		|-- rtc-core.h
		|-- rtc-sunxi.c
		└-- rtc-sunxi.h
linux-5.4
└-- drivers
	└-- rtc
		|-- class.c
		|-- hctosys.c
		|-- interface.c
		|-- dev.c
		|-- lib.c
		|-- proc.c
		|-- sysfs.c
		|-- systohc.c
		|-- rtc-core.h
		|-- rtc-sunxi.c
		└-- rtc-sunxi.h
```

