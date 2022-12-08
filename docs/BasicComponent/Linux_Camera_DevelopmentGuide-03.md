# Linux_Camera 模块开发
## 3 模块开发

### 3.1 模块体系结构描述

#### 3.1.1 VFE 框架

• 使用过程中可简单的看成是vfe 模块+ device 模块+af driver + flash 控制模块的方式；

• vfe.c 是驱动的主要功能实现，包括注册/注销、参数读取、与v4l2 上层接口、与各device 的下层接口、中断处理、buffer 申请切换等；

• device 文件夹里面是各个sensor 的器件层实现，一般包括上下电、初始化，各分辨率切换，yuv sensor 包括绝大部分的v4l2 定义的ioctrl 命令的实现；而raw 

sensor 的话大部分 ioctrl 命令在vfe 层调用isp 的库实现，少数如曝光/增益调节会透过vfe 层到实际器件层；

• actuator 文件夹内是各种vcm 的驱动；

• flash_light 文件夹内是闪光灯控制接口实现；

• csi 和mipi_csi 为对csi 接口和mipi 接口的控制文件；

• lib 文件夹为isp 的库文件；

• linux-3.0 前的版本相当于vivi.c+csi bsp 层

• linux-3.4 版本支持isp 驱动和双CSI

• linux-3.10 版本将mipi/csi/isp 模块化（由vfe.c 直接调用=>v4l2_subdev_ops）, 支持device tree

![image-20221122095738625](${media}/image-20221122095738625.png)

<center>图3-1: VFE</center>

#### 3.1.2 VIN 框架

• 使用过程中可简单的看成是vin 模块+ device 模块+af driver + flash 控制模块的方式；

• vin.c 是驱动的主要功能实现，包括注册/注销、参数读取、与v4l2 上层接口、与各device 的下层接口、中断处理、buffer 申请切换等；

• modules/sensor 文件夹里面是各个sensor 的器件层实现，一般包括上下电、初始化，各分辨率切换，yuv sensor 包括绝大部分的v4l2 定义的ioctrl 命令的实

现；而raw sensor 的话大部分ioctrl 命令在vin 层调用isp 库实现，少数如曝光/增益调节会透过vin 层到实际器件层；

• modules/actuator 文件夹内是各种vcm 的驱动；

• modules/flash 文件夹内是闪光灯控制接口实现；

• vin-csi 和vin-mipi 为对csi 接口和mipi 接口的控制文件；

• vin-isp 文件夹为isp 的库操作文件；

• vin-video 文件夹内主要是video 设备操作文件；

![image-20221122113602939](${media}/image-20221122113602939.png)

<center>图3-2: VIN</center>

#### 3.1.3 Camera 通路框架

• VIN 支持灵活配置单/双路输入双ISP 多通路输出的规格

• 引入media 框架实现pipeline 管理

• 将libisp 移植到用户空间解决GPL 问题

• 将统计buffer 独立为v4l2 subdev

• 将的scaler（vipp）模块独立为v4l2 subdev

• 将video buffer 修改为mplane 方式，使用户层取图更方便

• 采用v4l2-event 实现事件管理

• 采用v4l2-controls 新特性

![image-20221122113645959](${media}/image-20221122113645959.png)

<center>图3-3: camera Input</center>

### 3.2 驱动模块实现

#### 3.2.1 硬件部分

检查硬件电源，gpio 是否和原理图一致并且正确连接；检查sys_config.fex 或者board.dts 是否正确配置，包括使用的电源名称和电压，详见CSI 板级配置章节说

明；如果是电源选择有多个源头的请确认板子上的连接正确，比如0ohm 电阻是否正确的焊接为0ohm，NC 的电阻是否有正确断开等等。带补光灯的也需要检查

灯和driver IC 和控制io 是否连好。

#### 3.2.2 内核device 模块驱动

一般调试新模组的话建议以sdk 中的某个现成的驱动为基础修改：YUV 的并口模组以R40 平台(linux3.10) 的ov5640.c 为参考。

