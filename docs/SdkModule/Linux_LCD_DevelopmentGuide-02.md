## 4 模块介绍

### 4.1 添加屏驱动步骤

1. 对于linux4.9 及以下版本总共需要修改三处地方（即下列前三项），对于linux5.4 则需要修改四处地方，具体可参考屏驱动源码位置。

  • linux 源码仓库。

  • uboot 源码仓库。在uboot 中也有显示和屏驱动，目的是显示logo。

  • 板级dts 配置仓库。目的是通过board.dts 来配置一些通用的LCD 配置参数。对于linux4.9，该配置同时对内核及uboot 生效，对于linux-5.4，请参照下条。

  • 对于linux5.4，还需额外配置uboot 专用板级dts 配置仓库。

2. 确保全志显示框架的内核配置有使能，查看menuconfig 配置说明。

3. 前期准备以下资料和信息：

  • 屏手册。主要是描述屏基本信息和电气特性等，向屏厂索要。

  • Driver IC 手册。主要是描述屏IC 的详细信息。这里主要是对各个命令进行详解，对我们进行初始化定制有用，向屏厂索要。

  • 屏时序信息。请向屏厂索要。请看屏时序参数说明以了解更多信息。

  • 屏初始化代码。，请向屏厂索要。一般情况下DSI 和I8080 屏等都需要初始化命令对屏进行初始化。

  • 万用表。调屏避免不了测量相关电压。

4. 动手添加屏驱动之前，先了解屏驱动，请看屏驱动分解。

5. 通过第3 步的资料，定位该屏的类型，然后选择一个已有同样类型的屏驱动作为模板进行屏驱动添加或者直接在上面修改。

6. 修改屏驱动目录下的panel.c和panel.h。在全局结构体变量panel_array中新增刚才添加strcut_lcd_panel的变量指针。panel.h中新增strcut lcd_panel的声

  明。

7. 修改Makefile。在lcd 屏驱动目录的上一级的Makefile 文件中的disp-objs中新增刚才添加屏驱动.o。

8. 修改board.dts 中的lcd0。可以看RGB 接口，MIPI-DSI 接口，I8080 接口和LVDS 接口，里面有介绍各种接口典型配置。硬件参数说明，这一章有所有lcd0 节

  点下可配置属性详细解释。

9. 编译uboot，kernel，打包烧写。注意不同SDK，编译方式有所不同，部分SDK 默认不编译uboot。
10. 调试。通过调试方法我们可以初步定位问题，还有FAQ，对调屏也有帮助。

### 4.2 屏驱动说明

#### 4.2.1 屏驱动源码位置

linux 3.4 版本内核：

linux3-4/drivers/video/sunxi/disp2/disp/lcd/

linux 3.10 版本内核：

linux3-10/drivers/video/sunxi/disp2/disp/lcd/

linux 4.9 版本及其以上内核：

linux-4.9/drivers/video/fbdev/sunxi/disp2/disp/lcd/

uboot-2014:

brandy/u-boot-2014.07/drivers/video/sunxi/disp2/disp/lcd

uboot-2018:

brandy/brandy-2.0/u-boot-2018/drivers/video/sunxi/disp2/disp/lcd

板级配置，其中“芯片型号” 比如R818，和“板子名称” 比如demo，请根据实际替换。

device/config/chips/芯片型号/configs/板子名称/board.dts

针对linux5.4 时使用的uboot 板级配置：

device/config/chips/芯片型号/configs/板子名称/uboot-board.dts

针对linux5.4 时使用的kernel 板级配置：

device/config/chips/芯片型号/configs/板子名称/linux-5.4/board.dts

#### 4.2.2 menuconfig 配置说明

LCD 相关代码包含在disp 驱动模块中，进入内核根目录，执行make ARCH=arm menuconfig或者make ARCH=arm64 menuconfig(64bit 平台) 进入配置主界

面。并按以下步骤操作：

DE1.0 对应平台：R6(linux-3.10)、R16(linux-3.4)。

DE2.0 对应平台：除R6 和R16 之外的。

![image-20221129102051729](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_LCD_DevGuide_image-20221129102051729.png)

<center>图4-1: DE1.0 menuconfig 配置图</center>

![image-20221129102107009](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_LCD_DevGuide_image-20221129102107009.png)

<center>图4-2: DE2.0 menuconfig 配置图</center>

以R40 为例，具体配置目录为：Device Drivers->Graphics support->Support for frame buffer devices->Video Support for sunxi -> DISP Driver Support(sunxi-disp2)。



#### 4.2.3 屏驱动分解

在屏驱动源码位置中，主要分为四类文件

1. panel.c和panel.h，当用户添加新屏驱动时，是需要修改这两个文件的，需要将屏结构体变量添加到全局结构体变量panel_array中。

2. lcd_source.c和lcd_source.h，这两个文件实现的是给屏驱动使用的函数接口，比如电源开关，gpio，dsi 读写接口等，用户不需要修改只需要用。

3. 屏驱动。除了上面提到的源文件外，其它的一般一个c 文件和一个h 文件就代表一个屏驱动。

4. 在屏驱动源码位置的上一级，有用户需要修改的Makefile 文件。

  我们可以打开drivers/video/fbdev/sunxi/disp2/disp/lcd/default_panel.c作为屏驱动的例子，在该文件的最后

```
struct __lcd_panel default_panel = {
    /* panel driver name, must mach the lcd_drv_name in board.dts */
    .name = "default_lcd",
        .func = {
        .cfg_panel_info = LCD_cfg_panel_info,
        .cfg_open_flow = LCD_open_flow,
        .cfg_close_flow = LCD_close_flow,
        }
    ,
};
```

该全局变量default_panel的成员name与lcd_driver_name必须一致，这个关系到驱动能否找到指定的文件。

接下来是func成员的初始化，这里最主要实现三个回调函数。LCD_cfg_panel_info，LCD_open_flow和LCD_close_flow。

