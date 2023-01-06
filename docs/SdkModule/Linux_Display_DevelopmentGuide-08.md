## 9 Data Structure

### 9.1 disp_fb_info

• 原型

```
typedef struct
{
    unsigned long long addr[3]; /* address of frame buffer,
    single addr for interleaved fomart,
    double addr for semi-planar fomart
    triple addr for planar format */
    disp_rectsz size[3]; //size for 3 component,unit: pixels
    unsigned int align[3]; //align for 3 comonent,unit: bytes(align=2^n,i.e
    .1/2/4/8/16/32..
    disp_pixel_format format;
    disp_color_space color_space; //color space
    unsigned int trd_right_addr[3];/* right address of 3d fb,
    used when in frame packing 3d mode */
    bool pre_multiply; //true: pre-multiply fb
    disp_rect64 crop; //crop rectangle boundaries
    disp_buffer_flags flags; //indicate stereo or non-stereo buffer
    disp_scan_flags scan; //scan type & scan order
}disp_fb_info;
```

• 成员

| 变量           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| addr           | framebuffer 的内容地址，对于interleaved 类型，只有addr[0] 有效；planar 类型，三个都有；UV combined 的类型addr[0]，addr[1]有效 |
| size           | size of framebuffer, 单位为pixel                             |
| align          | 对齐位宽，为2 的指数                                         |
| format         | pixel format, 详见disp_pixel_format                          |
| color_space    | color space mode, 详见disp_cs_mode                           |
| b_trd_src      | 1:3D source; 0：2D source                                    |
| trd_mode       | source 3D mode, 详见disp_3d_src_mode                         |
| trd_right_addr | used when in frame packing 3d mode                           |
| crop           | 用于显示的buffer 裁减区                                      |
| flags          | 标识2D 或3D 的buffer                                         |
| scan           | 标识描述类型，progress, interleaved                          |

• 描述

disp_fb_info 用于描述一个display frambuffer 的属性信息。

### 9.2 disp_layer_info

• 原型

```
typedef struct
{
    disp_layer_mode mode;
    unsigned char zorder; /*specifies the front-to-back ordering of the layers on
    the screen,
    the top layer having the highest Z value
    can't set zorder, but can get */
    unsigned char alpha_mode; //0: pixel alpha; 1: global alpha; 2: global
    pixel alpha
    unsigned char alpha_value; //global alpha value
    disp_rect screen_win; //display window on the screen
    bool b_trd_out; //3d display
    disp_3d_out_mode out_trd_mode;//3d display mode
    union {
        unsigned int color; //valid when LAYER_MODE_COLOR
        disp_fb_info fb; //framebuffer, valid when LAYER_MODE_BUFFER
    };
    unsigned int id; /* frame id, can get the id of frame display currently
    by DISP_LAYER_GET_FRAME_ID */
}disp_layer_info;
```

• 成员

| 变量         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| mode         | 图层的模式，详见disp_layer_mode                              |
| zorder       | layer zorder, 优先级高的图层可能会覆盖优先级低的图层         |
| alpha_mode   | 0:pixel alpha, 1:global alpha, 2:global pixel alpha          |
| alpha_value  | layer global alpha value，valid while alpha_mode(1/2)        |
| screenn_win  | screen window，图层在屏幕上显示的矩形窗口                    |
| fb           | framebuffer 的属性，详见disp_fb_info,valid when BUFFER_MODE  |
| color        | display color, valid when COLOR_MODE                         |
| b_trd_out    | if output in 3d mode,used for scaler layer                   |
| out_trd_mode | output 3d mode, 详见disp_3d_out_mode                         |
| id           | frame id, 设置给驱动的图像帧号，可以通过DISP_LAYER_GET_FRAME_ID 获取当前显示的帧号，以做一下特定的处理，比如释放掉已经显示完成的图像帧buffer。 |

• 描述

disp_layer_info 用于描述一个图层的属性信息。

### 9.3 disp_layer_config

• 原型

```
typedef struct
{
    disp_layer_info info;
    bool enable;
    unsigned int channel;
    unsigned int layer_id;
}disp_layer_config;
```

• 成员

| 变量     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| info     | 图像的信息属性                                               |
| enable   | 使能标志                                                     |
| channel  | 图层所在的通道id（0/1/2/3）                                  |
| layer_id | 图层的id，此id 是在通道内的图层id。即(channel,layer_id)=(0,0) 表示通道0 中的图层0 之意 |

• 描述

disp_layer_config 用于描述一个图层配置的属性信息。

### 9.4 disp_color_info

• 原型

```
typedef struct
{
    u8 alpha;
    u8 red;
    u8 green;
    u8 blue;
}disp_color_info;
```

• 成员

| 变量说明 |              |
| -------- | ------------ |
| alpha    | 颜色的透明度 |
| red      | 红           |
| green    | 绿           |
| blue     | 蓝           |

