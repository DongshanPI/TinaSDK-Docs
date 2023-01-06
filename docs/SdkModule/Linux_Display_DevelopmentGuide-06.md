## 7 IOCTL 接口描述

sunxi 平台下显示驱动给用户提供了众多功能接口，可对图层、LCD、hdmi 等显示资源进行操作。

### 7.1 Global Interface

#### 7.1.1 DISP_SHADOW_PROTECT

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| hdle | 显示驱动句柄                                                 |
| cmd  | DISP_SHADOW_PROTECT                                          |
| arg  | arg[0] 为显示通道0/1；arg[1] 为protect 参数，1 表示protect, 0: 表示not protect |

• 返回值

如果成功，返回DIS_SUCCESS，否则，返回失败号。

• 描述

DISP_SHADOW_PROTECT（1）与DISP_SHADOW_PROTECT（0）配对使用，在protect期间，所有的请求当成一个命令序列缓冲起来, 等到调用

DISP_SHADOW_PROTECT（0）后将一起执行。

• 示例

```
//启动cache，disphd为显示驱动句柄
unsigned int arg[3];
arg[0] = 0;//disp0
arg[1] = 1;//protect
ioctl(disphd, DISP_SHADOW_PROTECT, (void*)arg);
//do somthing other
arg[1] = 0;//unprotect
ioctl(disphd, DISP_SHADOW_PROTECT, (void*)arg);
```

#### 7.1.2 DISP_SET_BKCOLOR

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| hdle | 显示驱动句柄                                                 |
| cmd  | DISP_SET_BKCOLOR                                             |
| arg  | arg[0] 为显示通道0/1；arg[1] 为backcolor 信息，指向disp_color 数据结构指针 |

• 返回值

如果成功，返回DIS_SUCCESS，否则，返回失败号。

• 描述

该函数用于设置显示背景色。

• 示例

```
//设置显示背景色，disphd为显示驱动句柄，sel为屏0/1
disp_color bk;
unsigned int arg[3];
bk.red = 0xff;
bk.green = 0x00;
bk.blue = 0x00;
arg[0] = 0;
arg[1] = (unsigned int)&bk;
ioctl(disphd, DISP_SET_BKCOLOR, (void*)arg);
```

#### 7.1.3 DISP_GET_BKCOLOR

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | **说明**                                                     |
| ---- | ------------------------------------------------------------ |
| hdle | 显示驱动句柄                                                 |
| cmd  | DISP_GET_BKCOLOR                                             |
| arg  | arg[0] 为显示通道0/1；arg[1] 为backcolor 信息，指向disp_color 数据结构指针 |

• 返回值

如果成功，返回DIS_SUCCESS，否则，返回失败号。

• 描述

该函数用于获取显示背景色。

• 示例

```
//获取显示背景色，disphd为显示驱动句柄，sel为屏0/1
disp_color bk;
unsigned int arg[3];
arg[0] = 0;
arg[1] = (unsigned int)&bk;
ioctl(disphd, DISP_GET_BKCOLOR, (void*)arg);
```

#### 7.1.4 DISP_GET_SCN_WIDTH

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明               |
| ---- | ------------------ |
| hdle | 显示驱动句柄       |
| cmd  | DISP_GET_SCN_WIDTH |
| arg  | arg[0] 显示通道0/1 |

• 返回值

如果成功，返回当前屏幕水平分辨率，否则，返回失败号。

• 描述

该函数用于获取当前屏幕水平分辨率。

• 示例

```
//获取屏幕水平分辨率
unsigned int screen_width;
unsigned int arg[3];
arg[0] = 0;
screen_width = ioctl(disphd, DISP_GET_SCN_WIDTH, (void*)arg);
```

#### 7.1.5 DISP_GET_SCN_HEIGHT

• 原型

int ioctl(int handle, unsigned int cmd, unsigned int *arg);

• 参数

| 参数 | 说明                |
| ---- | ------------------- |
| hdle | 显示驱动句柄        |
| cmd  | DISP_GET_SCN_HEIGHT |
| arg  | arg[0] 显示通道0/1  |

• 返回值

如果成功，返回当前屏幕垂直分辨率，否则，返回失败号。

• 描述

该函数用于获取当前屏幕垂直分辨率。

• 示例

```
//获取屏幕垂直分辨率
unsigned int screen_height;
unsigned int arg[3];
arg[0] = 0;
screen_height = ioctl(disphd, DISP_GET_SCN_HEIGHT, (void*)arg);
```