开关屏流程即屏上下电流程，屏手册或者driver IC 手册中里面的Power on Sequence 和Power off Sequence。

开关屏的操作流程如下图所示。

其中，LCD_open_flow 和LCD_close_flow 称为开关屏流程函数， 方框中的函数， 如LCD_power_on，称为开关屏步骤函数。

不需要进行初始化操作的LCD 屏，比如lvds 屏，RGB 屏等，LCD_panel_init 及LCD_panel_exit这函数可以为空。

![image-20221129102353374](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_LCD_DevGuide_image-20221129102353374.png)

<center>图4-3: LCD 开关屏流程</center>

函数：LCD_open_flow

功能：LCD_open_flow 函数只会系统初始化的时候调用一次，执行每个LCD_OPEN_FUNC即是把对应的开屏步骤函数进行注册，先注册先执行，但并没有立刻执

行该开屏步骤函数。

原型：

```
static s32 LCD_open_flow(u32 sel)
```


函数常用内容为：

```
static __s32 LCD_open_flow(__u32 sel)
{
    LCD_OPEN_FUNC(sel, LCD_power_on,10);
    LCD_OPEN_FUNC(sel, LCD_panel_init, 50);
    LCD_OPEN_FUNC(sel, sunxi_lcd_tcon_enable, 100);
    LCD_OPEN_FUNC(sel, LCD_bl_open, 0);
    return 0;
}
```

如上，调用四次LCD_OPEN_FUNC 注册了四个回调函数，对应了四个开屏流程, 先注册先执行。实际上注册多少个函数是用户自己的自由，只要合理即可。

1. LCD_power_on 即打开LCD 电源，再延迟10ms；这个步骤一般用于打开LCD 相关电源和相关管脚比如复位脚。这里一般是使用电源控制函数说明和管脚控制

  函数说明进行操作。

2. LCD_panel_init 即初始化屏，再延迟50ms；不需要初始化的屏，可省掉此步骤，这个函数一般用于发送初始化命令给屏进行屏初始化。如果是DSI 屏看DSI 相

  关函数说明，如果是I8080 屏用I8080 接口函数说明，如果是其它情况比如i2c 或者spi 可以看使用iic/spi 串行接口初始化，也可以用GPIO 来进行模拟。

3. sunxi_lcd_tcon_enable 打开TCON，再延迟100ms；这一步是固定的，表示开始发送图像信号。

4. LCD_bl_open 打开背光，再延迟0ms。前面三步搞定之后才开背光，这样不会看到闪烁。这里一般使用的函数请看背光控制函数说明。

  如下图，这是屏手册中典型的上电时序图，我们编写屏驱动的时候，也要注意，该延时就得延时。

![image-20221129102545133](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_LCD_DevGuide_image-20221129102545133.png)

<center>图4-4: power on</center>

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

LCD_OPEN_FUNC 的第二个参数是前后两个步骤的延时长度，单位ms，注意这里的数值请按照屏手册规定去填，乱填可能导致屏初始化异常或者开关屏时间过

长，影响用户体验。

与LCD_open_flow 对应的是LCD_close_flow 是，用于注册关屏函数，使用LCD_CLOSE_FUNC进行函数注册，先注册先执行，这里只是注册回调函数不是立刻执

行。

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

2. 关闭TCON，也就是停止发送数据，这是必要的。再延迟50ms。

3. 执行关屏代码，再延迟200ms（不需要初始化的屏，可省掉此步骤）。

4. 最后关闭电源，再延迟0ms。

  

  如下图是典型关屏时序图。

![image-20221129102917140](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_LCD_DevGuide_image-20221129102917140.png)

<center>图4-5: power off</center>

函数：LCD_cfg_panel_info

功能：配置的TCON 扩展参数，比如gamma 功能和颜色映射功能。

原型：

```
static void LCD_cfg_panel_info(__panel_extend_para_t * info)
```

TCON 的扩展参数只能在屏文件中配置，参数的定义见显示效果相关参数。

需要gamma 校正，或色彩映射，在board.dts 中将相应模块的enable 参数置1，lcd_gamma_en,lcd_cmap_en，并且填充3 个系数表，lcd_gamma_tbl, 

lcd_cmap_tbl，如下所示代码部分。注意的是：gamma，模板提供了18 段拐点值，然后再插值出所有的值（255 个）。如果觉得还不细，可以往相应表格里添加

子项。cmap_tbl 的大小是固定了，不能减小或增加表的大小。

最终生成的gamma 表项是由rgb 三个gamma 值组成的，各占8bit，目前提供的模板中，三个gamma 值是相同的。

```
static void LCD_cfg_panel_info(struct panel_extend_para *info)
{
    u32 i = 0, j = 0;
    u32 items;
    u8 lcd_gamma_tbl[][2] = {
        /* {input value, corrected value} */
        {0, 0},
        {15, 15},
        {30, 30},
        {45, 45},
        {60, 60},
        {75, 75},
        {90, 90},
        {105, 105},
        {120, 120},
        {135, 135},
        {150, 150},
        {165, 165},
        {180, 180},
        {195, 195},
        {210, 210},
        {225, 225},
        {240, 240},
        {255, 255},
    };
    u32 lcd_cmap_tbl[2][3][4] = {
    {
        {LCD_CMAP_G0, LCD_CMAP_B1, LCD_CMAP_G2, LCD_CMAP_B3},
        {LCD_CMAP_B0, LCD_CMAP_R1, LCD_CMAP_B2, LCD_CMAP_R3},
        {LCD_CMAP_R0, LCD_CMAP_G1, LCD_CMAP_R2, LCD_CMAP_G3},
        },
        {
        {LCD_CMAP_B3, LCD_CMAP_G2, LCD_CMAP_B1, LCD_CMAP_G0},
        {LCD_CMAP_R3, LCD_CMAP_B2, LCD_CMAP_R1, LCD_CMAP_B0},
        {LCD_CMAP_G3, LCD_CMAP_R2, LCD_CMAP_G1, LCD_CMAP_R0},
        },
    };
    items = sizeof(lcd_gamma_tbl) / 2;
    for (i = 0; i < items - 1; i++) {
    	u32 num = lcd_gamma_tbl[i + 1][0] - lcd_gamma_tbl[i][0];
        for (j = 0; j < num; j++) {
            u32 value = 0;
            value =
            lcd_gamma_tbl[i][1] +
            ((lcd_gamma_tbl[i + 1][1] -
            lcd_gamma_tbl[i][1]) * j) / num;
            info->lcd_gamma_tbl[lcd_gamma_tbl[i][0] + j] =
            (value << 16) + (value << 8) + value;
        }
    }
    info->lcd_gamma_tbl[255] =
        (lcd_gamma_tbl[items - 1][1] << 16) +
        (lcd_gamma_tbl[items - 1][1] << 8) + lcd_gamma_tbl[items - 1][1];
    memcpy(info->lcd_cmap_tbl, lcd_cmap_tbl, sizeof(lcd_cmap_tbl));
}
```

