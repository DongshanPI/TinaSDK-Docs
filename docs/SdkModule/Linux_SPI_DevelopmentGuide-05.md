## 5 FAQ

### 5.1 调试节点

#### 5.1.1 /sys/module/spi_sunxi/parameters/debug

默认情况下 debug 为 1，不打开调试信息。

```
echo 255 > /sys/module/spi_sunxi/parameters/debug
```

即可打开调试信息。



#### 5.1.2 /sys/devices/platform/soc/spi1/info

此节点文件可以打印出当前 SPI1 通道的一些硬件资源信息。

```
cat /sys/devices/platform/soc/spi1/info
```



#### 5.1.3 /sys/devices/platform/soc/spi1/status

此节点文件可以打印出当前 SPI1 通道的一些运行状态信息，包括控制器的各寄存器值。

```
cat /sys/devices/platform/soc/spi1/status
```



### 5.2 常见问题



### 5.3 dts 中设置使能不生效

问题现象：在 board.dts 中配置 spi 的 statue 状态为 “okay”，但是启动 Linux 内核却发现 spi控制器未使能。问题分析：可能状态配置有误，亦或者错误使用其他的控制器例如 spi0。

问题排查步骤：

*•* 步骤 1：这种问题一般是由于在设备树里，你的设备依赖了别的设备，但是这个设备没能 probe 成功，从而导致你的设备无法 probe。建议对 spi 依赖的 dma 模块进行排查，检查 dma 在 menuconfig 中是否被打开；

*•* 步骤 2：在 out/目录下搜索.sunxi.dts 并打开：

```
find -name ".sunxi.dts"
```

在文件里找到对应的节点，检查对应的 spi 是否配置成功。

*•* 步骤 3：在小机 uboot 控制台通过 fdt list spi* 命令查看 dts，是否使能 SPI 成功（status =“okay”），如果还是 disable，则可能 spi 在 uboot 阶段被 disable 掉了（一般 spi0 会保留给 flash 使用，spi0 会在 uboot 阶段关闭掉）。



### 5.4 SPI-Flash 数据传输异常

问题现象：写入与读出数据不一致。

*•* 步骤 1：进行兼容性排查。以 nor flash 为例，有些物料兼容性不好，会造成读写出错。这个时候可以先确认下次款物料是否在支持列表内。若不在，试着更换物料再做测试。

*•* 步骤 2：驱动调试。此类问题范围比较大，但是可以从基础调试手段着手跟踪调试。一般思路是打开数据打印，看写入的值是否传到 SPI 总线驱动处理，然后同样的看 SPI 总线驱动刚读出来的数据与前面写的打印数据是否一致，来判断是哪个环节造成读写出错，这个办法可以拓展到其他层次，以确认是文件系统层、MTD 层、SPI 总线驱动层的读或写问题。