#### 7.1.6 DISP_GET_OUTPUT_TYPE

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                 |
| ---- | -------------------- |
| hdle | 显示驱动句柄         |
| cmd  | DISP_GET_OUTPUT_TYPE |
| arg  | arg[0] 显示通道0/1   |

• 返回值

如果成功，返回当前显示输出类型，否则，返回失败号。

• 描述

该函数用于获取当前显示输出类型(LCD,TV,HDMI,VGA,NONE)。

• 示例

```
//获取当前显示输出类型
disp_output_type output_type;
unsigned int arg[3];
arg[0] = 0;
output_type = (disp_output_type)ioctl(disphd, DISP_GET_OUTPUT_TYPE, (void*)arg);
```

#### 7.1.7 DISP_GET_OUTPUT

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| hdle | 显示驱动句柄                                                 |
| cmd  | DISP_GET_OUTPUT                                              |
| arg  | arg[0] 为显示通道0/1；arg[1] 为指向disp_output 结构体的指针，用于保存返回值 |

• 返回值

如果成功，返回0，否则，返回失败号。

• 描述

该函数用于获取当前显示输出类型及模式(LCD,TV,HDMI,VGA,NONE)。

• 示例

```
//获取当前显示输出类型
unsigned int arg[3];
disp_output output;
disp_output_type type;
disp_tv_mode mode;
arg[0] = 0;
arg[1] = (unsigned long)&output;
ioctl(disphd, DISP_GET_OUTPUT, (void*)arg);
type = (disp_output_type)output.type;
mode = (disp_tv_mode)output.mode;
```

#### 7.1.8 DISP_VSYNC_EVENT_EN

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| hdle | 显示驱动句柄                                                 |
| cmd  | DISP_VSYNC_EVENT_EN                                          |
| arg  | arg[0] 为显示通道0/1；arg[1] 为enable 参数，0：disable, 1:enable |

• 返回值

如果成功，返回DIS_SUCCESS。

否则，返回失败号。

• 描述

该函数开启/关闭vsync 消息发送功能。

• 示例

```
//开启/关闭vsync消息发送功能，disphd为显示驱动句柄，sel为屏0/1
unsigned int arg[3];
arg[0] = 0;
arg[1] = 1;
ioctl(disphd, DISP_VSYNC_EVENT_EN, (void*)arg);
```

#### 7.1.9 DISP_DEVICE_SWITCH

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| hdle | 显示驱动句柄                                                 |
| cmd  | DISP_DEVICE_SWITCH                                           |
| arg  | arg[0] 为显示通道0/1；arg[1] 为输出类型；arg[2] 为输出模式，在输出类型不为LCD 时有效 |

• 返回值

如果成功，返回DIS_SUCCESS，否则，返回失败号。

• 描述

该函数用于切换输出类型。

• 示例

```
//切换
unsigned int arg[3];
arg[0] = 0;
arg[1] = （unsigned long)DISP_OUTPUT_TYPE_HDMI;
arg[2] = (unsigned long)DISP_TV_MOD_1080P_60HZ;
ioctl(disphd, DISP_DEVICE_SWITCH, (void*)arg);
```

说明：如果传递的type 是DISP_OUTPUT_TYPE_NONE，将会关闭当前显示通道的输出。

#### 7.1.10 DISP_DEVICE_SET_CONFIG

• 原型

```
int ioctl(int handle, unsigned int cmd,unsigned int *arg);
```

• 参数

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| hdle | 显示驱动句柄                                                 |
| cmd  | DISP_DEVICE_SET_CONFIG                                       |
| arg  | arg[0] 为显示通道0/1；arg[1] 为指向disp_device_config 的指针 |

• 返回值

如果成功，返回DIS_SUCCESS，否则，返回失败号。

• 描述

该函数用于切换输出类型并设置输出设备的属性参数。

• 示例

```
//切换输出类型并设置输出设备的属性参数
unsigned long arg[3];
struct disp_device_config config;
config.type = DISP_OUTPUT_TYPE_LCD;
config.mode = 0;
config.format = DISP_CSC_TYPE_RGB;
config.bits = DISP_DATA_8BITS;
config.eotf = DISP_EOTF_GAMMA22;
config.cs = DISP_BT709;
arg[0] = 0;
arg[1] = （unsigned long)&config;
ioctl(dispfd, DISP_DEVICE_SET_CONFIG, (void*)arg);
//说明：如果传递的type是DISP_OUTPUT_TYPE_NONE，将会关闭当前显示通道的输出。
```

