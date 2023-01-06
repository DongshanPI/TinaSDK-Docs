## 3 模块接口概述

模块使用主要通过ioctl 实现，对应的驱动节点是/dev/disp。

具体定义请仔细阅读头文件上面的注释，kernel/linux-4.9/include/video/sunxi_display2.h。

对于显示模块来说，把图层参数设置到驱动，让显示器显示为最重要。sunxi 平台的DE 接受用户设置图层参数，通过disp,channel,layer_id 三个索引确定需要设置的显示位置（disp:0/1,channel: 0/1/2/3，layer_id:0/1/2/3)，其中disp 表示显示器索引，channel 表示通道索引，layer_id 表示通道内的图层索引。

下面着重地把图层的参数从头文件中拿出来介绍。

```
truct disp_fb_info2 {
    int fd;
    struct disp_rectsz size[3];
    unsigned int align[3];
    enum disp_pixel_format format;
    enum disp_color_space color_space;
    int trd_right_fd;
    bool pre_multiply;
    struct disp_rect64 crop;
    enum disp_buffer_flags flags;
    enum disp_scan_flags scan;
    enum disp_eotf eotf;
    int depth;
    unsigned int fbd_en;
    int metadata_fd;
    unsigned int metadata_size;
    unsigned int metadata_flag;
};
```

• fd

显存的文件句柄。

• size 与crop

Size 表示buffer 的完整尺寸，crop 则表示buffer 中需要显示裁减区。如下图所示，完整的图像以size 标识，而矩形框住的部分为裁减区，以crop 标识，在屏幕上

只能看到crop 标识的部分，其余部分是隐藏的，不能在屏幕上显示出来的。

![image-20221123152906780](http://photos.100ask.net/tina-docs/Tina_Linux_Display_DevGuide_image-20221123152906780.png)

<center>图3-1: size 和crop 示意图</center>

• crop 和screen_win

crop 上面已经介绍过，Screen_win 为crop 部分buffer 在屏幕上显示的位置。如果不需要进行缩放的话，crop 和screen_win 的width,height 是相等的，如果需要

缩放，crop 和screen_win 的width,height 可以不相等。

![image-20221123152930138](http://photos.100ask.net/tina-docs/Tina_Linux_Display_DevGuide_image-20221123152930138.png)

<center>图3-2: crop 和screen win 示意图</center>

• alpha

Alpha 模式有三种:

1. gloabal alpha: 全局alpha，也叫面alpha，即整个图层共用一个alpha，统一的透明度。
2. pixel alpha: 点alpha，即每个像素都有自己单独的alpha，可以实现部分区域全透，部分区域半透，部分区域不透的效果。
3. global_pixel alpha: 可以是说以上两种效果的叠加，在实现pxiel alpha 的效果的同时，还可以做淡入浅出的效果。

![image-20221123153137180](http://photos.100ask.net/tina-docs/Tina_Linux_Display_DevGuide_image-20221123153137180.png)

<center>图3-3: alpha 叠加模式</center>

• align

显存的对齐字节数。

• format

输入图层的格式。Ui 通道支持的格式：

```
DISP_FORMAT_ARGB_8888
DISP_FORMAT_ABGR_8888
DISP_FORMAT_RGBA_8888
DISP_FORMAT_BGRA_8888
DISP_FORMAT_XRGB_8888
DISP_FORMAT_XBGR_8888
DISP_FORMAT_RGBX_8888
DISP_FORMAT_BGRX_8888
DISP_FORMAT_RGB_888
DISP_FORMAT_BGR_888
DISP_FORMAT_RGB_565
DISP_FORMAT_BGR_565
DISP_FORMAT_ARGB_4444
DISP_FORMAT_ABGR_4444
DISP_FORMAT_RGBA_4444
DISP_FORMAT_BGRA_4444
DISP_FORMAT_ARGB_1555
DISP_FORMAT_ABGR_1555
DISP_FORMAT_RGBA_5551
DISP_FORMAT_BGRA_5551
DISP_FORMAT_A2R10G10B10
DISP_FORMAT_A2B10G10R10
DISP_FORMAT_R10G10B10A2
DISP_FORMAT_B10G10R10A2
```

Video 通道支持的格式：

```
DISP_FORMAT_ARGB_8888
DISP_FORMAT_ABGR_8888
DISP_FORMAT_RGBA_8888
DISP_FORMAT_BGRA_8888
DISP_FORMAT_XRGB_8888
DISP_FORMAT_XBGR_8888
DISP_FORMAT_RGBX_8888
DISP_FORMAT_BGRX_8888
DISP_FORMAT_RGB_888
DISP_FORMAT_BGR_888
DISP_FORMAT_RGB_565
DISP_FORMAT_BGR_565
DISP_FORMAT_ARGB_4444
DISP_FORMAT_ABGR_4444
DISP_FORMAT_RGBA_4444
DISP_FORMAT_BGRA_4444
DISP_FORMAT_ARGB_1555
DISP_FORMAT_ABGR_1555
DISP_FORMAT_RGBA_5551
DISP_FORMAT_BGRA_5551
DISP_FORMAT_YUV444_I_AYUV
DISP_FORMAT_YUV444_I_VUYA
DISP_FORMAT_YUV422_I_YVYU
DISP_FORMAT_YUV422_I_YUYV
DISP_FORMAT_YUV422_I_UYVY
DISP_FORMAT_YUV422_I_VYUY
DISP_FORMAT_YUV444_P
DISP_FORMAT_YUV422_P
DISP_FORMAT_YUV420_P
DISP_FORMAT_YUV411_P
DISP_FORMAT_YUV422_SP_UVUV
DISP_FORMAT_YUV422_SP_VUVU
DISP_FORMAT_YUV420_SP_UVUV
DISP_FORMAT_YUV420_SP_VUVU
DISP_FORMAT_YUV411_SP_UVUV
DISP_FORMAT_YUV411_SP_VUVU
DISP_FORMAT_YUV444_I_AYUV_10BIT
DISP_FORMAT_YUV444_I_VUYA_10BIT
```

所有图层都支持缩放。对图层的操作如下所示：

1. 设置图层参数并使能，接口为DISP_LAYER_SET_CONFIG，图像格式，buffer size，
   buffer 地址，alpha 模式，enable，图像帧id 号等参数。
2. 关闭图层，依然通过DISP_LAYER_SET_CONFIG，将enable 参数设置为0 关闭。