下面以ov5640.c 为例说明调试新模组需要注意的两点：

1. 添加Makefile

```
[linux-3.10/drivers/media/platform/sunxi-vfe/device/Makefile]
添加
obj-m + = ov5640.o (详见1)
详注：
1.具体取决于使用的模组，如果是新模组则将驱动代码放置在该device目录下。
```

2. 配置模组参数

配置参数在linux-3.10/drivers/media/platform/sunxi-vfe/device/ov5640.c 中，只需注意下面两个参数。

```
#define SENSOR_NAME "ov5640" (详见1)
#define I2C_ADDR 0x78 (详见2)
详注：
1.该参数为模组名，必须和sys_config.fex的csi0_dev0_mname或者board.dts的sensor0_mname保持一致。
2.I2C_ADDR可参考相应模组的datasheet，sys_config.fex的csi0_dev0_twi_addr与此值保持一致。
```

##### 3.2.2.1 驱动宏定义

```
#define MCLK (24*1000*1000)
```

sensor 输入时钟频率，可查看模组厂提供的sensor datasheet，datasheet 中会有类似inputclock frequency: 6~27 MHz 信息，这个信息说明可提供给sensor 的

MCLK 可以在6 M 到27 M之间。其中MCLK 和使用的寄存器配置强相关，在模组厂提供寄存器配置时，可直接询问当前配置使用的MCLK 频率是多少。

```
#define VREF_POL V4L2_MBUS_VSYNC_ACTIVE_LOW
#define HREF_POL V4L2_MBUS_HSYNC_ACTIVE_HIGH
#define CLK_POL V4L2_MBUS_PCLK_SAMPLE_RISING
```

并口sensor 必须填写，MIPI sensor 无需填写，可在sensor 规格书找到，如下

![image-20221122115215017](${media}/image-20221122115215017.png)

<center>图3-4: timing</center>

从上述的图像可得到以下信息：

1. VSYNC 在低电平的时候，data pin 输出有效数据，所以VREF_POL 设置为V4L2_MBUS_VSYNC_ACTIVE_即低电平有效；

2. HREF 在高电平的时候，data pin 输出有效数据，所以HREF_POL 设置为V4L2_MBUS_HSYNC_ACTIVE_即高电平有效；

3. CLK_POL 则是表明SOC 是在sensor 输出的pclk 上升沿采集data pin 的数据还是下降沿采集数据，如果sensor 在pclk 上升沿改变data pin 的数据，那么SOC 

  应该在下降沿采集，CLK_POL 设置为V4L2_MBUS_PCLK_SAMPLE_FALLING；如果sensor在pclk 下降沿改变data pin 的数据，那么SOC 应该在上降沿采集，

  CLK_POL 设置为V4L2_MBUS_PCLK_SAMPLE_RISING。

```
#define V4L2_IDENT_SENSOR 0x2770
```

一般填写sensor ID，用于sensor 检测。sensor ID 可在sensor 规格书的找到，如下

![image-20221122115536252](${media}/image-20221122115536252.png)

<center>图3-5: sensorid</center>

```
#define I2C_ADDR 0x6c
```

sensor I2C 通讯地址，可在sensor 规格书找到，如下

![image-20221122115814628](${media}/image-20221122115814628.png)

<center>图3-6: sccbid</center>

```
#define SENSOR_NAME OV5640
```

定义驱动名字，与系统其他文件填写的名字要一致，比如需要和sys_config.fex 中的sensorname 一致。

##### 3.2.2.2 初始化代码

```
static struct regval_list sensor_default_regs[] = {}; /* 填写寄存器代码的公共部分*/
static struct regval_list sensor_XXX_regs[] = {}; /* 填写各模式的寄存器代码，不同的模式可以是分辨率、帧率等*/
```

上述部分的寄存器配置，公共部分可以忽略，直接在模式代码中配置sensor 即可，相应的寄存器配置，可让模组厂提供。