• 描述

disp_color_info 用于描述一个颜色的信息。

### 9.5 disp_rect

• 原型

```
typedef struct
{
    s32 x;
    s32 y;
    u32 width;
    u32 height;
}disp_rect;
```

• 成员

| 变量   | 参数     |
| ------ | -------- |
| ｘ     | 起点x 值 |
| y      | 起点y 值 |
| width  | 宽       |
| height | 高       |

• 描述

disp_rect 用于描述一个矩形窗口的信息。

### 9.6 disp_rect64

• 原型

```
typedef struct
{
    long long x;
    long long y;
    long long width;
    long long height;
}disp_rect64;
```

• 成员

| 变量   | 说明                                               |
| ------ | -------------------------------------------------- |
| x      | 起点x 值, 定点小数，高32bit 为整数，低32bit 为小数 |
| y      | 起点y 值, 定点小数，高32bit 为整数，低32bit 为小数 |
| width  | 宽, 定点小数，高32bit 为整数，低32bit 为小数       |
| height | 高, 定点小数，高32bit 为整数，低32bit 为小数       |

• 描述

disp_rect64 用于描述一个矩形窗口的信息。

### 9.7 disp_position

• 原型

```
typedef struct
{
    s32 x;
    s32 y;
}disp_posistion;
```

• 成员

| 变量 | 说明 |
| ---- | ---- |
| x    | x    |
| y    | y    |

• 描述

disp_position 用于描述一个坐标的信息。

### 9.8 disp_rectsz

• 原型

```
typedef struct
{
    u32 width;
    u32 height;
}disp_rectsz;
```

• 成员

| 变量   | 说明 |
| ------ | ---- |
| width  | 宽   |
| height | 高   |

• 描述

disp_rectsz 用于描述一个矩形尺寸的信息。

### 9.9 disp_pixel_format	

• 原型

```
typedef enum
{
    DISP_FORMAT_ARGB_8888 = 0x00,//MSB A-R-G-B LSB
    DISP_FORMAT_ABGR_8888 = 0x01,
    DISP_FORMAT_RGBA_8888 = 0x02,
    DISP_FORMAT_BGRA_8888 = 0x03,
    DISP_FORMAT_XRGB_8888 = 0x04,
    DISP_FORMAT_XBGR_8888 = 0x05,
    DISP_FORMAT_RGBX_8888 = 0x06,
    DISP_FORMAT_BGRX_8888 = 0x07,
    DISP_FORMAT_RGB_888 = 0x08,
    DISP_FORMAT_BGR_888 = 0x09,
    DISP_FORMAT_RGB_565 = 0x0a,
    DISP_FORMAT_BGR_565 = 0x0b,
    DISP_FORMAT_ARGB_4444 = 0x0c,
    DISP_FORMAT_ABGR_4444 = 0x0d,
    DISP_FORMAT_RGBA_4444 = 0x0e,
    DISP_FORMAT_BGRA_4444 = 0x0f,
    DISP_FORMAT_ARGB_1555 = 0x10,
    DISP_FORMAT_ABGR_1555 = 0x11,
    DISP_FORMAT_RGBA_5551 = 0x12,
    DISP_FORMAT_BGRA_5551 = 0x13,
    /* SP: semi-planar, P:planar, I:interleaved
    * UVUV: U in the LSBs; VUVU: V in the LSBs */
    DISP_FORMAT_YUV444_I_AYUV = 0x40,//MSB A-Y-U-V LSB
    DISP_FORMAT_YUV444_I_VUYA = 0x41,//MSB V-U-Y-A LSB
    DISP_FORMAT_YUV422_I_YVYU = 0x42,//MSB Y-V-Y-U LSB
    DISP_FORMAT_YUV422_I_YUYV = 0x43,//MSB Y-U-Y-V LSB
    DISP_FORMAT_YUV422_I_UYVY = 0x44,//MSB U-Y-V-Y LSB
    DISP_FORMAT_YUV422_I_VYUY = 0x45,//MSB V-Y-U-Y LSB
    DISP_FORMAT_YUV444_P = 0x46,//MSB P3-2-1-0 LSB, YYYY UUUU VVVV
    DISP_FORMAT_YUV422_P = 0x47,//MSB P3-2-1-0 LSB YYYY UU VV
    DISP_FORMAT_YUV420_P = 0x48,//MSB P3-2-1-0 LSB YYYY U V
    DISP_FORMAT_YUV411_P = 0x49,//MSB P3-2-1-0 LSB YYYY U V
    DISP_FORMAT_YUV422_SP_UVUV = 0x4a,//MSB V-U-V-U LSB
    DISP_FORMAT_YUV422_SP_VUVU = 0x4b,//MSB U-V-U-V LSB
    DISP_FORMAT_YUV420_SP_UVUV = 0x4c,
    DISP_FORMAT_YUV420_SP_VUVU = 0x4d,
    DISP_FORMAT_YUV411_SP_UVUV = 0x4e,
    DISP_FORMAT_YUV411_SP_VUVU = 0x4f,
}disp_pixel_format;
```