#### 7.1.11 DISP_DEVICE_GET_CONFIG

• 原型

```
int ioctl(int handle, unsigned int cmd,unsigned int *arg);
```

• 参数

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| hdle | 显示驱动句柄                                                 |
| cmd  | DISP_DEVICE_GET_CONFIG                                       |
| arg  | arg[0] 为显示通道0/1；arg[1] 为指向disp_device_config 的指针 |

• 返回值

如果成功，返回DIS_SUCCESS，否则，返回失败号。

• 描述

该函数用于获取当前输出类型及相关的属性参数。

• 示例

```
//获取当前输出类型及相关的属性参数
unsigned long arg[3];
struct disp_device_config config;
arg[0] = 0;
arg[1] = （unsigned long)&config;
ioctl(dispfd, DISP_DEVICE_GET_CONFIG, (void*)arg);
//说明：如果返回的type是DISP_OUTPUT_TYPE_NONE，表示当前输出显示通道为关闭状态
```

### 7.2 Layer Interface

#### 7.2.1 DISP_LAYER_SET_CONFIG

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| hdle | 显示驱动句柄                                                 |
| cmd  | DISP_CMD_SET_LAYER_CONFIG                                    |
| arg  | arg[0] 为显示通道0/1；arg[1] 为图层配置参数指针；arg[2] 为需要配置的图层数目 |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于设置多个图层信息。

• 示例

```
struct
{
	disp_layer_info info,
bool enable;
	unsigned int channel,
	unsigned int layer_id,
} disp_layer_config;
//设置图层参数，disphd为显示驱动句柄
unsigned int arg[3];
disp_layer_config config;
unsigned int width = 1280;
unsigned int height = 800;
unsigned int ret = 0;
memset(&info, 0, sizeof(disp_layer_info));
config.channel = 0; //channel 0
config.layer_id = 0;//layer 0 at channel 0
config.info.enable = 1;
config.info.mode = LAYER_MODE_BUFFER;
config.info.fb.addr[0] = (__u32)mem_in; //FB地址
config.info.fb.size.width = width;
config.info.fb.format = DISP_FORMAT_ARGB_8888; //DISP_FORMAT_YUV420_P
config.info.fb.crop.x = 0;
config.info.fb.crop.y = 0;
config.info.fb.crop.width = ((unsigned long)width) << 32;//定点小数。高32bit为整数，低32bit为小
数
config.info.fb.crop.height= ((uunsigned long)height)<<32;//定点小数。高32bit为整数，低32bit为小
数
config.info.fb.flags = DISP_BF_NORMAL;
config.info.fb.scan = DISP_SCAN_PROGRESSIVE;
config.info.alpha_mode = 1; //global alpha
config.info.alpha_value = 0xff;
config.info.screen_win.x = 0;
config.info.screen_win.y = 0;
config.info.screen_win.width = width;
config.info.screen_win.height= height;
config.info.id = 0;
arg[0] = 0;//screen 0
arg[1] = (unsigned int)&config;
arg[2 = 1; //one layer
ret = ioctl(disphd, DISP_CMD_LAYER_SET_CONFIG, (void*)arg);
```

#### 7.2.2 DISP_LAYER_GET_CONFIG

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| hdle | 显示驱动句柄                                                 |
| cmd  | DISP_LAYER_GET_CONFIG                                        |
| arg  | arg[0] 为显示通道0/1；arg[1] 为图层配置参数指针；arg[2] 为需要获取配置的图层数目 |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于获取图层参数。

• 示例

```
//获取图层参数，disphd为显示驱动句柄
unsigned int arg[3];
disp_layer_info info;
memset(&info, 0, sizeof(disp_layer_info));
config.channel = 0; //channel 0
config.layer_id = 0;//layer 0 at channel 0
arg[0] = 0;//显示通道0
arg[1] = 0;//图层0
arg[2 = (unsigned int)&info;
ret = ioctl(disphd, DISP_LAYER_GET_CONFIG, (void*)arg);
```

#### 7.2.3 DISP_LAYER_SET_CONFIG2

• 原型

```
int ioctl(int handle, unsigned int cmd,unsigned int *arg);
```

• 参数

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| hdle | 显示驱动句柄                                                 |
| cmd  | DISP_SET_LAYER_CONFIG2                                       |
| arg  | arg[0] 为显示通道0/1；arg[1] 为图层配置参数指针；arg[2] 为需要配置的图层数目 |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于设置多个图层信息，注意该接口只接受disp_layer_config2 的信息。

