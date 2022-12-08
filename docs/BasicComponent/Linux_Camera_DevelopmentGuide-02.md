# Linux_Camera 源码结构介绍
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

### 2.3 源码结构介绍

#### 2.3.1 linux3.4 VFE 框架

驱动路径位于linux-3.4/drivers/media/video/sunxi-vfe 下。

```
sunxi-vfe:.
│ bsp_common.c ;底层bsp共用的函数
│ bsp_common.h ;底层bsp共用函数头文件
│ config.c ;读取sys_config.fex的参数配置和isp参数
│ config.h ;读取sys_config.fex和isp参数函数的头文件
│ Kconfig
│ Makefile
│ platform_cfg.h ;区分各个平台的头文件
│ vfe.c ;v4l2驱动实现主体（包含视频接口和ISP部分）
│ vfe.h ;v4l2驱动头文件
│ vfe_os.c ;系统资源函数实现（pin，clock，memory）
│ vfe_os.h ;系统资源函数头文件
│ vfe_subdev.c ;sensor调用vfe资源函数
│ vfe_subdev.h ;sensor 调用vfe资源函数头文件
│
├─actuator
│ actuator.c ;vcm driver的一般行为
│ actuator.h ;vcm driver的头文件
│ dw9714_act.c ;具体vcm driver型号实现
│ Makefile
│
├─csi
│ bsp_csi.c ;底层csi bsp函数
│ bsp_csi.h ;底层csi bsp函数头文件
│ csi_reg.c ;csi硬件底层实现
│ csi_reg.h ;csi硬件底层实现头文件
│ csi_reg_i.h ;csi 寄存器资源头文件
│
├─device
│ camera.h ;camera公用结构体头文件
│ camera_cfg.h ;camera ioctl扩展命令头文件
│ Makefile
│ ov5640.c ;具体的sensor驱动
│
├─flash_light
│ flash.h ;led补光灯驱动头文件
│ flash_io.c ;led补光灯io控制实现
│ Makefile
│
├─lib
│ bsp_isp.h ;底层isp bsp函数头文件
│ bsp_isp_algo.h ;底层isp 算法bsp函数头文件
│ bsp_isp_comm.h ;底层isp共用函数头文件
│ isp_module_cfg.h ;isp里面各模块功能配置的头文件
│ libisp ;isp的函数库
│ lib_mipicsi2_v1 ;A31mipi库
│ lib_mipicsi2_v2 ;A80/A83mipi库
├─csi_cci
│ cci_helper.c ;cci 与设备相关初始化、注册以及通信等相关函数集实现
│ cci_helper.h ;cci 与设备相关初始化、注册以及通信等相关函数集实现头文件
│ bsp_cci.c ;cci 操作函数集实现
│ bsp_cci.h ;cci 操作函数集头文件
│ csi_cci_reg.c ;cci 底层实现
│ csi_cci_reg.h ;cci 底层实现头文件
│ csi_cci_reg_i.h ;cci寄存器资源头文件
│
├─mipi_csi
│ bsp_mipi_csi.c ;底层mipi bsp函数
│ bsp_mipi_csi.h ;底层mipi bsp函数头文件
│
└─utility
cfg_op.c ;读取ini文件的实现函数
cfg_op.h ;读取ini文件函数对应的头文件
```

#### 2.3.2 linux3.10 VFE 框架

驱动路径位于linux-3.10/drivers/media/platform/sunxi-vfe 下。

