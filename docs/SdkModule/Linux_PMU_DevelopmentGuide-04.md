## 4 FAQ

### 4.1 调试方法

在设备进行开发过程中，难免需要对各路电源进行调试，控制电源各路电压等操作，内核中提供了对电源调试的方式。

#### 4.1.1 调试工具

##### 4.1.1.1 power key 调试方式

在用户空间调用getevent 命令，通过标准input 系统上报的input 事件，可以确认power key是否能正常工作，是否能正常上报input 事件。

```
add device 2: /dev/input/event1
name: "axp2202-pek"
poll 4, returned 1
/dev/input/event1: 0001 0074 00000001
poll 4, returned 1
/dev/input/event1: 0000 0000 00000000
poll 4, returned 1
/dev/input/event1: 0001 0074 00000000
poll 4, returned 1
/dev/input/event1: 0000 0000 00000000
poll 4, returned 1
/dev/input/event1: 0001 0074 00000001
poll 4, returned 1
/dev/input/event1: 0000 0000 00000000
poll 4, returned 1
/dev/input/event1: 0001 0074 00000000
poll 4, returned 1
/dev/input/event1: 0000 0000 00000000
```

按下按钮为1，弹起为0。0074 为power 事件。

#### 4.1.2 调试节点

##### 4.1.2.1 /sys/kernel/debug/regulator/regulator_summary 节点

shell 命令查询regulator 状态。

kernel 提供调试结点供电源进行调试进行，我们可以通过kernel 的调试结点获取各路电源的各个详细状态。以AXP2101 的设备举例，首先需要mount debugfs 文

件系统。

```
mount -t debugfs none /sys/kernel/debug
cat /sys/kernel/debug/regulator/regulator_summary
regulator use open bypass voltage current min max
-------------------------------------------------------------------------------
regulator-dummy 0 8 0 0mV 0mA 0mV 0mV
dmic 0mV 0mV
uart2 0mV 0mV
uart1 0mV 0mV
twi3 0mV 0mV
twi2 0mV 0mV
twi1 0mV 0mV
twi0 0mV 0mV
twi6 0mV 0mV
usb0-vbus 0 0 0 5000mV 0mA 5000mV 5000mV
usb1-vbus 1 2 0 5000mV 0mA 5000mV 5000mV
5200000.ohci1-controller 0mV 0mV
5200000.ehci1-controller 0mV 0mV
axp2202-dcdc1 0 2 0 900mV 0mA 500mV 1540mV
cpu0 900mV 900mV
reg-virt-consumer.1 0mV 0mV
axp2202-dcdc2 0 1 0 950mV 0mA 500mV 3400mV
reg-virt-consumer.2 0mV 0mV
axp2202-dcdc3 0 1 0 1200mV 0mA 500mV 1840mV
reg-virt-consumer.3 0mV 0mV
axp2202-dcdc4 0 1 0 1000mV 0mA 1000mV 3700mV
reg-virt-consumer.4 0mV 0mV
axp2202-aldo1 0 3 0 2800mV 0mA 500mV 3500mV
sensor1 2800mV 3300mV
sensor0 2800mV 3300mV
reg-virt-consumer.5 0mV 0mV
axp2202-aldo2 1 4 0 1800mV 0mA 500mV 3500mV
sensor1 1800mV 3300mV
sensor0 1800mV 3300mV
sensor0 1800mV 3300mV
reg-virt-consumer.6 0mV 0mV
axp2202-aldo3 0 1 0 3300mV 0mA 500mV 3500mV
reg-virt-consumer.7 0mV 0mV
axp2202-aldo4 0 2 0 1800mV 0mA 500mV 3500mV
codec 1800mV 1800mV
reg-virt-consumer.8 0mV 0mV
axp2202-bldo1 2 3 0 3300mV 0mA 500mV 3500mV
soc@03000000:wlan@0 3300mV 3300mV
soc@03000000:wlan@0 3300mV 3300mV
reg-virt-consumer.9 0mV 0mV
axp2202-bldo2 0 1 0 2500mV 0mA 500mV 3500mV
reg-virt-consumer.10 0mV 0mV
axp2202-bldo3 1 1 0 2800mV 0mA 500mV 3500mV
reg-virt-consumer.11 0mV 0mV
axp2202-bldo4 0 3 0 1500mV 0mA 500mV 3500mV
sensor1 1500mV 1800mV
sensor0 1200mV 1800mV
reg-virt-consumer.12 0mV 0mV
axp2202-cldo1 3 3 0 1800mV 0mA 500mV 3500mV
codec 1800mV 1800mV
1-0036 1800mV 1800mV
axp2202-cldo2 0 1 0 3300mV 0mA 500mV 3500mV
reg-virt-consumer.14 0mV 0mV
axp2202-cldo3 3 4 0 3300mV 0mA 500mV 3500mV
1-0036 3300mV 3300mV
sdc2 0mV 0mV
uart0 0mV 0mV
reg-virt-consumer.15 0mV 0mV
axp2202-cldo4 0 1 0 3300mV 0mA 500mV 3500mV
reg-virt-consumer.16 0mV 0mV
axp2202-rtcldo 0 1 0 1800mV 0mA 1800mV 1800mV
reg-virt-consumer.17 0mV 0mV
axp2202-cpusldo 0 1 0 900mV 0mA 500mV 1400mV
reg-virt-consumer.18 0mV 0mV
axp2202-drivevbus 0 0 0 0mV 0mA 0mV 0mV
```

