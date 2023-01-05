## 1 概述

### 1.1 编写目的

介绍 Sunxi SPINand mtd/ubi 驱动设计, 方便相关驱动和应用开发人员

### 1.2 适用范围

本设计适用于所有 sunxi 平台

### 1.3 相关人员

 Nand 模块开发人员，及应用开发人员等

## 2 术语、缩略语及概念

**MTD**：（Memory Technology device）是用于访问存储设备的 linux 子系统。本模块是MTD 子系统的 flash 驱动部分

**UBI**：UBI 子系统是基于 MTD 子系统的，在 MTD 上实现 nand 特性的管理逻辑，向上屏蔽nand 的特性

**坏块** **(Bad Block)**：制作工艺和 nand 本身的物理性质导致在出厂和正常使用过程中都会产生坏块