```
sunxi-vfe:.
│ bsp_common.c ;底层bsp共用的函数
│ bsp_common.h ;底层bsp共用函数头文件
│ config.c ;读取sys_config.fex的参数配置和isp参数
│ config.h ;读取sys_config.fex和isp参数函数的头文件
│ Kconfig
│ Makefile
│ platform_cfg.h ;区分各个平台的头文件
│ vfe.c ;v4l2驱动实现主体（包含视频接口和ISP部分）
│ vfe.h ;v4l2驱动头文件
│ vfe_os.c ;系统资源函数实现（pin，clock，memory）
│ vfe_os.h ;系统资源函数头文件
│ vfe_subdev.c ;sensor调用vfe资源函数
│ vfe_subdev.h ;sensor 调用vfe资源函数头文件
│
├─actuator
│ actuator.c ;vcm driver的一般行为
│ actuator.h ;vcm driver的头文件
│ dw9714_act.c ;具体vcm driver型号实现
│ Makefile
│
├─csi
│ bsp_csi.c ;底层csi bsp函数
│ bsp_csi.h ;底层csi bsp函数头文件
│ csi_reg.c ;csi硬件底层实现
│ csi_reg.h ;csi硬件底层实现头文件
│ csi_reg_i.h ;csi 寄存器资源头文件
│
├─device
│ camera.h ;camera公用结构体头文件
│ camera_cfg.h ;camera ioctl扩展命令头文件
│ Makefile
│ ov5640.c ;具体的sensor驱动
│
├─flash_light
│ flash.h ;led补光灯驱动头文件
│ flash_io.c ;led补光灯io控制实现
│ Makefile
│
├─lib
│ bsp_isp.h ;底层isp bsp函数头文件
│ bsp_isp_algo.h ;底层isp 算法bsp函数头文件
│ bsp_isp_comm.h ;底层isp共用函数头文件
│ isp_module_cfg.h ;isp里面各模块功能配置的头文件
│ libisp ;isp的函数库
│ lib_mipicsi2_v1 ;A31mipi库
│ lib_mipicsi2_v2 ;A80/A83mipi库
├─csi_cci
│ cci_helper.c ;cci 与设备相关初始化、注册以及通信等相关函数集实现
│ cci_helper.h ;cci 与设备相关初始化、注册以及通信等相关函数集实现头文件
│ bsp_cci.c ;cci 操作函数集实现
│ bsp_cci.h ;cci 操作函数集头文件
│ csi_cci_reg.c ;cci 底层实现
│ csi_cci_reg.h ;cci 底层实现头文件
│ csi_cci_reg_i.h ;cci寄存器资源头文件
│
├─mipi_csi
│ bsp_mipi_csi.c ;底层mipi bsp函数
│ bsp_mipi_csi.h ;底层mipi bsp函数头文件
│
└─utility
cfg_op.c ;读取ini文件的实现函数
cfg_op.h ;读取ini文件函数对应的头文件
```

#### 2.3.3 linux4.4 VFE 框架

驱动路径位于linux-4.4/drivers/media/platform/sunxi-vfe 下。

```
sunxi-vfe:.
│ bsp_common.c ;底层bsp共用的函数
│ bsp_common.h ;底层bsp共用函数头文件
│ config.c ;读取sys_config.fex的参数配置和isp参数
│ config.h ;读取sys_config.fex和isp参数函数的头文件
│ Kconfig
│ Makefile
│ platform_cfg.h ;区分各个平台的头文件
│ vfe.c ;v4l2驱动实现主体（包含视频接口和ISP部分）
│ vfe.h ;v4l2驱动头文件
│ vfe_os.c ;系统资源函数实现（pin，clock，memory）
│ vfe_os.h ;系统资源函数头文件
│ vfe_subdev.c ;sensor调用vfe资源函数
│ vfe_subdev.h ;sensor 调用vfe资源函数头文件
│
├─actuator
│ actuator.c ;vcm driver的一般行为
│ actuator.h ;vcm driver的头文件
│ dw9714_act.c ;具体vcm driver型号实现
│ Makefile
│
├─csi
│ bsp_csi.c ;底层csi bsp函数
│ bsp_csi.h ;底层csi bsp函数头文件
│ csi_reg.c ;csi硬件底层实现
│ csi_reg.h ;csi硬件底层实现头文件
│ csi_reg_i.h ;csi 寄存器资源头文件
│
├─device
│ camera.h ;camera公用结构体头文件
│ camera_cfg.h ;camera ioctl扩展命令头文件
│ Makefile
│ ov5640.c ;具体的sensor驱动
│
├─flash_light
│ flash.h ;led补光灯驱动头文件
│ flash_io.c ;led补光灯io控制实现
│ Makefile
│
├─lib
│ bsp_isp.h ;底层isp bsp函数头文件
│ bsp_isp_algo.h ;底层isp 算法bsp函数头文件
│ bsp_isp_comm.h ;底层isp共用函数头文件
│ isp_module_cfg.h ;isp里面各模块功能配置的头文件
│ libisp ;isp的函数库
│ lib_mipicsi2_v1 ;A31mipi库
│ lib_mipicsi2_v2 ;A80/A83mipi库
├─csi_cci
│ cci_helper.c ;cci 与设备相关初始化、注册以及通信等相关函数集实现
│ cci_helper.h ;cci 与设备相关初始化、注册以及通信等相关函数集实现头文件
│ bsp_cci.c ;cci 操作函数集实现
│ bsp_cci.h ;cci 操作函数集头文件
│ csi_cci_reg.c ;cci 底层实现
│ csi_cci_reg.h ;cci 底层实现头文件
│ csi_cci_reg_i.h ;cci寄存器资源头文件
│
├─mipi_csi
│ bsp_mipi_csi.c ;底层mipi bsp函数
│ bsp_mipi_csi.h ;底层mipi bsp函数头文件
│
└─utility
cfg_op.c ;读取ini文件的实现函数
cfg_op.h ;读取ini文件函数对应的头文件
```

