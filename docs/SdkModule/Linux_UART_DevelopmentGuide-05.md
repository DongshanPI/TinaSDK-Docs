# 常见问题
## 6 FAQ

### 6.1 UART 调试打印开关

#### 6.1.1 通过 debugfs 使用命令打开调试开关

注：内核需打开 CONFIG_DYNAMIC_DEBUG 宏定义

```
1.挂载debugfs。 
mount -t debugfs none /sys/kernel/debug
2.打开uart模块所有打印。
echo "module sunxi_uart +p" > /mnt/dynamic_debug/control
3.打开指定文件的所有打印。
echo "file sunxi-uart.c +p" > /mnt/dynamic_debug/control
4.打开指定文件指定行的打印。
echo "file sunxi-uart.c line 615 +p" > /mnt/dynamic_debug/control
5.打开指定函数名的打印。
echo "func sw_uart_set_termios +p" > /mnt/dynamic_debug/control
6.关闭打印。
把上面相应命令中的+p 修改为-p 即可。
更多信息可参考linux 内核文档：linux-3.10/Documentation/dynamic-debug-howto.txt。
```



#### 6.1.2 代码中打开调试开关

```
1.定义CONFIG_SERIAL_DEBUG宏。
linux-4.9 内核版本中默认没有定义CONFIG_SERIAL_DEBUG ， 需要自行在
drivers/tty/serial/Kconfig 中添加CONFIG_SERIAL_DEBUG 定义，然后drivers/tty/serial/Makefile文件中添加代码ccflags-$(CONFIG_SERIAL_DEBUG) := -DDEBUG。 
注：使用该宏，需要内核关闭CONFIG_DYNAMIC_DEBUG宏。
```



#### 6.1.3 sysfs 调试接口

UART 驱动通过 sysfs 节点提供了几个在线调试的接口

```
1./sys/devices/platform/soc/uart0/dev_info

cupid-p2:/ # cat /sys/devices/platform/soc/uart0/dev_info
id = 0
name = uart0
irq = 247
io_num = 2
port->mapbase = 0x0000000005000000
port->membase = 0xffffff800b005000
port->iobase = 0x00000000
pdata->regulator = 0x (null)
pdata->regulator_id =

从该节点可以看到uart端口的一些硬件资源信息。

2./sys/devices/platform/soc/uart0/ctrl_info
cupid-p2:/ # cat /sys/devices/platform/soc/uart0/ctrl_info
ier : 0x05
lcr : 0x13
mcr : 0x03
fcr : 0xb1
dll : 0x0d
dlh : 0x00
last baud : 115384 (dl = 13)

TxRx Statistics:
tx : 61123
rx : 351
parity : 0
frame : 0
overrun: 0

此节点可以打印出软件中保存的一些控制信息，如当前UART 端口的寄存器值、收发数据的统计等。

3./sys/devices/platform/soc/uart0/status
cupid-p2:/ # cat /sys/devices/platform/soc/uart0/status
uartclk = 24000000
The Uart controller register[Base: 0xffffff800b005000]:
[RTX] 0x00 = 0x0000000d, [IER] 0x04 = 0x00000005, [FCR] 0x08 = 0x000000c1
[LCR] 0x0c = 0x00000013, [MCR] 0x10 = 0x00000003, [LSR] 0x14 = 0x00000060
[MSR] 0x18 = 0x00000000, [SCH] 0x1c = 0x00000000, [USR] 0x7c = 0x00000006
[TFL] 0x80 = 0x00000000, [RFL] 0x84 = 0x00000000, [HALT] 0xa4 = 0x00000002

此节点可以打印出当前UART 端口的一些运行状态信息，包括控制器的各寄存器值。

```
