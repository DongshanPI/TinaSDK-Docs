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

这个参数只有在lcd_if=0 时才有效。定义RGB 同步屏下的几种接口类型。

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

这个参数只有在lcd_if=0 时才有效。定义RGB 同步屏的clock 与data 之间的相位关系。总共有4 个相位可供调节。

设置相应值的对应含义为：

```
0: 0 degree
1: 90 degree
2: 180 degree
3: 270 degree
```

#### 5.1.6 lcd_hv_sync_polarity

Lcd HV panel Sync signals Polarity

这个参数只有在lcd_if=0 时才有效。定义RGB 同步屏的hsync 和vsync 的极性。

设置相应值的对应含义为：

```
0：vsync active low，hsync active low
1：vsync active high，hsync active low
2：vsync active low，hsync active high
3：vsync active high，hsync active high
```

#### 5.1.7 lcd_hv_srgb_seq

Lcd HV panel Serial RGB output Sequence

这个参数只有在lcd_if=0 且lcd_hv_if=8（Serial RGB）时才有效。

定义奇数行RGB 输出的顺序：

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

这个参数只有在lcd_if=0 且lcd_hv_if=12（Serial YUV）时才有效。

定义YUV 输出格式：

```
0：YUYV
1：YVYU
2：UYVY
3：VYUY
```

#### 5.1.9 lcd_hv_syuv_fdly

Lcd HV panel Serial YUV F line Delay

这个参数只有在lcd_if=0 且lcd_hv_if=12（Serial YUV）时才有效。

定义CCIR656 编码时F 相对有效行延迟的行数：

```
0：F toggle right after active video line
1：Delay 2 lines (CCIR PAL)
2：Delay 3 lines (CCIR NTSC)
```

#### 5.1.10 lcd_cpu_if

Lcd CPU panel Interface

这个参数只有在lcd_if=1 时才有效, 具体时序可参照RGB 和I8080 管脚配置示意图中CPU 那几列。

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

设置相应值的对应含义为，设置为0 时，刷屏间隔时间为lcd_ht × lcd_vt；设置为1 或2 时，刷屏间隔时间为两个te 脉冲：

Lcd CPU panel tear effect

设置相应值的对应含义为，设置为0 时，刷屏间隔时间为lcd_ht × lcd_vt；设置为1 或2 时，刷屏间隔时间为两个te 脉冲：

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

lcd_lvds_if 等于2 的场景是，接两个一模一样的屏，然后两个屏显示同样的内容，此时lcd 的其它timing 只需要填写一个屏的timing 即可。

#### 5.1.13 lcd_lvds_colordepth

Lcd LVDS panel color depth

设置相应值对应含义为：

```
0：8bit per color(4 data pair)
1：6bit per color(3 data pair)
```

5.1.14 lcd_lvds_mode

Lcd LVDS Mode

这个参数只有在lcd_lvds_bitwidth=0 时才有效。