#### 2.3.4 linux4.4 VIN 框架

驱动路径位于linux-4.4/drivers/media/platform/sunxi-vin 下。

```
sunxi-vin:
│ vin.c ;v4l2驱动实现主体（包含视频接口和ISP部分）
│ vin.h ;v4l2驱动头文件
│ top_reg.c ;vin对各v4l2 subdev管理接口实现主体
│ top_reg.h ;管理接口头文件
│ top_reg_i.h ;vin模块接口层部分结构体
│
├─modules
│ ├─actuator
│ │ actuator.c ;vcm driver的一般行为
│ │ actuator.h ;vcm driver的头文件
│ │ dw9714_act.c ;具体vcm driver型号实现
│ │ Makefile
│ ├─flash
│ │ flash.h ;led补光灯驱动头文件
│ │ flash_io.c ;led补光灯io控制实现
│ ├─sensor
│ │ camera.h ;camera公用结构体头文件
│ │ camera_cfg.h ;camera ioctl扩展命令头文件
│ │ sensor_helper.c ;sensor公用操作接口函数文件
│ │ sensor_helper.h ;sensor公用操作接口函数头文件
│ │ Makefile
│ │ ov5640.c ;具体的sensor驱动
│
├─platform
│ platform_cfg.h ;平台相关的配置接口
│
├─utility
│ bsp_common.h ;底层公用的格式配置函数头文件
│ bsp_common.c ;底层公用的格式配置函数文件
│ cfg_op.h ;解析配置文件接口头文件
│ cfg_op.c ;解析配置文件接口函数实现主体
│ config.h ;解析设备树的函数头文件
│ config.c ;解析设备树的接口函数主体
│ sensor_info.h ;sensor列表信息头文件
│ sensor_info.c ;获取sensor列表信息函数主体
│ vin_io.h ;vin框架io操作接口头文件
│ vin_io.c ;vin框架io操作接口文件
│ vin_os.h ;vin框架系统操作接口头文件
│ vin_os.c ;vin框架系统操作接口文件
│ vin_supply.h ;vin框架设置时钟频率等接口头文件
│ vin_supply.c ;vin框架设置时钟频率等接口函数主体
│
├─vin-cci
│ cci_helper.c ;cci 与设备相关初始化、注册以及通信等相关函数集实现
│ cci_helper.h ;cci 与设备相关初始化、注册以及通信等相关函数集实现头文件
│ bsp_cci.c ;cci 操作函数集实现
│ bsp_cci.h ;cci 操作函数集头文件
│ csi_cci_reg.c ;cci 底层实现
│ csi_cci_reg.h ;cci 底层实现头文件
│ csi_cci_reg_i.h ;cci寄存器资源头文件
│ sunxi_cci.c ;cci 接口封装实现
│ sunxi_cci.h ;cci 接口封装头文件
│
├─vin-csi
│ bsp_csi.c ;csi 操作函数集实现
│ bsp_csi.h ;csi 操作函数集头文件
│ csi_reg.c ;csi 底层实现
│ csi_reg.h ;csi 底层实现头文件
│ csi_reg_i.h ;csi寄存器资源头文件
│ parser_reg.c ;csi 底层实现
│ parser_reg.h ;csi 底层实现头文件
│ parser_reg_i.h ;csi寄存器资源头文件
│ sunxi_csi.c ;csi 接口封装实现
│ sunxi_csi.h ;csi 接口封装头文件
│
├─vin-isp
│ bsp_isp.c ;isp操作函数集实现
│ bsp_isp.h ;isp操作函数集头文件
│ bsp_isp_comm.h ;isp结构体定义
│ isp_default_tbl.h ;isp默认配置列表
│ isp_platform_drv.h ;isp平台操作集头文件
│ isp_platform_drv.c ;isp平台操作集实现
│ sunxi_isp.h ;sunxi平台isp操作集头文件
│ sunxi_isp.c ;sunxi平台isp操作集实现
│
├─vin-mipi
│ bsp_mipi_csi.c ;底层mipi bsp函数
│ bsp_mipi_csi.h ;底层mipi bsp函数头文件
│ bsp_mipi_csi_v1.c ;sunxi平台底层mipi bsp接口函数
│ sunxi_mipi.c ;sunxi平台mipi实现
│ bsp_mipi_csi.h ;sunxi平台mipi实现头文件
│ combo_common.c ;combo common头文件
│ protocol.h ;protocol头文件
├─vin-stat
│ vin_h3a.c ;vin 3a操作函数
│ vin_h3a.h ;vin 3a操作函数头文件
│ vin_ispstat.c ;sunxi isp stat操作函数
│ vin_ispstat.h ;sunxi isp stat操作函数头文件
│
├─vin-video
│ dma_reg.c ;csi模块dma操作函数
│ dma_reg.h ;csi模块dma操作函数头文件
│ dma_reg_i.h ;csi dma寄存器资源头文件
│ vin_core.c ;vin video核心函数
│ vin_core.h ;vin video核心函数头文件
│ vin_video.c ;vin video设备接口函数
│ vin_video.h ;vin video设备接口函数头文件
├─vin-vipp
│ sunxi_scaler.c ;scaler 子设备操作函数集
│ sunxi_scaler.h ;caler 子设备操作函数头文件
│ vipp_reg.c ;vipp操作函数集
│ vipp_reg.h ;vipp操作函数集头文件
│ vipp_reg_i.h ;vipp操作函数集资源头文件
```

