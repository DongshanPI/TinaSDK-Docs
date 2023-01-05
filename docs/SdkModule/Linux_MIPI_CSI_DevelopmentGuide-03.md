## 3 V4L2 接口描述

### 3.1 VIDIOC_QUERYCAP

#### 3.1.1 Parameters

```
Capability of csi driver（struct v4l2_capability * capability）
struct v4l2_capability {
    __u8 driver[16];    /* i.e. "bttv" */
    __u8 card[32];      /* i.e. "Hauppauge WinTV" */
    __u8 bus_info[32];  /* "PCI:" + pci_name(pci_dev) */
    __u32 version;      /* should use KERNEL_VERSION() */
    __u32 capabilities; /* Device capabilities */
    __u32 reserved[4];
};
```



#### 3.1.2 Returns

Success:0; Fail: Failure Number



#### 3.1.3 Description

获取驱动的名称、版本、支持的 capabilities 等，如 V4L2_CAP_STREAMIN,V4L2_BUF_TYPE_VIDEO_CAPTURE_MPLANE 等。



### 3.2 VIDIOC_ENUM_INPUT

#### 3.2.1 Parameters

```
input（struct v4l2_input *inp）
struct v4l2_input {
    __u32 index;    /* Which input */
    __u8 name[32];  /* Label */
    __u32 type;     /* Type of input */
    __u32 audioset; /* Associated audios (bitfield) */
    __u32 tuner;    /* Associated tuner */
    v4l2_std_id std;
    __u32 status;
    __u32 capabilities;
    __u32 reserved[3];
};
```



#### 3.2.2 Returns

Success:0; Fail: Failure Number



#### 3.2.3 Description

获取驱动支持的 input index。目前驱动只支持 input index = 0 或 index = 1。

Index = 0 表示 primary csi device

Index = 1 表示 secondary csi device

应用输入 index 参数，驱动返回 type。对于 VIN 设备来说，type 为 V4L2_INPUT_TYPE_CAMERA。



### 3.3 VIDIOC_S_INPUT

#### 3.3.1 Parameters

```
input（struct v4l2_input *inp）
The same as VIDIOC_ENUM_INPUT
```



#### 3.3.2 Returns

Success:0; Fail: Failure Number



#### 3.3.3 Description

通过 inp.index 设置当前要访问的 csi device 为 primary device 还是 secondary device。

Index = 0 （双摄像头配置中，一般对应后置摄像头。若只有一个摄像头设备，则 index 固定为0）

Index = 1（双摄像头配置中，一般对应前置摄像头）

调用该接口后，实际上会对 csi device 进行初始化工作。

在 A133 平台：Index 在 video0、1 时固定要设为 0；在 video2、3 要设为 1。





### 3.4 VIDIOC_G_INPUT

#### 3.4.1 Parameters

```
input（struct v4l2_input *inp）
The same as VIDIOC_ENUM_INPUT
```



#### 3.4.2 Returns

Success:0; Fail: Failure Number



#### 3.4.3 Description

获取 inp.index，判断当前设置的 csi device 为 primary device 还是 secondary device。

Index = 0 （双摄像头配置中，一般对应后置摄像头。若只有一个摄像头设备，则 index 固定为0）

Index = 1（双摄像头配置中，一般对应前置摄像头）



### 3.5 VIDIOC_S_PARM

#### 3.5.1 Parameters

```
Parameter（struct v4l2_streamparm *parms）
struct v4l2_streamparm {
    enum v4l2_buf_type type;
    union {
        struct v4l2_captureparm capture;
        struct v4l2_outputparm output;
        __u8 raw_data[200]; /* user-defined */
    } parm;
};
struct v4l2_captureparm {
    __u32 capability; /* Supported modes */
    __u32 capturemode; /* Current mode */
    struct v4l2_fract timeperframe; /* Time per frame in .1us units */
    __u32 extendedmode; /* Driver-specific extensions */
    __u32 readbuffers; /* # of buffers for read */
    __u32 reserved[4];
};
```



#### 3.5.2 Returns

Success:0; Fail: Failure Number



#### 3.5.3 Description

CSI 作为输入设备，只关注 parms.type 和 parms. capture。

