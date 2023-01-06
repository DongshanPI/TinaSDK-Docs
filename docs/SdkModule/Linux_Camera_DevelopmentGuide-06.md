## 6 camera 功能测试

Tina 系统可以通过SDK 中的camerademo 包来验证camera sensor（usb camera）是否移植成功，如果可以正常捕获保存图像数据，则底层驱动、板子硬件正常。

### 6.1 camerademo 配置

在命令行中进入Tina 根目录，执行make menuconfig 进入配置主界面，并按以下配置路径操作：

```
Allwinner
	└─>camerademo
```

首先，选择Allwinner 选项进入下一级配置，如下图所示：
![image-20221123141424322](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123141424322.png)

<center>图6-1: allwinner</center>

然后，选择camerademo 选项，可选择<*> 表示直接编译包含在固件，也可以选择表示仅编译不包含在固件。当平台的camera 框架是VIN 且需要使用ISP 时，将

需要在camerademo 的选项处点击回车进行以下界面选择使能ISP。（该选项只能在VIN 框架中，使用RAW sensor时使用，在修改该选项之后，需要先单独mm -

B 编译该package）。

![image-20221123141504379](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123141504379.png)

<center>图6-2: camerademo</center>

![image-20221123141522573](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123141522573.png)

<center>图6-3: vinisp</center>

### 6.2 源码结构

camerademo 的源代码位于package/allwinner/camerademo/目录下：

```
|---src
| camerademo.c //camea测试的主流程代码
| camerademo.h //camera demo相关数据结构
| common.c //实现共用的函数，转换时间、保存文件、测试帧率等
| common.h //共用函数头文件
| convert.c //实现图像格式转换函数
| convert.h //图像格式转换函数头文件
```

### 6.3 camerademo 使用方法

在小机端加载成功后输入camerademo help，假如驱动产生的节点video0（测试默认以/dev/video0 作为设备对象）可以打开则会出现下面提示：

通过提示我们可以得到一些提示信息，了解到该程序的运行方式、功能，可以查询sensor 支持的分辨率、sensor 支持的格式以及设置获取照片的数量、数据保存

的格式、路径、添加水印、测试数据输出的帧率、从open 节点到数据流打通需要的时间等，help 打印信息如下图：

![image-20221123142705806](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123142705806.png)

<center>图6-4: help</center>

Camerademo 共有4 种运行模式：

1. 默认方式：直接输入camerademo 即可，在这种运行模式下，将设置摄像头为640*480 的NV21 格式输出图像数据，并以BMP 和YUV 的格式保存在/tmp 目

  录下，而当输入camerademodebug 将会输出更详细的debug 信息；

2. 探测设置camerademo setting：将会在运行过程中根据具体camera 要求输入设置参数，当输入camerademo setting debug 的时候，将会输出详细的debug 信息；

3. 快速设置：camerademo argv[1] argv[2] argv[3] argv[4] argv[5] argv[6] argv[7]，将会按照输入参数设置图像输出，同样，当输入camerademo argv[1] 

  argv[2] argv[3] argv[4] argv[5] argv[6] argv[7] debug 时将会输出更详细的debug 信息。

4. 选择camera 设置：camerademo argv[1] argv[2] argv[3] argv[4] argv[5] argv[6] argv[7] argv[8]，将会按照输入参数设置图像输出，同样，当输入

  camerademo argv[1] argv[2] argv[3] argv[4] argv[5] argv[6] argv[7] argv[8] debug 时将会输出更详细的debug 信息。

#### 6.3.1 默认方式

当输入camerademo 之后，使用默认的参数运行，则会打印一下信息，如下图：

![image-20221123142833252](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123142833252.png)

<center>图6-5: camerademouser</center>

首先可以清楚的看到成功open video0 节点，并且知道照片数据的保存路径、捕获照片的数量以及当前设置：是否添加水印、输出格式、分辨率和从开启流传输到

第一帧数据达到时间间隔等信息。如果需要了解更多的详细信息，可以在运行程序的时候输入参数debug 即运行camerademo debug，将会打开demo 的debug 

