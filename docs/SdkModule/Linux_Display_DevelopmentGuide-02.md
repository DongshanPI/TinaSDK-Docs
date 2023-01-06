## 2 模块介绍

### 2.1 模块功能介绍

![image-20221123145359081](http://photos.100ask.net/tina-docs/Tina_Linux_Display_DevGuide_image-20221123145359081.png)

<center>图2-1: 模块框图</center>

本模块框图如上，由显示引擎（DE）和各类型控制器（tcon）组成。输入图层（layers）在DE中进行显示相关处理后，通过一种或多种接口输出到显示设备上显

示，以达到将众多应用渲染的图层合成后在显示器呈现给用户观看的作用。DE 有2 个独立单元（可以简称de0、de1），可以分别接受用户输入的图层进行合成，

输出到不同的显示器，以实现双显。DE 的每个独立的单元有1-4 个通道（典型地，de0 有4 个，de1 有2 个），每个通道可以同时处理接受4 个格式相同的

图层。sunxi 平台有视频通道和UI 通道之分。视频通道功能强大，可以支持YUV 格式和RGB图层。UI 通道只支持RGB 图层。

简单来说，显示模块的主要功能如下：

• 支持lcd(hv/lvds/cpu/dsi) 输出。
• 支持双显输出。
• 支持多图层叠加混合处理。
• 支持多种显示效果处理（alpha, colorkey, 图像增强，亮度/对比度/饱和度/色度调整）。
• 支持智能背光调节。
• 支持多种图像数据格式输入(arg,yuv)。
• 支持图像缩放处理。
• 支持截屏。
• 支持图像转换。



### 2.2 相关术语介绍

#### 2.2.1 硬件术语介绍

<center>表2-1: 硬件术语介绍表</center>

| 术语      | 解释                                                         |
| --------- | ------------------------------------------------------------ |
| de        | display engine，显示引擎，负责将输入的多图层进行叠加、混合、缩放等处理的硬件模块 |
| channel   | 一个硬件通道，包含若干图层处理单元，可以同时处理若干（典型4 个）格式相同的图层 |
| layer     | 一个图层处理单元，可以处理一张输入图像，按支持的图像格式分video 和ui类型 |
| capture   | 截屏，将de 的输出保存到本地文件                              |
| alpha     | 透明度，在混合时决定对应图像的透明度                         |
| transform | 图像变换，如平移、旋转等                                     |
| overlay   | 图像叠加，按顺序将图像叠加一起的效果。z 序大的靠近观察者，会把z 序小的挡住 |
| blending  | 图像混合，按alpha 比例将图像合成一起的效果                   |
| enhance   | 图像增强，有目的地处理图像数据以达到改善图像效果的过程或方法 |

#### 2.2.2 软件术语介绍

<center>表2-2: 软件术语介绍表</center>

| 术语     | 解释                                                         |
| -------- | ------------------------------------------------------------ |
| fb       | 帧缓冲（framebuffer）,Linux 为显示设备提供的一个接口，把显存抽象成的一种设备。有时也指一块显存 |
| al       | 抽象层，驱动中将底层硬件抽象成固定业务逻辑的软件层           |
| lowlevel | 底层，直接操作硬件寄存器的软件层                             |

### 2.3 模块配置介绍

#### 2.3.1 kenel_menuconfig 配置说明

```
make kenel_menuconfig
```

具体配置目录为：

```
Device Drivers --->
	Graphics support --->
		<*> Support for frame buffer devices --->
			Video support for sunxi --->
				<*> DISP Driver Support(sunxi-disp2)
```

![image-20221123150416330](http://photos.100ask.net/tina-docs/Tina_Linux_Display_DevGuide_image-20221123150416330.png)

<center>图2-2: disp 配置</center>

其中：

• DISP Driver Support(sunxi-disp2)

DE 驱动请选上。

• debugfs support for disp driver(sunxi-disp2)

调试节点，建议选上，方便调试。

• composer support for disp driver(sunxi-disp2)

disp2 的fence 处理。linux 系统可以不选择。

### 2.4 源码结构介绍

源码结构如下：

```
├─drivers
│ ├─video
│ │ ├─fbdev
│ │ │ ├─sunxi --display driver for sunxi
│ │ │ │ ├─disp2/ --disp2 的目录
│ │ │ │ │ ├─disp
│ │ │ │ │ │ ├─dev_disp.c --display driver 层
│ │ │ │ │ │ ├─dev_fb.c --framebuffer driver 层
│ │ │ │ │ │ ├─de --bsp层
│ │ │ │ │ │ │ ├─disp_lcd.c --disp_manager.c ..
│ │ │ │ │ │ │ ├─disp_al.c --al层
│ │ │ │ │ │ │ │ └─lowlevel_sun*i/ --lowlevel 层
│ │ │ │ │ │ │ │ ├─de_lcd.c ...
│ │ │ │ │ │ │ └─disp_sys_int.c --OSAL 层,与操作系统相关层
│ │ │ │ │ │ ├─lcd/ lcd driver
│ │ │ │ │ │ │ │─lcd_src_interface.c --与display 驱动的接口
│ │ │ │ │ │ │ │─default_panel.c... --平台已经支持的屏驱动
include
├─video video header dir
│ ├─sunxi_display2.h display header file
```

### 2.5 驱动框架介绍

![image-20221123151947713](http://photos.100ask.net/tina-docs/Tina_Linux_Display_DevGuide_image-20221123151947713.png)

<center>图2-3: 驱动框图</center>

显示驱动可划分为三个层面，驱动层，框架层及底层。底层与图形硬件相接，主要负责将上层配置的功能参数转换成硬件所需要的参数，并配置到相应寄存器中。

显示框架层对底层进行抽象封装成一个个的功能模块。驱动层对外封装功能接口，通过内核向用户空间提供相应的设备结点及统一的接口。在驱动层，分为四个驱

动，分别是framebuffer 驱动，disp 驱动，lcd 驱动，hdmi 驱动。Framebuffer 驱动与framebuffer core 对接，实现linux 标准的framebuffre 接口。

Disp 驱动是是整个显示驱动中的核心驱动模块，所有的接口都由disp 驱动来提供，包括lcd 的接口。