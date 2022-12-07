# LCD-硬件参数说明
## 5 硬件参数说明

### 5.1 LCD 接口参数说明

#### 5.1.1 lcd_driver_name

Lcd 屏驱动的名字（字符串），必须与屏驱动的名字对应。



#### 5.1.2 lcd_model_name

Lcd 屏模型名字，非必须，可以用于同个屏驱动中进一步区分不同屏。



#### 5.1.3 lcd_if

Lcd Interface

设置相应值的对应含义为：

```
0：HV RGB接口
1：CPU/I80接口
2：Reserved
3：LVDS接口
4：DSI接口
```



#### 5.1.4 lcd_hv_if

Lcd HV panel Interface

这个参数只有在 lcd_if=0 时才有效。定义 RGB 同步屏下的几种接口类型。

设置相应值的对应含义为：

```
0：Parallel RGB
8：Serial RGB
10：Dummy RGB
11：RGB Dummy
12：Serial YUV (CCIR656)
```



#### 5.1.5 lcd_hv_clk_phase

Lcd HV panel Clock Phase

这个参数只有在 lcd_if=0 时才有效。定义 RGB 同步屏的 clock 与 data 之间的相位关系。总共有 4 个相位可供调节。

设置相应值的对应含义为：

```
0: 0 degree
1: 90 degree
2: 180 degree
3: 270 degree
```



#### 5.1.6 lcd_hv_sync_polarity

Lcd HV panel Sync signals Polarity

这个参数只有在 lcd_if=0 时才有效。定义 RGB 同步屏的 hsync 和 vsync 的极性。

设置相应值的对应含义为：

```
0：vsync active low，hsync active low
1：vsync active high，hsync active low
2：vsync active low，hsync active high
3：vsync active high，hsync active high
```



#### 5.1.7 lcd_hv_srgb_seq

Lcd HV panel Serial RGB output Sequence

这个参数只有在 lcd_if=0 且 lcd_hv_if=8（Serial RGB）时才有效。

定义奇数行 RGB 输出的顺序：

```
0: Odd lines R-G-B; Even line R-G-B
1: Odd lines B-R-G; Even line R-G-B
2: Odd lines G-B-R; Even line R-G-B
4: Odd lines R-G-B; Even line B-R-G
5: Odd lines B-R-G; Even line B-R-G
6: Odd lines G-B-R; Even line B-R-G
8: Odd lines R-G-B; Even line B-R-G
9: Odd lines B-R-G; Even line G-B-R
10: Odd lines G-B-R; Even line G-B-R
```



#### 5.1.8 lcd_hv_syuv_seq

Lcd HV panel Serial YUV output Sequence

这个参数只有在 lcd_if=0 且 lcd_hv_if=12（Serial YUV）时才有效。

定义 YUV 输出格式：

```
0：YUYV
1：YVYU
2：UYVY
3：VYUY
```



#### 5.1.9 lcd_hv_syuv_fdly

Lcd HV panel Serial YUV F line Delay

这个参数只有在 lcd_if=0 且 lcd_hv_if=12（Serial YUV）时才有效。

定义 CCIR656 编码时 F 相对有效行延迟的行数：

```
0：F toggle right after active video line
1：Delay 2 lines (CCIR PAL)
2：Delay 3 lines (CCIR NTSC)
```



#### 5.1.10 lcd_cpu_if

Lcd CPU panel Interface

这个参数只有在 lcd_if=1 时才有效, 具体时序可参照**RGB 和 I8080 管脚配置示意**图中 CPU 那几列。

设置相应值的对应含义为：

```
0：18bit/1cycle (RGB666)
2: 16bit/3cycle (RGB666)
4：16bit/2cycle (RGB666)
6：16bit/2cycle (RGB666)
8：16bit/1cycle (RGB565)
10：9bit/1cycle (RGB666)
12：8bit/3cycle (RGB666)
14：8bit/2cycle (RGB565)
```



#### 5.1.11 lcd_cpu_te

Lcd CPU panel tear effect

设置相应值的对应含义为，设置为 0 时，刷屏间隔时间为 lcd_ht × lcd_vt；设置为 1 或 2 时，屏间隔时间为两个 te 脉冲：

```
0：frame trigged automatically
1：frame trigged by te rising edge
2：frame trigged by te falling edge
```



#### 5.1.12 lcd_lvds_if

Lcd LVDS panel Interface

设置相应值的对应含义为：

```
0：Single Link( 1 clock pair+3/4 data pair)
1：Dual Link(8 data lane，每4条lane接受一半像素，奇数像素或者偶数像素)
2: Dual Link (每4条lane接受全部像素，常用于物理双屏，且两个屏一样）
```

