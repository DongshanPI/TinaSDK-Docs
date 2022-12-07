# LCD-模块介绍


## 4 模块介绍

### 4.1 添加屏驱动步骤

1.对于 linux4.9 及以下版本总共需要修改三处地方（即下列前三项），对于 linux5.4 则需要修改四处地方，具体可参考**屏驱动源码位置**。 

*•* linux 源码仓库。

*•* uboot 源码仓库。在 uboot 中也有显示和屏驱动，目的是显示 logo。 

*•* 板级 dts 配置仓库。目的是通过 board.dts 来配置一些通用的 LCD 配置参数。对于linux4.9，该配置同时对内核及 uboot 生效，对于 linux-5.4，请参照下条。

*•* 对于 linux5.4，还需额外配置 uboot 专用板级 dts 配置仓库。



2.确保全志显示框架的内核配置有使能，查看menuconfig 配置说明

3.前期准备以下资料和信息：

- 屏手册。主要是描述屏基本信息和电气特性等，向屏厂索要。

- Driver IC 手册。主要是描述屏 IC 的详细信息。这里主要是对各个命令进行详解，对我们进行初始化定制有用，向屏厂索要。

- 屏时序信息。请向屏厂索要。请看**屏时序参数说明** 以了解更多信息。

- 屏初始化代码，请向屏厂索要。一般情况下 DSI 和 I8080 屏等都需要初始化命令对屏进行初始化。

- 万用表。调屏避免不了测量相关电压。



4.动手添加屏驱动之前，先了解屏驱动，请看**屏驱动分解**。

5.通过第 3 步的资料，定位该屏的类型，然后选择一个已有同样类型的屏驱动作为模板进行屏驱动添加或者直接在上面修改。

6.修改屏驱动目录下的panel.c和panel.h。在全局结构体变量panel_array中新增刚才添加strcut \__lcd_panel的变量指针。panel.h中新增strcut __lcd_panel的声明。

7.修改 Makefile。在 lcd 屏驱动目录的上一级的 Makefile 文件中的disp-objs中新增刚才添加屏驱动.o

8.修改 board.dts 中的 lcd0。可以看**RGB 接口，MIPI-DSI 接口，I8080 接口和LVDS 接口**，里面有介绍各种接口典型配置。**硬件参数说明**，这一章有所有 lcd0 节点下可配置属性详细解释。

9.编译 uboot，kernel，打包烧写。注意不同 SDK，编译方式有所不同，部分 SDK 默认不编译 uboot。

10.调试。通过**一些有用调试手段**我们可以初步定位问题，还有**FAQ**，对调屏也有帮助。



### 4.2 屏驱动说明

#### 4.2.1 屏驱动源码位置

**linux 3.4** **版本内核：**

linux3-4/drivers/video/sunxi/disp2/disp/lcd/

**linux 3.10** **版本内核：**

linux3-10/drivers/video/sunxi/disp2/disp/lcd/

**linux 4.9** **版本及其以上内核：**

linux-4.9/drivers/video/fbdev/sunxi/disp2/disp/lcd/

**uboot-2014:**

brandy/u-boot-2014.07/drivers/video/sunxi/disp2/disp/lcd

**uboot-2018:**

brandy/brandy-2.0/u-boot-2018/drivers/video/sunxi/disp2/disp/lcd

**板级配置**，其中 “芯片型号” 比如 T509，和 “板子名称” 比如 demo，请根据实际替换。

device/config/chips/芯片型号/configs/板子名称/

**针对** **linux5.4** **时使用的** **uboot** **板级配置：**

brandy/brandy-2.0/u-boot-2018/arch/arm/dts/芯片型号-板子名称-board.dts





#### 4.2.2 menuconfig 配置说明

lcd 相关代码包含在disp驱动模块中，在命令行中进入内核根目录，执行`make ARCH=arm menuconfig`或者`make ARCH=arm64 menuconfig`(64bit 平台) 进入配置主界面，其中。并按以下步骤操作：

具体配置目录为：

3.4 内核和 3.10 内核：

```
Device Drivers->Graphics support->Support for frame buffer devices->
Video Support for sunxi -> DISP Driver Support(sunxi-disp2)
```

如果是 linux-4.9 及其以上版本的内核路径是：

```
Device Drivers->Graphics support->Framebuffer Devices >
Video Support for sunxi -> DISP Driver Support(sunxi-disp2)
```





​                                                                      图 4-1: menuconfig



#### 4.2.3 屏驱动分解

在屏驱动源码位置中，主要分为四类文件

1. panel.c和panel.h，当用户添加新屏驱动时，是需要修改这两个文件的，需要将屏结构体变量添加到全局结构体变量panel_array中。

2. lcd_source.c和lcd_source.h，这两个文件实现的是给屏驱动使用的函数接口，比如电源开关，gpio，dsi 读写接口等，用户不需要修改只需要用。

3. 屏驱动。除了上面提到的源文件外，其它的一般一个 c 文件和一个 h 文件就代表一个屏驱动。

4. 在屏驱动源码位置的上一级，有用户需要修改的 Makefile 文件。

我们可以打开drivers/video/fbdev/sunxi/disp2/disp/lcd/default_panel.c作为屏驱动的例子，在该文件的最后

```
struct __lcd_panel default_panel = {
    /* panel driver name, must mach the lcd_drv_name in board.dts */
    .name = "default_lcd",
    .func = {
        .cfg_panel_info = LCD_cfg_panel_info,
        .cfg_open_flow = LCD_open_flow,
        .cfg_close_flow = LCD_close_flow,
    } ,
};
```

该全局变量default_panel的成员name与**lcd_driver_name**必须一致，这个关系到驱动能否找到指定的文件。

接下来是func成员的初始化，这里最主要实现三个回调函数。LCD_cfg_panel_info，LCD_open_flow和LCD_close_flow。

开关屏流程即屏上下电流程，屏手册或者 driver IC 手册中里面的 Power on Sequence 和 Power off Sequence，用于开关屏的操作流程如下图所示。

其中，LCD_open_flow 和 LCD_close_flow 称为开关屏流程函数，方框中的函数，如LCD_power_on，称为开关屏步骤函数。

不需要进行初始化操作的 LCD 屏，比如 lvds 屏，RGB 屏等，LCD_panel_init 及 LCD_panel_exit这函数可以空。







​																图 4-2: LCD 开关屏流程

函数：LCD_open_flow

功能：LCD_open_flow 函数只会系统初始化的时候调用一次，执行每个 LCD_OPEN_FUNC即是把对应的开屏步骤函数进行注册，先注册先执行，但并没有立刻执行该开屏步骤函数。

原型：

```
static __s32 LCD_open_flow(__u32 sel)
```

函数常用内容为：

```c
static __s32 LCD_open_flow(__u32 sel)
{
    LCD_OPEN_FUNC(sel, LCD_power_on, 10);
    LCD_OPEN_FUNC(sel, LCD_panel_init, 50);
    LCD_OPEN_FUNC(sel, sunxi_lcd_tcon_enable, 100);
    LCD_OPEN_FUNC(sel, LCD_bl_open, 0);
    return 0;
}
```

如上，调用四次 LCD_OPEN_FUNC 注册了四个回调函数，对应了四个开屏流程, 先注册先执行。实际上注册多少个函数是用户自己的自由，只要合理即可。

1. LCD_power_on 即打开 LCD 电源，再延迟 10ms；这个步骤一般用于打开 LCD 相关电源和相关管脚比如复位脚。这里一般是使用**电源控制函数说明**和**管脚控制函数说明**进行操作。

2. LCD_panel_init 即初始化屏，再延迟 50ms；不需要初始化的屏，可省掉此步骤，这个函数一般用于发送初始化命令给屏进行屏初始化。如果是 DSI 屏看**DSI 相关函数说明**，如果是 I8080 屏用 **I8080 接口函数说明**，如果是其它情况比如 i2c 或者 spi 可以看**使用 iic/spi 串行接口初始化**，也可以用 GPIO 来进行模拟。

3. sunxi_lcd_tcon_enable 打开 TCON，再延迟 100ms；这一步是固定的，表示开始发送图像信号。

4. LCD_bl_open 打开背光，再延迟 0ms。前面三步搞定之后才开背光，这样不会看到闪烁。这里一般使用的函数请看**背光控制函数说明**。



如下图，这是屏手册中典型的上电时序图，我们编写屏驱动的时候，也要注意，该延时就得延时。





​																	图 4-3: power on



函数：LCD_OPEN_FUNC

功能：注册开屏步骤函数到开屏流程中，记住这里是注册不是执行！

原型：

```
void LCD_OPEN_FUNC(__u32 sel, LCD_FUNC func, __u32 delay)
```

参数说明：

