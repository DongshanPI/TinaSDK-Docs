[ [中文](https://dongshanpi.com/DongshanPi-One/01-BoardIntroduction) | [English](https://dongshanpi.com/DongshanPi-One/01-BoardIntroduction_EN) ]

* 东山PI壹号问题反馈交流区 https://github.com/DongshanPI/Dongshanpi-Docs/discussions/2
# 东山Pi壹号-开发板

> 东山Pi壹号开发板是联合芯片原厂星宸科技一起推的最小Linux开发板，使用的是SSD202D主控芯片，最小开发板接口只保留了几个常见接口 包含LED灯 按键 FPCRGB显示接口 以及一路TYPE C USBHOST接口 还有一个专门用来供电和调试二合一的 TypeC接口，背面同时增加了SD卡卡槽 也保留了专门的烧录接口。

* 星宸科技( SigmaStar)成立于2017年，是一家专注于人工智能视觉系统芯片领域的科技公司，产品在智能安防、智能车载、智能物联等多个细分领域全球领先。

* 创始团队发源于晨星半导体高管团队（Mstar，显示及电视芯片领域全球NO.1，三星、LG、TCL、华星光电、康佳、海信、京东方等公司最大芯片供应商，已与联发科合并）。


## 硬件描述

* 最小开发板规格

    开发板大小参考了标准的MINI PCI-E规格，在此基础上针对芯片和器件做了相应的调整，大概尺寸是宽3.8cm x 长5.1cm。

* 最小开发板器件布局

![image-20211202174358899](https://cdn.staticaly.com/gh/codebug8/DongshanPi-Photos@master/BoardIntroduction-01.23mck7imbajk.png)

* 最小开发板规格参数
  * 主控芯片： 星辰科技 SSD202D 内置128MB DDR 支持H264/H265解码 支持MJPG编码。
  * 存储：板载128MB SPI NAND FLASH芯片 以及专门的SD card接口
  * LED灯：红色x1 表示pwr  蓝色 绿色 均为用户灯。
  * Key：硬件复位按键x1  用户按键x1
  * 显示：50Pin FPC RGB888显示输出
  * 供电&调试：板载专用USB转TTL芯片同时给整个板子供电。
  * usbHost:  TypeC接口的USB HOST 支持连接支持USB协议的设备。
  * 扩展接口： 使用MINI-PCI-E接口 用于连接底板。

* 东山Pi壹号 开发板原理图文件：https://cowtransfer.com/s/9b745ce262544c

## 开发资源描述
### 配套CPU开发手册

各位同学 期待已久的东山PI壹号 (SSD202D)部分常用外设CPU开发手册已经公开出来了，里面包含寄存器地址及描述哦，大家快来下载阅读。<br>
获取连接：：https://cowtransfer.com/s/0756a2af637745 里面包含<br>

* SSD202D_CHIPTOP_百问网.pdf
* SSD202D_CLKGEN_百问网.pdf
* SSD202D_EMAC_百问网.pdf
* SSD202D_GPIO_百问网.pdf
* SSD202D_GPI_INT_百问网.pdf
* SSD202D_I2C_百问网.pdf
* SSD202D_INTR_CPUINT_百问网.pdf
* SSD202D_INTR_CTRL_百问网.pdf
* SSD202D_IR_百问网.pdf
* SSD202D_PADTOP_百问网.pdf
* SSD202D_PWM_百问网.pdf
* SSD202D_RTC_百问网.pdf
* SSD202D_SAR_百问网.pdf
* SSD202D_SPI_百问网.pdf
* SSD202D_TIMER_百问网.pdf
* SSD202D_UART & FUART_百问网.pdf
* SSD202D_WDT_百问网.pdf

### 芯片原厂配套SDK

> 系统版本介绍,目前整套SDK基于星辰科技芯片原厂释放出来的 TAKOYAKI_DLC00V030 版本进行适配讲解。

| 名称        | 介绍     |
| :----------- | :------- |
|boot	| 存放适配好的uboot的源码，详见左侧编译烧写Boot|
|kernel	| 存放适配好的linux kernel的源码，详见左侧编译烧写kernel|
|project| 主要用于打包可供升级的image包，详见左侧编译烧写系统|
|sdk	| 存放一些简单的demo code，详见左侧 应用Demo示例|

* 芯片原厂SDK的优势: 

    这套框架是星辰科技针对于产品方案 项目方案等专门进行调整过的sdk 里面包含了许多实际产品中的demo示例，如果我们想快速落地产品 可以使用此套sdk。

* 芯片原厂SDK的劣势

    由于这套SDK是为了做产品，所以在此基础上做了很多框架性的改动，和我们常见的Linux开发有一定的区别，需要配合着原厂的开发文档才可以使用。

* 详细的获取及开发方式请参考后续 **原厂SDK开发入门** 章节。

### Linux社区版本SDK
> 社区版本使用的是基于 主线LinuxKernel Uboot 以及Buildroot进行开发。

* 社区版本优势

    非常方便用来学习Linux基础知识，熟悉整套开发流程，且符合Linux开发标准 有及其丰富的社区参考资料书籍。

* 社区版本劣势

    因为存在很多产品内用不到的组件驱动，并且比较消耗资源，所以不太适合用来做产品。


* 详细的获取及开发方式请参考左侧 **社区版本开发** 章节。