模式，输出更详细的信息，包括camera 的驱动类型，支持的输出格式以及对应的分辨率，申请buf 的信息，实际输出帧率等。

#### 6.3.2 选择方式

在选择模式下有两种运行方式，一种是逐步选择，在camera 的探测过程，知道其支持的输出格式以及分辨率之后再设置camera 的相关参数；另一种是直接在运

行程序的时候带上相应参数，程序按照输入参数运行（其中还可以选择camera 索引，从而测试不同的camera）。

1. 输入camerademo setting，则按照程序的打印提示输入相应选择信息即可。
   • 输入保存路径、照片数量、保存的格式等。

![image-20221123142929515](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123142929515.png)

<center>图6-6: info</center>

• 选择输出格式。

![image-20221123142950450](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123142950450.png)

<center>图6-7: format</center>

• 选择输出图像分辨率。

![image-20221123143011933](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123143011933.png)

<center>图6-8: size</center>

其它信息与默认设置一致，如需打印详细的信息，运行camerademo setting debug 即可。

2. 第二种是设置参数：
   • 默认的video 0 节点：camerademo argv[1] argv[2] argv[3] argv[4] argv[5] argv[6] argv[7]。
   输入参数代表意义如下：

```
argv[1]：camera输出格式---NV21 YUYV MJPEG等；
argv[2]：camera分辨率width；
argv[3]：camera分辨率height；
argv[4]：sensor输出帧率；
argv[5]：保存照片的格式：all---bmp和yuv格式都保存、bmp---仅以bmp格式保存、yuv---仅以yuv格式保存；
argv[6]：捕获照片的保存路径；
argv[7]：捕获照片的数量；
```

例如：camerademo NV21 640 480 30 yuv /tmp 2，将会输出640*480@30fps 的NV21格式照片以yuv 格式、不添加水印保存在/tmp 路径下，照片共2 张。

其它信息与默认设置一致，如需打印详细的信息，运行camerademo argv[1] argv[2] argv[3] argv[4] argv[5] argv[6] argv[7] debug 即可。

![image-20221123143239114](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123143239114.png)

<center>图6-9: run1</center>

• 选择其他的video 节点：camerademo argv[1] argv[2] argv[3] argv[4] argv[5] argv[6] argv[7] argv[8]。

输入参数代表意义如下：

```
argv[1]：camera输出格式---NV21 YUYV MJPEG等；
argv[2]：camera分辨率width；
argv[3]：camera分辨率height；
argv[4]：sensor输出帧率；
argv[5]：保存照片的格式：all---bmp和yuv格式都保存、bmp---仅以bmp格式保存、yuv---仅以yuv格式保存；
argv[6]：捕获照片的保存路径；
argv[7]：捕获照片的数量；
argv[8]：video节点索引；
```

例如：camerademo YUYV 640 480 30 yuv /tmp 1 1，将会打开/dev/video1 节点并输出640*480@30fps 的以yuv 格式、不添加水印保存在/tmp 路径下，照片共

1 张。

其它信息与默认设置一致，如需打印详细的信息，运行camerademo argv[1] argv[2] argv[3] argv[4] argv[5] argv[6] argv[7] argv[8] debug 即可。

![image-20221123143328632](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123143328632.png)

<center>图6-10: run2</center>

#### 6.3.3 camerademo 保存RAW 数据

当需要使用camerademo 保存RAW 数据时，只需要将输出格式设置为RAW 格式即可。先确认sensor 驱动中的mbus_code 设置为多少位， 假设驱动中， 配置为

mbus_code =MEDIA_BUS_FMT_SGRBG10_1X10，那么可以确认sensor 输出是RAW10，camerademo 的输出格式设置为RAW10 即可。比如输入camerademo 

RGGB10 1920 1080 30 bmp /tmp 5，以上命令输出配置sensor 输出RAW 数据并保存在/tmp 目录，命令的含义参考本章节的《选择方式》。

注意：RAW 数据文件的保存后缀是.raw 。

#### 6.3.4 debug 信息解析

以下debug 信息将说明sensor 驱动的相关信息，拍摄到的照片保存位置、数量、保存的格式以及水印使用情况等：

