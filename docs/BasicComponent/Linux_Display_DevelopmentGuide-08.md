# Linux_Display 调试
## 10 调试

### 10.1 查看显示模块的状态

```
cat /sys/class/disp/disp/attr/sys
```

示例如下：

```
# cat /sys/class/disp/disp/attr/sys
screen 0:
de_rate 432000000 Hz /* de 的时钟频率*/, ref_fps=50 /* 输出设备的参考刷新率*/
hdmi output mode(4) fps:50.5 1280x 720
err:0 skip:54 irq:21494 vsync:0
BUF enable ch[0] lyr[0] z[0] prem[N] a[globl 255] fmt[ 1] fb
[1920,1080;1920,1080;1920,1080] crop[ 0, 0,1920,1080] frame[ 32, 18,1216, 684]
addr[716da000, 0, 0] flags[0x 0] trd[0,0]
screen 1:
de_rate 432000000 Hz /* de 的时钟频率*/, ref_fps=50 /* 输出设备的参考刷新率*/
tv output mode(11) fps:50.5 720x 576 /* TV 输出| 模式为（11：PAL） | 刷新率为：50.5Hz | 分辨
率为：720x576 */
err:0 skip:54 irq:8372 vsync:0
BUF enable ch[0] lyr[0] z[0] prem[Y] a[globl 255] fmt[ 0] fb[ 720, 576; 720, 576; 720,
576] crop[ 0, 0, 720, 576] frame[ 18, 15, 684, 546]
addr[739a8000, 0, 0] flags[0x 0] trd[0,0]
acquire: 225, 2.6 fps
release: 224, 2.6 fps
display: 201, 2.5 fps
```

图层各信息描述如下：

```
BUF: 图层类型，BUF/COLOR，一般为BUF，即图层是带BUFFER 的。COLOR 意思是显示一个纯色的画面，不带
BUFFER。
enable: 显示处于enable 状态。
ch[0]: 该图层处于blending 通道0。
lyr[0]: 该图层处于当前blending 通道中的图层0。
z[0]: 图层z 序，越小越在底部，可能会被z 序大的图层覆盖住。
prem[Y]: 是否预乘格式，Y 是，N 否。
a: alpha 参数， globl/pixel/; alpha 值。
fmt: 图层格式，值64 以下为RGB 格式；以上为YUV 格式，常见的72 为YV12,76 为NV12。
fb: 图层buffer 的size，width,height，三个分量。
crop: 图像buffer 中的裁减区域， [x,y,w,h]。
frame: 图层在屏幕上的显示区域，[x,y,w,h]。
addr: 三个分量的地址。
flags: 一般为0， 3D SS 时0x4, 3D TB 时为0x1, 3D FP 时为0x2。
trd: 是否3D 输出，3D 输出的类型（HDMI FP 输出时为1）各counter 描述如下：
err： de 缺数的次数，de 缺数可能会出现屏幕抖动，花屏的问题。de 缺数一般为带宽不足引起。
skip: 表示de 跳帧的次数，跳帧会出现卡顿问题。跳帧是指本次中断响应较慢，de 模块判断在本次中断已经接近或
者超过了消隐区，将放弃本次更新图像的机会，选择继续显示原有的图像。
irq: 表示该通路上垂直消隐区中断执行的次数，一直增长表示该通道上的timing。
controller 正在运行当中。
vsync:表示显示模块往用户空间中发送的vsync 消息的数目，一直增长表示正在不断地发送中。
acquire/release/display 含义如下,只在android 方案中有效。
acquire： 是hw composer 传递给disp driver 的图像帧数以及帧率，帧率只要有在有图像更新时才有效，静止
时的值是不准确的。
release: 是disp driver 显示完成之后，返还给android 的图像帧数以及帧率，帧率只要有在有图像更新时才有
效，静止时的值是不准确的。
display： 是disp 显示到输出设备上的帧数以及帧率，帧率只要有在有图像更新时才有效，静止时的值是不准确的如
果acquire 与release 不一致，说明disp 有部分图像帧仍在使用，未返还，差值在1~2 之间为正常值。二者不能
相等，如果相等，说明图像帧全部返还，显示将会出。
现撕裂现象。如果display 与release 不一致，说明在disp 中存在丢帧情况，原因为在一个active 区内
hwcomposer 传递多于一帧的图像帧下来。
```

调试说明：