应用使用时，parms.type = V4L2_BUF_TYPE_VIDEO_CAPTURE_MPLANE；

其中通过设定 parms->capture.capturemode（V4L2_MODE_VIDEO 或 V4L2_MODE_IMAGE），

实现视频或图片的采集。通过设定 parms->capture.timeperframe，可以设置帧率。





### 3.6 VIDIOC_G_PARM

#### 3.6.1 Parameters

```
Parameter（struct v4l2_streamparm *parms）
The same as VIDIOC_S_PARM
```



#### 3.6.2 Returns

Success:0; Fail: Failure Number



#### 3.6.3 Description

应用使用时，parms.type = V4L2_BUF_TYPE_VIDEO_CAPTURE_MPLANE；

通过 parms->capture.capturemode 返回当前是 V4L2_MODE_VIDEO 或 V4L2_MODE_IMAGE；

通过 parms->capture.timeperframe, 返回当前设置的帧率。



### 3.7 VIDIOC_ENUM_FMT

#### 3.7.1 Parameters

```
V4L2 format（struct v4l2_fmtdesc * fmtdesc）
struct v4l2_fmtdesc {
    __u32 index; /* Format number */
    enum v4l2_buf_type type; /* buffer type */
    __u32 flags;
    __u8 description[32]; /* Description string */
    __u32 pixelformat; /* Format fourcc */
    __u32 reserved[4];
};
```



#### 3.7.2 Returns

Success:0; Fail: Failure Number



#### 3.7.3 Description

获取驱动支持的 V4L2 格式。

应用输入 type，index 参数，驱动返回 pixelformat 。对于 VIN 设备来说，type 为V4L2_BUF_TYPE_VIDEO_CAPTURE_MPLANE。



### 3.8 VIDIOC_TRY_FMT

#### 3.8.1 Parameters

```
Video type, format and size（struct v4l2_format * fmt）
struct v4l2_format {
    enum v4l2_buf_type type;
    union {
        struct v4l2_pix_format pix;
        struct v4l2_pix_format_mplane pix_mp;
        struct v4l2_window win;
        struct v4l2_vbi_format vbi;
        struct v4l2_sliced_vbi_format sliced;
        __u8 raw_data[200];
    } fmt;
};
struct v4l2_pix_format {
    __u32 width;
    __u32 height;
    __u32 pixelformat;
    enum v4l2_field field;
    __u32 bytesperline; /* for padding, zero if unused */
    __u32 sizeimage;
    enum v4l2_colorspace colorspace;
    __u32 priv; /* private data, depends on pixelformat */
};
```



#### 3.8.2 Returns

Success:0; Fail: Failure Number



#### 3.8.3 Description

根据捕捉视频的类型、格式和大小，判断模式、格式等是否被驱动支持。不会改变任何硬件设置。

对于 VIN 设备，type 为 V4L2_BUF_TYPE_VIDEO_CAPTURE_MPLANE。使用 struct v4l2_pix_format_mplane 进行参数传递。

应用程序输入 struct v4l2_pix_format_mplane 结构体里面的 width、height、pixelformat、field 等参数，驱动返回最接近的 width、height；若 pixelformat、field 不支持，则默认选择驱动支持的第一种格式。



### 3.9 VIDIOC_S_FMT

#### 3.9.1 Parameters

```
Video type, format and size（struct v4l2_format * fmt）
The same as VIDIOC_TRY_FMT
```



#### 3.9.2 Returns

Success:0; Fail: Failure Number



#### 3.9.3 Description

设置捕捉视频的类型、格式和大小，设置之前会调用 VIDIOC_TRY_FMT。

对于 VIN 设备，type 为 V4L2_BUF_TYPE_VIDEO_CAPTURE_MPLANE。使用 struct v4l2_pix_format_mplane 进行参数传递。应用程序输入 width、height、pixelformat、field 等，驱动返回最接近的 width、height； 若 pixelformat、field 不支持，则默认选择驱动支持的第一种格式。

应用程序应该以驱动返回的 width、height、pixelformat、field 等作为后续使用传递的参数。对于 OSD 设备，type 为 V4L2_BUF_TYPE_VIDEO_OVERLAY。使用 struct v4l2_window进行参数传递。