#### 4.2.4 延时函数说明

函数：sunxi_lcd_delay_ms / sunxi_lcd_delay_us

功能：延时函数，分别是毫秒级别/微秒级别的延时。

原型：s32 sunxi_lcd_delay_ms(u32 ms) / s32 sunxi_lcd_delay_us(u32 us)

#### 4.2.5 图像数据使能函数说明

函数：sunxi_lcd_tcon_enable / sunxi_lcd_tcon_disable

功能：打开LCD 控制器，开始刷新LCD 显示。关闭LCD 控制器，停止刷新数据。

原型：void sunxi_lcd_tcon_enable(u32 screen_id)

void sunxi_lcd_tcon_disable(u32 screen_id)

#### 4.2.6 背光控制函数说明

函数：sunxi_lcd_backlight_enable / sunxi_lcd_backlight_disable

功能：打开/关闭背光，操作的是board.dts 中lcd_bl 配置的gpio。见lcd_bl_en。

原型：void sunxi_lcd_backlight_enable(u32 screen_id)

void sunxi_lcd_backlight_disable(u32 screen_id)

函数：sunxi_lcd_pwm_enable / sunxi_lcd_pwm_disable

功能：打开/关闭pwm 控制器，打开时pwm 将往外输出pwm 波形。对应的是lcd_pwm_ch所对应的那一路pwm。

原型：s32 sunxi_lcd_pwm_enable(u32 screen_id)

s32 sunxi_lcd_pwm_disable(u32 screen_id)

#### 4.2.7 电源控制函数说明

函数：sunxi_lcd_power_enable / sunxi_lcd_power_disable

功能：打开/关闭Lcd 电源，操作的是board.dts 中的lcd_power/lcd_power1/lcd_power2（pwr_id 标识电源索引）。

原型：void sunxi_lcd_power_enable(u32 screen_id, u32 pwr_id)

void sunxi_lcd_power_disable(u32 screen_id, u32 pwr_id)

1. pwr_id = 0：对应于board.dts 中的lcd_power。
2. pwr_id = 1：对应于board.dts 中的lcd_power1。
3. pwr_id = 2：对应于board.dts 中的lcd_power2。
4. pwr_id = 3：对应于board.dts 中的lcd_power3。

函数：sunxi_lcd_pin_cfg

功能：配置lcd 的io。

原型：s32 sunxi_lcd_pin_cfg(u32 screen_id, u32 bon)

说明：配置lcd 的data/clk 等pin，对应board.dts 中的lcdd0-lcdd23/lcddclk/lcdde/lcdhsync/lcdvsync。

由于dsi 是专用pin, 所以dsi 接口屏不需要在board.dts 中配置这组pin，但同样会在此函数接口中打开与关闭对应的pin。

Bon: 1: 为开，0：为配置成disable 状态。

#### 4.2.8 DSI 相关函数说明

MIPI DSI 屏，大部分需要初始化，使用的是DSI-D0 通道的LP 模式进行初始化。提供的接口函数说明如下：

函数：sunxi_lcd_dsi_clk_enable / sunxi_lcd_dsi_clk_disble

功能：仅限dsi 接口屏使用，使能/关闭dsi 输出的高速时钟clk 信号，必须在初始化的时候调用。

原型：s32 sunxi_lcd_dsi_clk_enable(u32 scree_id)

s32 sunxi_lcd_dsi_clk_disable(u32 scree_id)

函数：sunxi_lcd_dsi_dcs_wr

功能：对屏的dcs 写操作。

原型：__s32 sunxi_lcd_dsi_dcs_wr(__u32 sel,__u8 cmd,__u8* para_p,__u32 para_num)

参数说明：

• cmd：dcs 写命令内容。

• para_p：dcs 写命令的参数起始地址。

• para_num：dcs 写命令的参数个数，单位为byte。

函数：sunxi_lcd_dsi_dcs_wr_2para

功能：对屏的dcs 写操作，该命令带有两个参数。

原型：__s32 sunxi_lcd_dsi_dcs_wr_2para(__u32 sel,__u8 cmd,__u8 para1,__u8 para2)

参数说明：

• cmd：dcs 写命令内容。

• para1：dcs 写命令的第一个参数内容。

• para2：dcs 写命令的第二个参数内容。

sunxi_dsi_dcs_wr_0para，sunxi_dsi_dcs_wr_1para，sunxi_dsi_dcs_wr_3para，sunxi_dsi_dcs_wr_4para，

sunxi_dsi_dcs_wr_5para定义与dsi_dcs_wr_2para类似，差别就是参数数量。

函数：sunxi_lcd_dsi_dcs_read

功能：dsi 读操作。

原型：s32 sunxi_lcd_dsi_dcs_read(u32 sel, u8 cmd, u8 result, u32 num_p)

参数说明：

• sel, 显示id。

• cmd, 要读取的寄存器。

• result，用于存放读取接口的数组，用户必须自行保证其有足够空间保存读取的接口。