```
1. 对于android 系统，可以dumpsys SurfaceFlinger 打印surface 的信息，如果信息与disp 中sys 中的信息不一致，很大可能是hwc 的转换存在问题。
2. 如果发现图像刷新比较慢，存在卡顿问题，可以看一下输出设备的刷新率，对比一下ref_fps 与fps 是否一致，如果不一致，说明tcon 的时钟频率或timing 没配置正确。如果ref_fps 与屏的spec 不一致，则需要检查sys_config 中的时钟频率和timing配置是否正确。屏一般为60Hz，而如果是TV 或HDMI，则跟模式有关，比较常
见的为60/50/30/24Hz。
如果是android 方案，还可以看一下display 与release 的counter 是否一致，如果相差太大，说明android送帧不均匀，造成丢帧。
3. 如果发现图像刷新比较慢，存在卡顿问题，也需要看一下skip counter，如果skip counter 有增长，说明现在的系统负荷较重，对vblank 中断的响应较慢，出现跳帧，导致了图像卡顿问题。
4. 如果屏不亮，怀疑背光时，可以看一下屏的背光值是否为0。如果为0，说明上层传递下来的背光值不合理；如果不为0，背光还是不亮，则为驱动或硬件问题了。硬件上可以通过测量bl_en 以及pwm 的电压值来排查问题。
5. 如果花屏或图像抖动，可以查看err counter，如果err counter 有增长，则说明de缺数，有可能是带宽不足，或者瞬时带宽不足问题。
```

10.2 截屏

```
echo 0 > /sys/class/disp/disp/attr/disp
echo /data/filename.bmp > /sys/class/disp/disp/attr/capture_dump
```

该调试方法用于截取DE 输出到TCON 前的图像，用于显示通路上分段排查。如果截屏没有问题而界面异常，可以确定TCON 到显示器间出错。

第一个路径接受显示器索引0 或1。

第二个路径接受文件路径。

### 10.3 colorbar

```
echo 0 > /sys/class/disp/disp/attr/disp
echo > /sys/class/disp/disp/attr/colorbar
```

第一个路径接受显示器索引0 或1。

第二个路径表示TCON 选择的输入源。1，DE 输出；2-7，TCON 自检用的colorbar；8,DE自检用的colorbar。

### 10.4 显示模块debugfs 接口

#### 10.4.1 总述

调试节点

```
mount -t debugfs none /sys/kernek/debug;
/sys/kernel/debug/dispdbg;
```

```
name command param start info
//name: 表示操作的对象名字
//command: 表示执行的命令
//param: 表示该命令接收的参数
//start: 输入1 开始执行命令
//info: 保存命令执行的结果
//只读，大小是1024 bytes。
```

#### 10.4.2 切换显示输出设备

```
name: disp0/1/2 //表示显示通道0/1/2
command: switch
param: type mode
//参数说明：type:0(none),1(lcd),2(tv),4(hdmi),8(vga)
//mode 详见disp_tv_mode 定义
/* 例子*/
/* 显示通道0 输出LCD */
echo disp0 > name;echo switch > command;echo 1 0 > param;echo 1 > start;
/* 关闭显示通道0 的输出*/
echo disp0 > name;echo switch > command;echo 0 0 > param;echo 1 > start;
```

#### 10.4.3 开关显示输出设备

```
name: disp0/1/2 //表示显示通道0/1/2
command: blank
param: 0/1
//参数说明：1 表示blank，即关闭显示输出；0 表示unblank，即开启显示输出
/* 例子*/
/* 关闭显示通道0 的显示输出*/
echo disp0 > name;echo blank > command;echo 1 > param;echo 1 > start;
/* 开启显示通道1 的显示输出*/
echo disp1 > name;echo blank > command;echo 0 > param;echo 1 > start;
```

#### 10.4.4 电源管理(suspend/resume) 接口

```
name: disp0/1/2 //表示显示通道0/1/2
command: suspend/resume //休眠，唤醒命令
param: 无
/* 例子*/
/* 让显示模块进入休眠状态*/
echo disp0 > name;echo suspend > command;echo 1 > start;
/* 让显示模块退出休眠状态*/
echo disp1 > name;echo resume > command;echo 1 > start;
```

#### 10.4.5 调节lcd 屏幕背光

```
name: lcd0/1/2 //表示lcd0/1/2
command: setbl //设置背光亮度
param: xx
//参数说明：背光亮度值，范围是0~255。
/* 例子*/
/* 设置背光亮度为100 */
echo lcd0 > name;echo setbl > command;echo 100 > param;echo 1 > start;
/* 设置背光亮度为0 */
echo lcd0 > name;echo setbl > command;echo 0 > param;echo 1 > start;
```

#### 10.4.6 vsync 消息开关