#### 2.3.5 linux4.9 VIN 框架

驱动路径位于linux-4.9/drivers/media/platform/sunxi-vin 下。

```
sunxi-vin:
│ vin.c ;v4l2驱动实现主体（包含视频接口和ISP部分）
│ vin.h ;v4l2驱动头文件
│ top_reg.c ;vin对各v4l2 subdev管理接口实现主体
│ top_reg.h ;管理接口头文件
│ top_reg_i.h ;vin模块接口层部分结构体
├── modules
│ ├── actuator ;vcm driver
│ │ ├── actuator.c
│ │ ├── actuator.h
│ │ ├── dw9714_act.c
│ │ ├── Makefile
│ ├── flash ;闪光灯driver
│ │ ├── flash.c
│ │ └── flash.h
│ └── sensor ;sensor driver
│ ├── ar0144_mipi.c
│ ├── camera_cfg.h ;camera ioctl扩展命令头文件
│ ├── camera.h ;camera公用结构体头文件
│ ├── Makefile
│ ├── ov2775_mipi.c
│ ├── ov5640.c
│ ├── sensor-compat-ioctl32.c
│ ├── sensor_helper.c ;sensor公用操作接口函数文件
│ ├── sensor_helper.h
├── platform ;平台相关的配置接口
├── utility
│ ├── bsp_common.c
│ ├── bsp_common.h
│ ├── cfg_op.c
│ ├── cfg_op.h
│ ├── config.c
│ ├── config.h
│ ├── sensor_info.c
│ ├── sensor_info.h
│ ├── vin_io.h
│ ├── vin_os.c
│ ├── vin_os.h
│ ├── vin_supply.c
│ └── vin_supply.h
├── vin-cci
│ ├── sunxi_cci.c
│ └── sunxi_cci.h
├── vin-csi
│ ├── parser_reg.c
│ ├── parser_reg.h
│ ├── parser_reg_i.h
│ ├── sunxi_csi.c
│ └── sunxi_csi.h
├── vin-isp
│ ├── sunxi_isp.c
│ └── sunxi_isp.h
├── vin-mipi
│ ├── sunxi_mipi.c
│ └── sunxi_mipi.h
├── vin-stat
│ ├── vin_h3a.c
│ ├── vin_h3a.h
│ ├── vin_ispstat.c
│ └── vin_ispstat.h
├── vin_test
├── vin-video
│ ├── vin_core.c
│ ├── vin_core.h
│ ├── vin_video.c
│ └── vin_video.h
└── vin-vipp
├── sunxi_scaler.c
├── sunxi_scaler.h
├── vipp_reg.c
├── vipp_reg.h
└── vipp_reg_i.h
```