func 是一个函数指针，其类型是：void (*LCD_FUNC) (__u32 sel)，用户自己定义的函数必须也要用统一的形式。比如：

```
void user_defined_func(__u32 sel)
{
	//do something
}
```

delay 是执行该步骤后，再延迟的时间，时间单位是毫秒。

LCD_OPEN_FUNC 的第二个参数是前后两个步骤的延时长度，单位 ms，注意这里的数值请按照屏手册规定去填，乱填可能导致屏初始化异常或者开关屏时间过长，影响用户体验。

与 LCD_open_flow 对应的是 LCD_close_flow 是，用于注册关屏函数，使用 LCD_CLOSE_FUNC 进行函数注册，先注册先执行，这里只是注册回调函数不是立刻执行。

```
static s32 LCD_close_flow(u32 sel)
{
    /* close lcd backlight, and delay 0ms */
    LCD_CLOSE_FUNC(sel, LCD_bl_close, 0);
    /* close lcd controller, and delay 0ms */
    LCD_CLOSE_FUNC(sel, sunxi_lcd_tcon_disable, 50);
    /* open lcd power, than delay 200ms */
    LCD_CLOSE_FUNC(sel, LCD_panel_exit, 100);
    /* close lcd power, and delay 500ms */
    LCD_CLOSE_FUNC(sel, LCD_power_off, 0);
    
    return 0;
}
```

1. 先关闭背光，这样整个关屏过程，用户不会看到闪烁的过程。

2. 关闭 TCON，也就是停止发送数据，这是必要的。再延迟 50ms；

3. 执行关屏代码，再延迟 200ms；（不需要初始化的屏，可省掉此步骤）

4. 最后关闭电源，再延迟 0ms。



如下图是典型关屏时序图。





​													    	图 4-4: power off



函数：LCD_cfg_panel_info

功能：配置的 TCON 扩展参数，比如 gamma 功能和颜色映射功能。

原型：

```
static void LCD_cfg_panel_info(__panel_extend_para_t * info)
```

TCON 的扩展参数只能在屏文件中配置，参数的定义见**显示效果相关参数**。

需要 gamma 校正，或色彩映射，在 board.dts 中将相应模块的 enable 参数置 1，lcd_gamma_en,

lcd_cmap_en，并且填充 3 个系数表，lcd_gamma_tbl, lcd_cmap_tbl，如下所示红色代码部分。注意的是：gamma，模板提供了 18 段拐点值，然后再插值出所有的值（255 个）。如果觉得还不细，可以往相应表格里添加子项。cmap_tbl 的大小是固定了，不能减小或增加表的大小。

最终生成的 gamma 表项是由 rgb 三个 gamma 值组成的，各占 8bit，目前提供的模板中，三个 gamma 值是相同的。



#### 4.2.4 延时函数说明

函数：**sunxi_lcd_delay_ms/sunxi_lcd_delay_us**

功能：延时函数，分别是毫秒级别/微秒级别的延时

原型：s32 sunxi_lcd_delay_ms(u32 ms); / s32 sunxi_lcd_delay_us(u32 us);



#### 4.2.5 图像数据使能函数说明

函数：**sunxi_lcd_tcon_enable /sunxi_lcd_tcon_disable**

功能：打开 LCD 控制器，开始刷新 LCD 显示。关闭 LCD 控制器，停止刷新数据。

原型：void sunxi_lcd_tcon_enable(u32 screen_id);/ void sunxi_lcd_tcon_disable(u32 screen_id);



#### 4.2.6 背光控制函数说明

函数：**sunxi_lcd_backlight_enable/ sunxi_lcd_backlight_disable**

功能：打开/关闭背光，操作的是 board.dts 中 lcd_bl 配置的 gpio。见 5.4.2 lcd_bl_en

原型：void sunxi_lcd_backlight_enable(u32 screen_id);

​		   void sunxi_lcd_backlight_disable(u32 screen_id);

函数：**sunxi_lcd_pwm_enable / sunxi_lcd_pwm_disable**

功能：打开/关闭 pwm 控制器，打开时 pwm 将往外输出 pwm 波形。对应的是 lcd_pwm_ch 所对应的那一路 pwm

原型：s32 sunxi_lcd_pwm_enable(u32 screen_id);

​            s32 sunxi_lcd_pwm_disable(u32 screen_id);



#### 4.2.7 电源控制函数说明

函数：**sunxi_lcd_power_enable / sunxi_lcd_power_disable**

功能：打开/关闭 Lcd 电源，操作的是 board.dts 中的 lcd_power/lcd_power1/lcd_power2。（pwr_id

标识电源索引）

原型：void sunxi_lcd_power_enable(u32 screen_id, u32 pwr_id);

​		   void sunxi_lcd_power_disable(u32 screen_id, u32 pwr_id);

1. pwr_id = 0：对应于 board.dts 中的 lcd_power

2. pwr_id = 1：对应于 board.dts 中的 lcd_power1

3. pwr_id = 2：对应于 board.dts 中的 lcd_power2

4. pwr_id = 3：对应于 board.dts 中的 lcd_power3

   

函数：**sunxi_lcd_pin_cfg**

功能：配置 lcd 的 io。

原型：s32 sunxi_lcd_pin_cfg(u32 screen_id, u32 bon);

说明：配置 lcd 的 data/clk 等 pin，对应 board.dts 中的 lcdd0-lcdd23/lcddclk/lcdde/lcdhsync/lcdvsync。

由于 dsi 是专用 pin, 所以 dsi 接口屏不需要在 board.dts 中配置这组 pin，但同样会在此函数接口中打开与关闭对应的 pin。

Bon: 1: 为开，0：为配置成 disable 状态。



#### 4.2.8 DSI 相关函数说明

MIPI DSI 屏，大部分需要初始化，使用的是 DSI-D0 通道的 LP 模式进行初始化。提供的接口函数说明如下：



函数：**sunxi_lcd_dsi_clk_enable / sunxi_lcd_dsi_clk_disble**

功能：仅限 dsi 接口屏使用，使能/关闭 dsi 输出的高速时钟 clk 信号，必须在初始化的时候调用。

原型：s32 sunxi_lcd_dsi_clk_enable(u32 scree_id);

​            s32 sunxi_lcd_dsi_clk_disable(u32 scree_id);



函数：**sunxi_lcd_dsi_dcs_wr**

功能：对屏的 dcs 写操作

原型：\__s32 sunxi_lcd_dsi_dcs_wr(\_\_u32 sel,\_\_u8 cmd,\_\_u8* para_p,__u32 para_num);

参数说明：

*•* cmd：dcs 写命令内容

*•* para_p：dcs 写命令的参数起始地址

*•* para_num：dcs 写命令的参数个数，单位为 byte



函数：**sunxi_lcd_dsi_dcs_wr_2para**

功能：对屏的 dcs 写操作，该命令带有两个参数

原型：\_\_s32 sunxi_lcd_dsi_dcs_wr_2para(\_\_u32 sel,\__u8 cmd,\_\_u8 para1,__u8 para2);

参数说明：

*•* cmd：dcs 写命令内容

*•* para1：dcs 写命令的第一个参数内容

*•* para2：dcs 写命令的第二个参数内容

sunxi_dsi_dcs_wr_0para，sunxi_dsi_dcs_wr_1para，sunxi_dsi_dcs_wr_3para，sunxi_dsi_dcs_wr_4para，

sunxi_dsi_dcs_wr_5para定义与dsi_dcs_wr_2para类似，差别就是参数数量。



函数：**sunxi_lcd_dsi_dcs_read**

功能：dsi 读操作。

原型：s32 sunxi_lcd_dsi_dcs_read(u32 sel, u8 cmd, u8\* result, u32* num_p);

参数说明：

*•* sel,  显示 id。 

*•* cmd,  要读取的寄存器

*•* result，用于存放读取接口的数组，用户必须自行保证其有足够空间保存读取的接口

*•* num_p，指针用于存放读取字节数，用户必须保证其非空指针。



#### 4.2.9 I8080 接口函数说明

显示驱动提供 5 个接口函数可供使用。如下：

函数：**sunxi_lcd_cpu_write**

功能：设定 CPU 屏的指定寄存器为指定的值

原型：void sunxi_lcd_cpu_write(\__u32 sel, \_\_u32 index, __u32 data)

函数内容为

```
void sunxi_lcd_cpu_write(__u32 sel, __u32 index, __u32 data)
{
    sunxi_lcd_cpu_write_index(sel, index);
    sunxi_lcd_cpu_wirte_data(sel, data);
}
```

实现了 8080 总线上的两个写操作。

sunxi_lcd_cpu_write_index 实现第一个写操作，这时 PIN 脚 RS（A1）为低电平，总线数据上的数据内容为参数 index 的值。

