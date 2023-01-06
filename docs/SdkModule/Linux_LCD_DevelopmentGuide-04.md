## 6 调试方法

系统起来之后可以读取sysfs 一些信息，来协助调试。

### 6.1 加快调试速度的方法

很明显，如果你在安卓上调试LCD 屏会比较不方便，安卓编译时间和安卓固件都太过巨大，每次修改内核后，可能都要经过10 几分钟都才能验证，这样效率就太

低下了，可用采用如下方法：

1. 使用linux 固件而不是安卓固件。SDK 是支持仅仅编译linux 固件，一般是配置lichee 或者longan 的时候选择linux，打包的时候，用lichee 或者longan 根目录

  下的build.sh 来打包就行。因为linux 内核小得多，编译更快，更方便调试。

2. 使用内核来调试LCD 屏。我们知道Uboot 和内核都需要添加LCD 驱动，这样才能快速显示logo，但是uboot 并不方便调试，所以有时候我们需要把uboot 的

  显示驱动关掉，专心调试内核的LCD 驱动，调好之后才移植到uboot，另外这样做的一个优点是，我可以非常方便的修改lcd timing 而不需要重烧固件。就是

  利用uboot 命令的fdt 命令修改device tree。
  比如说：

  ```
  fdt set lcd0 lcd_hbp <40>
  ```

  更多命令见fdt help。

如何关闭uboot 显示呢，一般是在uboot 源码路径下inlcude/configs/平台.h 中，注释掉CONFIG_SUNXI_MODULE_DISPLAY 即可，如果是uboot 2018 则是注释

掉configs/平台_defconfig 中CONFIG_DISP2_SUNXI。

### 6.2 查看显示信息

以下信息是所有信息中最重要的。

```
cat /sys/class/disp/disp/attr/sys
screen 0:
de_rate 297000000 hz, ref_fps:60
mgr0: 1280x800 fmt[rgb] cs[0x204] range[full] eotf[0x4] bits[8bits] err[0] force_sync[0]
unblank direct_show[false]
lcd output backlight( 50) fps:60.9 1280x 800
err:0 skip:31 irq:1942 vsync:0 vsync_skip:0
BUF enable ch[1] lyr[0] z[0] prem[N] a[globl 255] fmt[ 8] fb[1280, 800;1280,
800;1280, 800] crop[ 0, 0,1280, 800] frame[ 0, 0,1280, 800] addr[ 0,
0, 0] flags[0x 0] trd[0,0]
```

**lcd output**

表示当前显示接口是LCD 输出。

**1280x800**

表示当前LCD 的分辨率，与board.dts 中lcd0 的设置一样。

**ref_fps:60**

是根据你在board.dts的lcd0填的时序算出来的理论值。

**fps:60.9**

后面的数值是实时统计的，正常来说应该是在60(期望的fps) 附近，如果差太多则不正常，重新检查屏时序，和在屏驱动的初始化序列是否有被调用到。

**irq:1942**

这是vsync 中断的次数，每加1 都代表刷新了一帧，正常来说是一秒60（期望的fps）次，重复cat sys，如果无变化，则异常。

**BUF**

开头的表示图层信息，一行BUF 表示一个图层，如果一个BUF 都没有出现，那么将是黑屏，不过和屏驱动本身关系就不大了，应该查看应用层& 框架层。

**err:0**

这个表示缺数，如果数字很大且一直变化，屏幕会花甚至全黑，全红等。

**skip:31**

这个表示跳帧的数量，如果这个数值很大且一直变化，有可能卡顿，如果数字与irq 后面的数字一样，说明每一帧都跳，会黑屏（有背光）。

### 6.3 查看电源信息

查看axp 某一路电源是否有enable 可以通过下面命令查看。当然这个只是软件的，实际还是用万用表量为准。