##### 4.1.2.2 regmap registers 节点

shell 命令读写寄存器。

寄存器调试是指直接对PMIC 的寄存器进行读写操作，此操作应该对寄存器有了解的情况下进行操作，不正确的操作方式将会导致芯片烧毁。在终端中，对抛出的

调试结点进行读写操作，即可对寄存器进行读写操作。无论是读还是写寄存器，都应该首先挂载debugfs 文件系统。

由于PMIC 是通过regmap 进行读写操作，应该可以使用regmap 的调试结点进行对PMIC 的读写访问操作。regmap 的调试结点在debugfs 文件系统下面，通过对

regmap 调试结点的操作可以对PMIC 的寄存器进行读写访问操作。

• 写操作

```
寄存器调试挂载在debugfs文件系统。
mount -t debugfs none /sys/kernel/debug
echo ${reg} ${value} > /sys/kernel/debug/regmap/${dev-name}/registers
实例：
echo 0xff 0x01 > /sys/kernel/debug/regmap/4-0034/registers
写0xff寄存器值为0x01
```

• 读操作

```
寄存器调试挂载在debugfs文件系统。
mount -t debugfs none /sys/kernel/debug
cat /sys/kernel/debug/regmap/${dev-name}/registers
实例：
cat /sys/kernel/debug/regmap/4-0034/registers
读取pmic所有寄存器
```

##### 4.1.2.3 axp_reg 节点

另外，还支持axp 驱动自定义节点axp_reg 读写寄存器。但是这种用法是不推荐的，因为有标准regmap 方式来读写寄存器，根本没必要用私有非标的方式。示例

如下。

```
往axp寄存器0x0f写入值0x55：
echo 0x0f55 > /sys/class/axp/axp_reg
读出axp寄存器0x0f的值：
echo 0x0f > /sys/class/axp/axp_reg
cat /sys/class/axp/axp_reg
```

##### 4.1.2.4 debug_mask 节点

axp 驱动自定义节点debug_mask 打开和关闭调试信息。相关调试信息参考具体的PMIC 驱动。示例如下。

```
系统打印等级设置为8：
echo 8 > /proc/sys/kernel/printk
打开所有axp调试信息：
echo 0xf > /sys/class/axp/debug_mask
关闭所有axp调试信息：
echo 0x0 > /sys/class/axp/debug_mask
```

调试信息一般如下。

```
[ 712.458412] ic_temp = 45
[ 712.461311] vbat = 3977
[ 712.464082] ibat = -779
[ 712.466280] healthd: battery l=96 v=3977 t=30.0 h=2 st=3 c=-779 fc=5066880 chg=
[ 712.475174] charge_ibat = 0
[ 712.478448] dis_ibat = 779
[ 712.481545] ocv = 4073
[ 712.484239] rest_vol = 96
[ 712.487182] rdc = 123
[ 712.489862] batt_max_cap = 5066
[ 712.493472] coulumb_counter = 4857
[ 712.497583] AXP803_COULOMB_CTL = 0xe0
[ 712.501803] ocv_percentage = 86
[ 712.505436] col_percentage = 96
[ 712.509061] bat_current_direction = 0
[ 712.513386] ext_valid = 0
```