Sunxi_lcd_cpu_wirte_data 实现第二个写操作，这时 PIN 脚 RS（A1）为高电平，总线数据上的数据内容为参数 data 的值。



函数：**sunxi_lcd_cpu_write_index**

功能：设定 CPU 屏为指定寄存器

原型：

void sunxi_lcd_cpu_write_index(\__u32 sel,__u32 index);

具体说明见 sunxi_lcd_cpu_write。



函数：**sunxi_lcd_cpu_write_data**

功能：设定 CPU 屏寄存器的值为指定的值

原型：

void sunxi_lcd_cpu_write_data(\__u32 sel, __u32 data);



函数: tcon0_cpu_rd_24b_data

功能：读操作

原型：

s32 tcon0_cpu_rd_24b_data(u32 sel, u32 index, u32 *data, u32 size)

参数说明：

*•* sel：显示 id

*•* index: 要读取的寄存器

*•* data：用于存放读取接口的数组指针，用户必须保证其有足够空间存放数据

*•* size：要读取的字节数。



#### 4.2.10 管脚控制函数说明

函数：**sunxi_lcd_gpio_set_value**

功能：LCD_GPIO PIN 脚上输出高电平或低电平

原型：s32 sunxi_lcd_gpio_set_value(u32 screen_id, u32 io_index, u32 value);

参数说明：

*•* io_index = 0：对应于 board.dts 中的 lcd_gpio_0

*•* io_index = 1：对应于 board.dts 中的 lcd_gpio_1

*•* io_index = 2：对应于 board.dts 中的 lcd_gpio_2

*•* io_index = 3：对应于 board.dts 中的 lcd_gpio_3

*•* value = 0：对应 IO 输出低电平

*•* Value = 1：对应 IO 输出高电平

只用于该 GPIO 定义为输出的情形。



函数：**sunxi_lcd_gpio_set_direction**

功能：设置 LCD_GPIO PIN 脚为输入或输出模式

原型：

s32 sunxi_lcd_gpio_set_direction(u32 screen_id, u32 io_index, u32 direction);

参数说明：

*•* io_index = 0：对应于 board.dts 中的 lcd_gpio_0

*•* io_index = 1：对应于 board.dts 中的 lcd_gpio_1

*•* io_index = 2：对应于 board.dts 中的 lcd_gpio_2

*•* io_index = 3：对应于 board.dts 中的 lcd_gpio_3

*•* direction = 0：对应 IO 设置为输入

*•* direction = 1：对应 IO 设置为输出

一部分屏需要进行初始化操作，在开屏步骤函数中，对应于 LCD_panel_init 函数，提供了几种方式对屏的初始化。

对于 DSI 屏，是通过 DSI-D0 通道进行初始化。对于 CPU 屏，是通过 8080 总线的方式，使用的是 LCDIO（PD,PH）进行初始化。这种初始化方式，其总线的引脚位置定义与 CPU 屏一致。以下这些接口在 3.1 中提到路径的 lcd_source.c 和 lcd_source.h 中定义和实现。





#### 4.2.11 使用 iic/spi 串行接口初始化

需要在屏驱动中注册 iic/spi 设备对串行接口的访问。

使用硬件 spi 对屏或者转接 IC 进行初始化，如下代码片段。

首先调用 spi_init 函数对 spi 硬件进行初始化，spi_init 函数可以分为几个步骤，第一获取 master；根据实际的硬件连接，选择 spi（代码中选择了 spi1），如果这一步返回错误说 spi 没有配置好，找 spi 驱动负责人。第二步设置 spi device，这里包括最大速度，spi 传输模式，以及每个字包含的比特数。最后调用 spi_setup 完成 master 和 device 的关联。

comm_out 是一个 spi 传输的例子，核心就是 spi_sync_transfer 函数。

```
static int spi_init(void)
{
    int ret = -1;
    struct spi_master *master;

	master = spi_busnum_to_master(1);
    if (!master) {
        lcd_fb_wrn("fail to get master\n");
        goto OUT
    }

    spi_device = spi_alloc_device(master);
    if (!spi_device) {
        lcd_fb_wrn("fail to get spi device\n");
        goto OUT;
    }
    
    spi_device->bits_per_word = 8;
    spi_device->max_speed_hz = 60000000; /*50MHz*/
    spi_device->mode = SPI_MODE_0;
    
    ret = spi_setup(spi_device);
    if (ret) {
        lcd_fb_wrn("Faile to setup spi\n");
        goto FREE;
    }

    lcd_fb_inf("Init spi1:bits_per_word:%d max_speed_hz:%d mode:%d\n",
        spi_device->bits_per_word, spi_device->max_speed_hz,
        spi_device->mode);

    ret = 0;
    goto OUT;
    
FREE:
    spi_master_put(master);
    kfree(spi_device);
    spi_device = NULL;

OUT:
    return ret;
}
static int comm_out(unsigned int sel, unsigned char cmd)
{
    struct spi_transfer t;
    if (!spi_device)
        return -1;
    DC(sel, 0);
    memset(&t, 0, sizeof(struct spi_transfer));
    t.tx_buf = &cmd;
    t.len = 1;
    t.bits_per_word = 8;
    t.speed_hz = 24000000;
    return spi_sync_transfer(spi_device, &t, 1);
}
```

使用硬件 i2c 对 LCD& 转接 IC 进行初始化，初始化 i2c 硬件的核心函数是 i2c_add_driver，而你要做的是初始化好其参数 struct i2c_driver。

it66121_id 包含设备名字以及 i2c 总线索引（i2c0，i2c1…）

it66121_i2c_probe 能进到这个函数，你就可以开始使用 i2c 了。代码段里面仅仅将后面需要的参数 cilent 赋值给一个全局指针变量。

it66121_match，这是 dts 的 match table，由于你是给 disp2 加驱动，所以这里的 match table 就是 disp2 的 match table，这个 table 关系到能否使用 i2c，可别填错了。

tv_i2c_detect 函数，这里是非常关键的，这个函数早于 probe 函数被调用，只有成功被调用后才能开始使用 i2c，其中 strlcpy 的调用意味着成功。

normal_i2c 是从设备地址列表，填写的 LCD 或者转接 IC 的从设备地址以及 i2c 索引。

以 probe 函数是否被调用来决定你是否可以开始使用 I2C。 

用 i2c_smbus_write_byte_data 或者 i2c_smbus_read_byte_data 来读写可以满足大部分场景。

```
#define IT66121_SLAVE_ADDR 0x4c
#define IT66121_I2C_ID 0

static const struct i2c_device_id it66121_id[] = {
    { "IT66121", IT66121_I2C_ID },
    { /* END OF LIST */ }
};
MODULE_DEVICE_TABLE(i2c, it66121_id);
static int it66121_i2c_probe(struct i2c_client *client, const struct i2c_device_id *id)
{
    this_client = client;
    return 0;
}

static const struct of_device_id it66121_match[] = {
    {.compatible = "allwinner,sun8iw10p1-disp",},
    {.compatible = "allwinner,sun50i-disp",},
    {.compatible = "allwinner,sunxi-disp",},
    {},
};

static int tv_i2c_detect(struct i2c_client *client, struct i2c_board_info *info)
{
    const char *type_name = "IT66121";
    if (IT66121_I2C_ID == client->adapter->nr) {
    	strlcpy(info->type, type_name, 20);
    } else
   		pr_warn("%s:%d wrong i2c id:%d, expect id is :%d\n", __func__, __LINE__,
    				client->adapter->nr, IT66121_I2C_ID);
    return 0;
}

static unsigned short normal_i2c[] = {IT66121_SLAVE_ADDR, I2C_CLIENT_END};

static struct i2c_driver it66121_i2c_driver = {
    .class = I2C_CLASS_HWMON,
    .id_table = it66121_id,
    .probe = it66121_i2c_probe,
    .remove = it66121_i2c_remove,
    .driver = {
        .owner = THIS_MODULE,
        .name = "IT66121",
        .of_match_table = it66121_match,
    },
    .detect = tv_i2c_detect,
    .address_list = normal_i2c,
};
static void LCD_panel_init(u32 sel)
{
    int ret = -1;
    
    ret = i2c_add_driver(&it66121_i2c_driver);
    if (ret) {
        pr_warn("Add it66121_i2c_driver fail!\n");
        return;
	}

//start init chip with i2c
}
void it6612_twi_write_byte(it6612_reg_set* reg)
{
    u8 rdata = 0;
    u8 tmp = 0;
    
    rdata = i2c_smbus_read_byte_data(this_client, reg->offset);
    tmp = (rdata & (~reg->mask))|(reg->mask&reg->value);
    i2c_smbus_write_byte_data(this_client, reg->offset, tmp);
}
```



#### 4.2.12 U-boot 屏驱动注意事项