```
name: disp0/1/2 //表示显示通道0/1/2
command: vsync_enable //开启/关闭vsync 消息
param: 0/1
//参数说明：0：表示关闭； 1：表示开启
/* 例子*/
/* 关闭显示通道0 的vsync 消息*/
echo disp0 > name;echo vsync_enable > command;echo 0 > param;echo 1 > start;
/* 开启显示通道1 的vsync 消息*/
echo disp1 > name;echo vsync_enable > command;echo 1 > param;echo 1 > start;
```

#### 10.4.7 查看enhance 的状态

```
name: enhance0/1/2 //表示enhance0/1/2
command: getinfo //获取enhance 的状态
param: 无
/* 例子*/
/* 获取显示通道0 的enhance 状态信息*/
# echo enhance0 > name;echo getinfo > command;echo 1 > start;cat info;
# enhance 0: enable, normal
```

#### 10.4.8 查看智能背光的状态

```
name: smbl0/1/2 //表示显示通道0/1/2
command: getinfo //获取smart backlight 的状态
param: 无
/* 例子*/
/* 获取显示通道0 的smbl 状态信息*/
# echo smbl0 > name;echo getinfo > command;echo 1 > start;cat info;
# smbl 0: disable, window<0,0,0,0>, backlight=0, save_power=0 percent
//显示的是智能背光是否开启，有效窗口大小，当前背光值，省电比例
```

### 10.5 常见问题

#### 10.5.1 黑屏（无背光）

问题现象：机器接LCD 输出，发现LCD 没有任何显示，仔细查看背光也不亮。

问题分析：此现象说明LCD 背光供电不正常，不排除还有其他问题，但没背光的问题必须先解决。

问题排查步骤：

• 步骤一

使用电压表量LCD 屏的各路电压，如果背光管脚电压不正常，确定是PWM 问题。否则，尝试换个屏再试。

• 步骤二

先看看随sdk 有没发布PWM 模块使用指南，如果有按照里面步骤进行排查。

• 步骤三

如果sdk 没有发布PWM 模块使用指南。可以cat /sys/kernel/debug/pwm 看看有没输出。如果没有就是PWM 驱动没有加载，请检查一下menuconfig 有没打开。

• 步骤四

如果步骤三未解决问题，请排查dts 或board.dts 配置。如果还没有解决，可以寻求技术支持。

#### 10.5.2 黑屏（有背光）

问题现象：机器接LCD，发现有背光，界面输出黑屏。

问题分析：此现象说明没有内容输出，可能是DE、TCON 出错或应用没有送帧。

问题排查步骤：

• 步骤一

根据10.1 章节排查应用输入的图层信息是否正确。其中，宽高、显存的文件句柄出错问题最多。

• 步骤二

根据10.2 章节截屏，看看DE 输出是否正常。如果不正常，排查DE 驱动配置是否正确；如果正常，接着下面步骤。

• 步骤三
根据10.3 章节输出colorbar，如果TCON 自身的colorbar 也没有显示，排查硬件通路；如果
有显示，排查TCON 输入源选择的寄存器。后者概率很低，此时可寻求技术支持。

#### 10.5.3 绿屏

问题现象：显示器出现绿屏，切换界面可能有其他变化。

问题分析：此现象说明处理图层时DE 出错。可能是应用送显的buffer 内容或者格式有问题；也

可能DE 配置出错。

问题排查步骤：

• 步骤一

根据10.1 章节排查应用输入的图层信息是否正确。其中，图层格式填错的问题最多。

• 步骤二

导出DE 寄存器，排查异常。此步骤比较复杂，需要寻求技术支持。

#### 10.5.4 界面卡住

问题现象：界面定在一个画面，不再改变。

问题分析：此现象说明显示通路一般是正常的，只是应用没有继续送帧。

问题排查步骤：

• 步骤一

根据10.1 章节排查应用输入的图层信息有没改变，特别关注图层的地址。

• 步骤二

排查应用送帧逻辑，特别关注死锁，线程、进行异常退出，fence 处理异常。

#### 10.5.5 局部界面花屏

问题现象：画面切到特定场景时候，出现局部花屏，并不断抖动。

问题分析：此现象是典型的DE scaler 出错现象。

问题排查步骤：

根据10.1 章节查看问题出现时带缩放图层的参数。如果屏幕输出的宽x 高除以crop 的宽x 高小于1/16 或者大于32，那么该图层不能走DE 缩放，改用GPU，或应

用修改图层宽高。

#### 10.5.6 快速切换界面花屏

问题现象：快速切换界面花屏，变化不大的界面显示正常。

问题分析：此现象是典型的性能问题，与显示驱动关系不大。

问题排查步骤：

• 步骤一

排查DRAM 带宽是否满足场景需求。

• 步骤二

若是安卓系统，排查fence 处理流程；若是纯linux 系统，排查送帧流程、swap buffer、pandisplay流程。