```
cat /sys/class/regulator/dump
pmu1736_ldoio2 : disabled 0 700000 supply_name:
pmu1736_ldoio1 : disabled 0 700000 supply_name:
pmu1736_dc1sw : enabled 1 3300000 supply_name: vcc-lcd
pmu1736_cpus : enabled 0 900000 supply_name:
pmu1736_cldo4 : disabled 0 700000 supply_name:
pmu1736_cldo3 : disabled 0 700000 supply_name:
pmu1736_cldo2 : enabled 1 3300000 supply_name: vcc-pf
pmu1736_cldo1 : disabled 0 700000 supply_name:
pmu1736_bldo5 : enabled 2 1800000 supply_name: vcc-cpvin vcc-pc
pmu1736_bldo4 : disabled 0 700000 supply_name:
pmu1736_bldo3 : disabled 0 700000 supply_name:
pmu1736_bldo2 : disabled 0 700000 supply_name:
pmu1736_bldo1 : disabled 0 700000 supply_name:
pmu1736_aldo5 : enabled 0 2500000 supply_name:
pmu1736_aldo4 : enabled 0 3300000 supply_name:
pmu1736_aldo3 : enabled 1 1800000 supply_name: avcc
pmu1736_aldo2 : enabled 0 1800000 supply_name:
pmu1736_aldo1 : disabled 0 700000 supply_name:
pmu1736_rtc : enabled 0 1800000 supply_name:
pmu1736_dcdc6 : disabled 0 500000 supply_name:
pmu1736_dcdc5 : enabled 0 1480000 supply_name:
pmu1736_dcdc4 : enabled 1 900000 supply_name: vdd-sys
pmu1736_dcdc3 : enabled 0 900000 supply_name:
pmu1736_dcdc2 : enabled 0 1160000 supply_name:
pmu1736_dcdc1 : enabled 4 3300000 supply_name: vcc-emmc vcc-io vcc-io vcc-io
```

6.4 查看pwm 信息

pwm 的用处这里是提供背光电源。

```
cat /sys/kernel/debug/pwm
platform/7020c00.s_pwm, 1 PWM device
pwm-0 ((null) ): period: 0 ns duty: 0 ns polarity: normal
platform/300a000.pwm, 2 PWM devices
pwm-0 (lcd ): requested enabled period: 20000 ns duty: 3984 ns polarity:
normal
pwm-1 ((null) ): period: 0 ns duty: 0 ns polarity: normal
```

上面的“requested enabled” 表示请求并且使能了，括号里面的lcd 表示是由lcd 申请的。

### 6.5 查看管脚信息

```
cat /sys/kernel/debug/pinctrl/pio/pinmux-pins
pin 227 (PH3): twi1 (GPIO UNCLAIMED) function io_disabled group PH3
pin 228 (PH4): (MUX UNCLAIMED) (GPIO UNCLAIMED)
pin 229 (PH5): (MUX UNCLAIMED) pio:229
pin 230 (PH6): (MUX UNCLAIMED) pio:230
pin 231 (PH7): (MUX UNCLAIMED) pio:231
```

上面的信息我们知道PH5，PH6 这些IO 被申请为普通GPIO 功能，而PH3 被申请为twi1。

### 6.6 查看时钟信息

```
cat /sys/kernel/debug/clk/clk_summary
```

这个命令可以看哪个时钟是否使能，然后频率是多少。

与显示相关的是tcon，pll_video，mipi 等等。

```
cat /sys/kernel/debug/clk/clk_summary | grep tcon
cat /sys/kernel/debug/clk/clk_summary | grep pll_video
cat /sys/kernel/debug/clk/clk_summary | grep mipi
```

### 6.7 查看接口自带colorbar

显示是一整条链路，中间任何一个环节出错，最终的表现都是显示异常，图像显示异常几个可能

原因：

1. 图像本身异常。
2. 图像经过DE（Display Engine）后异常。
3. 图像经过接口模块后异常。这是我们关注的点。

有一个简单的方法可以初步判断，接口模块（tcon 和dsi 等）可以自己输出内置的一些patten（比如说彩条、灰阶图、棋盘图等），当接口输出这些内置patten 

的时候，如果这时候显示就异常，这说明了：

1. LCD 的驱动或者配置有问题。
2. LCD 屏由于外部环境导致显示异常。