U-boot 编写屏驱动的步骤和内核是一样的，代码路径文件组织方式都是一样的，这里要讲的是需要注意的事项。

1. 为了加快 U-boot 的显示速度，开屏的几个函数之间采取异步调用的方式，原理是利用 timer中断，定时调用开屏函数，所以这种情况下 bootGUI 框架加载完毕并不意味着开屏完成，而是当你见到LCD open finish的打印的时候。

建议：为了尽量利用异步调用的优点，请把需要的延时尽量在注册回调的时候指定，比如下面延时 10ms 就是利用 timer 异步来进行回调的，这 10ms 时间，uboot 就可以做其它事情，以达到异步调用的目的。

```
LCD_OPEN_FUNC(sel, LCD_power_on,10);
```

2. sunxi_lcd_power_enable 函数和 sunxi_lcd_pin_cfg 不能在LCD_power_on之外调用，否则uboot 会异常.

严格讲，只能在用LCD_OPEN_FUNC注册的回调第一个函数里面调用。





### 4.3 RGB 接口

#### 4.3.1 概述

下面介绍全志平台的 RGB 以及配置示例，至于 lcd0 下面每个属性的详解细节请看**硬件参数说明**。

RGB 接口在全志平台又称 HV 接口（Horizontal 同步和 Vertical 同步）。



对于 RGB 屏的初始化:

有些 LCD 屏支持高级的功能比如 gamma，像素格式的设置等，但是 RGB 协议本身不支持图像数据之外的传输，所以无法通过 RGB 管脚进行对 LCD 屏进行配置，所以拿到一款 RGB 接口屏，要么不需要初始化命令，要么这个屏会提供额外的管脚给 SoC 来进行配置，比如 SPI 和 I2C等。



#### 4.3.2 RGB 接口管脚



​																	图 4-5: RGB 管脚

上面这些脚具体到 SoC 哪根管脚以及第几个功能（管脚复用功能）请参考 pin mux 表格，管脚复用功能的名字一般以 “LCDX_” 开头，其中 X 是数字。

其中数据脚的数量不一定是 24 根。RGB 又细分几种接口，通过设置 **lcd_hv_if** 来选择。

​																	表 4-1: RGB 接口分类

| 位宽    | 时钟周期数 | 颜色数量和格式       |
| ------- | ---------- | -------------------- |
| 24 bits | 1 cycle    | 16.7M colors, RGB888 |
| 18 bits | 1 cycle    | 262K colors, RGB666  |
| 16 bits | 1 cycle    | 65K colors, RGB565   |
| 6 bits  | 3 cycles   | 262K colors, RGB666  |
| 6 bits  | 3 cycles   | 65K colors, RGB565   |



说明

**时钟周期数的意思：是一个像素需要用多少个时钟周期发送完毕的意思。**

**当时钟周期为 1 时，我们称这种 RGB 接口为并行接口，其它的情况则是串行接口, 更为普遍的原则就是只要需要多个时钟周期才能发送完一个像素的接口都是串行接口。**

**如何判断是否支持** **24bit** **的位宽，最简单的方式就是在** **pinmux** **表格中数一数数据脚的数量，如果有** **24** **根则支持** **24bit，如果只有 18 根则支持 18bit**。



**硬件连接**

对于并行 RGB 的接口，当位宽小于 24 时，硬件连接应该选择连接每个分量中的高位而放弃低位，这样做的原因是损失较少的颜色数量。

对于串行 RGB 接口，硬件连接可参考**RGB 和 I8080 管脚配置示意图**中 sync RGB 那几列。

RGB 接口有两种同步方式，根据经验来说尽量使用第二种方式，硬件上请保证连接好 DE 脚。

1. Hsync+Vsync

2. DE（Data Enable）



#### 4.3.3 并行 RGB 接口配置示例

当我们配置并行 RGB 接口时，在配置里面并不需要区分是 24 位，18 位和 16 位，最大位宽是哪种是参考 pin mux 表格，如果 LCD 屏本身支持的位宽比 SoC 支持的位宽少，当然只能选择少的一方。

因为不需要初始化，RGB 接口极少出现问题，重点关注 lcd 的 timing 的合理性，也就是**lcd_ht**，**lcd_hspw**，**lcd_hbp**，**lcd_vt**，**lcd_vspw** 和 **lcd_vbp** 这个属性的合理性。

**下面是典型并行** **RGB** **接口** **board.dts** **配置示例，其中用空行把配置分成几个部分**

1. 第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏驱动来初始化，这里是 default_lcd，是针对不需要初始化设置的 RGB 屏

2. 第二部分决定下面的配置是一个并行 RGB 的配置。

3. 第三部分决定了 SoC 中的 LCD 模块发送时序，请查看**屏时序参数说明**。

4. 第四部分决定了背光（pwm 和 lcd_bl_en）。请看**背光相关参数**。

5. 第五部分是显示效果部分的配置，如果非 24 位的 RGB，那么一般情况下需要设置**lcd_frm**。

6. 第六部分就是电源和管脚配置。是用 RGB666 还是 RGB888，需要根据实际 pinmux 表来决定，如果该芯片只有 18 根 rgb 数据则只能 rgb18。请看**电源和管脚参数**。

```
&lcd0 {
    /* part 1 */
    lcd_used = <1>;
    lcd_driver_name = "default_lcd";
    
    /* part 2 */
    lcd_if = <0>;
    lcd_hv_if = <0>;
    
    /* part 3 */
    lcd_width = <150>;
    lcd_height = <94>;
    lcd_x = <800>;
    lcd_y = <480>;
    lcd_dclk_freq = <33>;
    lcd_hbp = <46>;
    lcd_ht = <1055>;
    lcd_hspw = <0>;
    lcd_vbp = <23>;
    lcd_vt = <525>;
    lcd_vspw = <0>;
    
    /* part 4 */
    lcd_backlight = <50>;
    lcd_pwm_used = <1>;
    lcd_pwm_ch = <8>;
    lcd_pwm_freq = <10000>;
    lcd_pwm_pol = <1>;
    lcd_bl_en = <&pio PD 27 1 0 3 1>;
    lcd_bright_curve_en = <0>;
    
    /* part 5 */
    lcd_frm = <0>;
    lcd_io_phase = <0x0000>;
    lcd_gamma_en = <0>;
    lcd_cmap_en = <0>;
    lcd_hv_clk_phase = <0>;
    lcd_hv_sync_polarity= <0>;
    
    /* part 6 */
    lcd_power = "vcc-lcd";
    lcd_pin_power = "vcc-pd";
    pinctrl-0 = <&rgb24_pins_a>;
    pinctrl-1 = <&rgb24_pins_b>;
};
```



#### 4.3.4 串行 RGB 接口的典型配置

串行 RGB 是相对于并行 RGB 来说，而并不是说它只用一根线来发数据，只要通过多个时钟周期才能把一个像素的数据发完，那么这样的 RGB 接口就是串行 RGB。

同样与并行 RGB 接口一样，配置中并不需要也无法体现具体是哪种串行 RGB 接口，你要做的就是把硬件连接对就行。

**下面是典型串行** **RGB** **接口** **board.dts** **配置示例，它只有** **8** **根数据脚，其中用空行把配置分成几个部分**

1. 第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏驱动来初始化。

2. 第二分部决定下面的配置是一个串行 RGB 的配置。

3. 第三部分决定了 SoC 中的 LCD 模块发送时序，请查看屏时序参数说明。



技巧

这里需要注意的是，对于该接口，*SoC* 总共需要三个周期才能发完一个 *pixel*，所以我们配置时序的时候，需要满足*lcd_dclk_freq\*3=lcd_ht\*lcd_vt\*60*，或者*lcd_dclk_freq=lcd_ht\*3*\*lcd_vt\*60 要么 *3* 倍*lcd_ht*要么 *3* 倍*lcd_dclk_freq*。

4. 第四部分决定了背光。就是 pwm 和 lcd_bl_en。请看**背光相关参数**

5. 第五部分是显示效果方面的设置。

6. 第六部分管脚和电源的定义。请看**电源和管脚参数**。

说明

**下面实例的 lcd driver IC 是 stv7789v，是需要初始化，初始化的接口协议是  SPI，所以这多了几根 spi管脚配置，驱动里面用 gpio 模拟 spi 协议，所以这里都是配置 gpio 功能。**