##### 3.2.2.3 曝光增益接口函数

```
static int sensor_s_exp(struct v4l2_subdev *sd, unsigned int exp_val) /* 曝光函数*/
static int sensor_s_gain(struct v4l2_subdev *sd, unsigned int gain_val) /* 增益函数*/
```

AE 是同时控制曝光时间和增益的，所以需要在上面的函数中分别同时sensor 曝光和增益的寄存器。

![image-20221122174724278](${media}/image-20221122174724278.png)

<center>图3-7: expgain</center>

根据规格书中的寄存器说明，在相应的函数配置即可。若设置exp/gain 无效，可能的原因有：

• sensor 寄存器打开了AE；

• 设置值超出了有效范围

具体可根据模组厂提供的配置设置，如若检查之后设置仍失效，可与模组厂沟通，确认配置是否正确。

##### 3.2.2.4 上下电控制函数

```
static int sensor_power(struct v4l2_subdev *sd, int on)
```

控制sensor 上电、下电及进出待机状态，操作步骤须与规格书描述相同，注意power down 和reset pin 的电平变化。

![image-20221122174813656](${media}/image-20221122174813656.png)

<center>图3-8: powerup</center>

驱动中，按照规格书的上电时序进行配置，而如果上电之后测量硬件并没有相应的电压，这时候
检查硬件和软件配置是否一致。关于csi 电源的配置，操作流程可如下：

1. 先通过原理图确认sensor 模组的各路电源是连接到axp 的哪个ldo;
2. 查看sys_config.fex 的regulator 配置，在相应的ldo 后增加相应的字段，比如“csi-vdd”等；
3. 在sys_config.fex 的csi 部分，sensor 部分的电源后的字段再填写与上述一样的字段即可；
4. 根据sensor 规格书的要求，填写相应的电压即可；

以上图为例，确认sensor 驱动中的上电时序。

```
static int sensor_power(struct v4l2_subdev *sd, int on)
{
    int ret;
    ret = 0;
    switch (on) {
    /* STBY_ON 和STBY_OFF 基本不使用，可忽略这两个选项的配置*/
    case STBY_ON:
        ...
        break;
    case STBY_OFF:
        ...
        break;
    /* 上电操作*/
    case PWR_ON:
        sensor_print("PWR_ON!\n");
        cci_lock(sd);
        /* 将PWDN、RESET 引脚设置为输出*/
        vin_gpio_set_status(sd, PWDN, 1);
        vin_gpio_set_status(sd, RESET, 1);
        /* 按照上图知道，上电前PWDN、RESET 信号为低，所以将其设置为低电平*/
        vin_gpio_write(sd, PWDN, CSI_GPIO_LOW);
        vin_gpio_write(sd, RESET, CSI_GPIO_LOW);
        /* 延时*/
        usleep_range(1000, 1200);
        /* CAMERAVDD 为SOC 中的供电电源，部分板子可以忽略该电源，
        * 因为有些板子会通过一个vcc-pe 给上拉电阻等供电，所以需要
        * 使能该路电，有些是直接和iovdd 共用了，所以有部分会忽略该
        * 路电源配置.
        */
        vin_set_pmu_channel(sd, CAMERAVDD, ON);
        /* 将PWDN 设置为高电平*/
        vin_gpio_write(sd, PWDN, CSI_GPIO_HIGH);
        /*AF上电*/
        vin_set_pmu_channel(sd, AFVDD, ON);
        /* AVDD 上电*/
        vin_set_pmu_channel(sd, AVDD, ON);
        /* 延时，延时时长为T1，T1 的大小在datasheet 的上电时序图下面有标注*/
        usleep_range(1000, 1200);
        /* DOVDD 上电*/
        vin_set_pmu_channel(sd, IOVDD, ON);
        /* 延时，按照上电时序中的标注的T2 时间延时*/
        usleep_range(1000, 1200);
        /* DVDD 上电*/
        vin_set_pmu_channel(sd, DVDD, ON);
        /* 延时，按照上电时序中的标注的T3 时间延时*/
        usleep_range(1000, 1200);
        /* 将PWDN 设置为低电平*/
        vin_gpio_write(sd, PWDN, CSI_GPIO_LOW);
        /* 设置MCLK 频率并使能*/
        vin_set_mclk_freq(sd, MCLK);
        vin_set_mclk(sd, ON);
        /* 延时，按照上电时序中的标注的T4 时间延时*/
        usleep_range(1000, 1200);
        /* 将RESET 设置为高电平*/
        vin_gpio_write(sd, RESET, CSI_GPIO_HIGH);
        /* 延时，按照上电时序中的标注的T6 时间延时*/
        usleep_range(10000, 12000);
        cci_unlock(sd);
        break;
    /* 掉电操作*/
        case PWR_OFF:
        sensor_print("PWR_OFF!\n");
        cci_lock(sd);
        /* 具体的掉电操作同样的按照datasheet 的power off 操作即可*/
        vin_gpio_write(sd, PWDN, CSI_GPIO_HIGH);
        vin_gpio_write(sd, RESET, CSI_GPIO_LOW);
        vin_set_mclk(sd, OFF);
        vin_set_pmu_channel(sd, AFVDD, OFF);
        vin_set_pmu_channel(sd, AVDD, OFF);
        vin_set_pmu_channel(sd, DVDD, OFF);
        vin_set_pmu_channel(sd, IOVDD, OFF);
        vin_set_pmu_channel(sd, CAMERAVDD, OFF);
        vin_gpio_set_status(sd, PWDN, 0);
        cci_unlock(sd);
        break;
    default:
    	return -EINVAL;
    }
    return 0;
}
```

