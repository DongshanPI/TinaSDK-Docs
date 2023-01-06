## 1 概述

### 1.1 编写目的

简要介绍tina 平台功耗管理机制，为关注功耗的开发者，维护者和测试者提供使用和配置参考。

### 1.2 适用范围

<center>表1-1: 适用产品列表</center>

| 产品名称 | 内核版本  | 休眠类型                   | 参与功耗管理的协处理器 |
| -------- | --------- | -------------------------- | ---------------------- |
| R328     | Linux-4.9 | NormalStandby              | 无                     |
| R329     | Linux-4.9 | SuperStandby               | DSP0                   |
| D1-H     | Linux-5.4 | NormalStandby              | 无                     |
| V853     | Linux-4.9 | SuperStandby/NormalStandby | 无                     |

注：若同时支持多种休眠类型，则系统最终进入的休眠状态，根据唤醒源的配置自动确定。一般来说（无特别说明），唤醒源仅包括rtc, nmi(powerkey), 

gpio(wlan,usb 插拔等) 这些唤醒源，则最终进入super standby，否则包含任意的其他的唤醒源，则进入normal standby。

### 1.3 适用人员

tina 平台下功耗管理相关的开发、维护及测试相关人员。