lcd_lvds_if 等于 2 的场景是，接两个一模一样的屏，然后两个屏显示同样的内容，此时 lcd 的其它 timing 只需要填写一个屏的 timing 即可。



#### 5.1.13 lcd_lvds_colordepth

Lcd LVDS panel color depth

设置相应值对应含义为：

```
0：8bit per color(4 data pair)
1：6bit per color(3 data pair)
```



#### 5.1.14 lcd_lvds_mode

Lcd LVDS Mode

这个参数只有在 lcd_lvds_bitwidth=0 时才有效

设置相应值对应含义为 (见下图）：

```
0：NS mode
1：JEDIA mode
```





​																图 5-1: lvds mode jedia





​																图 5-2: lvds mode ns



#### 5.1.15 lcd_dsi_if

Lcd MIPI DSI panel Interface

这个参数只有在 lcd_if=4 时才有效。定义 MIPI DSI 屏的两种类型。

设置相应值的对应含义为：

```
0：Video mode
1：Command mode
2：video burst mode
```

注：Video mode 的 LCD 屏，是实时刷屏的，有 ht，hbp 等时序参数的定义；Command mode 的屏，屏上带有显示 Buffer，一般会有一个 TE 引脚



#### 5.1.16 lcd_dsi_lane

Lcd MIPI DSI panel Data Lane number

这个参数只有在 lcd_if=4 时才有效。

设置相应值的对应含义为：

```
1：1 data lane
2：2 data lane
3：3 data lane
4：4 data lane
```



#### 5.1.17 lcd_dsi_format

Lcd MIPI DSI panel Data Pixel Format

这个参数只有在 lcd_if=4 时才有效。

设置相应值的对应含义为：

```
0：Package Pixel Stream, 24bit RGB
1：Loosely Package Pixel Stream, 18bit RGB
2：Package Pixel Stream, 18bit RGB
3：Package Pixel Stream, 16bit RGB
```



#### 5.1.18 lcd_dsi_te

Lcd MIPI DSI panel Tear Effect

这个参数只有在 lcd_if=4 时才有效。

设置相应值的对应含义为：

```
0：frame trigged automatically
1：frame trigged by te rising edge
2：frame trigged by te falling edge
```

注：设置为 0 时，刷屏间隔时间为 lcd_ht × lcd_vt；设置为 1 或 2 时，刷屏间隔时间为两个 te 脉冲。

这个的作用就是屏一端发给 SoC 端的信号，用于同步信号，如果使能这个变量，那么 SoC 内部的显示中断将由这个外部脚来触发。



#### 5.1.19 lcd_dsi_port_num

DSI 屏 port 数量

这个参数只有在 lcd_if=4 时才有效。

设置相应值的对应含义为：

```
0：一个port
1：两个port
```



#### 5.1.20 lcd_tcon_mode

Tcon 模式

这个参数只有在 lcd_if=4 时才有效。

设置相应值的对应含义为：

```
0：normal mode
1：tcon master mode（在第一次发送数据同步）
2: tcon master mode（每一帧都同步）
3：tcon slave mode（依靠master mode来启动）
4：one tcon driver two dsi（8条lane）
```



#### 5.1.21 lcd_slave_tcon_num

Slave Tcon 的序号

这个参数只有在 lcd_if=4 时而且 lcd_tcon_mode 等于 1 或者 2 才有效。用于告诉 master 模式下的 tcon，从 tcon 的序号是多少。

设置相应值的对应含义为：

```
0：tcon_lcd0
1：tcon_lcd1
```



#### 5.1.22 lcd_tcon_en_odd_even_div

这个参数只有在 lcd_if=4 而且 lcd_tcon_mode=4 时才有效。

设置相应值的对应含义为：

```
0：tcon将一帧图像分左右两半来发送给两个DSI模块
1：tcon将一帧图像分奇偶像素来发给两个DSI模块
```



#### 5.1.23 lcd_sync_pixel_num

这个参数只有在 lcd_if=4 而且 lcd_tcon_mode 等于 2 或者 3 时才有效。

设置同步从 tcon 的起始 pixel

```
整数：不超过lcd_ht
```



#### 5.1.25 lcd_cpu_mode

Lcd CPU 模式，控制

设置相应值的对应含义为，设置为 0 时，刷屏间隔时间为 lcd_ht × lcd_vt；设置为 1 或 2 时，刷屏间隔时间为两个 te 脉冲：

```
0：中断自动根据时序，由场消隐信号内部触发
1：中断根据数据Block的counter触发或者由外部te触发。
```



#### 5.1.26 lcd_fsync_en

LCD 使能 fsync 功能，用于触发 sensor 出图, 目的是同步，部分 IC 支持。

```
0：disable
1：enable
```



#### 5.1.27 lcd_fsync_act_time

LCD 的 fsync 功能，其中的有效电平时间长度，单位：像素时钟的个数

```
0~lcd_ht-1
```



#### 5.1.28 lcd_fsync_dis_time

LCD 的 fsync 功能，其中的无效电平时间长度，单位：像素时钟的个数

```
0~lcd_ht-1
```



#### 5.1.29 lcd_fsync_pol

LCD 的 fsync 功能的有效电平的极性。

```
0：有效电平为低
1：有效电平为高
```



### 5.2 屏时序参数说明

下面几个参数对于调屏非常关键，决定了发送端（SoC）发送数据时序。由于涉及到发送端和接收端的调试，除了分辨率和尺寸之外，其它几个数值都不是绝对不变的，两款一样分辨率，同种接口的屏，它们的数值也有可能不一样。

获取途径如下：

1. 询问 LCD 屏厂。

2. 从屏手册或者 Driver IC 手册中查找（向屏厂索要这些文档），如下图所示。



​																图 5-3: lcd_info1



​																图 5-4: lcd_info2

3. 在前面两步都搞不定的情况下，可以根据 vesa 标准来设置，主要是 DMT 和 CVT 标准。

   其中 DMT，指的是《VESA and Industry Standards and Guidelines for Computer Display Monitor Timing(DMT)》，下载该标准，里面就有各种常用分辨率的 timing。

   其中的 CVT，指的是《VESA Coordinated Video Timings(CVT) Standard》，该标准提供一种通用公式用于计算出指定分辨率，刷新率等参数的 timing。

   可以下载这个 excel 表来计算VESA Coordinated Video Timing Generator。

由下面两条公式得知，我们不需要设置lcd_hfp和lcd_vfp参数，因为驱动会自动根据其它几个已知参数中算出lcd_hfp和lcd_vfp。

```
lcd_ht = lcd_x + lcd_hspw + lcd_hbp + lcd_hfp
lcd_vt = lcd_y + lcd_vspw + lcd_vbp + lcd_vfp
```



#### 5.2.1 lcd_x

显示屏的水平像素数量，也就是屏分辨率中的宽。



#### 5.2.2 lcd_y

显示屏的垂直行数，也就是屏分辨率中的高。



#### 5.2.3 lcd_ht

Horizontal Total time

指一行总的 dclk 的 cycle 个数。见下图：



​															图 5-5: lcdht



#### 5.2.4 lcd_hbp

Horizontal Back Porch

指有效行间，行同步信号（hsync）开始，到有效数据开始之间的 dclk 的 cycle 个数，包括同步信号区。见上图，注意的是包含了 hspw 段。

说明

**是包含了** **hspw** **段，也就是**

**lcd_hbp=** **实际的** **hbp+** **实际的** **hspw**



#### 5.2.5 lcd_hspw

Horizontal Sync Pulse Width

指行同步信号的宽度。单位为 1 个 dclk 的时间（即是 1 个 data cycle 的时间）。见上图。



#### 5.2.6 lcd_vt

Vertical Total time

指一场的总行数。见下图：





​															       图 5-6: lcdvt



#### 5.2.7 lcd_vbp

Vertical Back Porch

指场同步信号（vsync）开始，到有效数据行开始之间的行数，包括场同步信号区。

说明

**是包含了** **vspw** **段，也就是**

**lcd_vbp=** **实际的** **vbp+** **实际的** **vspw**

​											

#### 5.2.8 lcd_vspw

Vertical Sync Pulse Width

指场同步信号的宽度。单位为行。见上图。



#### 5.2.9 lcd_dclk_freq

Dot Clock Frequency

传输像素传送频率。单位为 MHz

fps = (lcd_dclk_freq×1000×1000) / (ht×vt)。

这个值根据以下公式计算：

```
lcd_dclk_freq=lcd_ht*lcd_vt*fps
```

注意：

1. 后面的三个参数都是从屏手册中获得，fps 一般是 60

2. 如果是串行接口，发完一个像素需要 2 到 3 个周期的，那么

   ```
   lcd_dclk_freq*cycles = lcd_ht*lcd_vt*fps
   ```

   或者

   ```
   lcd_dclk_freq = lcd_ht*cycles*lcd_vt*fps
   ```

   

#### 5.2.10 lcd_width

Width of lcd panel in mm

此参数描述 lcd 屏幕的物理宽度，单位是 mm。用于计算 dpi



#### 5.2.11 lcd_height

height of lcd panel in mm

此参数描述 lcd 屏幕的物理高度，单位是 mm。用于计算 dpi





### 5.3 背光相关参数

目前用得比较广泛的就是 pwm 背光调节，原理是利用 pwm 脉冲开关产生的高频率闪烁效应，通过调节占空比，达到欺骗人眼，调节亮暗的目的。



#### 5.3.1 lcd_pwm_used

是否使用 pwm

此参数标识是否使用 pwm 用以背光亮度的控制。



#### 5.3.2 lcd_pwm_ch

Pwm channel used

此参数标识使用的 Pwm 通道，这里是指使用 SoC 哪个 pwm 通道，通过查看原理图连接可知。



#### 5.3.3 lcd_pwm_freq

Lcd backlight PWM Frequency

这个参数配置 PWM 信号的频率，单位为 Hz。

说明

**频率不宜过低否则很容易就会看到闪烁，频率不宜过快否则背光调节效果差。部分屏手册会标明所允许的 pwm 频率范围，请遵循屏手册固定范围进行设置。**

**在低亮度的时候容易看到闪烁，是正常现象，目前已知用上** **pwm** **的背光都是如此。**



#### 5.3.4 lcd_pwm_pol

Lcd backlight PWM Polarity

这个参数配置 PWM 信号的占空比的极性。设置相应值对应含义为：

```
0：active high
1：active low
```



#### 5.3.5 lcd_pwm_max_limit

Lcd backlight PWM 最高限制，以亮度值表示

比如 150，则表示背光最高只能调到 150，0~255 范围内的亮度值将会被线性映射到 0~150 范围内。用于控制最高背光亮度，节省功耗。



#### 5.3.6 lcd_bl_en

背光使能脚，非必须，看原理图是否有，用于使能或者禁止背光电路的电压。

```
示例：lcd_bl_en = port:PD24<1><2><default><1>
```

含义：PD24 输出高电平时打开 LCD 背光；下拉，默认高电平

*•* 第一个尖括号：功能分配；1 为输出；

*•* 第二个尖括号：内置电阻；使用 0 的话，标示内部电阻高阻态，如果是 1 则是内部电阻上拉，

  2 就代表内部电阻下拉。使用 default 的话代表默认状态，即电阻上拉。其它数据无效。

*•* 第三个尖括号：驱动能力；default 表驱动能力是等级 1 

*•* 第四个尖括号：电平；0 为低电平，1 为高电平。

需要在屏驱动调用相应的接口进行开、关的控制。

说明

**一般来说，高电平是使能，在这个前提下，建议将内阻电阻设置成下拉，防止硬件原因造成的上拉，导致背光提前亮。默认电平请填写高电平，这是** uboot **显示过度到内核显示，平滑无闪烁的需要。**



#### 5.3.7 lcd_bl_n_percent

背光映射值，n 为 (0-100)

此功能是针对亮度非线性的 LCD 屏的，按照配置的亮度曲线方式来调整亮度变化，以使亮度变化更线性。

比如 lcd_bl_50_percent = 60，表明将 50% 的亮度值调整成 60%，即亮度比原来提高 10%。

说明

**修改此属性不当可能导致背光调节效果差。**



#### 5.3.8 lcd_backlight

背光默认值，0-255。

此属性决定在 uboot 显示 logo 阶段的亮度，进入都内核时则是读取保存的配置来决定亮度。

说明

**显示** **logo** **阶段，一般来说需要比较亮的亮度，业内做法都是如此。**



### 5.4 显示效果相关参数

#### 5.4.1 lcd_frm

Lcd Frame Rate Modulator

FRM 是解决由于 PIN 减少导致的色深问题。

这个参数设置相应值对应含义为：

```
0：RGB888 -- RGB888 direct
1：RGB888 -- RGB666 dither
2：RGB888 -- RGB565 dither
```

有些 LCD 屏的像素格式是 18bit 色深（RGB666）或 16bit 色深（RGB565），建议打开 FRM 功能，通过 dither 的方式弥补色深，使显示达到 24bit 色深（RGB888）的效果。如下图所示，上图是色深为 RGB66 的 LCD 屏显示，下图是打开 dither 后的显示，打开 dither 后色彩渐变的地方过度平滑。



​																	图 5-7: good



​																	图 5-8: bad



#### 5.4.2 lcd_gamma_en

Lcd Gamma Correction Enable

设置相应值的对应含义为：

0：Lcd的Gamma校正功能关闭

1：Lcd的Gamma校正功能开启

设置为 1 时，需要在屏驱动中对 lcd_gamma_tbl[256] 进行赋值。