应用程序输入水印的个数、窗口位置和大小、bitmap 地址、bitmap 格式以及 global_alpha 等。驱动保存这些参数，并在 VIDIOC_OVERLAY 命令传递使能命令时生效。



### 3.10 VIDIOC_G_FMT

#### 3.10.1 Parameters

```
Video type, format and size（struct v4l2_format * fmt）
The same as VIDIOC_TRY_FMT
```



#### 3.10.2 Returns

Success:0; Fail: Failure Number



#### 3.10.3 Description

获取捕捉视频的 width、height、pixelformat、field、bytesperline、sizeimage 等参数。



### 3.11 VIDIOC_OVERLAY

#### 3.11.1 Parameters

```
Overlay on/off（unsigned int i）
```



#### 3.11.2 Returns

Success:0; Fail: Failure Number



#### 3.11.3 Description

传递 1 表示使能，0 表示关闭。设置使能时会更新 osd 参数，使之生效。



### 3.12 VIDIOC_REQBUFS

#### 3.12.1 Parameters

```
Buffer type ,count and memory map type（struct v4l2_requestbuffers * req）
struct v4l2_requestbuffers {
    __u32 count;
    enum v4l2_buf_type type;
    enum v4l2_memory memory;
    __u32 reserved[2];
};
```



#### 3.12.2 Returns

Success:0; Fail: Failure Number



#### 3.12.3 Description

v4l2_requestbuffers 结构中定义了缓存的数量，驱动会据此申请对应数量的视频缓存。多个缓存可以用于建立 FIFO，来提高视频采集的效率。这些 buffer 通过内核申请，申请后需要通过 mmap 方法，映射到 User 空间。

Count：定义需要申请的 video buffer 数量；

Type：对于 VIN 设备，为 V4L2_BUF_TYPE_VIDEO_CAPTURE_MPLANE；

Memory：目前支持 V4L2_MEMORY_MMAP、V4L2_MEMORY_USERPTR、V4L2_MEMORY_DMABUF 方式。

应用程序传递上述三个参数，驱动会根据 VIDIOC_S_FMT 设置的格式计算供需要 buffer 的大小，并返回 count 数量。





### 3.13 VIDIOC_QUERYBUF

#### 3.13.1 Parameters

```
Buffer type ,index and memory map type（struct v4l2_buffer *buf）
struct v4l2_buffer {
    __u32 index;
    enum v4l2_buf_type type;
    __u32 bytesused;
    __u32 flags;
    enum v4l2_field field;
    struct timeval timestamp;
    struct v4l2_timecode timecode;
    __u32 sequence;
    
    /* memory location */
    enum v4l2_memory memory;
    union {
        __u32 offset;
        unsigned long userptr;
        struct v4l2_plane *planes;
    } m;
    __u32 length;
    __u32 input;
    __u32 reserved;
};
```



#### 3.13.2 Returns

Success:0; Fail: Failure Number



#### 3.13.3 Description

通过 struct v4l2_buffer 结构体的 index，访问对应序号的 buffer，获取到对应 buffer 的缓存信息。主要利用 length 信息及 m.offset 信息来完成 mmap 操作。



### 3.14 VIDIOC_DQBUF

#### 3.14.1 Parameters

```
Buffer type ,index and memory map type（struct v4l2_buffer *buf）
struct v4l2_buffer is the same as VIDIOC_QUERYBUF
```



#### 3.14.2 Returns

Success:0; Fail: Failure Number



#### 3.14.3 Description

将 driver 已经填充好数据的 buffer 出列，供应用使用。

应用程序根据 index 来识别 buffer，此时 m.offset 表示 buffer 对应的物理地址。





### 3.15 VIDIOC_QBUF

#### 3.15.1 Parameters

```
Buffer type ,index and memory map type（struct v4l2_buffer *buf）
```



#### 3.15.2 Returns

Success:0; Fail: Failure Number



#### 3.15.3 Description

将 User 空间已经处理过的 buffer，重新入队，移交给 driver，等待填充数据。

应用程序根据 index 来识别 buffer。