• 示例

```
struct
{
disp_layer_info info,
bool enable;
unsigned int channel,
unsigned int layer_id,
}disp_layer_config2;
//设置图层参数，dispfd 为显示驱动句柄
unsigned long arg[3];
struct disp_layer_config2 config;
unsigned int width = 1280;
unsigned int height = 800;
unsigned int ret = 0;
memset(&config, 0, sizeof(struct disp_layer_config2));
config.channnel = 0;//blending channel
config.layer_id = 0;//layer index in the blending channel
config.info.enable = 1;
config.info.mode = LAYER_MODE_BUFFER;
config.info.fb.addr[0] = (unsigned long long)mem_in; //FB 地址
config.info.fb.size[0].width = width;
config.info.fb.align[0] = 4;//bytes
config.info.fb.format = DISP_FORMAT_ARGB_8888; //DISP_FORMAT_YUV420_P
config.info.fb.crop.x = 0;
config.info.fb.crop.y = 0;
config.info.fb.crop.width = ((unsigned long)width) << 32;//定点小数。高32bit 为整数，低32bit 为小数
config.info.fb.crop.height= ((uunsigned long)height)<<32;//定点小数。高32bit 为整数，低32bit 为小数
config.info.fb.flags = DISP_BF_NORMAL;
config.info.fb.scan = DISP_SCAN_PROGRESSIVE;
config.info.fb.eotf = DISP_EOTF_SMPTE2084; //HDR
config.info.fb.metadata_buf = (unsigned long long)mem_in2;
config.info.alpha_mode = 2; //global pixel alpha
config.info.alpha_value = 0xff;//global alpha value
config.info.screen_win.x = 0;
config.info.screen_win.y = 0;
config.info.screen_win.width = width;
config.info.screen_win.height= height;
config.info.id = 0;
arg[0] = 0;//screen 0
arg[1] = (unsigned long)&config;
arg[2 = 1; //one layer
ret = ioctl(dispfd, DISP_LAYER_SET_CONFIG2, (void*)arg);
```

7.2.4 DISP_LAYER_GET_CONFIG2

• 原型

```
int ioctl(int handle, unsigned int cmd,unsigned int *arg);
```

• 参数

| 参数 | 说明                 |
| ---- | -------------------- |
| hdle | 显示驱动句柄         |
| cmd  | DISP_CAPTURE_START   |
| arg  | arg[0] 为显示通道0/1 |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于开启截屏功能。

• 示例

```
//启动截屏功能，dispfd 为显示驱动句柄
arg[0] = 0;//显示通道0
ioctl(dispfd, DISP_CAPTURE_START, (void*)arg);
```

#### 7.3.2 DISP_CAPTURE_COMMIT

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| hdle | 显示驱动句柄                                                 |
| cmd  | DISP_CAPTURE_COMMIT                                          |
| arg  | arg[0] 为显示通道0/1；arg[1] 为指向截屏的信息结构体，详见disp_capture_info |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于提交截屏的任务，提交一次，则会启动一次截屏操作。

• 示例

```
//提交截屏功能，dispfd 为显示驱动句柄
unsigned long arg[3];
struct disp_capture_info info;
arg[0] = 0;
screen_width = ioctl(dispfd, DISP_GET_SCN_WIDTH, (void*)arg);
screen_height = ioctl(dispfd, DISP_GET_SCN_HEIGHT, (void*)arg);
info.window.x = 0;
info.window.y = 0;
info.window.width = screen_width;
info.window.y = screen_height;
info.out_frame.format = DISP_FORMAT_ARGB_8888;
info.out_frame.size[0].width = screen_width;
info.out_frame.size[0].height = screen_height;
info.out_frame.crop.x = 0;
info.out_frame.crop.y = 0;
info.out_frame.crop.width = screen_width;
info.out_frame.crop.height = screen_height;
info.out_frame.addr[0] = fb_address; //buffer address
arg[0] = 0;//显示通道0
arg[1] = (unsigned long)&info;
ioctl(dispfd, DISP_CAPTURE_COMMIT, (void*)arg);
```

#### 7.3.3 DISP_CAPTURE_STOP

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                 |
| ---- | -------------------- |
| hdle | 显示驱动句柄         |
| cmd  | DISP_CAPTURE_STOP    |
| arg  | arg[0] 为显示通道0/1 |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于关闭截屏功能。