设置相应值对应含义为(见下图）：

```
0：NS mode
1：JEIDA mode
```

![image-20221129173201413](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221129173201413.png)

<center>图5-1: lvds mode</center>

#### 5.1.15 lcd_dsi_if

Lcd MIPI DSI panel Interface

这个参数只有在lcd_if=4 时才有效。定义MIPI DSI 屏的两种类型。

设置相应值的对应含义为：

```
0：Video mode
1：Command mode
2：video burst mode
```

注：Video mode 的LCD 屏，是实时刷屏的，有ht，hbp 等时序参数的定义；Command mode 的屏，屏上带有显示Buffer，一般会有一个TE 引脚。

#### 5.1.16 lcd_dsi_lane

Lcd MIPI DSI panel Data Lane number

这个参数只有在lcd_if=4 时才有效。

设置相应值的对应含义为：

```
1：1 data lane
2：2 data lane
3：3 data lane
4：4 data lane
```

#### 5.1.17 lcd_dsi_format

Lcd MIPI DSI panel Data Pixel Format

这个参数只有在lcd_if=4 时才有效。

设置相应值的对应含义为：

```
0：Package Pixel Stream, 24bit RGB
1：Loosely Package Pixel Stream, 18bit RGB
2：Package Pixel Stream, 18bit RGB
3：Package Pixel Stream, 16bit RGB
```

#### 5.1.18 lcd_dsi_te

Lcd MIPI DSI panel Tear Effect

这个参数只有在lcd_if=4 时才有效。

设置相应值的对应含义为：

```
0：frame trigged automatically
1：frame trigged by te rising edge
2：frame trigged by te falling edge
```

注：设置为0 时，刷屏间隔时间为lcd_ht × lcd_vt；设置为1 或2 时，刷屏间隔时间为两个te脉冲。

这个的作用就是屏一端发给SoC 端的信号，用于同步信号，如果使能这个变量，那么SoC 内部的显示中断将由这个外部脚来触发。

#### 5.1.19 lcd_dsi_port_num

DSI 屏port 数量

这个参数只有在lcd_if=4 时才有效。

设置相应值的对应含义为：

```
0：一个port
1：两个port
```

这个选项的一个作用是，单LCD 屏为8 条lane 的时候，如果只需要初始化其中一个driver IC，则这个设置1，如果两个driver IC 都要初始化，这里设置成0。并使用lcd_source.c 定义的函数来进行初始化。

#### 5.1.20 lcd_tcon_mode

Tcon 模式

这个参数只有在lcd_if=4 时才有效。

设置相应值的对应含义为：

```
0：normal mode
1：tcon master mode（在第一次发送数据同步）
2:：tcon master mode（每一帧都同步）
3：tcon slave mode（依靠master mode来启动）
4：one tcon driver two dsi（8条lane）
5.1.21 lcd_slave_tcon_num
```

#### 5.1.21 lcd_slave_tcon_num

Slave Tcon 的序号

这个参数只有在lcd_if=4 时而且lcd_tcon_mode 等于1 或者2 才有效。用于告诉master 模式下的tcon，从tcon 的序号是多少。

设置相应值的对应含义为：

```
0：tcon_lcd0
1：tcon_lcd1
```

#### 5.1.22 lcd_tcon_en_odd_even_div

这个参数只有在lcd_if=4 而且lcd_tcon_mode=4 时才有效。

设置相应值的对应含义为：

```
0：tcon将一帧图像分左右两半来发送给两个DSI模块
1：tcon将一帧图像分奇偶像素来发给两个DSI模块
```

#### 5.1.23 lcd_sync_pixel_num

这个参数只有在lcd_if=4 而且lcd_tcon_mode 等于2 或者3 时才有效。

设置同步从tcon 的起始pixel。

整数：不超过lcd_ht

#### 5.1.24 lcd_sync_line_num

这个参数只有在lcd_if=4 而且lcd_tcon_mode 等于2 或者3 时才有效。

设置同步从tcon 的起始行。

整数：不超过lcd_vt

#### 5.1.25 lcd_cpu_mode

Lcd CPU 模式，控制。

设置相应值的对应含义为，设置为0 时，刷屏间隔时间为lcd_ht × lcd_vt；设置为1 或2 时，刷屏间隔时间为两个te 脉冲：

```
0：中断自动根据时序，由场消隐信号内部触发。
1：中断根据数据Block的counter触发或者由外部te触发。
```

#### 5.1.26 lcd_fsync_en

LCD 使能fsync 功能，用于触发sensor 出图, 目的是同步，部分IC 支持。

```
0：disable
1：enable
```



#### 5.1.27 lcd_fsync_act_time

LCD 的fsync 功能，其中的有效电平时间长度，单位：像素时钟的个数。

```
0~lcd_ht-1
```

#### 5.1.28 lcd_fsync_dis_time

LCD 的fsync 功能，其中的无效电平时间长度，单位：像素时钟的个数。

```
0~lcd_ht-1
```

#### 5.1.29 lcd_fsync_pol

LCD 的fsync 功能的有效电平的极性。

```
0：有效电平为低
1：有效电平为高
```

### 5.2 屏时序参数说明

下面几个参数对于调屏非常关键，决定了发送端（SoC）发送数据时序。由于涉及到发送端和接收端的调试，除了分辨率和尺寸之外，其它几个数值都不是绝对不

变的，两款一样分辨率，同种接口的屏，它们的数值也有可能不一样。

获取途径如下：

1. 询问LCD 屏厂。
2. 从屏手册或者Driver IC 手册中查找（向屏厂索要这些文档），如下图所示。

![image-20221129174730302](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221129174730302.png)

<center>图5-2: lcd_info1</center>

![image-20221129174744511](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221129174744511.png)

<center>图5-3: lcd_info2</center>

3. 在前面两步都搞不定的情况下，可以根据vesa 标准来设置，主要是DMT 和CVT 标准。

其中DMT，指的是《VESA and Industry Standards and Guidelines for Computer Display Monitor Timing(DMT)》，下载该标准，里面就有各种常用分辨率的

timing。其中的CVT，指的是《VESA Coordinated Video Timings(CVT) Standard》，该标准提供一种通用公式用于计算出指定分辨率，刷新率等参数的timing。

可以下载这个excel 表来计算VESA Coordinated Video Timing Generator。

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

指一行总的dclk 的cycle 个数。见下图：

![image-20221129175141962](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221129175141962.png)

<center>图5-4: lcdht</center>

#### 5.2.4 lcd_hbp

Horizontal Back Porch

指有效行间，行同步信号（hsync）开始，到有效数据开始之间的dclk 的cycle 个数，包括同步信号区。见上图，注意的是包含了hspw 段。

**说明**
是包含了hspw 段，也就是lcd_hbp= 实际的hbp+ 实际的hspw

5.2.5 lcd_hspw

Horizontal Sync Pulse Width

指行同步信号的宽度。单位为1 个dclk 的时间（即是1 个data cycle 的时间）。见上图。

#### 5.2.6 lcd_vt

Vertical Total time

指一场的总行数。见下图：

![image-20221129182148157](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221129182148157.png)

<center>图5-5: lcdvt</center>

#### 5.2.7 lcd_vbp

Vertical Back Porch

指场同步信号（vsync）开始，到有效数据行开始之间的行数，包括场同步信号区。

**说明**

是包含了vspw 段，也就是lcd_vbp= 实际的vbp+ 实际的vspw

#### 5.2.8 lcd_vspw

Vertical Sync Pulse Width

指场同步信号的宽度。单位为行。见上图。

#### 5.2.9 lcd_dclk_freq

Dot Clock Frequency

传输像素传送频率。单位为MHz。

fps = (lcd_dclk_freq×1000×1000) / (ht×vt)。

这个值根据以下公式计算：

lcd_dclk_freq=lcd_ht*lcd_vt*fps

注意：

1. 后面的三个参数都是从屏手册中获得，fps 一般是60。
2. 如果是串行接口，发完一个像素需要2 到3 个周期的，那么。

```
lcd_dclk_freq * cycles = lcd_ht*lcd_vt*fps
```

或者

```
lcd_dclk_freq = lcd_ht*cycles*lcd_vt*fps
```

#### 5.2.10 lcd_width

Width of lcd panel in mm

此参数描述lcd 屏幕的物理宽度，单位是mm。用于计算dpi。

#### 5.2.11 lcd_height

height of lcd panel in mm

此参数描述lcd 屏幕的物理高度，单位是mm。用于计算dpi。

### 5.3 背光相关参数

目前用得比较广泛的就是pwm 背光调节，原理是利用pwm 脉冲开关产生的高频率闪烁效应，通过调节占空比，达到欺骗人眼，调节亮暗的目的。

#### 5.3.1 lcd_pwm_used

是否使用pwm。

此参数标识是否使用pwm 用以背光亮度的控制。

#### 5.3.2 lcd_pwm_ch

Pwm channel used

此参数标识使用的Pwm 通道，这里是指使用SoC 哪个pwm 通道，通过查看原理图连接可知。

#### 5.3.3 lcd_pwm_freq

Lcd backlight PWM Frequency

这个参数配置PWM 信号的频率，单位为Hz。

**说明**
频率不宜过低否则很容易就会看到闪烁，频率不宜过快否则背光调节效果差。部分屏手册会标明所允许的pwm 频率范围，请遵循屏手册固定范围进行设置。
在低亮度的时候容易看到闪烁，是正常现象，目前已知用上pwm 的背光都是如此。

#### 5.3.4 lcd_pwm_pol

Lcd backlight PWM Polarity

这个参数配置PWM 信号的占空比的极性。设置相应值对应含义为：

```
0：active high
1：active low
```

#### 5.3.5 lcd_pwm_max_limit

Lcd backlight PWM 最高限制，以亮度值表示

比如150，则表示背光最高只能调到150，0-255 范围内的亮度值将会被线性映射到0-150 范围内。用于控制最高背光亮度，节省功耗。

#### 5.3.6 lcd_bl_en

背光使能脚，非必须，看原理图是否有，用于使能或者禁止背光电路的电压。

示例：lcd_bl_en = port:PD24<1><2><default><1>

含义：PD24 输出高电平时打开LCD 背光；下拉，默认高电平

• 第一个尖括号：功能分配；1 为输出；

• 第二个尖括号：内置电阻；使用0 的话，标示内部电阻高阻态，如果是1 则是内部电阻上拉，2 就代表内部电阻下拉。使用default 的话代表默认状态，即电阻上

拉。其它数据无效。

• 第三个尖括号：驱动能力；default 表驱动能力是等级1

• 第四个尖括号：电平；0 为低电平，1 为高电平。

需要在屏驱动调用相应的接口进行开、关的控制。

**说明**

一般来说，高电平是使能，在这个前提下，建议将内阻电阻设置成下拉，防止硬件原因造成的上拉，导致背光提前亮。默认电平请填写高电平，这是uboot 显示过

度到内核显示，平滑无闪烁的需要。

#### 5.3.7 lcd_bl_n_percent

背光映射值，n 为(0-100)

此功能是针对亮度非线性的LCD 屏的，按照配置的亮度曲线方式来调整亮度变化，以使亮度变化更线性。

比如lcd_bl_50_percent = 60，表明将50% 的亮度值调整成60%，即亮度比原来提高10%。

**说明**

修改此属性不当可能导致背光调节效果差。

#### 5.3.8 lcd_backlight

背光默认值，0-255。

此属性决定在uboot 显示logo 阶段的亮度，进入都内核时则是读取保存的配置来决定亮度。

**说明**

显示logo 阶段，一般来说需要比较亮的亮度，业内做法都是如此。

### 5.4 显示效果相关参数

#### 5.4.1 lcd_frm

Lcd Frame Rate Modulator

FRM 是解决由于PIN 减少导致的色深问题。

这个参数设置相应值对应含义为：

```
0：RGB888 -- RGB888 direct
1：RGB888 -- RGB666 dither
2：RGB888 -- RGB565 dither
```

有些LCD 屏的像素格式是18bit 色深（RGB666）或16bit 色深（RGB565），建议打开FRM功能，通过dither 的方式弥补色深，使显示达到24bit 色深（RGB888）

的效果。如下图所示，上图是色深为RGB66 的LCD 屏显示，下图是打开dither 后的显示，打开dither 后色彩渐变的地方过度平滑。

![image-20221130172026584](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221130172026584.png)

<center>图5-6: lcd_frm 打开</center>

![image-20221130172041420](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221130172041420.png)

<center>图5-7: lcd_frm 关闭</center>

#### 5.4.2 lcd_gamma_en

Lcd Gamma Correction Enable

设置相应值的对应含义为：

```
0：Lcd的Gamma校正功能关闭
1：Lcd的Gamma校正功能开启
```

设置为1 时，需要在屏驱动中对lcd_gamma_tbl[256] 进行赋值。

#### 5.4.3 lcd_cmap_en

Lcd Color Map Enable

设置相应值的对应含义为：

```
0：Lcd的色彩映射功能关闭
1：Lcd的色彩映射功能开启
```

设置为1 时，需要对lcd_cmap_tbl [2][3][4] 进行赋值Lcd Color Map Table。

每个像素有R、G、B 三个单元，每四个像素组成一个选择项，总共有12 个可选。数组第一维表示奇偶行，第二维表示像素的RGB，第三维表示第几个像素，数组

的内容即表示该位置映射到的内容。

LCD CMAP 是对像素的映射输出功能，只有像素有特殊排布的LCD 屏才需要配置。

LCD CMAP 定义每行的4 个像素为一个总单元，每个像素分R、G、B 3 个小单元，总共有12个小单元。通过lcd_cmap_tbl 定义映射关系，输出的每个小单元可随

意映射到12 个小单元之一。

```
__u32 lcd_cmap_tbl[2][3][4] = {
    {
        {LCD_CMAP_G0,LCD_CMAP_B1,LCD_CMAP_G2,LCD_CMAP_B3},
        {LCD_CMAP_B0,LCD_CMAP_R1,LCD_CMAP_B2,LCD_CMAP_R3},
        {LCD_CMAP_R0,LCD_CMAP_G1,LCD_CMAP_R2,LCD_CMAP_G3},
    },
    {
        {LCD_CMAP_B3,LCD_CMAP_G2,LCD_CMAP_B1,LCD_CMAP_G0},
        {LCD_CMAP_R3,LCD_CMAP_B2,LCD_CMAP_R1,LCD_CMAP_B0},
        {LCD_CMAP_G3,LCD_CMAP_R2,LCD_CMAP_G1,LCD_CMAP_R0},
    },
};
```

如上，上三行代表奇数行的像素排布，下三行代表偶数行的像素排布。

每四个像素为一个单元，第一列代表每四个像素的第一个像素映射，第二列代表每四个像素的第二个像素映射，以此类推。

如上的定义，像素的输出格式如下图所示。

![image-20221130172721739](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221130172721739.png)

<center>图5-8: cmap</center>

#### 5.4.4 lcd_rb_swap

调换tcon 模块RGB 中的R 分量和B 分量。

```
0：不变
1：调换R分量和B分量
```

### 5.5 电源和管脚参数

#### 5.5.1 概述

如果需要使用某路电源必须现在[disp]节点中定义，然后[lcd]部分使用的字符串则要和disp 中定义的一致。比如下面的例子：

```
disp: disp@01000000 {
    disp_init_enable = <1>;
    disp_mode = <0>;
    /* VCC-LCD */
    dc1sw-supply = <&reg_sw>;
    /* VCC-LVDS and VCC-HDMI */
    bldo1-supply = <&reg_bldo1>;
    /* VCC-TV */
    cldo4-supply = <&reg_cldo4>;
}；
```

其中-supply是固定的，在-supply之前的字符串则是随意的，不过建议取有意义的名字。在=后面的像<&reg_sw>则必须在board.dtsi 的regulator0节点中找到。

然后lcd0 节点中，如果要使用reg_sw，则像下面这样写就行，dc1sw 对应dc1sw-supply。

```
lcd_power=”dc1sw”
```

由于u-boot 中也有axp 驱动和display 驱动，和内核，它们都是读取同份配置，为了能互相兼容，取名的时候，有以下限制：

在u-boot 2018 中，axp 驱动只认类似bldo1 这样从axp 芯片中定义的名字，所以命名xxxsupply的时候最好按照这个axp 芯片的定义来命名。

#### 5.5.2 lcd_power

见上面概述的注意事项。

```
示例：lcd_power = “vcc-lcd”
```

配置regulator 的名字。配置好之后，需要在屏驱动调用相应的接口进行开、关的控制。

注意：如果有多个电源需要打开，则定义lcd_power1，lcd_power2 等。

#### 5.5.3 lcd_pin_power

用法lcd_power一致，区别是用户设置之后，不需要在屏驱动中去操作，而是驱动框架自行在屏驱动之前使能，在屏驱动之后禁止。

```
示例：lcd_pin_power = “vcc-pd”
```

注意：如果需要多组，则添加lcd_pin_power1，lcd_pin_power2 等。除了lcddx 之外，这里的电源还有可能是pwm 所对应管脚的电源。

#### 5.5.4 lcd_gpio_0

```
示例：lcd_gpio_0 = port:PD25<0><0><default><0>
```

含义：lcd_gpio_0 引脚为PD25。

• 第一个尖括号：功能分配；0 为输入，1 为输出。

• 第二个尖括号：内置电阻；使用0 的话，标示内部电阻高阻态，如果是1 则是内部电阻上拉，2 就代表内部电阻下拉。使用default 的话代表默认状态，即电阻上

拉。其它数据无效。

• 第三个尖括号：驱动能力；default 表驱动能力是等级1。

• 第四个尖括号：表示默认值；即是当设置为输出时，该引脚输出的电平，0 为低电平，1 为高电平。

需要在屏驱动调用相应的接口进行拉高，拉低的控制。请看管脚控制函数说明

注意：如果有多个gpio 脚需要控制，则定义lcd_gpio_0，lcd_gpio_1 等。

#### 5.5.5 lcddx

示例：lcdd0 = port:PD00<3><0><default><default>

含义：lcdd0 这个引脚，即是PD0，配置为LVDS 输出。

• 第一个尖括号：功能分配；0 为输入，1 为输出，2 为LCD 输出，3 为LVDS 接口输出，7 为disable。

• 第二个尖括号：内置电阻；使用0 的话，标示内部电阻高阻态，如果是1 则是内部电阻上拉，2 就代表内部电阻下拉。使用default 的话代表默认状态，即电阻上

拉。其它数据无效。

• 第三个尖括号：驱动能力；default 表驱动能力是等级1。

• 第四个尖括号：表示默认值；即是当设置为输出时，该引脚输出的电平，0 为低电平，1 为高电平。

LCD PIN 的配置如下：

LCD 为HV RGB 屏，CPU/I80 屏时，必须定义相应的IO 口为LCD 输出（如果是0 路输出，第一个尖括号为2；如果是1 路输出，第一个尖括号为3）。

具体的IO 对应关系可参考user manual 手册进行配置。

LCD PIN 的所有IO，均可通过注释方式去掉其定义，显示驱动对注释IO 不进行初始化操作。

需要在屏驱动调用相应的接口进行开、关的控制。

注意：不是一定要叫做lcdd0 这个名字，你改成其它名字对驱动不会造成任何影响，这里只是为了方便记忆。

#### 5.5.6 pinctrl-0 和pinctrl-1

在配置lcd0节点时，当碰到需要配置管脚复用时，你只要把pinctrl-0和pinctrl-1赋值好就行，可以用提前定义好的，也可以用自己定义的，提前定义的管脚一般可

以在内核目录下arch/arm/boot/dts或者arch/arm64/boot/dts下找：平台-pinctrl.dtsi 中找到定义。

例子:

```
pinctrl-0 = <&rgb24_pins_a>;
pinctrl-1 = <&rgb24_pins_b>;//休眠时候的定义，io_disable
```

<center>表5-1: 提前定义好的管脚名称</center>

| 管脚名称                                      | 描述                                        |
| --------------------------------------------- | ------------------------------------------- |
| rgb24_pins_a 和rgb24_pins_b                   | RGB 屏接口，而且数据位宽是24，RGB888        |
| rgb18_pins_a 和rgb18_pins_b                   | RGB 屏接口，而且数据位宽是16，RGB666        |
| lvds0_pins_a 和lvds0_pins_b                   | Single link LVDS 接口0 管脚定义（主显lcd0） |
| lvds1_pins_a 和lvds1_pins_b                   | Single link LVDS 接口1 管脚定义（主显lcd0） |
| lvds2link_pins_a 和lvds2link_pins_b           | Dual link LVDS 接口管脚定义（主显lcd0）     |
| lvds2_pins_a 和lvds2_pins_b                   | Single link LVDS 接口0 管脚定义（主显lcd1） |
| lvds3_pins_a 和lvds3_pins_b                   | Single link LVDS 接口1 管脚定义（主显lcd1） |
| lcd1_lvds2link_pins_a 和lcd1_lvds2link_pins_b | Dual link LVDS 接口管脚定义（主显lcd1）     |
| dsi4lane_pins_a 和dsi4lane_pins_b             | 4 lane DSI 屏接口管脚定义                   |

自定义一组脚

写在board.dtsi 中，只要名字不要和现有名字重复就行，首先判断自己需要用的管脚，属于大cpu 域还是小cpu 域，以此判断需要将管脚定义放在pio（大cpu 

域）下面还是r_pio（小cpu域）下面。

例子：

```
&pio {
    I8080_8bit_pins_a: I8080_8bit@0 {
        allwinner,pins = "PD1", "PD2", "PD3", "PD4", "PD5", "PD6", "PD7", "PD8", "PD18", "
        	PD19", "PD20", "PD21";
        allwinner,pname = "PD1", "PD2", "PD3", "PD4", "PD5", "PD6", "PD7", "PD8", "PD18", "
            PD19", "PD20", "PD21";
            allwinner,function = "I8080_8bit";
            allwinner,muxsel = <2>;
            allwinner,drive = <3>;
            allwinner,pull = <0>;
        };
        I8080_8bit_pins_b: I8080_8bit@1 {
        allwinner,pins = "PD1", "PD2", "PD3", "PD4", "PD5", "PD6", "PD7", "PD8", "PD18", "
        	PD19", "PD20", "PD21";
        allwinner,pname = "PD1", "PD2", "PD3", "PD4", "PD5", "PD6", "PD7", "PD8", "PD18", "
        	PD19", "PD20", "PD21";
            allwinner,function = "I8080_8bit_suspend";
            allwinner,muxsel = <7>;
            allwinner,drive = <3>;
            allwinner,pull = <0>;
    };
};
```

• pins，具体管脚。

• pname，管脚名称，随便取。

• function，管脚功能名称，随便取。

• muxsel，管脚功能选择。根据port spec 来选择对应功能。

• drive，驱动能力，数值越大驱动能力越大。

• pull，上下拉，使用0 的话，标示内部电阻高阻态，如果是1 则是内部电阻上拉，2 就代表内部电阻下拉。使用default 的话代表默认状态，即电阻上拉。其它数

据无效。

为了规范，我们将在所有平台保持一致的名字，其中后缀为a 为管脚使能，b 的为io_disable 用于设备关闭时。

有时候，你需要用两组不同功能的管脚，可以像下面这样定义即可。

```
pinctrl-0 = <&rgb24_pins_a>, <&xxx_pins_a>;
pinctrl-1 = <&rgb24_pins_b>, <&xxx_pins_b>;//休眠时候的定义，io_disable
```

5.6 ESD 静电检测自动恢复功能

这个功能在linux4.9 以及linux 3.10 sunxi-product 分支上实现了，如果需要这个功能，需要完成以下步骤。

首先打开如下内核配置：

![image-20221130174114112](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221130174114112.png)

<center>图5-9: ESD 内核配置</center>

修改屏驱动，实现三个回调函数：

如下示例，在屏he0801a068 上添加esd 相关的回调函数。
（linux-4.9/drivers/video/fbdev/sunxi/disp2/disp/lcd/he0801a068.c）。

![image-20221130174231109](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221130174231109.png)

<center>图5-10: ESD 屏驱动添加函数</center>

esd_check 函数原型：

```
S32 esd_check(u32 sel)
```

作用：是给上层反馈当前屏的状态。

返回值：如果屏正常的话就返回0，不正常的话就返回非0。

sel：显示索引。

由于屏的类型接口众多，不同屏检测屏的状态各异，一般来说是通过驱动接口读取屏的内部信息（id 或者其它寄存器），如果获取正常则认为屏是正常的，获取失

败则认为屏是异常的。比如下面dsi 屏的做法：

![image-20221130174303084](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221130174303084.png)

<center>图5-11: ESD 屏驱动函数实现</center>

此外，一般情况下，也会通过dsi 接口读取0x0A 命令（获取power 模式）来判断屏是否正常。

```
sunxi_lcd_dsi_dcs_read(sel, 0x0A, result, &num)
```

![image-20221130174341400](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221130174341400.png)

<center>图5-12: ESD MIPI 状态寄存器</center>

reset_panel 函数原型：

```
s32 reset_panel(u32 sel)
```

作用：当屏幕异常的时候所需要的复位操作。

返回值：复位成功就是0，复位失败非0。

sel：显示索引。

每个屏的初始化都不同，顺序步骤都不一样，总的来说就是执行部分或者完整的屏驱动里面的close_flow 和open_flow 所定义的回调函数。根据实际情况灵活编写

这个函数。

值得注意的是：某些dsi 屏中，需要至少执行过一次sunxi_lcd_dsi_clk_disable（dsi 高速时钟禁止）和sunxi_lcd_dsi_clk_enable（高速时钟使能），否则可能导致

dsi 的读函数异常。

下图是复位函数示例：

![image-20221130174745808](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221130174745808.png)

<center>图5-13: ESD 复位函数1</center>

set_esd_info 函数原型：

```
s32 set_esd_info(struct disp_lcd_esd_info *p_info)
```

作用：控制esd 检测的具体行为。比如间隔多长时间检测一次，复位的级别，以及检测函数被调用的位置。

返回值：成功设置返回0，否则非0。

p_info：需要设置的esd 行为结构体。

示例：下面图所示，每隔60 次显示中断检测一次（调用esd_check 函数，如果显示帧率是60fps 的话，那么就是1 秒一次），然后将在显示中断处理函数里面执行

检测函数，由esd_check_func_pos 成员决定调用esd_check 函数的位置，如果是0 则在中断之外执行检测函数，之所以有这个选项是因为显示中断资源（中断处

理时间）是非常珍贵的资源，关系到显示帧率的问题。下图中的level 为1 表示复位全志SoC 的LCD 相关模块以及reset_panel 里面的操作，level 为0 的时候表示

仅仅执行reset_panel 里面的操作。

![image-20221130174832573](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221130174832573.png)

<center>图5-14: ESD 设置信息函数</center>

可以通过cat /sys/class/disp/disp/attr/sys 获取当前的esd info。

```
screen 0:
de_rate 594000000 hz, ref_fps:60
mgr0: 2560x1600 fmt[rgb] cs[0x204] range[full] eotf[0x4] bits[8bits] unblank err[0]
force_sync[0]
dmabuf: cache[0] cache max[0] umap skip[0] overflow[0]
capture: dis req[0] runing[0] done[0,0]
lcd output(enable) backlight( 50) fps:60.9 esd level(1) freq(300) pos(1)
reset(244) 2560x1600
err:0 skip:0 skip T.O:50 irq:73424 vsync:0 vsync_skip:0
BUF en ch[1] lyr[0] z[0] prem[N] fbd[N] a[globl 255] fmt[ 0] fb
[2560,1600;2560,1600;2560,1600] crop[ 0, 0,2560,1600] frame[ 0, 0,2560,1600]
addr[98100000,00000000,00000000] right[00000000,00000000,00000000] flags[0x00] trd[0,0]
depth[ 0]
acquire: 0, 25.5 fps
release: 0, 25.5 fps
display: 0, 25.5 fps
```

esd level(1) freq(300) pos(1) reset(244)

esd levele 和freq 和pos 的意思请看上面set_esd_info 函数原型的解释。

Reset 后面的数字表示屏复位的次数（也就是esd 导致屏挂掉之后，并且成功检测到并复位的次数）。

此功能可能遇到的问题。

1. 打静电挂了，但是读出来的值仍然是正确的，此问题无解。

2. Dsi 读操作卡住了，卡在中断里面了。此问题可能和DSI 的lp 模式下的速率有关系，而lp 的速率又和dclk 的频率有关系。此时可以尝试修改de_dsi_28.c() 文件

  中的dsi_basic_cfg 函数，如下图所示红框所示的寄存器值，这个寄存器是Lp 模式时钟分频值，一般来说这个值越小，lp 速率越快，尝试改小看是否还会卡

  住。

![image-20221130175036108](https://photos.100ask.net/Tina-Sdk/Linux_LCD_DevGuide_image-20221130175036108.png)

<center>图5-15: Lp 模式时钟分频值</center>