### 3.16 VIDIOC_STREAMON

#### 3.16.1 Parameters

```
Buffer type（enum v4l2_buf_type *type）
```



#### 3.16.2 Returns

Success:0; Fail: Failure Number



#### 3.16.3 Description

此处的 buffer type 为 V4L2_BUF_TYPE_VIDEO_CAPTURE_MPLANE。运行此 IOCTL， 将 buffer 队列中所有 buffer 入队，并开启 CSIC DMA 硬件中断，每次中断便表示完成一帧 buffer 数据的填入。



### 3.17 VIDIOC_STREAMOFF

#### 3.17.1 Parameters

```
Buffer type（enum v4l2_buf_type *type）
```



#### 3.17.2 Returns

Success:0; Fail: Failure Number



#### 3.17.3 Description

此处的 buffer type 为 V4L2_BUF_TYPE_VIDEO_CAPTURE_MPLANE。运行此 IOCTL，停止捕捉视频，将 frame buffer 队列清空，以及 video buffer 释放。



### 3.18 VIDIOC_QUERYCTRL

#### 3.18.1 Parameters

```
Control id and value（struct v4l2_queryctrl *qc）
struct v4l2_queryctrl {
    __u32 id;
    enum v4l2_ctrl_type type;
    __u8 name[32]; /* Whatever */
    __s32 minimum; /* Note signedness */
    __s32 maximum;
    __s32 step;
    __s32 default_value;
    __u32 flags;
    __u32 reserved[2];
};
```



#### 3.18.2 Returns

Success:0; Fail: Failure Number



#### 3.18.3 Description

应用程序通过 id 参数，驱动返回需要调节参数的 name，minmum，maximum，default_value 以及步进 step。（由 v4l2 conctrols framework 完成）目前可能支持的 id 请参考 VIDIOC_S_CTRL。



### 3.19 VIDIOC_S_CTRL

#### 3.19.1 Parameters

```
Control id and value（struct v4l2_queryctrl *qc）
The same as VIDIOC_QUERYCTRL
```



#### 3.19.2 Returns

Success:0; Fail: Failure Number



#### 3.19.3 Description

应用程序通过 id，value 等参数，对 camera 驱动对应的参数进行设置。

驱动内部会先调用 vidioc_queryctrl，判断 id 是否支持，value 是否在 minimum 和 maximum 之间。（由 v4l2 conctrols framework 完成）目前可能支持的 id 和 value 参考附件。





### 3.20 VIDIOC_G_CTRL

#### 3.20.1 Parameters

```
Control id and value（struct v4l2_queryctrl *qc）
The same as VIDIOC_QUERYCTRL
```



#### 3.20.2 Returns

Success:0; Fail: Failure Number



#### 3.20.3 Description

应用程序通过 id，驱动返回对应 id 当前设置的 value。



### 3.21 VIDIOC_ENUM_FRAMESIZES

#### 3.21.1 Parameters

```
index,type,format（struct v4l2_frmsizeenum）
enum v4l2_frmsizetypes {
    V4L2_FRMSIZE_TYPE_DISCRETE = 1,
    V4L2_FRMSIZE_TYPE_CONTINUOUS = 2,
    V4L2_FRMSIZE_TYPE_STEPWISE = 3,
};

struct v4l2_frmsize_discrete {
    __u32 width; /* Frame width [pixel] */
    __u32 height; /* Frame height [pixel] */
};

struct v4l2_frmsize_stepwise {
    __u32 min_width;   /* Minimum frame width [pixel] */
    __u32 max_width;   /* Maximum frame width [pixel] */
    __u32 step_width;  /* Frame width step size [pixel] */
    __u32 min_height;  /* Minimum frame height [pixel] */
    __u32 max_height;  /* Maximum frame height [pixel] */
    __u32 step_height; /* Frame height step size [pixel] */
};

struct v4l2_frmsizeenum {
    __u32 index;        /* Frame size number */
    __u32 pixel_format; /* Pixel format */
    __u32 type;         /* Frame size type the device supports. */
    union {             /* Frame size */
        struct v4l2_frmsize_discrete discrete;
        struct v4l2_frmsize_stepwise stepwise;
	};
	__u32 reserved[2]; /* Reserved space for future use */
};
```