• num_p，指针用于存放读取字节数，用户必须保证其非空指针。

#### 4.2.9 I8080 接口函数说明

显示驱动提供5 个接口函数可供使用。如下：

函数：sunxi_lcd_cpu_write

功能：设定CPU 屏的指定寄存器为指定的值。

原型：void sunxi_lcd_cpu_write(__u32 sel, __u32 index, __u32 data)

函数内容为：

```
Void sunxi_lcd_cpu_write(__u32 sel, __u32 index, __u32 data)
{
    sunxi_lcd_cpu_write_index(sel, index);
    sunxi_lcd_cpu_wirte_data(sel, data);
}
```

实现了8080 总线上的两个写操作。

sunxi_lcd_cpu_write_index 实现第一个写操作，这时PIN 脚RS（A1）为低电平，总线数据上的数据内容为参数index 的值。

Sunxi_lcd_cpu_wirte_data 实现第二个写操作，这时PIN 脚RS（A1）为高电平，总线数据上的数据内容为参数data 的值。

函数：sunxi_lcd_cpu_write_index

功能：设定CPU 屏为指定寄存器。

原型：

```
void sunxi_lcd_cpu_write_index(__u32 sel,__u32 index)
```

具体说明见sunxi_lcd_cpu_write。

函数：sunxi_lcd_cpu_write_data

功能：设定CPU 屏寄存器的值为指定的值。

原型：

```
void Sunxi_lcd_cpu_write_data(__u32 sel, __u32 data);
```

函数: tcon0_cpu_rd_24b_data

功能：读操作。

原型：

```
s32 tcon0_cpu_rd_24b_data(u32 sel, u32 index, u32 *data, u32 size)
```

参数说明：

• sel：显示id。

• index: 要读取的寄存器。

• data：用于存放读取接口的数组指针，用户必须保证其有足够空间存放数据。

• size：要读取的字节数。

#### 4.2.10 管脚控制函数说明

函数：sunxi_lcd_gpio_set_value

功能：LCD_GPIO PIN 脚上输出高电平或低电平。

原型：s32 sunxi_lcd_gpio_set_value(u32 screen_id, u32 io_index, u32 value)

参数说明：

• io_index = 0：对应于board.dts 中的lcd_gpio_0。

• io_index = 1：对应于board.dts 中的lcd_gpio_1。

• io_index = 2：对应于board.dts 中的lcd_gpio_2。

• io_index = 3：对应于board.dts 中的lcd_gpio_3。

• value = 0：对应IO 输出低电平。

• Value = 1：对应IO 输出高电平。

只用于该GPIO 定义为输出的情形。

函数：sunxi_lcd_gpio_set_direction

功能：设置LCD_GPIO PIN 脚为输入或输出模式。

原型：

```
s32 sunxi_lcd_gpio_set_direction(u32 screen_id, u32 io_index, u32 direction);
```

参数说明：

• io_index = 0：对应于board.dts 中的lcd_gpio_0。

• io_index = 1：对应于board.dts 中的lcd_gpio_1。

• io_index = 2：对应于board.dts 中的lcd_gpio_2。

• io_index = 3：对应于board.dts 中的lcd_gpio_3。

• direction = 0：对应IO 设置为输入。

• direction = 1：对应IO 设置为输出。

一部分屏需要进行初始化操作，在开屏步骤函数中，对应于LCD_panel_init 函数，提供了几种方式对屏的初始化。

对于DSI 屏，是通过DSI-D0 通道进行初始化。对于CPU 屏，是通过8080 总线的方式，使用的是LCDIO（PD,PH）进行初始化。这种初始化方式，其总线的引脚位

置定义与CPU 屏一致。

以下这些接口在屏驱动分解中提到路径的lcd_source.c 和lcd_source.h 中定义和实现。

#### 4.2.11 使用iic/spi 串行接口初始化

需要在屏驱动中注册iic/spi 设备对串行接口的访问。

使用硬件spi 对屏或者转接IC 进行初始化，如下代码片段。

首先调用spi_init 函数对spi 硬件进行初始化，spi_init 函数可以分为几个步骤，第一获取master；根据实际的硬件连接，选择spi（代码中选择了spi1），如果这一

步返回错误说spi 没有配置好，找spi 驱动负责人。第二步设置spi device，这里包括最大速度，spi 传输模式，以及每个字包含的比特数。最后调用spi_setup 完成

master 和device 的关联。

comm_out 是一个spi 传输的例子，核心就是spi_sync_transfer 函数。

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

使用硬件i2c 对LCD& 转接IC 进行初始化，初始化i2c 硬件的核心函数是i2c_add_driver，而你要做的是初始化好其参数struct i2c_driver。

it66121_id 包含设备名字以及i2c 总线索引（i2c0，i2c1…）。

it66121_i2c_probe 能进到这个函数，你就可以开始使用i2c 了。代码段里面仅仅将后面需要的参数cilent 赋值给一个全局指针变量。

it66121_match，这是dts 的match table，由于你是给disp2 加驱动，所以这里的match table 就是disp2 的match table，这个table 关系到能否使用i2c，注意不

要填错。
tv_i2c_detect 函数，这里是非常关键的，这个函数早于probe 函数被调用，只有成功被调用后才能开始使用i2c，其中strlcpy 的调用意味着成功。

normal_i2c 是从设备地址列表，填写的LCD 或者转接IC 的从设备地址以及i2c 索引。

以probe 函数是否被调用来决定你是否可以开始使用I2C。

用i2c_smbus_write_byte_data 或者i2c_smbus_read_byte_data 来读写可以满足大部分场景。

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

1.为了加快U-boot 的显示速度，开屏的几个函数之间采取异步调用的方式，原理是利用timer中断，定时调用开屏函数，所以这种情况下bootGUI 框架加载完毕并

不意味着开屏完成，而是当你见到LCD open finish的打印的时候。