• 示例

```
//停止截屏功能，dispfd 为显示驱动句柄
unsigned long arg[3];
arg[0] = 0;//显示通道0
ioctl(dispfd, DISP_CAPTURE_STOP, (void*)arg);
```

#### 7.3.4 DISP_CAPTURE_QUERY

• 原型

```
int ioctl(int handle, unsigned int cmd,unsigned int *arg);
```

• 参数

命令DISP_CAPTURE_QUERY 是查询功能。

| 参数 | 说明                 |
| ---- | -------------------- |
| hdle | 显示驱动句柄         |
| cmd  | DISP_CAPTURE_QUERY   |
| arg  | arg[0] 为显示通道0/1 |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数查询刚结束的图像帧是否截屏成功。

• 示例

```
//查询截屏是否成功，dispfd 为显示驱动句柄
unsigned long arg[3];
arg[0] = 0;//显示通道0
ioctl(dispfd, DISP_CAPTURE_QUERY, (void*)arg);
```

### 7.4 LCD Interface

#### 7.4.1 DISP_LCD_SET_BRIGHTNESS

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                                                 |
| ---- | ---------------------------------------------------- |
| hdle | 显示驱动句柄                                         |
| cmd  | DISP_LCD_SET_BRIGHTNESS                              |
| arg  | arg[0] 为显示通道0/1；arg[1] 为背光亮度值，（0~255） |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于设置LCD 亮度。

• 示例

```
//设置LCD的背光亮度，disphd为显示驱动句柄
unsigned int arg[3];
unsigned int bl = 197;
arg[0] = 0;//显示通道0
arg[1] = bl;
ioctl(disphd, DISP_LCD_SET_BRIGHTNESS, (void*)arg);
```

#### 7.4.2 DISP_LCD_GET_BRIGHTNESS

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                    |
| ---- | ----------------------- |
| hdle | 显示驱动句柄            |
| cmd  | DISP_LCD_GET_BRIGHTNESS |
| arg  | arg[0] 为显示通道0/1    |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于获取LCD 亮度。

• 示例

```
//获取LCD的背光亮度，disphd为显示驱动句柄
unsigned int arg[3];
unsigned int bl;
arg[0] = 0;//显示通道0
bl = ioctl(disphd, DISP_LCD_GET_BRIGHTNESS, (void*)arg);
```

### 7.5 Enhance interface

#### 7.5.1 DISP_ENHANCE_ENABLE

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                 |
| ---- | -------------------- |
| hdle | 显示驱动句柄         |
| cmd  | DISP_ENHANCE_ENABLE  |
| arg  | arg[0] 为显示通道0/1 |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于使能图像后处理功能。

• 示例

```
//开启图像后处理功能，disphd为显示驱动句柄
unsigned int arg[3];
arg[0] = 0;//显示通道0
ioctl(disphd, DISP_ENHANCE_ENABLE, (void*)arg);
```

#### 7.5.2 DISP_ENHANCE_DISABLE

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                 |
| ---- | -------------------- |
| hdle | 显示驱动句柄         |
| cmd  | DISP_ENHANCE_DISABLE |
| arg  | arg[0] 为显示通道0/1 |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于关闭图像后处理功能。

• 示例

```
//关闭图像后处理功能，disphd为显示驱动句柄
unsigned int arg[3];
arg[0] = 0;//显示通道0
ioctl(disphd, DISP_ENHANCE_DISABLE, (void*)arg);
```

#### 7.5.3 DISP_ENHANCE_DEMO_ENABLE

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                     |
| ---- | ------------------------ |
| hdle | 显示驱动句柄             |
| cmd  | DISP_ENHANCE_DEMO_ENABLE |
| arg  | arg[0] 为显示通道0/1     |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于开启图像后处理演示模式，开启后，在屏幕会出现左边进行后处理，右边未处理的图像画面，方便对比效果。演示模式需要在后处理功能开启之后才有

效。

• 示例

```
//开启图像后处理演示模式，disphd为显示驱动句柄
unsigned int arg[3];
arg[0] = 0;//显示通道0
ioctl(disphd, DISP_ENHANCE_DEMO_ENABLE, (void*)arg);
```

