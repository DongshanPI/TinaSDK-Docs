# Linux_Camera 调试常见问题
## 5 模块调试常见问题

初次调试建议打开device 中的DEV_DBG_EN 为1，方便调试。

Camera 模块调试一般可以分为三步：

1. 使用lsmod 命令查看驱动是否加载，查看/lib/modules/内核版本号目录下是否存在相应的ko，如果没有，确认modules.mk 是否修改正确，配置了开机自动

加载。如果存在相应的ko，可手动加载测试确认ko 是否正常，手动加载成功，则确认内核的版本是否一致，导致开机时没有找到相应的ko 从而没有加载。

2. 使用ls /dev/v* 查看是否有video0/1 节点生成

3. 在adb shell 中使用cat /proc/kmsg 命令，或者是使用串口查看内核的打印信息，查看不能正常加载的原因。一般情况下驱动加载不成功的原因有：一是读取

  的sys_config.fex 文件中的配置信息与加载的驱动不匹配，二是probe 函数遇到某些错误没能正确的完成probe 的时候返回。

### 5.1 移植一款sensor 需要进行哪些操作

移植camera sensor，主要进行以下操作：
1. 根据主板的原理图，确认与sensor 模组的接口是否一致，一致才可以保证配置和数据的正常接收。

2. 根据产品的需求，让sensor 模组厂提供产品所需的分辨率、帧率的寄存器配置，这一步需要注意，提供的配置需要是和模组匹配的。比如模组的mipi 接口只

  引出2lane，而提供的寄存器配置却是配置为4lane 输出的，那么该配置在该模组无法正常使用，让模组厂提供该模组可以正常使用的正确配置。注意，该寄存

  器配置SOC 原厂没有，需要sensor 厂提供。

3. 拿到寄存器配置之后，按照本文档《驱动模块实现》章节完成sensor 驱动的编写。

4. 在完成驱动的编写之后，按照本文档《Tina 配置》章节完成modules.mk 的修改。

5. 根据板子的原理图与模组的硬件连接，参照本文档《sys_config.fex 配置》或者《MR813/R818平台配置》章节完成sys_config.fex 或者board.dts 的修改。

6. 完成上述操作之后，按照本文档《menuconfig 配置说明》章节，选上camera 驱动模块，按照《camera 功能测试》章节选上camera 的测试程序，测试驱动

  移植是否正常。

### 5.2 I2C 通信出现问题

#### 5.2.1 R16 R11 R40 等

I2C 出现问题内核一般会伴随打印” cci_write_aX_dX error! slave = 0xXX, addr = 0xXX,value = 0xXX“。

如果与此同时，内核出现打印“chip found is not an target chip.”，则说明在初始化camera前，读取camera 的ID 已经失败。此时，一般是如下几点出现问题。

```
a. 最先考虑应该是更换一个camera模组试试。
b. 电源
    检查sys_config.fex
    vip_dev0_iovdd = "axp22_eldo3"
    vip_dev0_iovdd_vol = 2800000
    vip_dev0_avdd = "axp22_dldo4"
    vip_dev0_avdd_vol = 2800000
    vip_dev0_dvdd = "axp22_eldo2"
    vip_dev0_dvdd_vol = 1500000
    一定要与原理图设计保持一致。必要时，需要用万用表测量camera模组的各路电压是否正常。
c. reset和power down脚
    检查sys_config.fex配置
    vip_dev0_reset = port:PH2<1><default><default><default>
    vip_dev0_pwdn = port:PH1<1><default><default><default>
    是否与原理图设计保持一致。必要时，需要用示波器测量reset，pwdn脚，在camera加载时，是否有动作。
d. mclk
    检查sys_config.fex配置
    vip_csi_mck = port:PE01<3><default><default><default>
    pin脚是否与原理图设计保持一致。必要时，在加载camera时，测量mclk，看是否有正确输出（一般是24MHz或27MHz
    ）
```

如果已经能够正确通过camera 的id 读取，只是在使用过程当中，偶尔出现I2C 的读写错误，此时需要从打印里面，将报错的地址和读写值，结合camera 具体的

