# UART开发指南介绍
## 1 概述

### 1.1 编写目的

介绍 Linux 内核中 UART 驱动的接口及使用方法，为 UART 设备的使用者提供参考。



### 1.2 适用范围

​														  			表 1-1: 适用产品列表

| 内核版本         | 驱动文件     |
| ---------------- | ------------ |
| Linux-4.9 及以上 | sunxi-uart.c |



### 1.3 相关人员

UART 驱动、及应用层的开发/维护人员。





## 2 模块介绍

### 2.1 模块功能介绍

Linux 内核中,UART 驱动的结构图 1 所示, 可以分为三个层次: 





​																图 2-1: Linux UART 体系结构图

1. Sunxi UART Driver, 负责 SUNXI 平台 UART 控制器的初始化、数据通信等, 也是我们要实现的部分。

2. UART Core, 为 UART 驱动提供了一套 API, 完成设备和驱动的注册等。

3. TTY core, 实现了内核中所有 TTY 设备的注册和管理。



### 2.2 相关术语介绍

​															表 2-1: UART 模块相关术语介绍

| 术语    | 解释说明                                                     |
| ------- | ------------------------------------------------------------ |
| Sunxi   | 指 Allwinner 的一系列 SoC 硬件平台                           |
| UART    | Universal Asynchronous Receiver/Transmitter，通用异步收发传输器 |
| Console | 控制台，Linux 内核中用于输出调试信息的 TTY 设备              |
| TTY     | TeleType/TeleTypewriters 的一个老缩写，原来指的是电传打字机，现在泛指和计算机串行端口连接的终端设备。TTY 设备还包括虚拟控制台，串口以及伪终端设备 |





### 2.3 源码结构介绍

```
linux4.9
|-- drivers
|     |-- tty
|     |    |-- serial
|     |    |-- serial_core.c
|     |    |-- sunxi-uart.c
|     |    |-- sunxi-uart.h
```
