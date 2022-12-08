# MPP_Sample_使用说明
## 1 前言 

### 1.1 概述 

整理 MPP sample 使用说明文档的目的是：使 MPP sample 更好用。 

### 1.2 读者对象 

本文档（本指南）主要适用于以下人员： • 技术支持工程师 • 软件开发工程师 

#### 1.3 约定 

#### 1.3.1 符号约定 

本文中可能出现的符号如下：



## 2 简介

MPP sample 一般存放在 MPP Middleware 的 sample 目录下。此外，MPP Framework 的 demo 目录下也有一些 sample。 本文档主要介绍 MPP Middleware 各 sample 的基本使用方法：配置、编译、测试以及 sample 类别、各平台方案上的支持情况和测试方法等。 文末 FAQ 部分对音视频编解码功能的测试方法和测试工具的使用做了详细介绍。	

## 3 配置

​	本章节主要介绍 MPP sample 的配置方法。目前 Tina 和 Melis 上存在差异。

### 3.1 Tina 上 MPP sample 的配置方法 

Tina V853、V833 和 V536 方案上，MPP sample 支持从 menuconfig 中配置。选中 [*] select mpp sample 之后，当前支持测试的 MPP sample 就会显示出来，默认都是未选中的状态。此时， 可以勾选想要测试的 sample，然后保存配置，重新编译 MPP 即可。

```
$ make menuconfig
	Allwinner --->
		eyesee-mpp --->
			[*] select mpp sample
```

Tina 其他方案（如：V316 等）上，目前 MPP sample 不支持从 menuconfig 中配置。若想开启 或关闭 sample，只能修改 tina.mk 文件，然后重新编译 MPP。

1. 各 sample 对组件的依赖关系详情，请参考文件 tina\package\allwinner\eyesee-mpp\middleware \Config.in。
2. 因为每个 MPP sample 编译时，依赖的 MPP 组件不一样，所以，只有当支持的 MPP 组 件打开后，相关的 MPP sample 才会显示出来。否则，不可见。 
3. 由于在编译 MPP 基础库libaw_mpp.a 或 libmedia_mpp.so 时，mpi region 是默认开启的， 且 mpi region 依赖 mpi_vi 和 mpi_venc，所以 mpi_vi 和 mpi_venc 是运行 MPP sample 的基础组件，需要保持常开。

### 3.2 Melis 上 MPP sample 的配置方法

 Melis 上各方案，目前 MPP sample 不支持从 menuconfig 中配置。若想开启或关闭 sample，只 能修改 Makefile 文件，然后重新编译 MPP。