#### 2.3.6 linux5.4 VIN 框架

驱动路径位于linux-5.4/drivers/media/platform/sunxi-vin 下。

```
sunxi-vin:
├── Kconfig
├── Makefile
├── modules
│ ├── actuator ;vcm driver
│ │ ├── actuator.c
│ │ ├── actuator.h
│ │ ├── dw9714_act.c
│ │ ├── Makefile
│ ├── flash ;flash driver
│ │ ├── flash.c
│ │ └── flash.h
│ ├── sensor ;cmos sensor driver
│ │ ├── camera_cfg.h
│ │ ├── camera.h
│ │ ├── gc0310_mipi.c
│ │ ├── Makefile
│ │ ├── ov2710_mipi.c
│ │ ├── ov5640.c
│ │ ├── sensor-compat-ioctl32.c
│ │ ├── sensor_helper.c
│ │ ├── sensor_helper.h
│ ├── sensor-list
│ │ ├── sensor_list.c
│ │ └── sensor_list.h
│ └── sensor_power ;sensor上下电接口函数
│ ├── Makefile
│ ├── sensor_power.c
│ └── sensor_power.h
├── platform
├── top_reg.c
├── top_reg.h
├── top_reg_i.h
├── utility ;驱动通用接口
│ ├── bsp_common.c
│ ├── bsp_common.h
│ ├── cfg_op.c
│ ├── cfg_op.h
│ ├── config.c
│ ├── config.h
│ ├── vin_io.h
│ ├── vin_os.c
│ ├── vin_os.h
│ ├── vin_supply.c
│ └── vin_supply.h
├── vin.c ;sunxi-vin驱动注册入口
├── vin-cci ;i2c操作相关接口
│ ├── bsp_cci.c
│ ├── bsp_cci.h
│ ├── cci_helper.c
│ ├── cci_helper.h
│ ├── Kconfig
│ ├── sunxi_cci.c
│ └── sunxi_cci.h
├── vin-csi ;csi操作相关接口
│ ├── sunxi_csi.c
│ └── sunxi_csi.h
├── vin.h
├── vin-isp ;isp驱动
│ ├── sunxi_isp.c
│ └── sunxi_isp.h
├── vin-mipi ;mipi驱动
│ ├── sunxi_mipi.c
│ └── sunxi_mipi.h
├── vin-stat
│ ├── vin_h3a.c
│ └── vin_h3a.h
├── vin-tdm
│ ├── vin_tdm.c
│ └── vin_tdm.h
├── vin-video ;video节点相关的接口定义
│ ├── vin_core.c
│ ├── vin_core.h
│ ├── vin_video.c
│ └── vin_video.h
└── vin-vipp
├── sunxi_scaler.c
├── sunxi_scaler.h
```