##### 3.2.2.5 检测函数

```
static int sensor_detect(struct v4l2_subdev *sd)
```

在开机加载驱动的时候，将会检测sensor ID，用于测试I2C 通讯是否正常和sensor 识别。

```
#define V4L2_IDENT_SENSOR 0x7750
    sensor_read(sd, 0x300A, &rdval);
    if (rdval != (V4L2_IDENT_SENSOR >> 8))
    	return -ENODEV;
    sensor_read(sd, 0x300B, &rdval);
    if (rdval != (V4L2_IDENT_SENSOR & 0xff))
    	return -ENODEV;
```

![image-20221122175111168](${media}/image-20221122175111168.png)

<center>图3-9: sensordetect</center>

##### 3.2.2.6 SENSOR 相关的IOCTL

```
static long sensor_ioctl(struct v4l2_subdev *sd, unsigned int cmd, void *arg)
```

用于应用层获取曝光增益，及进行与sensor 相关模块的驱动控制，如对焦，闪光等

```
case VIDIOC_VIN_SENSOR_EXP_GAIN:/*设置sensor的曝光增益*/
    ret = sensor_s_exp_gain(sd, (struct sensor_exp_gain *)arg);
    break;
case VIDIOC_VIN_SENSOR_CFG_REQ:/*获取sensor驱动的基础配置信息*/
    sensor_cfg_req(sd, (struct sensor_config *)arg);
    break;
case VIDIOC_VIN_ACT_SET_CODE:/*设置对焦马达配置参数，在配置AF模块时，需要此ioctl*/
	actuator_set_code(sd, (struct actuator_ctrl *)arg);
```

##### 3.2.2.7 与CSI 的接口

```
static struct sensor_format_struct sensor_formats[] = {};
RAW sensor:
.desc = "Raw RGB Bayer",
.mbus_code = MEDIA_BUS_FMT_SGRBG10_1X10,
.regs = sensor_fmt_raw,
.regs_size = ARRAY_SIZE(sensor_fmt_raw),
.bpp = 1
YUV sensor:
.desc = "YUYV 4:2:2",
.mbus_code = MEDIA_BUS_FMT_YUYV8_2X8,
.regs = sensor_fmt_yuyv422_yuyv,
.regs_size = ARRAY_SIZE(sensor_fmt_yuyv422_yuyv),
.bpp = 2
```