![image-20221123143422192](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123143422192.png)

<center>图6-11: debug1</center>

以下debug 信息将说明驱动框架支持的格式以及sensor 支持的输出格式：

![image-20221123143618199](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123143618199.png)

<center>图6-12: debug2</center>

类似以下的信息代表这相应格式支持的分辨率信息：

![image-20221123143642323](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123143642323.png)

<center>图6-13: debug3</center>

以下信息将会提示将要设置到sensor 的格式和分辨率等信息：

![image-20221123143703117](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123143703117.png)

<center>图6-14: debug4</center>

以下信息将会提示设置格式的情况，buf 的相应信息等：

![image-20221123143723798](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123143723798.png)

<center>图6-15: debug5</center>

以下信息将提示当前拍照的照片索引以及从开启流传输到dqbuf 成功的时间间隔：

![image-20221123143742621](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123143742621.png)

<center>图6-16: debug6</center>

以下信息提示该sensor 的实际测量帧率信息：

![image-20221123143807349](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123143807349.png)

<center>图6-17: debug7</center>

以下信息提示从open 节点到可以得到第一帧数据的时间间隔，默认设置为测试拍照的相应设置：

![image-20221123143825189](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123143825189.png)

<center>图6-18: debug8</center>

#### 6.3.5 文件保存格式

设置完毕之后，将会在所设路径（默认/tmp）下面保存图像数据，数据分别有两种格式，一种是YUV 格式，以source_ 格式.yuv 名称保存；一种是BMP 格式，以

bmp_ 格式.bmp 格式保存，如下图所示。

查看图像数据时，需要通过adb pull 命令将相应路径下的图像数据pull 到PC 端查看。