• 成员

| 变量                       | 说明                                                   |
| -------------------------- | ------------------------------------------------------ |
| DISP_FORMAT_ARGB_8888      | 32bpp, A 在最高位，B 在最低位                          |
| DISP_FORMAT_YUV420_P       | planar yuv 格式，分三块存放，需三个地址，P3在最高位    |
| DISP_FORMAT_YUV422_SP_UVUV | semi-planar yuv 格式，分两块存放，需两个地址，U 在低位 |
| DISP_FORMAT_YUV422_SP_VUVU | semi-planar yuv 格式，分两块存放，需两个地址，V 在低位 |

• 描述

disp_pixel_format 用于描述像素格式。

### 9.10 disp_buffer_flags

• 原型

```
typedef enum
{
    DISP_BF_NORMAL = 0,//non-stereo
    DISP_BF_STEREO_TB = 1 << 0,//stereo top-bottom
    DISP_BF_STEREO_FP = 1 << 1,//stereo frame packing
    DISP_BF_STEREO_SSH = 1 << 2,//stereo side by side half
    DISP_BF_STEREO_SSF = 1 << 3,//stereo side by side full
    DISP_BF_STEREO_LI = 1 << 4,//stereo line interlace
}disp_buffer_flags;
```

• 成员

| 变量                         | 说明                         |
| ---------------------------- | ---------------------------- |
| DISP_BF_NORMAL               | 2d                           |
| DISP_BF_STEREO_TB top bottom | 模式                         |
| DISP_BF_STEREO_FP            | framepacking                 |
| DISP_BF_STEREO_SSF           | side by side full, 左右全景  |
| DISP_BF_STEREO_SSH           | side by side half, 左右半景  |
| DISP_BF_STEREO_LI            | line interleaved, 行交错模式 |

• 描述

disp_buffer_flags 用于描述3D 源模式。

### 9.11 disp_3d_out_mode

• 原型

```
typedef enum
{
    //for lcd
    DISP_3D_OUT_MODE_CI_1 = 0x5,//column interlaved 1
    DISP_3D_OUT_MODE_CI_2 = 0x6,//column interlaved 2
    DISP_3D_OUT_MODE_CI_3 = 0x7,//column interlaved 3
    DISP_3D_OUT_MODE_CI_4 = 0x8,//column interlaved 4
    DISP_3D_OUT_MODE_LIRGB = 0x9,//line interleaved rgb
    //for hdmi
    DISP_3D_OUT_MODE_TB = 0x0,//top bottom
    DISP_3D_OUT_MODE_FP = 0x1,//frame packing
    DISP_3D_OUT_MODE_SSF = 0x2,//side by side full
    DISP_3D_OUT_MODE_SSH = 0x3,//side by side half
    DISP_3D_OUT_MODE_LI = 0x4,//line interleaved
    DISP_3D_OUT_MODE_FA = 0xa,//field alternative
}disp_3d_out_mode;
```

• 成员

for lcd：

| 变量                   | 说明   |
| ---------------------- | ------ |
| DISP_3D_OUT_MODE_CI_1  | 列交织 |
| DISP_3D_OUT_MODE_CI_2  | 列交织 |
| DISP_3D_OUT_MODE_CI_3  | 列交织 |
| DISP_3D_OUT_MODE_CI_4  | 列交织 |
| DISP_3D_OUT_MODE_LIRGB | 行交织 |

for hdmi：

| 变量                           | 说明                        |
| ------------------------------ | --------------------------- |
| DISP_3D_OUT_MODE_TB top bottom | 上下模式                    |
| DISP_3D_OUT_MODE_FP            | framepacking                |
| DISP_3D_OUT_MODE_SSF           | side by side full, 左右全景 |
| DISP_3D_OUT_MODE_SSH           | side by side half, 左右半景 |
| DISP_3D_OUT_MODE_LI            | line interleaved, 行交织    |
| DISP_3D_OUT_MODE_FA            | field alternate 场交错      |

• 描述

disp_3d_out_mode 用于描述3D 输出模式。

### 9.12 disp_color_space

• 原型

```
typedef enum
{
    DISP_BT601 = 0,
    DISP_BT709 = 1,
    DISP_YCC = 2,
}disp_color_mode;
```

• 成员

| 变量       | 说明         |
| ---------- | ------------ |
| DISP_BT601 | 用于标清视频 |
| DISP_BT709 | 用于高清视频 |
| DISP_YCC   | 用于图片     |

• 描述

disp_color_space 用于描述颜色空间类型。