```
&lcd0 {
    /* part 1 */
    lcd_used = <1>;
    lcd_driver_name = "st7789v";
    
    /* part 2 */
    lcd_if = <0>;
    lcd_hv_if = <8>;
    
    /* part 3 */
    lcd_x = <240>;
    lcd_y = <320>;
    lcd_width = <108>;
    lcd_height = <64>;
    lcd_dclk_freq = <19>;
    lcd_hbp = <120>;
    ;10 + 20 + 10 + 240*3 = 760 real set 1000
    lcd_ht = <850>;
    lcd_hspw = <2>;
    lcd_vbp = <13>;
    lcd_vt = <373>;
    lcd_vspw = <2>;
    
    /* part 4 */
    lcd_backlight = <50>;
    lcd_pwm_used = <1>;
    lcd_pwm_ch = <8>;
    lcd_pwm_freq = <50000>;

    lcd_pwm_pol = <1>;
    lcd_pwm_max_limit = <255>;
    lcd_bl_en = <&pio PB 1 1 0 3 1>;
    lcd_bright_curve_en = <1>;
    
    /* part 5 */
    lcd_frm = <1>;
    lcd_hv_clk_phase = <0>;
    lcd_hv_sync_polarity= <0>;
    lcd_hv_srgb_seq = <0>;
    lcd_io_phase = <0x0000>;
    lcd_gamma_en = <0>;
    lcd_cmap_en = <0>;
    lcd_rb_swap = <0>;
    
    /* part 6 */
    lcd_power = "vcc-lcd";
    lcd_pin_power = "vcc-pd";
    
    /*reset */
    lcd_gpio_0 = <&pio PD 9 1 0 3 1>;
    /* cs */
    lcd_gpio_1 = <&pio PD 10 1 0 3 0>;
    /*sda */
    lcd_gpio_2 = <&pio PD 13 1 0 3 0>;
    /*sck */
    lcd_gpio_3 = <&pio PD 12 1 0 3 0>;
    
    pinctrl-0 = <&rgb8_pins_a>;
    pinctrl-1 = <&rgb8_pins_b>;
};
```



### 4.4 MIPI-DSI 接口

#### 4.4.1 概述

MIPI-DSI，即 Mobile Industry Processor Interface Display Serial Interface，移动通信行业处理器接口显示串行接口。

对于用户来说，需要了解：

1. Command mode，类似 MPU 接口，需要 IC 内部有 GRAM 来缓冲。

2. Video mode。类似 RGB 接口，没有 GRAM，需要不停往 panel 刷数据。其中 video mode 又分为三个子 mode

​		*•* Non-burst mode with sync pulses

​		*•* Non Burst mode with sync Events

​		*•* Burst mode。简单理解就是有效数据比率更高，传输效率更高。

3. lane 的意思是指一对差分管脚。



#### 4.4.2 MIPI-DSI 的管脚

MIPI-DSI 的管脚是在大部分 IC 中是专用，在 board.dtsi 里面不需要配置，只要硬件上连接好就行。

但是有一部分 IC 的 DSI 管脚不是专用的，与其它功能的脚复用，这个时候就需要配置好pinctrl-0和pinctrl-1。

mipi-dsi 的管脚是差分的，分为两种管脚，一种是时钟管脚，另外一种是数据管脚，数据管脚的数量是可变的，数量的单位是 lane，每一条 lane 实际包含两条线。一般来说 LCD 屏说明书里面的说的 lane 的数量是指数据管脚的数量不包括时钟管脚。比如说某 4 lane MIPI-DSI 屏就总共有(4+1)*2根脚。



#### 4.4.3 MIPI-DSI 的电源

一般都有一路电源供给 MIPI-DSI 这个模块，你可以理解为管脚电，也可以理解成模块电，不同 IC 这路电的电压要求可能不同，一旦确定 IC 型号之后，这路电的电压就不变，如果擅自改变此路电的电压可能导致模块异常。



​																图 4-6: pinmux



#### 4.4.4 判断是否支持某款 MIPI-DSI 屏

1. 分辨率限制。有 lane 的速度限制，我们可以得到最大分辨率的限制，计算公式如下，只要lane_speed 不超过上面 IC 规格规定的速度，那么理论上是支持的，请查看**IC 规格**。lane_speed=lcd_vt\*lcd_ht\*fps\*bit_per_pixel/lane_num/1e9

​		*•* 单位：Gbps。 

​		*•* fps: 期望刷新率，通过屏手册可知道，一般是 60。请看**lcd_dclk_freq**。 

​		*•* bit_per_pixel: 每个像素包含的比特数量，一般是 24 或者 18，通过**lcd_dsi_format**来设置。

​		*•* lane_num:lane 数量，通过**lcd_dsi_lane**来设置。

​		*•* 1e9:1000000000 的科学计数写法。

2. 选择分辨率的同时需要考虑系统带宽，DE 能力，所以即使接口方面支持这个分辨率，对于整个系统来说不一定支持，比如说硬件为了节省成本选择了一款速度很慢的 DDR 内存然后同时又想选择高分辨率的屏幕，很明显这是不现实的。

3. lane 数量限制。绝大部分全志科技 IC 最大支持 4 lane 的 MIPI-DSI，如果你看到该款屏超过 4 lane 就肯定不支持了。少数 IC 最大支持 8 lane，应该选择该款 IC。

4. MIPI-DSI 标准不兼容。请查看**IC 规格**。



#### 4.4.5 计算 MIPI-DSI 时钟 lane 频率

使用示波器测量 MIPI-DSI 的时钟信号，确定其频率是否满足屏的需求。

首先，我们由给定的像素时钟和 lane 数量，可以计算出理论 CLK 信号的频率，如下公式：

Freq_dsi_clk = (Dclk * colordepth*3 / lane )/2

1. Freq_dsi_clk：我们要测量的 dsi 时钟脚的频率。单位 MHz。

2. Dclk：像素时钟。由 lcd_ht\*lcd_vt*fps/1e6 公式算出来。

3. Colordepth：颜色深度，一般是 8 或者 6。

4. 乘以 3 表示 RGB 分量 3 个。

5. Lane：dsi 的 lane 数量。

6. 除以 2：是因为 dsi 时钟是双沿采样。



#### 4.4.6 MIPI-DSI Video mode 屏配置示例

绝大多数 MIPI-DSI 屏的配置都是用 video mode。

**下面是典型MIPI-DSI video mode 的 board.dts 配置示例，其中用空行把配置分成几个部分**

1. 第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏驱动来初始化。

2. 第二部分，决定该配置是 dsi 接口，而且 dsi 接口使用的是 video mode。

3. 第三部分，决定了 SoC 中的 LCD 模块发送时序，请查看**屏时序参数说明**。

4. 第四部分，背光相关的设置。请看**背光相关参数**。

5. 第五部分，dsi 接口的详细设置。

6. 第六部分，显示效果相关的设置。

7. 第七部分，管脚和电源设置。请看**电源和管脚参数**。

```
&lcd0 {
    /* part 1 */
    lcd_used = <1>;
    lcd_driver_name = "k101im2qa04";

	/* part 2 */
    lcd_if = <4>;
    lcd_dsi_if = <0>;
    
    /* part 3 */
    lcd_x = <800>;
    lcd_y = <1280>;
    lcd_width = <135>;
    lcd_height = <216>;
    lcd_dclk_freq = <68>;
    lcd_hbp = <36>;
    lcd_ht = <854>;
    lcd_hspw = <18>;
    lcd_vbp = <12>;
    lcd_vt = <1320>;
    lcd_vspw = <4>;
    
    /* part 4 */
    lcd_backlight = <50>;
    lcd_pwm_used = <1>;
    lcd_pwm_ch = <0>;
    lcd_pwm_freq = <50000>;
    lcd_pwm_pol = <1>;
    lcd_pwm_max_limit = <255>;
    lcd_bl_en = <&pio PB 8 1 0 3 1>;
    lcd_bright_curve_en = <0>;
    
    /* part 5 */
    lcd_dsi_lane = <4>;
    lcd_dsi_format = <0>;
    lcd_dsi_te = <0>;
    
    /* part 6 */
    lcd_frm = <0>;
    lcd_gamma_en = <0>;
    lcd_cmap_en = <0>;
    
    /* part 7 */
    lcd_pin_power = "dcdc1";
    lcd_pin_power1 = "eldo3";
    lcd_power = "dc1sw";
    lcd_gpio_0 = <&pio PD 22 1 0 3 1>;
    pinctrl-0 = <&dsi4lane_pins_a>;
    pinctrl-1 = <&dsi4lane_pins_b>;
};
```



#### 4.4.7 MIPI-DSI 超高分辨率屏配置示例

根据分辨率的高低通常分为几种模式来配置。1080p 分辨率及其以下：只需要设置 lcd_dsi_if 来控制就行。Command mode 一般是低分辨率屏，而 video mode 和 burst mode 则是用于高分辨率的。如果分辨率达到 2k，则需要额外的设置。

分辨率达到 2k 以上的屏，实际上需要多达 8 条数据 lane 才能正常显示，其中四条 lane 发送一副图像中的奇像素，另外一副图像发送偶像素。

说明：