spec 来分析，到底是操作了camera 哪些寄存器带来的问题。



#### 5.2.2 其他平台

出错时一般出现以下信息：

```
[ 5.556579] sunxi_i2c_do_xfer()1942 - [i2c1] incomplete xfer (status: 0x20, dev addr: 0
x30)
[ 5.566234] sunxi_i2c_do_xfer()1942 - [i2c1] incomplete xfer (status: 0x20, dev addr: 0
x30)
[ 5.575963] sunxi_i2c_do_xfer()1942 - [i2c1] incomplete xfer (status: 0x20, dev addr: 0
x30)
[ 5.585375] [VIN_DEV_I2C]sc031gs_mipi sensor read retry = 2
[ 5.591666] [sensorname_mipi] error, chip found is not an target chip.
```

出现上述错误打印时，可按以下操作逐步debug。

1. 确认sys_config.fex 中配置的sensor I2C 地址是否正确（sensor datasheet 中标注，读地
址为0x6d，写地址为0x6c，那么sys_config.fex 配置sensor I2C 地址为0x6c）；
2. 在完成以上操作之后，在senor 上电函数中，将掉电操作屏蔽，保持sensor 一直上电状态，
方便debug；
3. 确认I2C 地址正确之后，测量sensor 的各路电源电压是否正确且电压幅值达到datasheet
标注的电压要求；
4. 测量MCLK 的电压幅值与频率，是否正常；
5. 测量senso r 的reset、pown 引脚电平配置是否正确，I2C 引脚SCK、SDA 是否已经硬件
上拉；
6. 确认I2C 接口使用正确并使能（CCI / TWI）；
7. 如果还是I2C 出错，协调硬件同事使用逻辑分析仪等仪器进行debug；

#### 5.2.3 经典错误

##### 5.2.3.1 I2C 没有硬件上拉

```
twi_start()450 - [i2c2] START can't sendout!
twi_start()450 - [i2c2] START can't sendout!
twi_start()450 - [i2c2] START can't sendout!
[VFE_DEV_I2C_ERR]cci_write_a16_d16 error! slave = 0x1e, addr = 0xa03e, value = 0x1
```

出现上述的问题是因为SDA、SCK 没有拉上，导致在进行I2C 通信时，发送开始信号失败，SDA、SCK 添加上拉即可。

##### 5.2.3.2 没有使能I2C

```
[VFE]Sub device register "ov2775_mipi" i2c_addr = 0x6c start!
[VFE_ERR]request i2c adapter failed!
[VFE_ERR]vfe sensor register check error at input_num = 0
```

出现上述的错误，是因为使用twi 进行I2C 通信但没有使能twi 导致的错误，此时需要确认sys_config.fex 中，[twiX] 中的twiX_used 是否已经设置为1。

### 5.3 图像异常

#### 5.3.1 运行camerademo 可以成功采集图像，但图像全黑(RAWsensor)

当camerademo 成功采集到图像时，最起码整条数据通路已经正常，而发现图像时全黑的，注意以下几点：

1. 在编译camerademo 之前，是根据平台(MR813/R818/MR133/R311) 正确的选上了“Enable vin isp support”，选上之后，重新编译camerademo

  (建议cd package/allwinner/-camerademo 目录后执行mm -B 编译)；

2. 通过上述操作之后， 执行新编译的camerademo 可执行程序， 运行过程应可看到类似”[ISP]create isp0 server thread¡‘信息，则正确运行isp，这时再查看新

  抓取的图像数据；

3. 执行运行camerademo 只会抓取5 张图像数据，由于isp 计算合适的图像曝光需要一定的帧数，所以可能存在前面几张图像黑的情况，修改camerademo 运行

  参数，抓取多几张图像数据查看（20 张）；

4. 如果是没有移植isp 的环境，则可修改sensor 驱动中寄存器组中的曝光参数配置，增加初始化时曝光时间，从而使初始输出的图像亮度较合适；

#### 5.3.2 camerademo 采集的图像颜色异常

