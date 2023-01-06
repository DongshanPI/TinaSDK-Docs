## 5 FAQ

### 5.1 调试方法

#### 5.1.1 调试工具

##### 5.1.1.1 i2c-tools 调试工具

i2c-tools 是一个开源工具，专门用来调试 I2C 设备。可以用 i2c-tools 来获取 i2c 设备的相关信息（默认集成在内核里面），并且读写相关的 i2c 设备的数据。i2c-tools 主要是通过读写/dev/i2c-* 文件获取 I2C 设备，所以需要在 kernel/linux-4.9 的 menuconfig 里面把 I2C的 device interface 节点打开，具体的 i2c-tools 使用方法如下。

```
i2cdetect -l //获取i2c设备信息
i2cdump -y i2c-number i2c-reg //把相关的i2c设备数据dump出来，如i2cdump -y 1 0x50
i2cget -y i2c-number i2c-reg data_rege //读取i2c设备某个地址的数据，如i2cget -y 1 0x50 1
i2cset -y i2c-number i2c-reg data_rege data //往i2c设备某个地址写数据，如i2cset -y 1 0x50 1 1
```



#### 5.1.2 调试节点

##### 5.1.2.1 /sys/module/i2c_sunxi/parameters/transfer_debug

此节点文件的功能是打开某个 TWI 通道通信过程的调试信息。缺省值是-1，不会打印任何通道的通信调试信息。

打开通道 x 通信过程调试信息的方法：

```
echo x > /sys/module/i2c_sunxi/parameters/transfer_debug
```

关闭通信过程调试信息的方法：

```
echo -1 > /sys/module/i2c_sunxi/parameters/transfer_debug
```



##### 5.1.2.2 /sys/devices/soc.2/1c2ac00.twi.0/info

此节点文件可以打印出当前 TWI 通道的一些硬件资源信息。

```
cat /sys/devices/soc.2/1c2ac00.twi.0/info
```



**5.1.2.3 /sys/devices/soc.2/1c2ac00.twi/status**

此节点文件可以打印出当前 TWI 通道的一些运行状态信息，包括控制器的各寄存器值。

```
cat /sys/devices/soc.2/1c2ac00.twi/status
```



### 5.2 常见问题

#### 5.2.1 TWI 数据未完全发送

**问题现象**：incomplete xfer。具体的 log 如下所示：

```
[ 1658.926643] sunxi_i2c_do_xfer()1936 - [i2c0] incomplete xfer (status: 0x20, dev addr: 0x50)
[ 1658.926643] sunxi_i2c_do_xfer()1936 - [i2c0] incomplete xfer (status: 0x48, dev addr: 0x50)
```

**问题分析**：此错误表示主控已经发送了数据（status 值为 0x20 时，表示发送了 SLAVE ADDR \+ WRITE；status 值为 0x48 时，表示发送了 SLAVE ADDR + READ），但是设备没有回ACK，这表明设备无响应，应该检查是否未接设备、接触不良、设备损坏和上电时序不正确导致的设备未就绪等问题。



**问题排查步骤**： 

*•* 步骤 1：通过设备树里面的配置信息，核对引脚配置是否正确。每组 TWI 都有好几组引脚配置。

*•* 步骤 2：更换 TWI 总线下的设备为 at24c16，用 i2ctools 读写 at24c16 看看是否成功，成功则表明总线工作正常。

*•* 步骤 3：排查设备是否可以正常工作以及设备与 I2C 之间的硬件接口是否完好。

*•* 步骤 4：详细了解当前需要操作的设备的初始化方法，工作时序，使用方法，排查因初始化设备不正确导致通讯失败。

*•* 步骤 5：用示波器检查 TWI 引脚输出波形，查看波形是否匹配。





#### 5.2.2 TWI 起始信号无法发送

**问题现象**：START can’t sendout!。具体的 log 如下所示：

```
sunxi_i2c_do_xfer()1865 - [i2c1] START can't sendout!
```

**问题分析**：此错误表示 TWI 无法发送起始信号，一般跟 TWI 总线的引脚配置以及时钟配置有关。应该检查引脚配置是否正确，时钟配置是否正确，引脚是否存在上拉电阻等等。

**问题排查步骤**： 

*•* 步骤 1：重新启动内核，通过查看 log，分析 TWI 是否成功初始化，如若存在引脚配置问题，应核对引脚信息是否正确。

*•* 步骤 2：根据原理图，查看 TWI-SCK 和 TWI-SDA 是否经过合适的上拉电阻接到 3.3v 电压。