**注意只有部分 IC 支持超高分辨率，具体查看芯片规格中的 MIPI-DSI 部分下面是 MIPI-DSI 高分辨超高分辨率（大于 2k）board.dts配置示例，其中用空行把配置分成几个部分**

1. 第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏驱动来初始化。

2. 第二部分，决定该配置是 dsi 接口，而且 dsi 接口使用的是 video mode。

3. 第三部分，决定了 SoC 中的 LCD 模块发送时序，请查看屏时序参数说明。

4. 第四部分，背光相关的设置，请看背光相关参数。

5. 第五部分，dsi 接口的详细设置。



说明

**lcd_dsi_lane 依旧设置成 4 条 lane 的原因，是因为这个是设置一个 dsi 的 lane 数量，这个屏要用两个 dsi。加起来就是8条 lane。**

**此时** **lcd_tcon_mode, lcd_dsi_port_num** **和** **lcd_tcon_en_odd_even_div** **三个选项需要特别设置，点击查看具体含义，如果是 1080p 及其以下分辨率的屏 (只用 4lane 或者以下的)，那么蓝色下划线三个配置默认** **0** **即可。**



6. 第六部分，显示效果部分的设置。

7. 第七部分，是管脚和电源的配置。请根据电路图来配置。请看**电源和管脚参数**。

```
&lcd0 {
    /* part 1 */
    lcd_used = <1>;
    lcd_driver_name = "lq101r1sx03";

	/* part 2 */
    lcd_if = <4>;
    lcd_dsi_if = <0>;
    
    /* part 3 */
    lcd_x = <2560>;
    lcd_y = <1600>;
    lcd_width = <216>;
    lcd_height = <135>;
    lcd_dclk_freq = <268>;
    lcd_hbp = <80>;
    lcd_ht = <2720>;
    lcd_hspw = <32>;
    lcd_vbp = <37>;
    lcd_vt = <1646>;
    lcd_vspw = <6>;
    
    /* part 4 */
    lcd_backlight = <50>;
    lcd_pwm_used = <1>;
    lcd_pwm_ch = <0>;
    lcd_pwm_freq = <50000>;
    lcd_pwm_pol = <1>;
    lcd_pwm_max_limit = <255>;
    lcd_bl_en = <&pio PH 10 1 0 3 1>;
    
    /* part 5 */
    lcd_dsi_lane = <4>;
    lcd_dsi_format = <0>;
    lcd_dsi_te = <0>;
    lcd_dsi_port_num = <1>;
    lcd_tcon_mode = <4>;
    lcd_tcon_en_odd_even_div = <1>;
    
    /* part 6 */
    lcd_frm = <0>;
    lcd_io_phase = <0x0000>;
    lcd_gamma_en = <0>;
    lcd_bright_curve_en = <0>;
    lcd_cmap_en = <0>;
    
    /* part 7 */
    lcd_power = "vcc18-lcd";
    lcd_power1 = "vcc33-lcd";
    lcd_pin_power = "vcc-pd";
    lcd_gpio_0 = <&pio PH 11 1 0 3 1>;
    lcd_gpio_1 = <&pio PH 12 1 0 3 1>;
};
```



#### 4.4.8 MIPI-DSI Command mode 屏配置示例

Command mode 下的 DSI 屏类似与 I8080 接口，屏内部带 RAM 用于缓冲和图像处理，这种情况一般都需要用屏的 te 脚来触发 vsync 中断，所以与其它类型的 DSI 屏不同的是，这里需要设置 lcd_vsync 脚，屏的 te 脚就连到 lcd_vsync 上，并且 lcd_dsi_te 设置成 1。te 脚的设置非常关键，一般来说如果屏有 te 脚，则必须连上，否则在显示动态画面的时候会画面会撕裂，而且软件无法解决，直接造成最终硬件无法量产的结果。



这里只列举出与 MIPI-DSI video mode 不同的关键之处，其它参考上一小节。

1. 第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏驱动来初始化。

2. 第二部分，决定该配置是 dsi 接口，而且**lcd_dsi_if**设置成 1 表明 command mode。

3. 第三部分，决定了 SoC 中的 LCD 模块发送时序，请查看**屏时序参数说明**。

4. 第四部分，背光相关的设置。请看**背光相关参数**。

5. 第五部分，dsi 接口的详细设置。**lcd_dsi_te**，这里设置为 1 表示使能 te 触发。

6. 第六部分，显示效果相关的设置。

7. 第七部分，管脚和电源设置。lcd_vsync，这里是 te 脚，硬件上需要将这根脚连接到屏的 te 脚，软件上需要将其设置为 vsync 功能。请看**电源和管脚参数**。

```
&lcd0 {
    /* part 1 */
    lcd_used = <1>;
    lcd_driver_name = "h245qbn02";
    
    /* part 2 */
    lcd_if = <4>;
    lcd_dsi_if = <1>;
    
    /* part 3 */
    lcd_x = <240>;
    lcd_y = <432>;
    lcd_width = <52>;
    lcd_height = <52>;
    lcd_dclk_freq = <18>;
    lcd_hbp = <96>;
    lcd_ht = <480>;
    lcd_hspw = <2>;
    lcd_vbp = <21>;
    lcd_vt = <514>;
    lcd_vspw = <2>;
    
    /* part 4 */
    lcd_backlight = <100>;
    lcd_pwm_used = <1>;
    lcd_pwm_ch = <0>;
    lcd_pwm_freq = <50000>;
    lcd_pwm_pol = <1>;
    lcd_pwm_max_limit = <255>;
    lcd_bright_curve_en = <0>;
    lcd_bl_en = <&pio PB 3 1 0 3 1>;
    
    /* part 5 */
    lcd_dsi_lane = <1>;
    lcd_dsi_format = <0>;
    lcd_dsi_te = <1>;
    lcd_frm = <0>;
    lcd_io_phase = <0x0000>;
    lcd_gamma_en = <0>;
    lcd_cmap_en = <0>;
    
    /* part 7 */
    lcd_power = "axp233_dc1sw"
    lcd_power1 = "axp233_eldo1"
    lcd_gpio_0 = <&pio PB 2 1 0 3 0>;
    lcd_vsync = <&pio PD 21 2 0 3 0>;
};
```



#### 4.4.9 MIPI-DSI VR 双屏配置示例

实际场景是两个物理屏，每个屏是 1080p，每个屏都是 4 条 lane，要求的是两个屏各自显示一帧图像的左右一半，由于宽高比和横竖屏以及 DE 处理能力的因素，一个 DE+ 一个 tcon+ 两个DSI 已经无法满足，必须用两个 tcon 各自驱动一个 dsi，但是两路显示必须要同步，这就需要用到两个 tcon 的同步模式。

1. LCD0 标记为 slave tcon，它由 master tcon 来驱动（设置lcd_tcon_mode）

2. LCD1 标记为 master tcon，并且负责两个屏的所有电源，背光，管脚的开关。

3. 把管脚，电源等都放到 LCD1 开，LCD0 先开，对应模块寄存器都初始化，但是电源不开，然后开 LCD1，LCD1 使能就会触发 LCD0 一起发数据。这样做到同时亮灭。

说明

**注意：仅有极少 IC 支持该模式**

