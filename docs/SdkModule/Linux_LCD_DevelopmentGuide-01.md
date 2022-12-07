# Linux_LCD_开发指南
## 1 概述

### 1.1 编写目的

介绍 sunxi 平台 display2 框架平台中 LCD 模块中

1. LCD 调试方法，调试手段

2. LCD 驱动编写

3. lcd0 节点下各个属性的解释

4. 典型 LCD 接口配置



###  1.2 适用范围

​														         表 1-1: 适用产品列表

| 内核版本  | 驱动文件                                   |
| --------- | ------------------------------------------ |
| Linux-4.9 | drivers/video/fbdev/sunxi/disp2/disp/lcd/* |
| Linux-5.4 | drivers/video/fbdev/sunxi/disp2/disp/lcd/* |



### 1.3 相关人员

系统整合人员，显示开发相关人员。





## 2 相关术语介绍

​                                                                  表 2-1: LCD 相关术语

| 术语  | 解释说明                                                     |
| ----- | ------------------------------------------------------------ |
| SUNXI | Allwinner 一系列 SoC 硬件平台                                |
| LCD   | Liquid Crystal Display, 液晶显示器                           |
| MIPI  | Mobile Industry Processor Interface                          |
| DSI   | Display Serial Interface，显示串行接口                       |
| I8080 | Intel 8080LCD 接口                                           |
| RGB   | 这里指一种 LCD 接口，该接口发送不经过任何编码的 RGB 分量     |
| LVDS  | Low-Voltage Differential Signaling 一种 LCD 接口，低压和差分传输是其特点 |