*•* 步骤 3：用万用表量 SDA 与 SCL 初始电压，看电压是否在 3.3V 附近（断开此 TWI 控制器所有外设硬件连接与软件通讯进程）。

*•* 步骤 4：核查引脚配置以及 clk 配置是否进行正确设置。

*•* 步骤 5：测试 PIN 的功能是否正常，利用寄存器读写的方式，将 PIN 功能直接设为 INPUT 功能（echo [reg] [val] > /sys/class/sunxi_dump/write），然后将 PIN 上拉和接地改变 PIN状态，读 PIN 的状态 (echo [reg,reg] > /sys/class/sunxi_dump/dump;cat dump)，看是否匹配。

*•* 步骤 6：测试 CLK 的功能是否正常，利用寄存器读写的方式，将 TWI 的 CLK gating 等打开，（echo [reg] [val] > /sys/class/sunxi_dump/write），然后读取相应 TWI 的寄存器信息，读 TWI 寄存器的数据（echo [reg] ,[len]> /sys/class/sunxi_dump/dump)，查看寄存器数据是否正常。



#### 5.2.3 TWI 终止信号无法发送

**问题现象**：STOP can’t sendout。具体的 log 如下所示：

```
twi_stop()511 - [i2c4] STOP can't sendout!
sunxi_i2c_core_process()1726 - [i2c4] STOP failed!
```

**问题分析**：此错误表示 TWI 无法发送终止信号，一般跟 TWI 总线的引脚配置。应该检查引脚配置是否正确，引脚电压是否稳定等等。

**问题排查步骤**： 

*•* 步骤 1：根据原理图，查看 TWI-SCK 和 TWI-SDA 是否经过合适的上拉电阻接到 3.3v 电压。

*•* 步骤 2：用万用表量 SDA 与 SCL 初始电压，看电压是否在 3.3V 附近（断开此 TWI 控制器所有外设硬件连接与软件通讯进程）。

*•* 步骤 3：测试 PIN 的功能是否正常，利用寄存器读写的方式，将 PIN 功能直接设为 INPUT 功能（echo [reg] [val] > /sys/class/sunxi_dump/write），然后将 PIN 上拉和接地改变 PIN状态，读 PIN 的状态 (echo [reg,reg] > /sys/class/sunxi_dump/dump;cat dump)，看是否匹配。

*•* 步骤 4: 查看设备树配置，把其他用到 SCK/SDA 引脚的节点关闭，重新测试 I2C 通信功能。



#### 5.2.4 TWI 传送超时

**问题现象**：xfer timeout。具体的 log 如下所示：

```
[123.681219] sunxi_i2c_do_xfer()1914 - [i2c3] xfer timeout (dev addr:0x50)
```

**问题分析**: 此错误表示主控已经发送完起始信号，但是在与设备通信的过程中无法正常完成数据发送与接收，导致最终没有发出终止信号来结束 I2C 传输，导致的传输超时问题。应该检查引脚配置是否正常，CLK 配置是否正常，TWI 寄存器数据是否正常，是否有其他设备干扰，中断是否正常等问题。



**问题排查步骤**： 

*•* 步骤 1：核实 TWI 控制器配置是否正确。

*•* 步骤 2：根据原理图，查看 TWI-SCK 和 TWI-SDA 是否经过合适的上拉电阻接到 3.3v 电压。

*•* 步骤 3：用万用表量 SDA 与 SCL 初始电压，看电压是否在 3.3V 附近（断开此 TWI 控制器所有外设硬件连接与软件通讯进程）。

*•* 步骤 4：关闭其他 TWI 设备，重新进行烧录测试 TWI 功能是否正常。

*•* 步骤 4：测试 PIN 的功能是否正常，利用寄存器读写的方式，将 PIN 功能直接设为 INPUT 功能（echo [reg] [val] > /sys/class/sunxi_dump/write），然后将 PIN 上拉和接地改变 PIN状态，读 PIN 的状态 (echo [reg,reg] > /sys/class/sunxi_dump/dump;cat dump)，看是否匹配。

*•* 步骤 5：测试 CLK 的功能是否正常，利用寄存器读写的方式，将 TWI 的 CLK gating 等打开，（echo [reg] [val] > /sys/class/sunxi_dump/write），然后读取相应 TWI 的寄存器信息，读 TWI 寄存器的数据（echo [reg] ,[len]> /sys/class/sunxi_dump/dump)，查看寄存器数据是否正常。

*•* 步骤 7：根据相关的 LOG 跟踪 TWI 代码执行流程，分析报错原因。





