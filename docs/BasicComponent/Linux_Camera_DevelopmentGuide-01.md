# Linux_Camera_开发指南
## 1 概述

### 1.1 编写目的

介绍camera 模块在sunxi 平台上的开发流程。

### 1.2 适用范围

本文档目前适用于tina3.0 以上具备camera 的硬件平台。

### 1.3 相关人员

公司开发人员、客户。

## 2 模块介绍

### 2.1 模块功能介绍

用于接收并行或者mipi 接口的sensor 信号或者是bt656 格式的信号。

### 2.2 硬件介绍

目前Tina 系统的各平台camera 硬件接口、linux 内核版本以及camera 驱动框架如下表所示：

<center>表2-1: 平台CSI 框架</center>

| 平台  | 支持接口      | 是否具备ISP模块 | linux 内核版本 | camera 驱动框架 |
| ----- | ------------- | --------------- | -------------- | --------------- |
| F35   | 并口csi、mipi | 否              | 3.4            | VFE             |
| R16   | 并口csi       | 否              | 3.4            | VFE             |
| R18   | 并口csi       | 否              | 4.4            | VFE             |
| R30   | 并口csi       | 否              | 4.4            | VFE             |
| R40   | 并口csi       | 否              | 3.10           | VFE             |
| R311  | mipi csi      | 是              | 4.9            | VIN             |
| MR133 | mipi csi      | 是              | 4.9            | VIN             |
| R818  | mipi csi      | 是              | 4.9            | VIN             |
| MR813 | mipi csi      | 是              | 4.9            | VIN             |
| R528  | 并口csi       | 否              | 5.4            | VIN             |
| V536  | 并口csi、mipi | 是              | 4.9            | VIN             |
| V533  | 并口csi、mipi | 是              | 4.9            | VIN             |
| V831  | 并口csi、mipi | 是              | 4.9            | VIN             |
| V833  | 并口csi、mipi | 是              | 4.9            | VIN             |
| V851  | 并口csi、mipi | 是              | 4.9            | VIN             |
| V853  | 并口csi、mipi | 是              | 4.9            | VIN             |

注意：

1. 如果平台没有ISP 模块，那么将不支持RAW sensor（即sensor 只输出采集到的原始数据），文档中提到的RAW 等相关信息不用理会；
2. 如果平台没有支持mipi 接口，文档中提到的mipi 相关信息忽略；
3. 不同平台可能将使用不同的camera 驱动框架，这点注意区分；
