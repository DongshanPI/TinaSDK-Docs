## 1 前言

### 1.1 编写目的

介绍 U-Boot 的编译打包、基本配置、常用命令的使用、基本调试方法等, 为 U-BOOT 的移植及应用开发提供了基础。

### 1.2 适用范围

本文档适用于 brandy2.0, 即 U-Boot-2018 平台。

### 1.3 相关人员

U-Boot 开发/维护人员，内核开发人员。



## 2 LICHEE 类宏关键字解释

请到 longan 目录下的.buildconfig 查看目前使用了以下 LICHEE 类宏。

```
LICHEE_IC    ——> IC名\
LICHEE_CHIP  ——> 平台名\
LICHEE_BOARD ——> 板级名\
LICHEE_ARCH  ——> 所属架构\
LICHEE_BOARD_CONFIG_DIR ——> 板级目录\
LICHEE_BRANDY_OUT_DIR   ——> bin文件所在目录\
LICHEE_PLAT_OUT         ——> 平台临时bin所在目录\
LICHEE_CHIP_CONFIG_DIR  ——> IC目录
```