#### 3.21.2 Returns

Success:0; Fail: Failure Number



#### 3.21.3 Description

根据应用传进来的 index，pixel_format，驱动返回 type，并根据 type 填写 discrete 或 step-wise 的值。Discrete 表示分辨率固定的值；stepwise 表示分辨率有最小值和最大值，并根据step 递增。上层根据返回的 type，做对应不同的操作。



### 3.22 VIDIOC_ENUM_FRAMEINTERVALS

#### 3.22.1 Parameters

```
Index，format，size，type（struct v4l2_frmivalenum）
enum v4l2_frmivaltypes {
    V4L2_FRMIVAL_TYPE_DISCRETE = 1,
    V4L2_FRMIVAL_TYPE_CONTINUOUS = 2,
    V4L2_FRMIVAL_TYPE_STEPWISE = 3,
};
struct v4l2_frmival_stepwise {
    struct v4l2_fract min;  /* Minimum frame interval [s] */
    struct v4l2_fract max;  /* Maximum frame interval [s] */
    struct v4l2_fract step; /* Frame interval step size [s] */
};
struct v4l2_frmivalenum {
    __u32 index;        /* Frame format index */
    __u32 pixel_format; /* Pixel format */
    __u32 width;        /* Frame width */
    __u32 height;       /* Frame height */
    __u32 type;         /* Frame interval type the device supports. */
    union {             /* Frame interval */
        struct v4l2_fract discrete;
        struct v4l2_frmival_stepwise stepwise;
    };
    __u32 reserved[2]; /* Reserved space for future use */
};
```



#### 3.22.2 Returns

Success:0; Fail: Failure Number



#### 3.22.3 Description

应用程序通过 pixel_format、width、height、驱动返回 type，并根据 type 填写

V4L2_FRMIVAL_TYPE_DISCRETE、V4L2_FRMIVAL_TYPE_CONTINUOUS 或V4L2_FRMIVAL_TYPE_STEPWISE。Discrete 表示支持单一的帧率；stepwise 表示支持步进的帧率。





### 3.23 VIDIOC_ISP_EXIF_REQ

作用: 得到当前照片的 EXIF 信息，填写到相应的编码域中。目的：对于 raw sensor 尽量填写正规的 EXIF 信息，yuv sensor 该 IOCTRL 也可以使用，不过驱动中填写的也是固定值。相关参数：

```
struct v4l2_fract {
    __u32 numerator;
    __u32 denominator;
};

struct isp_exif_attribute {
    struct v4l2_fract exposure_time;
    struct v4l2_fract shutter_speed;
    __u32 aperture;
    __u32 focal_length;
    __s32 exposure_bias;
    __u32 iso_speed;
    __u32 flash_fire;
    __u32 brightness;
};

struct v4l2_fract exposure_time;
曝光时间：分数类型，例如numerator = 1，denominator = 200，则表示1/200秒的曝光时间。

struct v4l2_fract shutter_speed;
快门速度：分数类型，例如numerator = 1，denominator = 200，则表示1/200秒的快门速度。（实际上和曝光时间数值相同）

__u32 aperture;
光圈大小：FNumber，例如aperture = 22，则表示，光圈大小为2.2，即FNumber = 22/10;

__u32 focal_length;
焦距：例如focal_length = 1400，则表示焦距为14mm，即FocalLength = 1400/100( mm);

__s32 exposure_bias;
曝光补偿：范围 -4~4

__u32 iso_speed;
感光速度：50~3200

__u32 flash_fire;
闪光灯是否开启：flash_fire = 1 表示闪光灯开启，flash_fire = 0 表示闪光灯未开启。

__u32 brightness;
图像亮度：0~255.

使用示例：
int V4L2CameraDevice::getExifInfo(struct isp_exif_attribute *exif_attri)
{
    int ret = -1;
    if (mCameraFd == NULL)
    {
	    return 0xFF000000;
    }
    ret = ioctl(mCameraFd, VIDIOC_ISP_EXIF_REQ, exif_attri);
    return ret;
}
```