```
;slave
&lcd0 {
    lcd_used = <1>;
    lcd_driver_name = "lpm025m475a";
    ;lcd_bl_0_percent = <0>;
    ;lcd_bl_40_percent = <23>;
    ;lcd_bl_100_percent = <100>;
    lcd_backlight = <50>;
    lcd_if = <4>;
    lcd_x = <1080>;
    lcd_y = <1920>;
    lcd_width = <31>;
    lcd_height = <56>;
    lcd_dclk_freq = <141>;
    lcd_pwm_used = <0>;
    lcd_pwm_ch = <0>;
    lcd_pwm_freq = <20000>;
    lcd_pwm_pol = <0>;
    lcd_pwm_max_limit = <255>;
    lcd_hbp = <100>;
    lcd_ht = <1212>;
    lcd_hspw = <5>;
    lcd_vbp = <8>;
    lcd_vt = <1936>;
    lcd_vspw = <2>;
    lcd_dsi_if = <0>;
    lcd_dsi_lane = <4>;
    lcd_dsi_format = <0>;
    lcd_dsi_te = <0>;
    lcd_dsi_eotp = <0>;
    lcd_frm = <0>;
    lcd_io_phase = <0x0000>;
    lcd_hv_clk_phase = <0>;
    lcd_hv_sync_polarity= <0>;
    lcd_gamma_en = <0>;
    lcd_bright_curve_en = <0>;
    lcd_cmap_en = <0>;
    lcd_dsi_port_num = <0>;
    lcd_tcon_mode = <3>;
    lcd_slave_stop_pos = <0>;
    lcd_sync_pixel_num = <0>;
    lcd_sync_line_num = <0>;
};

&lcd1 {
    lcd_used = <1>;
    lcd_driver_name = "lpm025m475a";
    ;lcd_bl_0_percent = <0>;
    ;lcd_bl_40_percent = <23>;
    ;lcd_bl_100_percent = <100>;
    lcd_backlight = <50>;
    lcd_if = <4>;
    lcd_x = <1080>;
    lcd_y = <1920>;
    lcd_width = <31>;
    lcd_height = <56>;
    lcd_dclk_freq = <141>;
    lcd_pwm_used = <1>;
    lcd_pwm_ch = <0>;
    lcd_pwm_freq = <20000>;
    lcd_pwm_pol = <0>;
    lcd_pwm_max_limit = <255>;
    lcd_hbp = <100>;
    lcd_ht = <1212>;
    lcd_hspw = <5>;
    lcd_vbp = <8>;
    lcd_vt = <1936>;
    lcd_vspw = <2>;
    lcd_dsi_if = <0>;
    lcd_dsi_lane = <4>;
    lcd_dsi_format = <0>;
    lcd_dsi_te = <0>;
    lcd_dsi_eotp = <0>;
    lcd_frm = <0>;
    lcd_io_phase = <0x0000>;
    lcd_hv_clk_phase = <0>;
    lcd_hv_sync_polarity= <0>;
    lcd_gamma_en = <0>;
    lcd_bright_curve_en = <0>;
    lcd_cmap_en = <0>;
    lcd_dsi_port_num = <0>;
    lcd_tcon_mode = <1>;
    lcd_tcon_slave_num = <0>;
    lcd_slave_stop_pos = <0>;
    lcd_sync_pixel_num = <0>;
    lcd_sync_line_num = <0>;
    lcd_bl_en = <&pio PH 10 1 0 3 1>;
    lcd_power = "vcc-dsi";
    lcd_power1 = "vcc18-lcd";
    lcd_power2 = "vcc33-lcd";
    lcd_gpio_0 = <&pio PH 8 1 0 3 1>;
    lcd_gpio_1 = <&pio PH 11 1 0 3 1>;
    lcd_gpio_2 = <&pio PH 12 1 0 3 1>;
    lcd_pin_power = "vcc-ph"
};
```



### 4.5 I8080 接口

#### 4.5.1 概述

Intel 8080 接口屏 (又称 MCU 接口) 很老的协议，一般用在分辨率很小的屏上。

管脚的控制脚有 6 种：

*•* CS 片选信号，决定该芯片是否工作. 

*•* RS 寄存器选择信号，低表示选择 index 或者 status 寄存器，高表示选择控制寄存器。实际场景中一般接 SoC 的 LCD_DE 脚（数据使能脚）

*•* WR （低表示写数据) 数据命令区分信号，也就是写时钟信号，一般接 SoC 的 LCD_CLK 脚 

*•* RD （低表示读数据）数据读信号，也就是读时钟信号，一般接 SoC 的 LCD_HSYNC 脚 

*•* RESET 复位 LCD（用固定命令系列 0 1 0 来复位) 

*•* Data 是双向的

I8080 根据的数据位宽接口有 8/9/16/18，连哪些脚参考，即使位宽一样，连的管脚也不一样，还要考虑的因素是 rgb 格式。

1. RGB565，总共有 65K 这么多种颜色

2. RGB666，总共有 262K 那么多种颜色

3. 9bit 固定为 262K

从屏手册得知：数据位宽，颜色数量之和，参考 **RGB 和 I8080 管脚配置示意图**，进行硬件连接。



#### 4.5.2 I8080 接口屏典型配置示例

**下面是典型是一个** **RGB565** **的，位宽为** **8** **位的** **I8080** **接口的屏的** **board.dts** **配置示例。**

1. 第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏驱动来初始化。

2. 第二部分，决定该配置是 I8080 接口，而且是 8bit/2cycle 格式 RGB565。

技巧

为什么叫做 *8bit/2cycle RGB565* 呢，首先它的格式是 *RGB565*，也就是一个像素是 *16bit*，然后它是 *8bit* 的位宽，就需要两个时钟周期才能发完一个像素，所以才叫 *2 cycle*。

3. 第三部分，决定了 SoC 中的 LCD 模块发送时序，请查看**屏时序参数说明**。这里比较特殊的是设置像素时钟要满足以下公式：lcd_dclk_freq\*2>=lcd_ht\*lcd_vt\*fps，或者lcd_dclk_freq=lcd_ht\*2\*lcd_vt*60, 也就是要么双倍lcd_ht要么双倍lcd_dclk_freq

4. 第四部分，背光相关的设置。请看**背光相关参数**。

5. 第五部分，cpu 接口的详细设置。这里使能了**lcd_cpu_te**和**lcd_cpu_mode**，意思是使用 te触发和规定了触发间隔。这是非常关键的设置。