运行camerademo 采集图像之后，发现拍摄得到的轮廓正确但颜色不对，比如红蓝互换、画面整体偏红或偏蓝等颜色异常的情况，出现这样的问题，首先考虑是

sensor 驱动中配置的RAW 数据RGB 顺序错误导致的。在sensor 驱动中有类似以下的配置：

```
static struct sensor_format_struct sensor_formats[] = {
    {
        .desc = "Raw RGB Bayer",
        .mbus_code = MEDIA_BUS_FMT_SBGGR10_1X10,
        .regs = sensor_fmt_raw,
        .regs_size = ARRAY_SIZE(sensor_fmt_raw),
        .bpp = 1
    },
};
```

以上配置表明sensor 输出的图像数据是RAW10，RGB 排列顺序是BGGR，出现颜色异常时，一般就是RGB 的排列顺序配置错误导致的，RGB 排列顺序一共有4 种

(MEDIA_BUS_FMT_SBGGR10_1X10/MEDIA_BUS_FMT_SGBRG10_1X10/MEDIA_BUS_FMT_SGRBG10_修改驱动中的mbus_code 为上述的4 种之一，确认哪一种

颜色比较正常，则驱动配置正确。

如果颜色还有细微的不够艳丽、准确等问题，需要进行isp 效果调试，改善图像色彩。上述是以10bit sensor 为例进行介绍，其他的8bit、12bit、14bit 类似，参

考上述即可。

### 5.4 调试camera 常见现象和功能检查

1. insmod 之后首先看内核打印，看加载有无错误打印，部分驱动在加载驱动进行上下电时候会进行i2c 操作，如果此时报错的话就不需要再进入camera 了，先

  检查是否io 或电源配置不对。或者是在复用模组时候有可能是另外一个模组将i2c 拉住了。

2. 如果i2c 读写没有问题的话，一般就可以认为sensor 控制是ok 的，只需要根据sensor 的配置填好H/VREF、PCLK 的极性就能正常接收图像了。这个时候可以

  在进入camera 应用之后用示波器测量sensor 的各个信号，看h/vref、pclk 极性、幅度是否正常（2.8V 的vpp）。

3. 如果看到画面了，但是看起来是绿色和粉红色的，但是有轮廓，一般是YUYV 的顺序设置反了，可检查yuyv 那几个寄存器是否填写正确配置，其次，看是否是

  在配置的其他地方有填写同一个寄存器的地方导致将yuyv fmt 的寄存器被改写。

4. 如果画面颜色正常，但是看到有一些行是粉红或者绿色的，往往是sensor 信号质量不好所致，通常在比较长的排线中出现这个情况。在信号质量不好并且

  yuyv 顺序不对的时候也会看见整个画面的是绿色的花屏。

5. 当驱动能力不足的时候增强sensor 的io 驱动能力有可能解决这个问题。此时用示波器观察pclk 和数据线可能会发现：pclk 波形摆幅不够IOVDD 的幅度，或者

  是data 输出波形摆幅有时候能高电平达到IOVDD 的幅度，有时候可能连一半都不够。

6. 如果是两个模组复用数据线的话，不排除是另外一个sensor 在进入standby 时候没有将其数据线设置成高阻，也会影响到当前模组的信号摆幅，允许的话可

  以剪断另一个模组来证实。

7. 当画面都正常之后检查前置摄像头垂直方向是否正确，水平方向是否是镜像，后置水平垂直是否正确，不对的话可以调节sys_config.fex 中的hflip 和vflip 参数

  来解决，但如果屏幕上看到的画面与人眼看到的画面是成90 度的话，只能是通过修改模组的方向来解决。

8. 之后可以检查不同分辨率之间的切换是否ok，是否有切换不成功的问题；以及拍照时候是否图形正常，亮度颜色是否和预览一致；双摄像头的话需要检查前后

  切换是否正常。

9. 如果上述都没有问题的话，可认为驱动无大问题，接下来可以进行其他功能（awb/exp bias/-color effect 等其他功能的测试）。

10. 测试对焦功能，单次点触屏幕，可正确对上不同距离的物体；不点屏幕时候可以自动对焦对上画面中心物体，点下拍照后拍出来的画面能清晰。

    

11. 打开闪光灯功能，检查在单次对焦时候能打开灯，对完之后无论成功失败或者超时能够关闭，在点下拍照之后能打开，拍完之后能关闭。

12. 如果加载模块后，发现dev/videoX 节点没有生成，请检查下面几点。

```
a. 模块加载的顺序
    一定要按照以下顺序加载模块
    insmod videobuf-core.ko
    insmod videobuf-dma-contig.ko
    ;如果有对应的vcm driver，在这里加载，如果没有，请省略。
    insmod actuator.ko
    insmod ad5820_act.ko
    ;以下是camera驱动和vfe驱动的加载，先安装一些公共资源。
    insmod vfe_os.ko
    insmod vfe_subdev.ko
    insmod cci.ko
    insmod ov5640.ko
    insmod gc0308.ko
    ;如果一个csi接两个camera，所有camera对应的ko都要在vfe_v4l2.ko之前加载。
    insmod vfe_v4l2.ko
b. sys_config.fex配置
    vip_used = 1 ;确保used为1
    vip_dev_qty = 2 ;确保csi接口上接的camera数量与ko加载情况相同
    vip_dev0_mname = "ov5640" ;确保camera型号与ko加载情况相同
    vip_dev0_twi_id = 1 ;确保camera使用的i2c总线id与配置一样
    vip_dev1_mname = "gc0308" ;确保camera型号与ko加载情况相同
    vip_dev1_twi_id = 1 ;确保camera使用的i2c总线id与配置一样
```

### 5.5 画面大体轮廓正常，颜色出现大片绿色和紫红色

一般可能是csi 采样到的yuyv 顺序出现错位。

确认camera 输出的yuyv 顺序的设置与camera 的spec 一致
若camera 输出的yuyv 顺序没有问题，则可能是由于走线问题，导致pclk 采样data 时发生错位，此时可以调整pclk 的采样沿。具体做法如下：

在对应的camara 驱动源码，如ov5640.c 里面，找到宏定义#define CLK_POL。此宏定义可以有两个值V4L2_MBUS_PCLK_SAMPLE_RISING 和

V4L2_MBUS_PCLK_SAMPLE_FALLING。若原来是其中一个值，则修改成另外一个值，便可将PCLK 的采样沿做反相。

### 5.6 画面大体轮廓正常，但出现不规则的绿色紫色条纹

一般可能是pclk 驱动能力不足，导致某个时刻采样data 时发生错位。

解决办法：

• 若pclk 走线上有串联电阻，尝试将电阻阻值减小。

• 增强pclk 的驱动能力，需要设置camera 的内部寄存器。



### 5.7 画面看起来像油画效果，过渡渐变的地方有一圈一圈

一般是CSI 的data 线没有接好，或短路，或断路。



### 5.8 出现[VFE_WARN] Nobody is waiting on thisvideo buffer

上层还回来所有的buffer，但是没有再来取buffer。



### 5.9 出现[VFE_WARN] Only three buffer left for csi

上层占用了大部分buffer，没有还回，驱动部分只有三个buffer 此时驱动不再进行buffer 切换，直到有buffer 还回为止。



### 5.10 sensor 的硬件接口注意事项

1. 如果是使用并口的sensor 模组，会使用到720p@30fps 或更高速度的，必须在mclk/pclk/-
data/vsync/hsync 上面串33ohm 电阻，5M 的sensor 一律串电阻；
2. 使用Mipi 模组时候PCB layout 需要尽量保证clk/data 的差分对等长，过孔数相等，特征
阻抗100ohm；
3. 如果使用并口复用pin 的模组时候，不建议reset 脚的复用；
4. 并口模组的排线长度加上pcb 板上走线长度不超过10cm，mipi 模组排线长度加上pcb 板上
走线长度不超过20cm，超过此距离不保证能正常使用。
5. 主控并口数据线有D11~D0 共12bit，并口的sensor 输出一般为8/10bit，原理图连接需
  要做高位对齐。