![image-20221123143900168](http://photos.100ask.net/tina-docs/Linux_Camera_DevGuide_image-20221123143900168.png)

<center>图6-19: save</center>

### 6.4 select timeout 了，如何操作？

在完成sensor 驱动的移植，驱动模块正常加载，I2C 正常通信，将会在/dev 目录下创建相应的video 节点，之后可以使用camerademo 进行捕获测试，如果出现

select timeout,end capture thread!，这个情况可按照以下操作进行debug。

1. 先和模组厂确认，当前提供的寄存器配置是否可以正常输出图像数据。有些模组厂提供的寄存器配置还需要增加一个使能寄存器，这些可以在sensor 

  datasheet 上查询得到或者与模组厂沟通；

2. 通过dmesg 命令， 查看在运行camerademo 的过程中内核是否有异常的打印。在MR813/R818 平台，内核出现tdm 相关字段的连续打印，则需要确认，

  board.dts 中的isp配置是否正确，单摄的配置，isp_sel 和tdm_rx_sel 都需要配置为0；双摄的则需要先运行配置为isp0 的video 节点才能再运行isp1 的video 

  节点；

3. 其他的按照是并口还是mipi 接口进行相应的debug；

#### 6.4.1 DVP sensor

1. 确定sensor 的出图data 配置正确，是8 位的、10 位的、12 位的？确认之后，检查驱动中的sensor_formats 和sys_config.fex 中的csi data pin 设置是否正

  确；

2. MCLK 的频率配置是否正确；

3. sensor 驱动的sensor_g_mbus_config() 函数配置为DVP sensor，type 需要设置为V4L2_MBUS_PARALLEL；

4. 确定输出的data 是高8 位、高10 位，确定硬件引脚配置没有问题；

5. 示波器测量VSYNC、HSYNC 有没有波形输出，这两个标记着有一场数据、一行数据信号产
   生；

6. 测量data 脚有没有波形，电压幅值是否正常；

7. 如果没有波形，检查一下sensor 的寄存器配置，看看有没有软件复位的操作，如果有，在该寄存器配置后面加上”{REG_DLY，0xff}“进行相应的延时，防止在

  软件复位的时候，sensor还没有准备好就I2C 配置寄存器；

8. 如果上面都还是没有接收到数据，那么在sensor 的驱动文件，有以下配置，这三个宏定义的具体值。每个都有两种配置，将这三个宏的配置两两组合，共8 种

  配置，都尝试一下；

```
#define VREF_POL V4L2_MBUS_VSYNC_ACTIVE_HIGH
#define HREF_POL V4L2_MBUS_HSYNC_ACTIVE_HIGH
#define CLK_POL V4L2_MBUS_PCLK_SAMPLE_RISING
```

#### 6.4.2 mipi sensor

如果mipi sensor 没有正常出图，做以下debug 操作：

1. mipi 接口和主控板子连接不要飞线，mipi 信号本身就是高频差分信号，布线时都要求高，飞线更会影响其信号质量，导致无法正常接收数据；
2. 确认sensor 驱动设置的mipi 格式，同样是查看sensor_g_mbus_config() 函数（lane 和通道数）；
3. 示波器测量mipi 接口的data 线、时钟线，看看有没有数据输出；
4. 检查一下寄存器配置方面有没有软件复位的，增加相应的延时；
5. 和模组厂商确认sensor 驱动中对应分辨率的sensor_win_sizes 以下参数配置是否与寄存器组配合的，因为这些参数将会影响mipi 接收数据；

.hts = 3550, .vts = 1126, .pclk = 120 * 1000 * 1000, .mipi_bps = 480 * 1000 * 1000,

上面的hts，又称line_length_pck，VTS 又称frame_length_lines，与寄存器的值要一致，Pclk(Pixel clock) 的值由PLL 寄存器计算得出，可简单计算，pclk = hts × 

vts × fps；而mipi_bps 为mipi 数据速率，mipi_bps = hts × vts × fps ×（12bit/10bit/8bit）/ lane。

有些sensor 的datasheet 没有标注hts 和vts 的，但是他们有H Blanking 和Vertical blanking，他们的转换公式是：

hts = H Blanking + output_width

vts = Vertical blanking + output_height

Output_width 就是输出的一行的大小，output_height 就是输出的一列的大小。

gc 厂的sensor，vts = VB + win_height + 16；VB 和win_height 都是可以从寄存器中获取得到的，注意，win_height 是寄存器值，而不是输出的高。



#### 6.4.3 其他注意事项

##### 6.4.3.1 R311、MR133

**sensor_sel**

在vin 框架中，sys_config.fex 有以下配置：

```
[vind0/sensor0]
...
[vind0/sensor1]
...
[vind0/vinc0]
vinc0_used = 1
vinc0_csi_sel = 0
vinc0_mipi_sel = 0
vinc0_isp_sel = 0
vinc0_rear_sensor_sel = 0
vinc0_front_sensor_sel = 0
vinc0_sensor_list = 0
```

在这里主要是需要注意vinc0_rear_sensor_sel 和vinc0_front_sensor_sel 的配置，当它们都配置为0，表明vind0/vinc0 配置的video0 节点，使用的是

vind0/sensor0 节点中配置的sensor 输出图像数据；当它们配置为0、1，表明vind0/vinc0 配置的video0 节点，可以使用vind0/sensor0 和vind0/sensor1 两个

sensor 输出图像数据，可以通过ioctl 的VIDIOC_S_INPUT 的index 选择使用哪个sensor 的输出；当它们都配置为1 的时候，表明vind0/vinc0 配置的video0 节

点，使用的是vind0/sensor1 的输出。



**mipi AB 配置**

mipi 配置方面，还有一个需要注意的，该部分在R311、MR133 平台才有这种情况。一般情况，我们使用的是MCSIB 组的mipi 接口，这个按照一般配置使用即

可，MCSIA、MCSIB，这两个会在原理图上表明使用的是哪一组接口，如果单独使用MCSIA 组的mipi 接口，在sys_config.fex 中配置如下：

```
由于只是使用MCSIA，所以应该配置的是[vind0/sensor1] 组sensor，vinc0_rear_sensor_sel 和
vinc0_front_sensor_sel 都配置为1.
[vind0/vinc0]
vinc0_used = 1
vinc0_csi_sel = 0
vinc0_mipi_sel = 0
vinc0_isp_sel = 0
vinc0_rear_sensor_sel = 1
vinc0_front_sensor_sel = 1
vinc0_sensor_list = 0
```

