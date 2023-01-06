## 5 FAQ

### 5.1 调试方法

#### 5.1.1 调试节点

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxMIPICSIDevelopmentGuide_006.png)

​														        	     图 5-1: vi 节点	

当系统打开 DEBUG_FS 编译宏时，可以 cat /sys/kernel/debug/mpp/vi 查看；否则可以

cat /sys/devices/platform/soc@2900000/2000800.vind/vi。

vi 节点保存的是当前或上一次工作（当前没有工作）的状态。下面对 vi 节点的关键信息进行说明。

CSI_TOP、CSI_ISP 分别是对应 CSI、和 ISP 的工作频率；input 一行表示 CSI 接收到的图片尺寸，fmt 表示输入数据的格式；

output 表示 CSI 出尺寸，如果使用了缩放或者裁剪，那么输入输出尺寸会不一致，fmt 表示数据的输出格式；

最后一行分别表示平均帧间隔、最大帧间隔、最小帧间隔，可以计算得出帧率，调试帧率时可以参考。



#### 5.1.2 settle time

方式一：修改对应 sensor 驱动中的 sensor_probe 函数，可以添加或修改 info->time_hs 的值即可。

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxMIPICSIDevelopmentGuide_007.png)

​																  	图 5-2: info->time_hs



方式二：通过 mipi 子设备的 settle_time 节点在线进行修改，settle_time 节点路径：/sys/devices/platform/soc/5800800.vind/5810100.mipi。

进入节点路径后，可以看到当前目录下存在 settle_time 节点：

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxMIPICSIDevelopmentGuide_008.png)

​																	图 5-3: settle time 节点



可以通过 cat、echo 命令，对 settle_time 节点进行读写操作：

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxMIPICSIDevelopmentGuide_009.png)

​																	图 5-4: settle time 节点读写



调整策略：settle time 的值慢慢增大调整，调大直到不能出图，再取一个略低于最大值的数值即可。调整范围：0x00-0xff。



#### 5.1.3 信号状态

介绍如何观测 SOC 主控的接收数据的信号状态，分别对 MIPI 和并口做出说明。



##### 5.1.3.1 MIPI

MIPI 传输模式有两种：

LP（Low-Power）模式：用于传输控制信号，最高速率 10 MHz。

HS（High-Speed）模式：用于高速传输数据，以 MIPI DPHY V1.1 版本为例，速率范围 [80Mbps - 1.5Gbps] per Lane。

可以通过查看 user manual MIPI PHY 部分寄存器，观测 SOC 识别到的 clock lane 和 data lane 的 LP、HS 状态。



##### 5.1.3.2 并口

对于并口接口的 sensor，可以查看 user manual CSI PARSER 部分的 parser signal 寄存器，观测 sensor 端 PCLK、DATA 的信号状态。以此判断 parser 是否有识别到 sensor 端发送的数据。



### 5.2 常见问题

#### 5.2.1 I2C 不通

如下图打印：

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxMIPICSIDevelopmentGuide_0010.png)

​																		图 5-5: i2c 不通



【分析步骤一】：确认供电、MCLK、i2c 上拉等外围电路信号是否正常。使用万用表测量板子上AVDD、DVDD、IOVDD 供电电压、MCLK 频率、幅度、RESET、PWDN 的电平是否符合要求。

【分析步骤二】：确认 i2c 地址，TWI 通道是否和原理图一致。

【分析步骤三】：以上都正常就用示波器或者逻辑分析仪测量分析主控发出 i2c 波形是否正确、有无回应；最后可以考虑 sensor 损坏或者接口错位等问题。



#### 5.2.2 sensor 不出图

【分析步骤一】：确认 chip id 和 datasheet 上一致。

在对应 sensor 驱动的 sensor_detect 函数中读 chip id 寄存器，这一步也能检验 i2c 的读写是否正确。

【分析步骤二】：确认配置已经配置到 sensor 里。

可以把写进去的寄存器读出来和写入值对比是否一致。

【分析步骤三】：确认配置正确并且 sensor 已经输出图像。

和原厂确认寄存器配置、用示波器测量 sensor 端的 mipi 数据 lane 和时钟 lane 波形，分析是否正在发送数据。

【分析步骤四】：确认 SOC 是否接收到 sensor 数据。

1. mipi 的 clock lane 存在两种工作模式，一种是连续时钟模式，传输过程不会切换 LP 状态；另一种是非连续时钟信号模式，每传输完一帧图像数据，帧 blanking 时将会切换为 LP 状态。目前大部分 MIPI sensor 一般都是非连续时钟模式。

2. 如果 sensor 是连续时钟模式，要保证 MIPI 在 sensor 之前初始化，需要在 sensor 驱动 sensor_probe() 中配置 info->stream_seq = MIPI_BEFORE_SENSOR;

3. 如果 sensor 是非连续时钟模式，可以通过判断 SOC 识别到的 LP、HS 模式状态是否在不断切换，来间接判断 SOC 的 MIPI 的接收状态。

4. 查看 user manual MIPI PHY 部分寄存器，观测 clock lane 和 data lane 的 LP、HS 状态是否有在不断切换，有则说明 MIPI 已经接收到了 sensor 发送的数据。如果没有切换则说明 MIPI 没有正确接收 sensor 数据。此时应该检查 MIPI 相关配置是否正确。

【分析步骤五】：尝试修改 settle time。

如果可以确定 sensor 已经在正确发送数据，只是 MIPI 这边一直接收不到导致无法出图，可以尝试修改 settle time（参考调试方法章节）。



#### 5.2.3 已出图但画面是绿色或者粉红色

一般是 YUYV 顺序反了，可以修改 sensor 驱动中 sensor_formats 结构体的 mbus_code 参数，修改 YUV 顺序即可。



#### 5.2.4 I2c 已通，但是读所有 sensor 寄存器值都为 0

【分析步骤一】检查 i2c 通讯 addr 和 data 的位宽。

检查 sensor 驱动中 cci_drv 结构体中定义的值是否符合 datasheet 要求。

【分析步骤二】检查 i2c 通讯数据大小端是否不一致。

可以在读 sensor id 时把地址高低位相反来快速验证一下。



#### 5.2.5 画面旋转 180 度

可以修改 board.dts 里面的 hflip 和 vflip 来解决，如果画面和人眼成 90 度的话，只能通过修改 sensor 配置来解决（只有部分 sensor 支持）。



#### 5.2.6 没有 video 节点

【问题解析】没有加载 ko 或者 ko 加载失败。

【分析步骤一】检查模块加载顺序是否正确。

lsmod 看一下模块是否加载正确，如果报的错误是 [VIN_ERR]registering gc2355_mipi, No such device! 则表明 sensor 模块 gc2355_mipi 没有加载。

【分析步骤二】检查 board.dts 文件配置是否配置了 vind0，且 status 为 okay。

【分析步骤三】如果是加载失败检查加载失败的原始是 i2c 不通还是没有 ko。

i2c 不通参考前面的分析，没有 ko 请检查是否有对应的驱动并且在 Makefile 中使能了编译。