6. 第六部分，显示效果相关的设置。这里使能了**lcd_frm**(#lcd_frm] 也是比较关键的设置，详细意思点击查看。

7. 第七部分，管脚和电源设置。这里为了用 te 触发，同样需要设置 lcd_vsync，该脚功能定义已经包括在 pinctrl-0 中。这里自定义了一组管脚。参考**RGB 和 I8080 管脚配置示意图**，通过确定 I8080 的位宽，像素格式（颜色数量），在表中确定需要连接哪些管脚。请看**电源和管脚参数**。

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

&lcd0 {
    /* part 1 */
    lcd_used = <1>;
    lcd_driver_name = "s2003t46g";
    
    /* part 2 */
    lcd_if = <1>;
    lcd_cpu_if = <14>;
    
    /* part 3 */
    lcd_x = <240>;
    lcd_y = <320>;
    lcd_width = <108>;
    lcd_height = <64>;
    lcd_dclk_freq = <16>;
    lcd_hbp = <20>;
    lcd_ht = <298>;
    lcd_hspw = <10>;
    lcd_vbp = <8>;
    lcd_vt = <336>;
    lcd_vspw = <4>;
    
    /* part 4 */
    lcd_pwm_used = <1>;
    lcd_pwm_ch = <8>;
    lcd_pwm_freq = <50000>;
    lcd_pwm_pol = <1>;
    lcd_pwm_max_limit = <255>;
    lcd_bright_curve_en = <1>;
    
    /* part 5 */
    lcd_cpu_mode = <1>;
    lcd_cpu_te = <1>;
    
    /* part 6 */
    lcd_frm = <1>;
    lcd_gamma_en = <0>;
    lcd_cmap_en = <0>;
    lcd_rb_swap = <0>;
    
    /* part 7 */
    lcd_power = "vcc-lcd";
    lcd_pin_power = "vcc-pd";
    ;reset pin
    lcd_gpio_0 = <&pio PD 9 1 0 3 1>;
    ;cs pin
    lcd_gpio_1 = <&pio PD 10 1 0 3 0>;
    pinctrl-0 = <&I8080_8bit_pins_a>;
    pinctrl-1 = <&I8080_8bit_pins_a>;
};
```



### 4.6 LVDS 接口

#### 4.6.1 概述

LVDS 即 Low Voltage Differential Signaling 是一种低压差分信号接口。

#### 4.6.2 LVDS Single link 典型配置

LVDS 接口，lcd0 对应的 lvds 管脚和 lcd1 对应的 lvds 管脚是固定而且不一样。

由于 lvds 协议不具备传输数据之外的能力，一般屏端不需要任何初始化，只需要初始化 SoC 端即可。所以这里的 lcd_driver_name 依旧是”default_lcd”，当然你可以为初始化的启动延时做专门的优化。



**下面是典型是** **single link lvds** **屏的** **board.dts** **配置示例，其中用空行把配置分成几个部分**

1. 第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏驱动来初始化。

2. 第二部分，决定该配置是 lvds 接口，而且是 single link。

   技巧

   如果 *Dual Link* 的屏，那么除了要改 *lcd_lvds_if* 为 *1* 之外，管脚方面还要把 *lcd1* 的管脚一起搬到下面去，也就是总共需要配置 *PD0* 到 *PD9*，和配置 *PD10* 到 *PD19* 总共二十根脚为 *lvds* 管脚功能（功能 *3*）。当然屏的 *timing* 也是要根据屏来改的。

3. 第三部分，决定了 SoC 中的 LCD 模块发送时序，请查看屏时序参数说明。

4. 第四部分，背光相关的设置。请看背光相关参数。

5. 第五部分，lvds 接口的详细设置。

6. 第六部分，显示效果相关的设置。

7. 第七部分，管脚和电源设置。请看电源和管脚参数。

```
&lcd0 {
    /* part 1 */
    lcd_used = 1
    lcd_driver_name = "default_lcd";

	/* part 2 */
    lcd_if = 3
    lcd_lvds_if = 0
    
    /* part 3 */
    lcd_x = 1280
    lcd_y = 800
    lcd_width = 150
    lcd_height = 94
    lcd_dclk_freq = 70
    lcd_hbp = 20
    lcd_ht = 1418
    lcd_hspw = 10
    lcd_vbp = 10
    lcd_vt = 814
    lcd_vspw = 5
    
    /* part 4 */
    lcd_pwm_used = 1
    lcd_pwm_ch = 0
    lcd_pwm_freq = 50000
    lcd_pwm_pol = 0
    lcd_pwm_max_limit = 255
    lcd_backlight = 50
    lcd_bright_curve_en = 0
    lcd_bl_en = <&pio PD 21 1 0 3 1>;
    
    /* part 5 */
    lcd_lvds_colordepth = 1
    lcd_lvds_mode = 0
    
    /* part 6 */
    lcd_frm = 1
    lcd_hv_clk_phase = 0
    lcd_hv_sync_polarity= 0
    lcd_gamma_en = 0
    lcd_cmap_en = 0
    
    /* part 7 */
    lcd_power = "vcc-lcd"
    pinctrl-0 = <&lvds0_pins_a>;
    pinctrl-1 = <&lvds0_pins_b>;
};
```

4.6.3 LVDS dual link 典型配置

如果 Dual Link 的屏:

1. lcd_lvds_if设置为 1（场景 1）或者 2（场景 2）

2. 管脚配置方面，也从 4 data lane 变成 8 data lane，包括 clk lane 总共 20 根管脚。场景 1，物理上连接一个屏，8 data lane，SoC 向每 4 条 lane 传输一半的像素，奇数像素或者偶数像素

```
&lcd1 {
    lcd_used = <1>;
    
    lcd_driver_name = "bp101wx1";
    lcd_backlight = <50>;
    lcd_if = <3>;
    
    lcd_x = <2560>;
    lcd_y = <800>;
    lcd_width = <150>;
    lcd_height = <94>;
    lcd_dclk_freq = <138>;
    
    lcd_pwm_used = <0>;
    lcd_pwm_ch = <2>;
    lcd_pwm_freq = <50000>;
    lcd_pwm_pol = <1>;
    lcd_pwm_max_limit = <255>;
    
    lcd_hbp = <40>;
    lcd_ht = <2836>;
    lcd_hspw = <20>;
    lcd_vbp = <10>;
    lcd_vt = <814>;
    lcd_vspw = <5>;
    
    lcd_lvds_if = <1>;
    lcd_lvds_colordepth = <0>;
    lcd_lvds_mode = <0>;
    lcd_frm = <0>;
    lcd_hv_clk_phase = <0>;
    lcd_hv_sync_polarity= <0>;
    lcd_gamma_en = <0>;
    lcd_bright_curve_en = <0>;
    lcd_cmap_en = <0>;
    lcd_fsync_en = <0>;
    lcd_fsync_act_time = <1000>;
    lcd_fsync_dis_time = <1000>;
    lcd_fsync_pol = <0>;
    
    deu_mode = <0>;
    lcdgamma4iep = <22>;
    smart_color = <90>;
    lcd_bl_en = <&pio PJ 27 1 0 3 1>;
    lcd_gpio_0 = <&pio PI 1 1 0 3 1>;
    
    lcd_pin_power = "bldo5";
    
    lcd_power = "dc1sw";
    
    pinctrl-0 = <&lcd1_lvds2link_pins_a>;
    pinctrl-1 = <&lcd1_lvds2link_pins_b>;
};
```

场景 2（部分 IC 支持），物理上连接两个屏，每个屏各自 4 条 lane，两个屏是一样型号，分辨率和 timing 一样，这时候部分 IC 支持将全部像素发到每个屏上，实现双显（信号上的双显），注意这时候 lcd timing 是一个屏的 timing, lcd_lvds_if 为 2.

```
lcd1: lcd1@01c0c001 {
    lcd_used = <1>;
    
    lcd_driver_name = "bp101wx1";
    lcd_backlight = <50>;
    lcd_if = <3>;
    
    lcd_x = <1280>;
    lcd_y = <800>;
    lcd_width = <150>;
    lcd_height = <94>;
    lcd_dclk_freq = <70>;
    
    lcd_pwm_used = <0>;
    lcd_pwm_ch = <2>;
    lcd_pwm_freq = <50000>;
    lcd_pwm_pol = <1>;
    lcd_pwm_max_limit = <255>;
    
    lcd_hbp = <20>;
    lcd_ht = <1418>;
    lcd_hspw = <10>;
    lcd_vbp = <10>;
    lcd_vt = <814>;
    lcd_vspw = <5>;
    
    lcd_lvds_if = <2>;
    lcd_lvds_colordepth = <0>;
    lcd_lvds_mode = <0>;
    lcd_frm = <0>;
    lcd_hv_clk_phase = <0>;
    lcd_hv_sync_polarity= <0>;
    lcd_gamma_en = <0>;
    lcd_bright_curve_en = <0>;
    lcd_cmap_en = <0>;
    lcd_fsync_en = <0>;
    lcd_fsync_act_time = <1000>;
    lcd_fsync_dis_time = <1000>;
    lcd_fsync_pol = <0>;
    
    deu_mode = <0>;
    lcdgamma4iep = <22>;
    smart_color = <90>;
    lcd_bl_en = <&pio PJ 27 1 0 3 1>;
    lcd_gpio_0 = <&pio PI 1 1 0 3 1>;
    
    lcd_pin_power = "bldo5";
    
    lcd_power = "dc1sw";
    
    pinctrl-0 = <&lcd1_lvds2link_pins_a>;
    pinctrl-1 = <&lcd1_lvds2link_pins_a>;
};
```



### 4.7 RGB 和 I8080 管脚配置示意图









​														  	图 4-7: pinmux



### 4.8 多屏兼容使用说明

#### 4.8.1 功能说明

所谓多屏兼容，其实就是单一固件同时支持多款接口、型号不一的 LCD 屏幕模组（主要特指屏驱动、时序不能共用），即支持动态识别 LCD 模组型号，并加载对应 LCD 驱动并以相应的 LCD 屏幕进行显示。

对于多屏兼容的实现，除了将需要兼容的屏全部在对应的平台调通点亮外，另一个关键是如何将当前正在使用的屏识别出来并告知驱动框架以加载对应的屏驱动，一般而言有如下几种方法：

*•* 当使用不同屏时，硬件上也将某些 io 同时拉高或拉低，用 io 标识屏模组。

*•* 出厂时根据使用的屏型号，往 flash 中同时烧录一个屏模组型号标识。

*•* 对 mipi 等支持通信的屏，直接读屏 id。



#### 4.8.2 使用方法

该功能目前只支持在单屏显示时兼容多屏（也就是 lcd0），并且使用前为了避免不必要的麻烦需要先确保兼容的屏在不使用屏兼容功能时能正常点亮。多屏兼容的具体使用方法如下：

1. Uboot 的 dts 中增加节点 lcd0_1（或者 lcd0_2、lcd0_3，根据需要按顺序增加），并填入需要对应的屏参数及配置。

2. 在默认的屏，也就是 lcd0 的屏驱动中添加读屏 id(或其他方法) 然后切换的逻辑，切换的接口为 sunxi_lcd_switch_compat_panel。具体可参照下图 “屏驱动切换示例”。

3. 在调用 sunxi_lcd_switch_compat_panel 前，确保执行了之前调用的开屏流程对应的关屏流程，类似图 “屏驱动切换示例” 中的 LCD_power_off。

4. 确保在 LCD_open_flow 中，调用 sunxi_lcd_switch_compat_panel 所在的开屏函数以及之前的开屏函数的延时都是 0，具体参照下图 “屏驱动切换延时”。



​																	图 4-8: 屏驱动切换示例



​																	图 4-9: 屏驱动切换延时



参考配置：

1.brandy/brandy-2.0/u-boot-2018/drivers/video/sunxi/disp2/disp/lcd/K080_IM2HYL802R_800X1280.c

2.brandy/brandy-2.0/u-boot-2018/arch/arm/dts/a133-b4-board.dts或device/config/chips/a133/configs/b4/

uboot-board.dts

其他说明：

1. 由于实现原理是通过 uboot 替换 kernel 的 lcd 相关的 dts 配置，kernel 的屏驱动及 dts 无须做任何兼容处理，屏兼容对内核是完全透明的。

2. 为了加快机器的启动速度，默认在第一次启动时记录当前使用的兼容屏，后续启动时将不执行try 屏驱动的逻辑而直接使用之前记录的屏驱动。如不需要该功能，需要手动关闭 uboot 配项“COMPATIBLE_PANEL_RECORD”。

