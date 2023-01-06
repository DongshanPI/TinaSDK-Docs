## 1 概述

* 编写目的
  本文档将介绍sunxi 平台Display Engine 模块中LCD 的调试方法。

  1. LCD 调试方法，调试手段。
  2. LCD 驱动编写。
  3. lcd0 节点下各个属性的解释。
  4. 典型LCD 接口配置。

适用范围：

sunxi 平台DE1.0/DE2.0 中LCD 屏幕参数设置。





## 2 相关术语介绍

<center>表2-1: LCD 相关术语</center>

| 术语  | 解释说明                                                     |
| ----- | ------------------------------------------------------------ |
| SUNXI | Allwinner 一系列SoC 硬件平台                                 |
| LCD   | Liquid Crystal Display, 液晶显示器                           |
| MIPI  | Mobile Industry Processor Interface                          |
| DSI   | Display Serial Interface，显示串行接口                       |
| I8080 | Intel 8080LCD 接口                                           |
| RGB   | 这里指一种LCD 接口，该接口发送不经过任何编码的RGB 分量       |
| LVDS  | Low-Voltage Differential Signaling 一种LCD 接口，低压和差分传输是其特点 |

## 3 IC 规格

LCD 接口相关规格：

1. 支持双显，异显。也就是显示内容可以不一样，显示分辨率可以不一样，屏接口也可以不一样。

2. 支持MIPI-DSI 接口, 数量一个。最大支持1920x1200@60 分辨率，满足宽高不要超过2048，像素时钟不超过180MHz 都支持。

3. 支持RGB 接口，数量2 个。其中主显支持并行RGB666，副显并行支持RGB888, 并行RGB 接口最大支持1920x1200@60 分辨率，满足宽高不要超过2048，像

素时钟不超过180MHz 都支持。或者两个串行RGB 接口，串行RGB 的最高分辨率最大不超过800*480@60

4. 支持两个dual-link LVDS 接口, 最大支持1920x1200@60 分辨率，满足宽高不要超过2048，像素时钟不超过180MHz 都支持。或者4 个single-link LVDS 接

  口，分辨率最高支持1366*768@60。

5. 两个I8080 接口。分辨率最高支持800*480@60。

6. LVDS 接口支持信号同显。每两个single link LVDS 接口必须连接到完全一样的LVDS 接口的屏上，将完全一样的数据发送到这两个屏上，做到信号一样。所以

  理论上，T509 能做到4显，其中前2 个和后2 个分辨率可以不一样，2 个之间的分辨率必须一样，且必须连接一样的LCD 屏。

**说明**

在多显的场景下，以上接口可以自由搭配，除了MIPI-DSI 必须用在主显上。

**技巧**

一个dual link LVDS 接口共20 条线，它可以拆分成两个single link 的LVDS 接口，假设为lvds0 和lvds1，选择一个single link 的时候做显示的时候，必须选择lvds0。