其中，mbus_code 中BGGR 可以根据sensor raw data 输出顺序修改为GBRG/RGGB/-GRBG。若填错, 会导致色彩偏紫红和出现网格状纹理。10_1X10 表示10 bit 

并口输出, 若是12 bit MIPI 输出, 则改为12_12X1。其他情况类推。对于DVP YUV sensor, 需根据yuv 输出顺序选择yuyv/vyuy/uyvy/yvyu 其中一种。

```
static int sensor_g_mbus_config(struct v4l2_subdev *sd,struct v4l2_mbus_config *cfg)
DVP sensor:
    cfg->type = V4L2_MBUS_PARALLEL;
    cfg->flags = V4L2_MBUS_MASTER | VREF_POL | HREF_POL | CLK_POL;
MIPI sensor:
    cfg->type = V4L2_MBUS_CSI2;
    cfg->flags = 0 | V4L2_MBUS_CSI2_1_LANE | V4L2_MBUS_CSI2_CHANNEL_0;
```

其中，MIPI sensor 须根据实际使用的lane 数，修改V4L2_MBUS_CSI2_X_LANE 中的X值。如果使用LVDS 接口，需要将cfg->type 配置为V4L2_MBUS_SUBLVDS。

##### 3.2.2.8 分辨率配置

```
static struct sensor_win_size sensor_win_sizes[] = {
{
    .width = VGA_WIDTH,
    .height = VGA_HEIGHT,
    .hoffset = 0,
    .voffset = 0,
    .hts = 928,
    .vts = 1720,
    .pclk = 48 * 1000 * 1000,
    .mipi_bps = 480 * 1000 * 1000,
    .fps_fixed = 30,
    .bin_factor = 1,
    .intg_min = 1 << 4,
    .intg_max = (1720) << 4,
    .gain_min = 1 << 4,
    .gain_max = 16 << 4,
    .regs = sensor_VGA_regs,
    .regs_size = ARRAY_SIZE(sensor_VGA_regs),
    .set_size = NULL,
},
{
    /* 定义图像输出的大小*/
    .width = VGA_WIDTH,
    .height = VGA_HEIGHT,
    /* 定义输入ISP 的偏移量，用于截取所需的Size，丢弃不需要的部分图像*/
    .hoffset = 0,
    .voffset = 0,
    /*
    定义行长(以pclk 为单位)、帧长(以hts 为单位) 和像素时钟频率。hts 又称line_length_pck,vts 又称frame_length_lines，与寄存器的值要一致。pclk(pixel clock)的值由PLL 寄存器计算得出。
    */
    .hts = 928,
    .vts = 1720,
    .pclk = 48 * 1000 * 1000,
    /* 定义MIPI 数据速率,MIPI sensor 必需，其他sensor 忽略*/
    /* mipi_bps = hts * vts * fps * raw bit / lane num */
    .mipi_bps = 480 * 1000 * 1000,
    /* 定义帧率，fps * hts * vts = pclk */
    .fps_fixed = 30,
    /*
    定义曝光行数最小值和最大值,增益最小值和最大值,以16 为1 倍。最值的设置应在sensor 规格和
    曝光函数限定的范围内,若超出会导致画面异常。此外,若AE table 中的最值超出这里的限制,会使得
    AE table 失效。
    */
    .intg_min = 1 << 4,
    .intg_max = (1720) << 4,
    .gain_min = 1 << 4,
    .gain_max = 16 << 4,
    /* (必需)说明这部分的配置对应哪个寄存器初始化代码*/
    .regs = sensor_VGA_regs,
    .regs_size = ARRAY_SIZE(sensor_VGA_regs),
    },
};
```

根据应用的需求，在这里配置驱动能输出的不同尺寸帧率组合，注意，一种分辨率、帧率配置为一个数组成员，不要混淆。

### 3.2.3 LVDS 接口须知