建议：为了尽量利用异步调用的优点，请把需要的延时尽量在注册回调的时候指定，比如下面延时10ms 就是利用timer 异步来进行回调的，这10ms 时间，uboot 

就可以做其它事情，以达到异步调用的目的。

```
LCD_OPEN_FUNC(sel, LCD_power_on,10);
```



2.sunxi_lcd_power_enable 函数和sunxi_lcd_pin_cfg 不能在LCD_power_on之外调用，否则uboot 会异常。



严格讲，只能在用LCD_OPEN_FUNC注册的回调第一个函数里面调用。



### 4.3 RGB 接口

#### 4.3.1 概述

下面介绍全志平台的RGB 以及配置示例，至于lcd0 下面每个属性的详解细节请看硬件参数说明。

RGB 接口在全志平台又称HV 接口（Horizontal 同步和Vertical 同步）。

**对于RGB 屏的初始化:**

有些LCD 屏支持高级的功能比如gamma，像素格式的设置等，但是RGB 协议本身不支持图像数据之外的传输，所以无法通过RGB 管脚进行对LCD 屏进行配置，

所以拿到一款RGB 接口屏，要么不需要初始化命令，要么这个屏会提供额外的管脚给SoC 来进行配置，比如SPI 和I2C等。

#### 4.3.2 RGB 接口管脚

![image-20221129104718608](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_LCD_DevGuide_image-20221129104718608.png)

<center>图4-6: RGB 管脚</center>

上面这些脚具体到SoC 哪根管脚以及第几个功能（管脚复用功能）请参考pin mux 表格，管脚复用功能的名字一般以“LCDX_” 开头，其中X 是数字。

其中数据脚的数量不一定是24 根。RGB 又细分几种接口，通过设置lcd_hv_if来选择。

<center>表4-1: RGB 接口分类</center>

| 位宽    | 时钟周期 | 数颜色数量和格式     |
| ------- | -------- | -------------------- |
| 24 bits | 1 cycle  | 16.7M colors, RGB888 |
| 18 bits | 1 cycle  | 262K colors, RGB666  |
| 16 bits | 1 cycle  | 65K colors, RGB565   |
| 6 bits  | 3 cycles | 262K colors, RGB666  |
| 6 bits  | 3 cycles | 65K colors, RGB565   |

**说明**

时钟周期数的意思：是一个像素需要用多少个时钟周期发送完毕的意思。

当时钟周期为1 时，我们称这种RGB 接口为并行接口，其它的情况则是串行接口，更为普遍的原则就是只要需要多个时钟周期才能发送完一个像素的接口都是串行

接口。

如何判断是否支持24bit 的位宽，最简单的方式就是在pinmux 表格中数一数数据脚的数量，如果有24 根则支持24bit，如果只有18 根则支持18bit。



**硬件连接**

对于并行RGB 的接口，当位宽小于24 时，硬件连接应该选择连接每个分量中的高位而放弃低位，这样做的原因是损失较少的颜色数量。

对于串行RGB 接口，硬件连接可参考RGB 和I8080 管脚配置示意图中sync RGB 那几列。

RGB 接口有两种同步方式，根据经验来说尽量使用第二种方式，硬件上请保证连接好DE 脚。

1. Hsync+Vsync
2. DE（Data Enable）

#### 4.3.3 并行RGB 接口配置示例

当我们配置并行RGB 接口时，在配置里面并不需要区分是24 位，18 位和16 位，最大位宽是哪种是参考pin mux 表格，如果LCD 屏本身支持的位宽比SoC 支持的

位宽少，当然只能选择少的一方。

因为不需要初始化，RGB 接口极少出现问题，重点关注lcd 的timing 的合理性，也就是lcd_ht，lcd_hspw，lcd_hbp，lcd_vt，lcd_vspw和lcd_vbp这个属性的合理

性。

下面是典型并行RGB 接口board.dts 配置示例，其中用空行把配置分成几个部分

1. 第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏驱动来初始化，这里是default_lcd，是针对不需要初始化设置的

  RGB 屏

2. 第二部分决定下面的配置是一个并行RGB 的配置。

3. 第三部分决定了SoC 中的LCD 模块发送时序，请查看屏时序参数说明。

4. 第四部分决定了背光（pwm 和lcd_bl_en）。请看背光相关参数。

5. 第五部分是显示效果部分的配置，如果非24 位的RGB，那么一般情况下需要设置lcd_frm。

6. 第六部分就是电源和管脚配置。是用RGB666 还是RGB888，需要根据实际pinmux 表来决定，如果该芯片只有18 根rgb 数据则只能rgb18。请看电源和管脚

  参数。

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

4.3.4 串行RGB 接口的典型配置

串行RGB 是相对于并行RGB 来说，而并不是说它只用一根线来发数据，只要通过多个时钟周期才能把一个像素的数据发完，那么这样的RGB 接口就是串行RGB。

同样与并行RGB 接口一样，配置中并不需要也无法体现具体是哪种串行RGB 接口，需要做的就是把硬件连接对。

下面是典型串行RGB 接口board.dts 配置示例，它只有8 根数据脚，其中用空行把配置分成几个部分

1. 第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏驱动来初始化。

2. 第二分部决定下面的配置是一个串行RGB 的配置。

3. 第三部分决定了SoC 中的LCD 模块发送时序，请查看屏时序参数说明。

  **技巧**
  这里需要注意的是，对于该接口，SoC 总共需要三个周期才能发完一个pixel，所以我们配置时序的时候，需要满足lcd_dclk_freq*3=lcd_ht*lcd_vt*60，或者*

  *lcd_dclk_freq=lcd_ht*3*lcd_vt*60要么3 倍lcd_ht要么3倍lcd_dclk_freq。

4. 第四部分决定了背光。就是pwm 和lcd_bl_en。请看背光相关参数

5. 第五部分是显示效果方面的设置。

6. 第六部分管脚和电源的定义。请看电源和管脚参数。

**说明**