### 9.13 disp_output_type

• 原型

```
typedef enum
{
    DISP_OUTPUT_TYPE_NONE = 0,
    DISP_OUTPUT_TYPE_LCD = 1,
    DISP_OUTPUT_TYPE_TV = 2,
    DISP_OUTPUT_TYPE_HDMI = 4,
    DISP_OUTPUT_TYPE_VGA = 8,
}disp_output_type;
```

• 成员

| 变量                  | 说明       |
| --------------------- | ---------- |
| DISP_OUTPUT_TYPE_NONE | 无显示输出 |
| DISP_OUTPUT_TYPE_LCD  | LCD 输出   |
| DISP_OUTPUT_TYPE_TV   | TV 输出    |
| DISP_OUTPUT_TYPE_HDMI | HDMI 输出  |
| DISP_OUTPUT_TYPE_VGA  | VGA 输出   |

• 描述

disp_output_type 用于描述显示输出类型。

### 9.14 disp_tv_mode

• 原型

```
typedef enum
{
    DISP_TV_MOD_480I = 0,
    DISP_TV_MOD_576I = 1,
    DISP_TV_MOD_480P = 2,
    DISP_TV_MOD_576P = 3,
    DISP_TV_MOD_720P_50HZ = 4,
    DISP_TV_MOD_720P_60HZ = 5,
    DISP_TV_MOD_1080I_50HZ = 6,
    DISP_TV_MOD_1080I_60HZ = 7,
    DISP_TV_MOD_1080P_24HZ = 8,
    DISP_TV_MOD_1080P_50HZ = 9,
    DISP_TV_MOD_1080P_60HZ = 0xa,
    DISP_TV_MOD_1080P_24HZ_3D_FP = 0x17,
    DISP_TV_MOD_720P_50HZ_3D_FP = 0x18,
    DISP_TV_MOD_720P_60HZ_3D_FP = 0x19,
    DISP_TV_MOD_1080P_25HZ = 0x1a,
    DISP_TV_MOD_1080P_30HZ = 0x1b,
    DISP_TV_MOD_PAL = 0xb,
    DISP_TV_MOD_PAL_SVIDEO = 0xc,
    DISP_TV_MOD_NTSC = 0xe,
    DISP_TV_MOD_NTSC_SVIDEO = 0xf,
    DISP_TV_MOD_PAL_M = 0x11,
    DISP_TV_MOD_PAL_M_SVIDEO = 0x12,
    DISP_TV_MOD_PAL_NC = 0x14,
    DISP_TV_MOD_PAL_NC_SVIDEO = 0x15,
    DISP_TV_MOD_3840_2160P_30HZ = 0x1c,
    DISP_TV_MOD_3840_2160P_25HZ = 0x1d,
    DISP_TV_MOD_3840_2160P_24HZ = 0x1e,
    DISP_TV_MODE_NUM = 0x1f,
}disp_tv_mode;
```

• 成员

• 描述

disp_tv_mode 用于描述TV 输出模式。

### 9.15 disp_output

• 原型

```
typedef struct
{
    unsigned int type;
    unsigned int mode;
}disp_output;
```

• 成员

| 变量 | 说明                     |
| ---- | ------------------------ |
| Type | 输出类型                 |
| Mode | 输出模式，480P/576P, etc |

• 描述

disp_output 用于描述显示输出类型，模式。

### 9.16 disp_layer_mode

• 原型

```
typedef enum
{
    LAYER_MODE_BUFFER = 0,
    LAYER_MODE_COLOR = 1,
}disp_layer_mode;
```

• 成员

| 变量              | 说明                                                    |
| ----------------- | ------------------------------------------------------- |
| LAYER_MODE_BUFFER | buffer 模式，带buffer 的图层                            |
| LAYER_MODE_COLOR  | 单色模式，无buffer 的图层，只需要一个颜色值表示图像内容 |

• 描述

disp_layer_mode 用于描述图层模式。

### 9.17 disp_scan_flags

• 原型

```
typedef enum
{
    DISP_SCAN_PROGRESSIVE = 0,//non interlace
    DISP_SCAN_INTERLACED_ODD_FLD_FIRST = 1 << 0,//interlace ,odd field first
    DISP_SCAN_INTERLACED_EVEN_FLD_FIRST = 1 << 1,//interlace,even field first
}disp_scan_flags;
```

• 成员

| 变量                                | 说明                 |
| ----------------------------------- | -------------------- |
| DISP_SCAN_PROGRESSIVE               | 逐行模式             |
| DISP_SCAN_INTERLACED_ODD_FLD_FIRST  | 隔行模式，奇数行优先 |
| DISP_SCAN_INTERLACED_EVEN_FLD_FIRST | 隔行模式，偶数行优先 |

• 描述
disp_scan_flags 用于描述显示Buffer 的扫描方式。