显示自带patten 的方式：

在linux-4.9 及其以上版本的内核，disp 的sysfs 中有一个attr 可以直接操作显示:

```
echo X > /sys/class/disp/disp/attr/colorbar
```

上面的操作是显示colorbar，其中的X 可以是0 到8，对应的含义如下图所示：

![image-20221130180522997](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_LCD_DevGuide_image-20221130180522997.png)

<center>图6-1: colorbar</center>

如果有多个显示设备，想让第二个显示设备显示colorbar 的话，那么先：

```
echo 1 > /sys/class/disp/disp/attr/disp
```

然后再执行上面操作。

如果没有这个attr 的话，可以直接操作寄存器，也就是操作tcon 寄存器的040 偏移的最低3位。

在linux 下，cd /sys/class/sunxi_dump 然后：

```
echo 0x06511040 > dump;cat dump
```

这样会打印当前tcon 的040 偏移寄存器的值，然后在上面值的基础上修改最低3 位为上图的值即可，修改方式示例：

```
echo 0x06511040 0x800001f1 > write
```

注意tcon 的基地址不一定是0x06511000，不同平台不一样，请参考SoC 文档获取tcon 的基地址。

### 6.8 DE 截屏

显示出现异常的时候，有可能是下面三个原因：

1. SoC 端屏接口模块+LCD 屏出现了问题。
2. 图像经过SoC 端图像合成模块(DE）处理后出现了问题。
3. 图像源本身就有问题。
   本节介绍，确认图像源本身没问题的前提下，如何进一步确认是否经过DE 处理之后图像是否有问题。

语法：

```
echo 屏幕索引> /sys/class/disp/disp/attr/disp
echo 路径/bmp文件名> /sys/class/disp/disp/attr/capture_dump
```

第一个echo 的作用是确定捕捉哪个显示的，“屏幕索引” 可选取值是0 或者1。

第二个echo 作用是生成截屏bmp 文件，确保“路径” 是可写的，有剩余空间的。

比如：

```
echo 0 > /sys/class/disp/disp/attr/disp
echo /data/xx.bmp > /sys/class/disp/disp/attr/capture_dump
```

这样就会在/data 目录下生成screen 0 的bmp 截图，文件名是xx.bmp。

除了bmp 之外，还支持保存raw 数据，包括RGB 和YUV 颜色空间，它们以后缀来区分，用法

如下：

```
# 截取yuv420p颜色空间的raw数据。
echo /data/xx.yuv420_p > /sys/class/disp/disp/attr/capture_dump
# 截取yuv420_sp_uvuv颜色空间的raw数据。
echo /data/xx.yuv420_sp_uvuv > /sys/class/disp/disp/attr/capture_dump
# 截取yuv420_sp_vuvu颜色空间的raw数据。
echo /data/xx.yuv420_sp_vuvu > /sys/class/disp/disp/attr/capture_dump
# 截取yuv420_sp_vuvu颜色空间的raw数据。
echo /data/xx.yuv420_sp_vuvu > /sys/class/disp/disp/attr/capture_dump
# 截取argb8888颜色空间的raw数据。
echo /data/xx.argb8888 > /sys/class/disp/disp/attr/capture_dump
# 截取abgr8888颜色空间的raw数据。
echo /data/xx.abgr8888 > /sys/class/disp/disp/attr/capture_dump
# 截取rgb888颜色空间的raw数据。
echo /data/xx.rgb888 > /sys/class/disp/disp/attr/capture_dump
# 截取bgr888颜色空间的raw数据。
echo /data/xx.bgr888 > /sys/class/disp/disp/attr/capture_dump
# 截取rgba8888颜色空间的raw数据。
echo /data/xx.rgba8888 > /sys/class/disp/disp/attr/capture_dump
# 截取bgra8888颜色空间的raw数据。
echo /data/xx.bgra8888 > /sys/class/disp/disp/attr/capture_dump
```

注意：这个功能只有linux-4.9 以及后续的内核才支持这个功能。