下面实例的lcd driver IC 是stv7789v，是需要初始化，初始化的接口协议是SPI，所以这多了几根spi 管脚配置，驱动里面用gpio 模拟spi 协议，所以这里都是配置

gpio 功能。

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

MIPI-DSI，即Mobile Industry Processor Interface Display Serial Interface，移动通信行业处理器接口显示串行接口。

对于用户来说，需要了解：

1. Command mode，类似MPU 接口，需要IC 内部有GRAM 来缓冲。

2. Video mode。类似RGB 接口，没有GRAM，需要不停往panel 刷数据。其中video mode又分为三个子mode。

  • Non-burst mode with sync pulses

  • Non Burst mode with sync Events

  • Burst mode。简单理解就是有效数据比率更高，传输效率更高。

3. lane 的意思是指一对差分管脚。

#### 4.4.2 MIPI-DSI 的管脚

MIPI-DSI 的管脚是在大部分IC 中是专用，在board.dtsi 里面不需要配置，只要硬件上连接好就行。

但是有一部分IC 的DSI 管脚不是专用的，与其它功能的脚复用，这个时候就需要配置好pinctrl-0和pinctrl-1。

mipi-dsi 的管脚是差分的，分为两种管脚，一种是时钟管脚，另外一种是数据管脚，数据管脚的数量是可变的，数量的单位是lane，每一条lane 实际包含两条线。

一般来说LCD 屏说明书里面的说的lane 的数量是指数据管脚的数量不包括时钟管脚。比如说某4 lane MIPI-DSI 屏就总共有(4+1)*2根脚。

#### 4.4.3 MIPI-DSI 的电源

一般都有一路电源供给MIPI-DSI 这个模块，你可以理解为管脚电，也可以理解成模块电，不同IC 这路电的电压要求可能不同，一旦确定IC 型号之后，这路电的电

压就不变，如果擅自改变此路电的电压可能导致模块异常。

![image-20221129105502705](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_LCD_DevGuide_image-20221129105502705.png)

<center>图4-7: pinmux</center>

#### 4.4.4 判断是否支持某款MIPI-DSI 屏

​	1.分辨率限制。有lane 的速度限制，我们可以得到最大分辨率的限制，计算公式如下，只要lane_speed 不超过上面IC 规格规定的速度，那么理论上是支持的，

请查看IC 规格。

lane_speed=lcd_vt * lcd_ht * fps * bit_per_pixel / lane_num / 1e9

• 单位：Gbps。

• fps: 期望刷新率，通过屏手册可知道，一般是60。请看lcd_dclk_freq。

• bit_per_pixel: 每个像素包含的比特数量，一般是24 或者18，通过lcd_dsi_format来设置。

• lane_num:lane 数量，通过lcd_dsi_lane来设置。

• 1e9:1000000000 的科学计数写法。

2. 选择分辨率的同时需要考虑系统带宽，DE 能力，所以即使接口方面支持这个分辨率，对于整个系统来说不一定支持，比如说硬件为了节省成本选择了一款速

  度很慢的DDR 内存然后同时又想选择高分辨率的屏幕，很明显这是不现实的。

3. lane 数量限制。绝大部分全志科技IC 最大支持4 lane 的MIPI-DSI，如果你看到该款屏超过4 lane 就肯定不支持了。少数IC 最大支持8 lane，应该选择该款

  IC。

4. MIPI-DSI 标准不兼容。请查看IC 规格。

#### 4.4.5 计算MIPI-DSI 时钟lane 频率

使用示波器测量MIPI-DSI 的时钟信号，确定其频率是否满足屏的需求。

首先，我们由给定的像素时钟和lane 数量，可以计算出理论CLK 信号的频率，如下公式：

```
Freq_dsi_clk = (Dclk * colordepth * 3 / lane ) / 2
```

1. Freq_dsi_clk：我们要测量的dsi 时钟脚的频率。单位MHz。
2. Dclk：像素时钟。由lcd_ht*lcd_vt*fps/1e6公式算出来。
3. Colordepth：颜色深度，一般是8 或者6。
4. 乘以3 表示RGB 分量3 个。
5. Lane：dsi 的lane 数量。
6. 除以2：是因为dsi 时钟是双沿采样。

#### 4.4.6 MIPI-DSI Video mode 屏配置示例

绝大多数MIPI-DSI 屏的配置都是用video mode。

下面是典型MIPI-DSI video mode 的board.dts 配置示例，其中用空行把配置分成几个部分

1. 第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏驱动来初始化。
2. 第二部分，决定该配置是dsi 接口，而且dsi 接口使用的是video mode。
3. 第三部分，决定了SoC 中的LCD 模块发送时序，请查看屏时序参数说明。
4. 第四部分，背光相关的设置。请看背光相关参数。
5. 第五部分，dsi 接口的详细设置。
6. 第六部分，显示效果相关的设置。
7. 第七部分，管脚和电源设置。请看电源和管脚参数。

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

根据分辨率的高低通常分为几种模式来配置。1080p 分辨率及其以下：只需要设置lcd_dsi_if 来控制就行。Command mode 一般是低分辨率屏，而video mode 

和burst mode 则是用于高分辨率的。如果分辨率达到2k，则需要额外的设置。

分辨率达到2k 以上的屏，实际上需要多达8 条数据lane 才能正常显示，其中四条lane 发送一副图像中的奇像素，另外一副图像发送偶像素。

**说明**

注意只有部分IC 支持超高分辨率，具体查看芯片规格中的MIPI-DSI 部分

下面是MIPI-DSI 高分辨超高分辨率（大于2k）board.dts 配置示例，其中用空行把配置分成几个部分

1. 第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏驱动来初始化。
2. 第二部分，决定该配置是dsi 接口，而且dsi 接口使用的是video mode。
3. 第三部分，决定了SoC 中的LCD 模块发送时序，请查看屏时序参数说明。
4. 第四部分，背光相关的设置，请看背光相关参数。
5. 第五部分，dsi 接口的详细设置。

