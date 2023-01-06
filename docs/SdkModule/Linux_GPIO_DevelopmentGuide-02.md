## 2 模块介绍

Pinctrl 框架是 linux 系统为统一各 SoC 厂商 pin 管理，避免各 SoC 厂商各自实现相同 pin 管理子系统而提出的。目的是为了减少 SoC 厂商系统移植工作量。



### 2.1 模块功能介绍

许多 SoC 内部都包含 pin 控制器，通过 pin 控制器，我们可以配置一个或一组引脚的功能和特性。在软件上，Linux 内核 pinctrl 驱动可以操作 pin 控制器为我们完成如下工作：

*•* 枚举并且命名 pin 控制器可控制的所有引脚；

*•* 提供引脚的复用能力

*•* 提供配置引脚的能力，如驱动能力、上拉下拉、数据属性等。

*•* 与 gpio 子系统的交互

*•* 实现 pin 中断



### 2.2 相关术语介绍

<center>表 2-1: Pinctrl 模块相关术语介绍</center>

| 术语           | 解释说明                                                     |
| -------------- | ------------------------------------------------------------ |
| SUNXI          | Allwinner 一系列 SOC 硬件平台                                |
| Pin controller | 是对硬件模块的软件抽象，通常用来表示硬件控制器。能够处理引脚复用、属性配置等功能 |
| Pin            | 根据芯片不同的封装方式，可以表现为球形、针型等。软件上采用常用一组无符号的整数 [0-maxpin] 来表示 |
| Pin groups     | 外围设备通常都不只一个引脚，比如 SPI，假设接在 SoC 的 {0,8,16,24} 管脚，而另一个设备 I2C 接在 SoC 的 {24,25} 管脚。我们可以说这里有两个pin groups。很多控制器都需要处理 pin groups。因此管脚控制器子系统需要一个机制用来枚举管脚组且检索一个特定组中实际枚举的管脚 |
| Pinconfig      | 管脚可以被软件配置成多种方式，多数与它们作为输入/输出时的电气特性相关。例如，可以设置一个输出管脚处于高阻状态，或是 “三态”（意味着它被有效地断开连接）。或者可以通过设置将一个输入管脚与 VDD 或 GND 相连 (上拉/下拉)，以便在没有信号驱动管脚时使管脚拥有确认值 |
| Pinmux         | 引脚复用功能，使用一个特定的物理管脚（ball/pad/finger/等等）进行多种扩展复用，以支持不同功能的电气封装习惯 |
| Device tree    | 犹如它的名字，是一棵包括 cpu 的数量和类别、内存基地址、总线与桥、外设连接，中断控制器和 gpio 以及 clock 等系统资源的树，Pinctrl 驱动支持从device tree 中定义的设备节点获取 pin 的配置信息 |



### 2.3 总体框架

Sunxi Pinctrl 驱动模块的框架如下图所示，整个驱动模块可以分成 4 个部分：pinctrl api、pinctrl common frame、sunxi pinctrl driver，以及 board configuration。（图中最上面一层 device driver 表示 Pinctrl 驱动的使用者）

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxGPIODevelopmentGuide_001.png)

​                                                               图 2-1: pinctrl 驱动整体框架图

Pinctrl api: pinctrl 提供给上层用户调用的接口。

Pinctrl framework：Linux 提供的 pinctrl 驱动框架。

Pinctrl sunxi driver：sunxi 平台需要实现的驱动。

Board configuration：设备 pin 配置信息，一般采用设备树进行配置。



### 2.4 state/pinmux/pinconfig

Pinctrl framework 主要处理 pinstate、pinmux 和 pinconfig 三个功能，pinstate 和 pinmux、pinconfig 映射关系如下图所示。

![](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/LinuxGPIODevelopmentGuide_002.png)

<center>图 2-2: pinctrl 驱动 framework 图</center>

系统运行在不同的状态，pin 配置有可能不一样，比如系统正常运行时，设备的 pin 需要一组配置，但系统进入休眠时，为了节省功耗，设备 pin 需要另一组配置。Pinctrl framwork 能够有效管理设备在不同状态下的引脚配置。



### 2.5 源码结构介绍

```
linux
|
|-- drivers
| |-- pinctrl
| | |-- Kconfig
| | |-- Makefile
| | |-- core.c
| | |-- core.h
| | |-- devicetree.c
| | |-- devicetree.h
| | |-- pinconf.c
| | |-- pinconf.h
| | |-- pinmux.c
| | `-- pinmux.h
| `-- sunxi
| |-- pinctrl-sunxi-test.c
| |-- pinctrl-sun*.c
| `-- pinctrl-sun*-r.c
`-- include
`-- linux
`-- pinctrl
|-- consumer.h
|-- devinfo.h
|-- machine.h
|-- pinconf-generic.h
|-- pinconf.h
|-- pinctrl-state.h
|-- pinctrl.h
`-- pinmux.h
```