#### 7.5.4 DISP_ENHANCE_DEMO_DISABLE

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```


• 参数

| 参数 | 说明                      |
| ---- | ------------------------- |
| hdle | 显示驱动句柄              |
| cmd  | DISP_ENHANCE_DEMO_DISABLE |
| arg  | arg[0] 为显示通道0/1      |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于关闭图像后处理演示模式。

• 示例

```
//开启图像后处理演示模式，disphd为显示驱动句柄
unsigned int arg[3];
arg[0] = 0;//显示通道0
ioctl(disphd, DISP_ENHANCE_DEMO_ENABLE, (void*)arg);
```

### 7.6 Smart backlight

#### 7.6.1 DISP_SMBL_ENABLE

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                 |
| ---- | -------------------- |
| hdle | 显示驱动句柄         |
| cmd  | DISP_SMBL_ENABLE     |
| arg  | arg[0] 为显示通道0/1 |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于使能智能背光功能。

• 示例

```
//开启智能背光功能，disphd为显示驱动句柄
unsigned int arg[3];
arg[0] = 0;//显示通道0
ioctl(disphd, DISP_SMBL_ENABLE, (void*)arg);
```

#### 7.6.2 DISP_SMBL_DISABLE

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数



| 参数 | 说明                 |
| ---- | -------------------- |
| hdle | 显示驱动句柄         |
| cmd  | DISP_SMBL_ENABLE     |
| arg  | arg[0] 为显示通道0/1 |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于关闭智能背光功能。

• 示例

```
//关闭智能背光功能，disphd为显示驱动句柄
unsigned int arg[3];
arg[0] = 0;//显示通道0
ioctl(disphd, DISP_SMBL_DISABLE, (void*)arg);
```

#### 7.6.3 DISP_SMBL_SET_WINDOW

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                                                       |
| ---- | ---------------------------------------------------------- |
| hdle | 显示驱动句柄                                               |
| cmd  | DISP_SMBL_SET_WINDOW                                       |
| arg  | arg[0] 为显示通道0/1；arg[1] 为指向struct disp_rect 的指针 |

• 返回值

如果成功，则返回DIS_SUCCESS；如果失败，则返回失败号。

• 描述

该函数用于设置智能背光开启效果的窗口，智能背光在设置的窗口中有效。

• 示例

```
//设置智能背光窗口，disphd为显示驱动句柄
unsigned int arg[3];
unsigned int screen_width, screen_height;
struct disp_rect window;
　　
screen_width = ioctl(disphd, DISP_GET_SCN_WIDTH, (void*)arg);
screen_height = ioctl(disphd, DISP_GET_SCN_HEIGHT, (void*)arg);
window.x = 0;
window.y = 0;
window.width = screen_width / 2;
widnow.height = screen_height;
arg[0] = 0;//显示通道0
arg[1] = (unsigned long)&window;
ioctl(disphd, DISP_SMBL_SET_WINDOW, (void*)arg);
```

### 7.7 Hdmi interface

#### 7.7.1 DISP_HDMI_SUPPORT_MODE

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| hdle | 显示驱动句柄                                                 |
| cmd  | DISP_HDMI_SUPPORT_MODE                                       |
| arg  | arg[0] 为显示通道0/1；arg[1] 为需要查询的模式，详见disp_tv_mode |

• 返回值

如果支持，则返回1；如果失败，则返回0。

• 描述

该函数用于查询指定的HDMI 模式是否支持。

• 示例

```
//查询指定的HDMI模式是否支持
unsigned int arg[3];
arg[0] = 0;//显示通道0
arg[1] = (unsigned long)DISP_TV_MOD_1080P_60HZ;
ioctl(disphd, DISP_HDMI_SUPPORT_MODE, (void*)arg);
```

#### 7.7.2 DISP_HDMI_GET_HPD_STATUS

• 原型

```
int ioctl(int handle, unsigned int cmd, unsigned int *arg);
```

• 参数

| 参数 | 说明                     |
| ---- | ------------------------ |
| hdle | 显示驱动句柄             |
| cmd  | DISP_HDMI_GET_HPD_STATUS |
| arg  | arg[0] 为显示通道0/1     |

• 返回值

如果HDMI 插入，则返回1；如果未插入，则返回0。

• 描述

该函数用于指定HDMI 是否处于插入状态。

• 示例

```
//查询HDMI是否处于插入状态
unsigned int arg[3];
arg[0] = 0;//显示通道0
if (ioctl(disphd, DISP_HDMI_GET_HPD_STATUS, (void*)arg) == 1)
printf("hdmi plug in\n");
else
printf("hdmi plug out\n");
```