**说明**
lcd_dsi_lane 依旧设置成4 条lane 的原因，是因为这个是设置一个dsi 的lane 数量，这个屏要用两个dsi。加起来就是8 条lane。

此时lcd_tcon_mode, lcd_dsi_port_num 和lcd_tcon_en_odd_even_div 三个选项需要特别设置，点击查看具体含义，如果是1080p 及其以下分辨率的屏(只用

4lane 或者以下的)，那么这三个配置默认0 即可。

6. 第六部分，显示效果部分的设置。
7. 第七部分，是管脚和电源的配置。请根据电路图来配置。请看电源和管脚参数。

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

Command mode 下的DSI 屏类似与I8080 接口，屏内部带RAM 用于缓冲和图像处理，这种情况一般都需要用屏的te 脚来触发vsync 中断，所以与其它类型的DSI 

屏不同的是，这里需要设置lcd_vsync 脚，屏的te 脚就连到lcd_vsync 上，并且lcd_dsi_te 设置成1。

te 脚的设置非常关键，一般来说如果屏有te 脚，则必须连上，否则在显示动态画面的时候会画面会撕裂，而且软件无法解决，直接造成最终硬件无法量产的结

果。

这里只列举出与MIPI-DSI video mode 不同的关键之处，其它参考上一小节。

1. 第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏驱动来初始化。
2. 第二部分，决定该配置是dsi 接口，而且lcd_dsi_if设置成1 表明command mode。
3. 第三部分，决定了SoC 中的LCD 模块发送时序，请查看屏时序参数说明。
4. 第四部分，背光相关的设置。请看背光相关参数。
5. 第五部分，dsi 接口的详细设置。lcd_dsi_te，这里设置为1 表示使能te 触发。
6. 第六部分，显示效果相关的设置。
7. 第七部分，管脚和电源设置。lcd_vsync，这里是te 脚，硬件上需要将这根脚连接到屏的te脚，软件上需要将其设置为vsync 功能。请看电源和管脚参数。



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

实际场景是两个物理屏，每个屏是1080p，每个屏都是4 条lane，要求的是两个屏各自显示一帧图像的左右一半，由于宽高比和横竖屏以及DE 处理能力的因素，

一个DE+ 一个tcon+ 两个DSI 已经无法满足，必须用两个tcon 各自驱动一个dsi，但是两路显示必须要同步，这就需要用到两个tcon 的同步模式。

1. LCD0 标记为slave tcon，它由master tcon 来驱动（设置lcd_tcon_mode）。

2. LCD1 标记为master tcon，并且负责两个屏的所有电源，背光，管脚的开关。

3. 把管脚，电源等都放到LCD1 开，LCD0 先开，对应模块寄存器都初始化，但是电源不开，然后开LCD1，LCD1 使能就会触发LCD0 一起发数据。这样做到同时

  亮灭。

**说明**

注意：仅有极少IC 支持该模式

