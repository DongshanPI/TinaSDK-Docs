## 1 概述

### 1.1 编写目的

介绍TinaLinux 下安全方案的功能。安全完整的方案基于normal 方案扩展，覆盖硬件安全、安全启动（Secure Boot）、安全系统（Secure OS）、安全存储

（Secure Storage）、安全应用（Trust Application）、完整性保护（Dm-Verity）、强制访问控制（MAC）等方面。

### 1.2 适用范围

适用于基于硬件平台: 全志R18、R30、R311、MR133、R328、MR813、MR813B、R329、R818、R818B、R528、V853 芯片。

软件平台: Tina V3.5 及其后续版本。

![image-20230103095336137](https://photos.100ask.net/Tina-Sdk/Linux_Security_DevGuide_image-20230103095336137.png)

### 1.3 相关人员

适用于TinaLinux 平台的客户及相关技术人员。

### 1.4 配置文件

本文涉及到一些配置文件，在此进行说明。

注意：新SDK 配置文件优先级高于旧SDK 文件优先级。

**env*.cfg配置文件路径**

```
tina/device/config/chips/<chip>/configs/<board>/env.cfg #新SDK，优先级高
tina/device/config/chips/<chip>/configs/<board>/linux/env-<kernel-version>.cfg #新SDK，优先
级中
tina/device/config/chips/<chip>/configs/default/env.cfg #新SDK，优先级低
tina/target/allwinner/<board>/configs/env-<kernel-version>.cfg #旧SDK，优先级最低
```

**sys_config.fex路径：**

```
tina/device/config/chips/<chip>/configs/<board>/sys_config.fex #新SDK
tina/target/allwinner/<board>/configs/sys_config.fex #旧SDK，优先级最低
```

说明：

如果存在uboot-board.dts，uboot 会使用uboot-board.dts 中配置。不存在uboot-board.dts，uboot 会使用sys_config.fex 中的配置。

**dragon_toc*.cfg配置文件路径：**

```
tina/device/config/chips/<chip>/configs/default/dragon_toc*.cfg #新SDK，优先级高
tina/device/config/common/sign_config/dragon_toc*.cfg #新SDK，优先级低
tina/target/allwinner/<chip>-common/sign_config/dragon_toc*.cfg #旧SDK，优先级高
tina/target/allwinner/generic/sign_config/dragon_toc*.cfg #旧SDK，优先级低
```

**version_base.mk配置文件路径：**

```
tina/device/config/chips/<chip>/configs/default/version_base.mk #新SDK，优先级高
tina/device/config/common/version/version_base.mk #新SDK，优先级低
tina/target/allwinner/<chip>-common/version/version_base.mk #旧SDK，优先级高
tina/target/allwinner/generic/version/version_base.mk #旧SDK，优先级低
```