除了完成以上函数的实现，LVDS Sensor 驱动还需要完成combo 同步校验函数和combo 数据线映射函数。combo 校验码可以在sensor 规格书获取，combo 数

据线映射关系需要查看原理图设计进行配对，可参考imx274_slvds.c 完成开发。

![image-20221122180028623](${media}/image-20221122180028623.png)

<center>图3-10: SYNC_CODE</center>

```
static void sensor_g_combo_sync_code(struct v4l2_subdev *sd,
struct combo_sync_code *sync)
{
    int i;
    for (i = 0; i < 12; i++) {
    sync->lane_sof[i].low_bit = 0x0000ab00;
    sync->lane_sof[i].high_bit = 0xFFFF0000;
    sync->lane_sol[i].low_bit = 0x00008000;
    sync->lane_sol[i].high_bit = 0xFFFF0000;
    sync->lane_eol[i].low_bit = 0x00009d00;
    sync->lane_eol[i].high_bit = 0xFFFF0000;
    sync->lane_eof[i].low_bit = 0x0000b600;
    sync->lane_eof[i].high_bit = 0xFFFF0000;
}
}
static void sensor_g_combo_lane_map(struct v4l2_subdev *sd,
	struct combo_lane_map *map)
	{
	struct sensor_info *info = to_state(sd);
    if (info->isp_wdr_mode == ISP_DOL_WDR_MODE) {
        map->lvds_lane0 = LVDS_MAPPING_A_D0_TO_LANE0;
        map->lvds_lane1 = LVDS_MAPPING_A_D1_TO_LANE1;
        map->lvds_lane2 = LVDS_MAPPING_B_D2_TO_LANE2;
        map->lvds_lane3 = LVDS_MAPPING_B_D0_TO_LANE3;
        map->lvds_lane4 = LVDS_MAPPING_B_D3_TO_LANE4;
        map->lvds_lane5 = LVDS_MAPPING_C_D2_TO_LANE5;
        map->lvds_lane6 = LVDS_LANE6_NO_USE;
        map->lvds_lane7 = LVDS_LANE7_NO_USE;
        map->lvds_lane8 = LVDS_LANE8_NO_USE;
        map->lvds_lane9 = LVDS_LANE9_NO_USE;
        map->lvds_lane10 = LVDS_LANE10_NO_USE;
        map->lvds_lane11 = LVDS_LANE11_NO_USE;
} else {
        map->lvds_lane0 = LVDS_MAPPING_A_D1_TO_LANE0;
        map->lvds_lane1 = LVDS_MAPPING_B_D2_TO_LANE1;
        map->lvds_lane2 = LVDS_MAPPING_B_D0_TO_LANE2;
        map->lvds_lane3 = LVDS_MAPPING_B_D3_TO_LANE3;
        map->lvds_lane4 = LVDS_LANE4_NO_USE;
        map->lvds_lane5 = LVDS_LANE5_NO_USE;
        map->lvds_lane6 = LVDS_LANE6_NO_USE;
        map->lvds_lane7 = LVDS_LANE7_NO_USE;
        map->lvds_lane8 = LVDS_LANE8_NO_USE;
        map->lvds_lane9 = LVDS_LANE9_NO_USE;
        map->lvds_lane10 = LVDS_LANE10_NO_USE;
        map->lvds_lane11 = LVDS_LANE11_NO_USE;
	}
}
```

### 3.2.4 内核代码注意事项

驱动中一般禁止使用mdelay 或者msleep 实现延时，例如使用msleep 实现10~20ms的延时，通常会因为系统调度而变成延时更长的时间，这种做法精度较差。

所以如果需要使用ms 级别延时，则使用usleep_range(a, b)，比如原来mdelay(1)、mdelay(10) 可改为usleep_range(1000, 2000)、usleep_range(10000, 

12000)。如果是长达30ms 或以上的延时可选择使用msleep()；

中断过程中不能使用msleep 和usleep_range，除了特殊情况必须加延时之外，mdelay 一般也不可使用。