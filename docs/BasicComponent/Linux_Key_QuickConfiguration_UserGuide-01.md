# Linux_Key_快速配置_使用指南
## 1 前言

### 1.1 文档简介

本文介绍Tina 平台key 相关的快速配置和使用方法。

### 1.2 目标读者

Allwinner key 驱动驱动层/应用层的使用/开发/维护人员。

### 1.3 适用范围

<center>表1-1: 适用产品列表</center>

| 产品名称 | 内核版本    | 平台架构                    |
| -------- | ----------- | --------------------------- |
| R18      | Linux-4.4   | cortex-a53(64 位)           |
| R30      | Linux-4.4   | cortex-a53(64 位)           |
| R328     | Linux-4.9   | cortex-a7(32 位)            |
| R329     | Linux-4.9   | cortex-a53(64 位)           |
| R818     | Linux-4.9   | cortex-a53(64 位)           |
| R818B    | Linux-4.9   | cortex-a53(64 位)           |
| MR813    | Linux-4.9 x | Linux-4.9 cortex-a53(64 位) |
| MR813B   | Linux-4.9   | cortex-a53(64 位)           |
| R528     | Linux-5.4   | cortex-a7(32 位)            |
| D1       | Linux-5.4   | risc-v(64 位)               |
| H133     | Linux-5.4   | cortex-a7(32 位)            |

## 2 模块介绍

### 2.1 Key 配置

Allwinner 平台支持三种不同类型的Key：GPIO-Key，ADC-Key，AXP-Key。其中，GPIOKey又包括普通的gpio 按键和矩阵键盘。

按键相关配置根据平台不同内核会有部分差异，下面作详细介绍。

**说明**

若板子上没有使用我司的带有按键功能的PMU，则就没有对应的AXP 按键。

### 2.2 相关术语介绍

#### 2.2.1 软件术语

<center>表2-1: Key 软件术语列表</center>

| 术语     | 解释说明                       |
| -------- | ------------------------------ |
| Key      | 按键                           |
| GPIO-Key | 使用GPIO 检测按键的设备        |
| ADC      | 模数转换器                     |
| ADC-Key  | 通过ADC 读取电压检测按键的设备 |
| LRADC    | 精度为6 位的单通道ADC          |
| GPADC    | 精度为12 位的多通道ADC         |
| PMU      | 电源管理单元                   |
| AXP-Key  | 连接在电源芯片的按键           |