```
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

Intel 8080 接口屏(又称MCU 接口) 很老的协议，一般用在分辨率很小的屏上。

信号线：

• CS 片选信号，决定该芯片是否工作。

• RS 寄存器选择信号，低表示选择index 或者status 寄存器，高表示选择控制寄存器。实际场景中一般接SoC 的LCD_DE 脚（数据使能脚）。

• /WR （低表示写数据) 数据命令区分信号，也就是写时钟信号，一般接SoC 的LCD_CLK 脚。

• /RD （低表示读数据）数据读信号，也就是读时钟信号，一般接SoC 的LCD_HSYNC 脚。

• RESET 复位LCD（用固定命令系列0 1 0 来复位)。

• Data 双向传输的数据总线。

I8080 根据的数据位宽接口有8/9/16/18，连哪些脚参考，即使位宽一样，连的管脚也不一样，还要考虑的因素是rgb 格式。

1. RGB565，总共有65K 这么多种颜色。

2. RGB666，总共有262K 那么多种颜色。

3. 9bit 固定为262K。

  

  从屏手册得知：数据位宽，颜色数量之和，参考RGB 和I8080 管脚配置示意图，进行硬件连接。

  

#### 4.5.2 I8080 接口屏典型配置示例

下面是典型是一个RGB565 的，位宽为8 位的I8080 接口的屏的board.dts 配置示例

​	1.第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏驱动来初始化。

​	2.第二部分，决定该配置是I8080 接口，而且是8bit/2cycle 格式RGB565。

**技巧**
为什么叫做8bit/2cycle RGB565 呢，首先它的格式是RGB565，也就是一个像素是16bit，然后它是8bit 的位宽，就需要两个时钟周期才能发完一个像素，所以才

叫2 cycle。

3. 第三部分，决定了SoC 中的LCD 模块发送时序，请查看屏时序参数说明。这里比较特殊的是设置像素时钟要满足以下公式：lcd_dclk_freq*2>=lcd_ht*lcd_vt*fps，或者lcd_dclk_freq=lcd_ht
   *2*lcd_vt*60, 也就是要么双倍lcd_ht要么双倍lcd_dclk_freq。

4. 第四部分，背光相关的设置。请看背光相关参数。

5. 第五部分，cpu 接口的详细设置。这里使能了lcd_cpu_te和lcd_cpu_mode，意思是使用te触发和规定了触发间隔。这是非常关键的设置。

6. 第六部分，显示效果相关的设置。这里使能了lcd_frm也是比较关键的设置，详细意思点击查看。

7. 第七部分，管脚和电源设置。这里为了用te 触发，同样需要设置lcd_vsync，该脚功能定义已经包括在pinctrl-0 中。这里自定义了一组管脚。参考RGB 和

I8080 管脚配置示意图，通过确定I8080 的位宽，像素格式（颜色数量），在表中确定需要连接哪些管脚。请看电源和管脚参数。

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

LVDS 即Low Voltage Differential Signaling 是一种低压差分信号接口。

#### 4.6.2 LVDS Single link 典型配置

LVDS 接口，lcd0 对应的lvds 管脚和lcd1 对应的lvds 管脚是固定而且不一样。

由于lvds 协议不具备传输数据之外的能力，一般屏端不需要任何初始化，只需要初始化SoC 端即可。所以这里的lcd_driver_name 依旧是”default_lcd”，当然你可

以为初始化的启动延时做专门的优化。

下面是典型是single link lvds 屏的board.dts 配置示例，其中用空行把配置分成几个部分



1. 第一部分，决定该配置是否使用，以及使用哪个屏驱动，lcd_driver_name 决定了用哪个屏
   驱动来初始化。

2. 第二部分，决定该配置是lvds 接口，而且是single link。
   **技巧**
   如果Dual Link 的屏，那么除了要改lcd_lvds_if 为1 之外，管脚方面还要把lcd1 的管脚一起搬到下面去，也就是总共需要配置PD0 到PD9，和配置PD10 到

  PD19 总共二十根脚为lvds 管脚功能（功能3）。当然屏的timing 也是要根据屏来改的。

3. 第三部分，决定了SoC 中的LCD 模块发送时序，请查看屏时序参数说明。

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

如果Dual Link 的屏:

1. lcd_lvds_if设置为1（场景1）或者2（场景2）。
2. 管脚配置方面，也从4 data lane 变成8 data lane，包括clk lane 总共20 根管脚。

场景1，物理上连接一个屏，8 data lane，SoC 向每4 条lane 传输一半的像素，奇数像素或者偶数像素。

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

场景2（部分IC 支持），物理上连接两个屏，每个屏各自4 条lane，两个屏是一样型号，分辨率和timing 一样，这时候部分IC 支持将全部像素发到每个屏上，实现

双显（信号上的双显），注意这时候lcd timing 是一个屏的timing, lcd_lvds_if 为2。

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

### 4.7 RGB 和I8080 管脚配置示意图

![image-20221129120014562](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_LCD_DevGuide_image-20221129120014562.png)

<center>图4-8: pinmux</center>

### 4.8 从sys_config.fex 到board.dtsi 的迁移注意事项

为了规范等原因，部分平台将配置放在board.dtsi 中实现。下面说明了修改board.dtsi 的注意事项。

#### 4.8.1 管脚定义

在配置RGB 屏或者LVDS 屏时，现在不再需要复杂的定义，也不需要了解到底哪些管脚需要配置，也不需要lcd0_suspend节点了。其中rgb24_pins_a这个名字是

定义好的，直接用即可，一般LCD屏直接可用的配置会在注释中写明，你可以在内核目录下arch/arm/boot/dts或者arch/arm64/boot/dts下的平台-pinctrl.dtsi 文

件中找。

例子:

```
pinctrl-0 = <&rgb24_pins_a>;
pinctrl-1 = <&rgb24_pins_b>;//休眠时候的定义，io_disable
```

当然，你也可以自定义一组脚，写在board.dtsi 中，只要名字不要和现有名字重复就行。

为了规范，我们将在所有平台保持一致的名字，其中后缀为a 即为管脚使能，b 的为io_disable用于设备关闭时。

目前有以下管脚定义可用：

<center>表4-2: 显示管脚名称表</center>

| 管脚名称                                      | 描述                                               |
| --------------------------------------------- | -------------------------------------------------- |
| rgb24_pins_a 和rgb24_pins_b                   | RGB 屏接口，而且数据位宽是24，RGB888               |
| rgb18_pins_a 和rgb18_pins_b                   | RGB 屏接口，而且数据位宽是16，RGB666               |
| lvds0_pins_a和lvds0_pins_b                    | Single link LVDS 接口0 管脚定义（主显lcd0）        |
| lvds1_pins_a 和lvds1_pins_b                   | Single link LVDS 接口1 管脚定义（主显lcd0)         |
| lvds2link_pins_a 和lvds2link_pins_b           | Dual link LVDS 接口管脚定义（主显lcd0）            |
| lvds2_pins_a 和lvds2_pins_b                   | Single link LVDS 接口0 管脚定义（主显lcd1）        |
| lvds3_pins_a 和lvds3_pins_b                   | Single link LVDS 接口1 管脚定义（主显lcd1）        |
| lcd1_lvds2link_pins_a 和lcd1_lvds2link_pins_b | Dual link LVDS 接口管脚定义（主显lcd1）            |
| dsi4lane_pins_a 和dsi4lane_pins_b             | DSI 屏接口管脚定义，4lane，如果是其它lane 数量，只 |

#### 4.8.2 电源定义

电源定义在旧的SDK 中并不需要注意什么，还是直接把axp 的别名字符串赋值给想lcd_power这样的属性上即可，但是新的SDK 中，如果需要使用某路电源必须先

在disp 节点中定义，然后lcd 部分使用的字符串则要和disp 中定义的一致。比如下面的例子：

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

其中”-supply” 是固定的，它之前的字符串则是随意的，不过建议取有意义的名字。而后面的像<&reg_sw> 则必须在board.dtsi 的regulator0 节点中找到。

然后lcd0 节点中，如果要使用reg_sw，则类似下面这样写就行，dc1sw 对应dc1sw-supply。

```
lcd_power=”dc1sw”
```

由于u-boot 中也有axp 驱动和display 驱动，它们和内核一样，都是读取同份配置，为了能互相兼容，取名的时候，有以下限制。

在u-boot 2018 中，axp 驱动只认类似bldo1 这样从axp 芯片中定义的名字，所以命名xxxsupply的时候最好按照这个axp 芯片的定义来命名。

#### 4.8.3 其它注意事项

board.dtsi 里面可能只有lcd0 没有lcd1，或只有tv0 没有tv1，这时候你要添加的话，需要参考内核目录arch/arm/boot/dts 或者arch/arm64/boot/dts 下对应的平

台.dtsi 文件。其中最关键的是@ 后面那串地址必须与内核中定义一致，比如：

```
lcd1: lcd1@01c0c000
```

