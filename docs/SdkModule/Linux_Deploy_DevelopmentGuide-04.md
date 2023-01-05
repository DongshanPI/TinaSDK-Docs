## 5 设备树使用

### 5.1 引言

#### 5.1.1 编写目的

介绍Device Tree配置、设备驱动如何获取Device Tree配置信息等内容，让用户明确掌握Device Tree配置与使用方法。

#### 5.1.2 术语与缩略语.

| 术语/缩略语    | 解释说明                         |
| :------------- | :------------------------------- |
| DTS Device     | Tree Source File，设备树源码文件 |
| DTB Device     | Tree Blob File，设备树二进制文件 |
| sys_config.fex | Allwinner配置文件                |

### 5.2 模块介绍

Device Tree是一种描述硬件的数据结构，可以把嵌入式系统资源抽象成一颗树形结构，可以直观查看系统资源分布；内核可以识别这棵树，并根据它展开出Linux内核中的platform_device等。

#### 5.2.1 模块功能介绍.

Device Tree改变了原来用hardcode方式将HW配置信息嵌入到内核代码的方法，消除了arch/arm64下大量的冗余编码。使得各个厂商可以更专注于driver开发，开发流程遵从main-line kernel的规范。

#### 5.2.2 相关术语介绍.


| 术语/缩略语 | 解释说明                                                     |
| :---------- | :----------------------------------------------------------- |
| FDT         | 嵌入式PowerPC中，为了适应内核发展&&嵌入式PowerPC平台的千变万化，推出了Standard for EmbeddedPower Architecture Platform Requirements（ePAPR）标准，吸收了Open Firmware的优点，在U-boot引入了扁平设备树FDT接口，使用一个单独的FDT blob对象将系统硬件信息传递给内核。 |
| DTS         | device tree源文件，包含用户配置信息。对于32bit Arm架构，dts文件存放在arch/arm/boot/dts路径下。对于64bitArm架构，dts文件存放在arch/arm64/boot/dts路径下。对于Tina3.5.1及之后版本，会使用device目录下的board.dts，即device/config/chips/ chip / conf igs /{borad}/board.dts。 |
| DTB         | DTS被DTC编译后二进制格式的Device Tree描述，可由Linux内核解析，并为设备驱动提供硬件配置信息。 |

### 5.3 如何配置

#### 5.3.1 配置文件位置.

设备树文件，存放在具体内核的目录下。

- ARMv7架构下，dts文件放置在内核的arch/arm/boot/dts/目录。
- ARMv8架构下，dts文件放置在内核的arch/arm64/boot/dts/目录。
- RISCV架构下，dts文件放置在内核的arch/riscv/boot/dts/目录。

对于Tina3.5.1及之后版本，会使用device目录下的board.dts，即：

```
device/config/chips/${chip}/configs/${borad}/board.dts
```

lunch选择具体方案后，可以使用快捷命令跳到该目录：

```
cdts
```

要确认具体的设备树，首先确定chip，在lunch选择方案之后，会有打印：

```
TARGET_CHIP=xxx
```

如，TARGET_CHIP=sun8iw18p1，则使用sun8iw18p1开头的配置文件。

对应CHIP开头的设备树有多份。编译内核的时候都会进行编译，生成中间文件。在打包的时候，才结合sys_config，生成最终的设备树。


在打包脚本，即scripts/pack_img.sh中，do_ini_to_dts函数进行相关的处理。

对于Tina3.5.0及之前版本，如果有方案同名的设备树，则优先使用方案同名的设备树。没有的话，则使用通用的设备树。

如 cowbell-perf1 方案，会先找 sun8iw18p1-cowbell-perf1.dts，如果找不到，就使用sun8iw18p1-soc.dts。

对于Tina3.5.1及之后版本，会使用device目录下的board.dts，即：

```
device/config/chips/${chip}/configs/${borad}/board.dts
```

#### 5.3.2 配置文件关系.

##### 5.3.2.1 不存在sys_config.fex配置情况

当不存在sys_config.fex时，一份完整的配置可以包括两个部分(以sun50iw1p1平台为例)：

- soc级配置文件：定义了SOC级配置，如设备时钟、中断等资源，如图sun50iw1p1.dtsi[1]。
- board级配置文件：定义了板级配置，包含一些板级差异信息，如图sun50iw1p1-t1.dtsi。

示例如图：


![OpenRemoved_Tina_Linux_Configuration_Development_Guide-5-1](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_Configuration_Development_Guide-5-1.jpg)

```
说明
注，对应图中为 sun50iw1p1.dts ， .dts 与 .dtsi 的区别在于 dtsi 是 dts 公共部分的提炼，应用时 dts 包含 dtsi 即可。
在本文中并不作严格区分。
```

上图以R18为例子，展示了三个方案的设备树配置信息，其中：


- 每个方案 dtb 文件，依赖于 sun50iw1p1- _board_. _dtssun_ 50 _iw_ 1 _p_ 1 −{board}.dtsi 又包含sun50iw1p1.dtsi，当board级配置文件跟 soc级配置文件出现相同节点的同名属性时，Board级配置文件的属性值会去覆盖soc级的相同属性值。

- 图示中sun50iw1p1-soc.dts文件跟sun50iw1p1-t1.dts与sun50iw1p1-perf1.dts一样，都属于board配置文件。该配置文件定义为一种通用的board配置文

  件，主要为了防止客户移植新的方案时，没有在内核linux-3.10/arch/arm64/boot/dts/目录下定义客户方案的board级配置文件。如果出现这样的情况，内核编译的时候，就会采用sun50iw1p1-soc.dts，作为该客户方案的board级配置文件。注：Tina3.5.1及之后的版本，方案dts配置，迁移到了device/config/chips/ _chip_ / _conf igs_ /{board}/board.dts

##### 5.3.2.2 存在sys_config.fex配置情况

当存在sys_config.fex时，一份完整的配置可以包括三个部分(以sun50iw1p1平台为例)：

- soc级配置文件：定义了SOC级配置，如设备时钟、中断等资源，如图sun50iw1p1.dtsi。
- board级配置文件：定义了板级配置，包含一些板级差异信息，如图sun50iw1p1-t1.dtsi。
- sys_config.fex配置文件，为方便客户使用而定义，优先级比board级配置、soc级配置都高。

示例如图：


![OpenRemoved_Tina_Linux_Configuration_Development_Guide-5-1](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_Configuration_Development_Guide-5-1-1672044892618.jpg)

以R18为例子，上图展示了三个方案的设备树配置信息，其中：

- 每个方案 dtb 文件，既包含 sys_config.fex 配置信息，同时又依赖于 sun50iw1p1-
  _board_. _dtssun_ 50 _iw_ 1 _p_ 1 −{board}.dtsi又包含sun50iw1p1.dtsi，sys_config.fex配置文件的


```
优先级别最高，sys_config.fex跟devicetree文件都存在配置项时，sys_config.fex的配置
项内容会更新到board级配置文件或者soc级配置文件对应的配置项上去。
```

- 图示中sun50iw1p1-soc.dts文件跟sun50iw1p1-t1.dts与sun50iw1p1-perf1.dts一样，都属于board配置文件。该配置文件定义为一种通用的board配置文件，主要为了防止客户移植新的方案时，没有在内核 linux-4.4/arch/arm64/boot/dts/sunxi/目录下定义客户方案的board级配置文件。如果出现这样的情况，内核编译的时候，就会采用sun50iw1p1-soc.dts，作为该客户方案的board级配置文件。注：Tina3.5.1及之后的版本，方案dts配置，迁移到了device/config/chips/ _chip_ / _conf igs_ /{board}/board.dts

##### 5.3.2.3 soc级配置文件与board级配置文件.

soc级配置文件与board级配置文件都是dts配置文件，对于相同设备节点的描述可能存在重合关系。因此，需要对重合的部分采取合并或覆盖的特殊处理，我们一般考虑两种情况：

- 1 、一般地，soc级配置文件保存公共配置，board级配置文件保存差异化配置，如果公共配置不完善或需要变更，则一般需要通过board级配置文件修改补充，那么只需在board级配置文件中创建相同的路径的节点，补充差异配置即可。此时采取的合并规则是：两个配置文件中不同的属性都保留到最终的配置文件，即合并不同属性配置项；相同的属性，则优先选取board级配置文件中属性值保留，即board覆盖soc级相同属性配置项。如下：

```
soc级定义：
/｛
soc {
thermal-zones {
xxx {
aaa = "1";
bbb = "2";
}
}
}
｝
board级定义：
/｛
soc {
thermal-zones {
xxx {
aaa = "3";
ccc = "4";
}
}
}
｝
最终生成：
/｛
soc {
thermal-zones {
xxx {
aaa = "3";
```

```
bbb = "2";
ccc = "4";
}
}
}
｝
```

- 2 、如果soc级保存的公共配置无法满足部分方案的特殊要求，且使用这项公共配置的其他方案众多，直接修改难度较大。那么我们考虑在board级配置文件中，使用/delete-node/语句删除soc级的配置，并重新定义。如下：

```
soc级定义：
/｛
soc {
thermal-zones {
xxx {
aaa = "1";
bbb = "2";
}
}
}
｝
board级定义：
/｛
soc {
/delete-node/ thermal-zones;
thermal-zones {
xxx {
aaa = "3";
ccc = "4";
}
}
}
｝
最终生成：
/｛
soc {
thermal-zones {
xxx {
aaa = "3";
ccc = "4";
}
}
}
｝
```

删除节点的语法如下：

```
/delete-node/ 节点名；
```

需要注意的是注意：（ 1 ）/delete-node/与节点名之间有空格。

（ 2 ）如果节点中有地址信息，节点名后也需要加上。


删除属性的语法如下：

```
/delete-property/属性名；
```

#### 5.3.3 配置sys_config.fex


![OpenRemoved_Tina_Linux_Configuration_Development_Guide-5-3](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_Configuration_Development_Guide-5-3.jpg)

#### 5.3.4 配置device tree

```
Vdevice: vdevice@0{ (详见(1))：(详见(2))
compatible=”allwinner,sun50i-vdevice”;
device_type=”Vdevice”; (详见(3))
pinctrl_name=”default”; (详见(4))
pinctrl_0=<&vdevice_pins_a>
test_prop=”adb” (详见(5))
status=”okay” (详见(6))
};
```

详注：

(1) label，此处名字必须与sys_config.fex主键一致。

(2) 节点名字。

(3) 特定属性表示设备类型，必须与label一致。

(4) 特定属性，用来PIN配置。

(5) 普通属性。

(6) 特定属性，用来表示设备是否使用。


### 5.4 接口描述

Linux系统为device tree提供了标准的API接口。

#### 5.4.1 常用外部接口.

使用内核提供的device tree接口，必须引用Linux系统提供的device tree接口头文件，包含
且不限于以下头文件：

```
#include<linux/of.h>
#include<linux/of_address.h>
#include<linux/of_irq.h>
#include<linux/of_gpio.h>
```

##### 5.4.1.1 irq_of_parse_and_map

| 类别 |  介绍 |

```
函数原型 unsigned int irq_of_parse_and_map(struct device_node
*dev, int index)
参数 dev：要解析中断号的设备；index：dts源文件中节点
interrupt属性值索引；
返回 如果解析成功，返回中断号，否则返回 0 。
```

DEMO：

```
以timer节点为例子：
Dts配置：
/{
timer0: timer@1c20c00 {
...
interrupts = <GIC_SPI 18 IRQ_TYPE_EDGE_RISING>;
...
};
};
驱动代码片段：
static void __init sunxi_timer_init(struct device_node *node){
int irq;
....
irq = irq_of_parse_and_map(node, 0);
if (irq <= 0)
panic("Can't parse IRQ");
}
```

##### 5.4.1.2 of_iomap

| 类别     | 介绍                                                         |
| :------- | :----------------------------------------------------------- |
| 函数原型 | void __iomem *of_iomap(struct device_node *np, intindex);    |
| 参数     | np：要映射内存的设备节点，index：dts源文件中节点reg属性值索引； |
| 返回     | 如果映射成功，返回IO memory的虚拟地址，否则返回NULL。        |

DEMO：

```
以timer节点为例子，dts配置：
/{
timer0: timer@1c20c00 {
reg = <0x0 0x01c20c00 0x0 0x90>;
};
};
以timer为例子，驱动代码片段：
static void __init sunxi_timer_init(struct device_node *node){
timer_base = of_iomap(node, 0);
}
```

##### 5.4.1.3 of_property_read_u32

| 类别     | 介绍                                                         |
| :------- | :----------------------------------------------------------- |
| 函数原型 | static inline int of_property_read_u32(const structdevice_node *np, const char *propname, u32*out_value) |
| 参数     | np：想要获取属性值的节点; propname：属性名称; out_value：属性值 |
| 返回     | 如果取值成功，返回 0 。                                      |

DEMO：

```
//以timer节点为例子，dts配置例子：
/{
soc_timer0: timer@1c20c00 {
clock-frequency = <24000000>;
timer-prescale = <16>;
```

```
};
};
//以timer节点为例子，驱动中获取clock-frequency属性值的例子：
int rate=0;
if (of_property_read_u32(node, "clock-frequency", &rate)) {
pr_err("<%s> must have a clock-frequency property\n",node->name);
return;
}
```

##### 5.4.1.4 of_property_read_string

| 类别     | 介绍                                                         |
| :------- | :----------------------------------------------------------- |
| 函数原型 | static inline int of_property_read_string(structdevice_node *np, const char *propname, const char**output) |
| 参数     | np：想要获取属性值的节点；propname：属性名称；output：用来存放返回字符串 |
| 返回     | 如果取值成功，返回 0                                         |
| 功能描述 | 该函数用于获取节点中属性值。（针对属性值为字符串）           |

DEMO：

```
//例如获取string-prop的属性值，Dts配置：
/{
soc@01c20800{
vdevice: vdevice@0{
string_prop = "abcd";
};
};
};
例示代码：
test{
const char *name;
err = of_property_read_string(np, "string_prop", &name);
if (WARN_ON(err))
return;
}
```

##### 5.4.1.5 of_property_read_string_index


| 类别     | 介绍                                                         |
| :------- | :----------------------------------------------------------- |
| 函数原型 | static inline int of_property_read_string_index(structdevice_node *np, const char *propname,int index, constchar **output) |
| 参数     | np：想要获取属性值的节点Propname：属性名称; Index：用来索引配置在dts中属性为propname的值。Output：用来存放返回字符串 |
| 返回     | 如果取值成功，返回 0 。                                      |
| 功能描述 | 该函数用于获取节点中属性值。（针对属性值为字符串）。         |

DEMO：

```
//例如获取string-prop的属性值，Dts配置：
/{
soc@01c20800{
vdevice: vdevice@0{
string_prop = "abcd";
};
};
};
例示代码：
test{
const char *name;
err = of_property_read_string_index(np, "string_prop", 0, &name);
if (WARN_ON(err))
return;
}
```

##### 5.4.1.6 of_find_node_by_name

| 类别     | 介绍                                                         |
| :------- | :----------------------------------------------------------- |
| 函数原型 | extern struct device_node *of_find_node_by_name(struct device_node *from, constchar *name); |
| 参数     | clk：待操作的时钟句柄；From：从哪个节点开始找起 Name：想要查找节点的名字 |
| 返回     | 如果成功，返回节点结构体，失败返回null。                     |
| 功能描述 | 该函数用于获取指定名称的节点。                               |

DEMO：


```
//获取名字为vdeivce的节点，dts配置
/{
soc@01c20800{
vdevice: vdevice@0{
string_prop = "abcd";
};
};
};
例示代码片段：
test{
struct device_node *node;
node = of_find_node_by_name(NULL, "vdevice");
if (!node){
pr_warn("can not get node.\n");
};
of_node_put(node);
}
```

##### 5.4.1.7 of_find_node_by_type.

| 类别     | 介绍                                                         |
| :------- | :----------------------------------------------------------- |
| 函数原型 | extern struct device_node *of_find_node_by_name(struct device_node *from, const char *type); |
| 参数     | clk：待操作的时钟句柄；From：从哪个节点开始找起type：想要查找节点中device_type包含的字符串 |
| 返回     | 如果成功，返回节点结构体，失败返回null。                     |
| 功能描述 | 该函数用于获取指定device_type的节点。                        |

DEMO：

```
//获取名字为vdeivce的节点，dts配置。
/{
soc@01c20800{
vdevice: vdevice@0{
...
device_type = "vdevice";
string_prop = "abcd";
};
};
};
例示代码片段：
test{
struct device_node *node;
....
node = of_find_node_by_type(NULL, "vdevice");
```

```
if (!node){
pr_warn("can not get node.\n");
};
of_node_put(node);
}
```

##### 5.4.1.8 of_find_node_by_path.

| 类别     | 介绍                                                         |
| :------- | :----------------------------------------------------------- |
| 函数原型 | extern struct device_node *of_find_node_by_path(const char *path); |
| 参数     | path：通过指定路径查找节点；                                 |
| 返回     | 如果成功，返回节点结构体，失败返回null。                     |
| 功能描述 | 该函数用于获取指定路径的节点。                               |

DEMO：

```
//获取名字为vdeivce的节点，dts配置。
/{
soc@01c20800{
vdevice: vdevice@0{
device_type = "vdevice";
string_prop = "abcd";
};
};
};
例示代码片段：
test{
struct device_node *node;
node = of_find_node_by_path("/soc@01c2000/vdevice@0");
if (!node){
pr_warn("can not get node.\n");
};
of_node_put(node);
}
```

##### 5.4.1.9 of_get_named_gpio_flags

| 类别     | 介绍                                                         |
| :------- | :----------------------------------------------------------- |
| 函数原型 | int of_get_named_gpio_flags(struct device_node *np, const char *propname, int index, enum of_gpio_flags* flags) |
| 参数     | np：包含所需要查找GPIO的节点propname：包含GPIO信息的属性Index：属性propname中属性值的索引Flags：用来存放gpio的flags |
| 返回     | 如果成功，返回gpio编号，flags存放gpio配置信息，失败返回null。 |
| 功能描述 | 该函数用于获取指定名称的gpio信息。                           |

DEMO：

```
//获取名字为vdeivce的节点，dts配置。
/{
soc@01c20800{
vdevice: vdevice@0{
...
device_type = "vdevice";
string_prop = "abcd";
};
};
};
例示代码片段：
test{
struct device_node *node;
....
node = of_find_node_by_path("/soc@01c2000/vdevice@0");
if (!node){
pr_warn("can not get node.\n");
};
of_node_put(node);
}
/{
soc@01c20800{
vdevice: vdevice@0{
...
test-gpios=<&pio PA 1 1 1 1 0>;
};
};
};
static int gpio_test(struct platform_device *pdev)
{
struct gpio_config config;
....
node=of_find_node_by_type(NULL, "vdevice");
if(!node){
printk(" can not find node\n");
}
ret = of_get_named_gpio_flags(node, "test-gpios", 0, (enum of_gpio_flags *)&config)
;
if (!gpio_is_valid(ret)) {
return -EINVAL;
}
};
```

#### 5.4.2 sys_config接口&&dts接口映射

##### 5.4.2.1 获取子键内容.

| script API： |                                                              |
| :----------- | :----------------------------------------------------------- |
| 原型         | script_item_value_type_e script_get_item(char *main_key, char *sub_key, script_item_u *item); |

作用 |通过主键名和子键名字，获取子键内容（该接口可以自己识别子键的类型）。|

| dts API： |                                                              |
| :-------- | :----------------------------------------------------------- |
| 说明      | dts标准接口支持通过节点和属性名，获取属性值（用户需要知道属性值得类型） |
| 原型      | int of_property_read_u32(const struct device_node *np,const char *propname, u32 *out_value) |
| 作用      | 获取属性值，使用于属性值为整型数据。                         |
| 原型      | int of_property_read_string(struct device_node *np,const char *propname, const char **out_string) |
| 作用      | 获取属性值，使用于属性值为字符串。                           |
| 原型      | int of_get_named_gpio_flags(struct device_node *np,const char *list_name, int index, enum of_gpio_flags*flags) |
| 作用      | 获取GPIO信息。                                               |

##### 5.4.2.2 获取主键下GPIO列表

| script API： |                                                              |
| :----------- | :----------------------------------------------------------- |
| 原型         | int script_get_pio_list(char *main_key, script_item_u **list); |
| 作用         | 获取主键下GPIO列表。                                         |

| dts API：        |      |
| :--------------- | :--- |
| 说明无对应接口。 |      |


##### 5.4.2.3 获取主键数量.

|script API：||
|:---|:---|
|原型| unsigned int script_get_main_key_count(void);、
|作用 |获取主键数量。|

| dts API：        |      |
| :--------------- | :--- |
| 说明无对应接口。 |      |

##### 5.4.2.4 获取主键名称.

| script API： |                                                              |
| :----------- | :----------------------------------------------------------- |
| 原型         | char *script_get_main_key_name(unsigned int main_key_index); |
| 作用         | 通过主键索引号，获取主键名字。                               |

| dts API：        |      |
| :--------------- | :--- |
| 说明无对应接口。 |      |

##### 5.4.2.5 判断主键是否存在

| script API： |                                                |
| :----------- | :--------------------------------------------- |
| 原型         | bool script_is_main_key_exist(char *main_key); |
| 作用         | 判断主键是否存在。                             |

dts API：
|:---|:---|
|说明| dts标准接口支持四种方式判断节点是否存在。
|原型| struct device_node *of_find_node_by_name(struct device_node *from, const char *name)、
|作用 |通过节点名字。|
|原型 |struct device_node *of_find_node_by_path(const char *path)|
|作用 |通过节点路径。|
|原型 |struct device_node *of_find_node_by_phandle(phandle handle)|
|作用 |通过节点phandle属性。|
|原型 |struct device_node *of_find_node_by_type(struct device_node *from, const char *type)|
|作用 |通过节点device_type属性。|

### 5.5 接口使用例子.

#### 5.5.1 配置比较

下表展示了设备vdevice在sys_config.fex与dts中的配置，两种配置形式不一样，但实现的功能是等价的。

dts：

```
/*
device config in dts:
/{
soc@01c20000{
vdevice@0{
compatible = "allwinner,sun50i-vdevice";
device_type= "vdevice";
vdevice_0=<&pio 1 1 1 1 1 0>;
vdevice_1=<&pio 1 2 1 1 1 0>;
vdevice-prop-1=<0x1234>;
vdevice-prop-3="device-string";
status = "okay";
};
};
};
```

sys_config.fex：

```
device config in sys_config.fex
[vdevice]
compatible = "allwinner,sun50i-vdevice";
vdevice_used = 1
vdevice_0 = port:PB01<1><1><2><default>
vdevice_1 = port:PB02<1><1><2><default>
vdevice-prop-1 = 0x1234
```

```
vdevice-prop-3 = "device-string"
*/
```

说明：GPIO_IN/GPIO_OUT/EINT采用下边的配置方式，PIN采用另外配置，参考pinctrl使用说明文档。

```
vdevice_0=<&pio 1 1 1 1 1 0>;
| | | | | | | |-------------------电平
| | | | | | |----------------------上下拉
| | | | | |-------------------------驱动力
| | | | |----------------------------复用类型，0-GPIOIN 1-GPIOOUT..
| | | |------------------------------pin bank内偏移.
| | |---------------------------------哪个bank，PA=0，PB=1...以此类推
| |--------------------------------------指向哪个pio，属于cpus要用&r_pio
|-----------------------------------------------------属性名字，相当sys_config子键名
```

#### 5.5.2 获取整形属性值

通过script接口：

```
#include <linux/sys_config.h>
int get_subkey_value_int(void)
{
script_item_u script_val;
script_item_value_type_e type;
type = script_get_item("vdevice", "vdevice-prop-1", &script_val);
if (SCIRPT_ITEM_VALUE_TYPE_INT != type) {
return -EINVAL;
}
return 0;
}
```

通过dts接口：

```
#include <linux/of.h>
int get_subkey_value_int(void)
{
int ret;
u32 value;
struct device_node *node;
node = of_find_node_by_type(NULL,"vdevice");
if(!node){
return -EINVAL;
}
ret = of_property_read_u32(node, "vdevice-prop-1", &value);
if(ret){
return -EINVAL;
}
printk("prop-value=%x\n", value);
return 0;
}
```

#### 5.5.3 获取字符型属性值

通过script接口：

```
#include <linux/sys_config.h>
int get_subkey_value_string(void)
{
script_item_u script_val;
script_item_value_type_e type;
type = script_get_item("vdevice", "vdevice-prop-3", &script_val);
if (SCIRPT_ITEM_VALUE_TYPE_STR!= type) {
return -EINVAL;
}
return 0;
}
```

通过dts接口：

```
#include <linux/of.h>
int get_subkey_value_string(void)
{
int ret;
const char *string;
struct device_node *node;
node = of_find_node_by_type(NULL,"vdevice");
if(!node){
return -EINVAL;
}
ret = of_property_read_string(node, "vdevice-prop-3", &string);
if(ret){
return -EINVAL;
}
printk("prop-vlalue=%s\n", string);
return 0;
}
```

#### 5.5.4 获取gpio属性值

通过script接口：

```
#include <linux/sys_config.h>
int get_gpio_info(void)
{
script_item_u script_val;
```

```
script_item_value_type_e type;
type = script_get_item("vdevice", "vdevice_0", &script_val);
if (SCIRPT_ITEM_VALUE_TYPE_PIO!= type) {
return -EINVAL;
}
return 0;
}
```

通过dts接口：

```
#include <linux/sys_config.h>
#include <linux/of.h>
#include <linux/of_gpio.h>
int get_gpio_info(void)
{
unsigned int gpio;
struct gpio_config config;
struct device_node *node;
node = of_find_node_by_name(NULL,"vdevice");
if(!node){
return -EINVAL;
}
gpio = of_get_named_gpio_flags(node, "vdevice_0", 0, (enum of_gpio_flags *)&config);
if (!gpio_is_valid(gpio)) {
return -EINVAL;
}
printk("pin=%d mul-sel=%d drive=%d pull=%d data=%d gpio=%d\n",
config.gpio,
config.mul_sel,
config.drv_level,
config.pull,
config.data,
gpio);
return 0;
}
```

#### 5.5.5 获取节点

通过scritp接口：

```
#include <linux/sys_config.h>
int check_mainkey_exist(void)
{
int ret;
ret = script_is_main_key_exist("vdevice");
if(!ret){
return -EINVAL;
}
}
```

通过dts接口：


```
int check_mainkey_exist(void)
{
struct device_node *node_1, *node_2;
/* mode 1*/
node_1 = of_find_node_by_name(NULL,"vdevice");
if(!node_1){
printk("can not find node in dts\n");
return -EINVAL;
}
/*mode 2 */
node_2 = of_find_node_by_type(NULL,"vdevice");
If(!node_2){
return -EINVAL;
}
return 0;
}
```

### 5.6 其他

#### 5.6.1 sysfs设备节点.

device tree会解析dtb文件中，并在/sys/devices目录下会生成对应设备节点，其节点命名规则如下：

##### 5.6.1.1 “单元地址.节点名”

节点名的结构是“单元地址.节点名”，例如1c28000.uart、1f01400.prcm。

形成这种节点名的设备，在device tree里的节点配置具有reg属性。

```
uart0: uart@01c28000 {
compatible = "allwinner,sun50i-uart";
reg = <0x0 0x01c28000 0x0 0x400>;
};
prcm {
compatible = "allwinner,prcm";
reg = <0x0 0x01f01400 0x0 0x400>;
};
```

##### 5.6.1.2 “节点名.编号”

节点名的结构是“节点名.编号”，例如soc.0、usbc0.5。

形成这种节点名的设备，在device tree里的节点配置没有reg属性。


```
soc: soc@01c00000 {
compatible = "simple-bus";
......
};
usbc0:usbc0@0 {
compatible = "allwinner,sunxi-otg-manager";
......
};
```

编号是按照在device tree中的出现顺序从 0 开始编号，每扫描到这样一个节点，编号就增加1 ，如soc节点是第 1 个出现的，所以编号是 0 ，而usbc0是第 6 个出现的，所以编号是 5 。

device tree之所以这么做，是因为device tree中允许配置同名节点，所以需要通过单元地址或者编号来区分这些同名节点。可以参见内核的具体实现代码：

```
arm64_device_init()
->of_platform_populate()
->of_platform_bus_create()
->of_platform_device_create_pdata()
->of_device_alloc()
->of_device_make_bus_id()
of_device_make_bus_id()
{
......
reg = of_get_property(node, "reg", NULL);
if (reg) {
......
dev_set_name(dev, "%llx.%s", (unsigned long long)addr, node->name);
return;
}
magic = atomic_add_return(1, &bus_no_reg_magic);
dev_set_name(dev, "%s.%d", node->name, magic - 1);
}
```

## 6 设备树调试

介绍在不同阶段Device Tree配置信息查看方式。

### 6.1 测试环境

Kernel Menuconfig配置：

```
Device Drivers-->
Device Tree and Open Firmware support-->
Suppot for device tree in /proc
```

### 6.2 Build阶段.

#### 6.2.1 输出文件描述.

用户在tina根目录下执行make命令，执行编译固件的动作，该动作会生成device tree的二进制文件。

这些文件会存放到：

```
out/<方案名字>/compile_dir/target/linux-<方案名字>/<内核版本>/arch/arm/boot/dts
```

以sitar方案为例：

```
(R6平台)
1.sun3iw1p1-sitar-soc.dtb
2.sun3iw1p1-sitar-pd4.dtb
3.sun3iw1p1-sitar-mic.dtb
3.sun3iiw1p1-sitar-cuckoo.dtb
```

这些文件存放在：out/ sitar-{board} /compile_dir/target/linux-sitar-{board} /linux-3.10.65+/arch/arm/boot/dts路径下边

这个阶段的 dtb 文件只包含 linux-3.10.65+/arch/arm/boot/dts 的配置信息，不包括sys_config.fex配置信息。


#### 6.2.2 配置信息查看.

查看dtb配置信息的方法（在tina根目录下输入）：

以R6平台为例：

```
./lichee/linux-3.10/scripts/dtc/dtc -I dtb -O dts -o output_sunxi.dts \
./out/sitar -{board} /compile_dir/target/linux-sitar –{board} /linux-3.10.65+/arch/arm/boot
/dts /sun3iw1p1-soc.dtb
```

解析出来的output_sunxi.dts文件包含的信息，跟lichee/linux-3.10/arch/arm/boot/dts的配置信息完全一致。

### 6.3 Pack阶段

#### 6.3.1 输出文件描述.

用户在tina根目录下执行pack命令，执行打包生成固件的动作，该动作会生成dtb文件：

以R6平台为例：

sunxi.dtb存放在out/sitar-{board}/image。

这个阶段 sunxi.dtb 文件包含 lichee/linux-3.10/arch/arm/boot/dts/的配置信息，还包含
sys_config.fex配置信息。

#### 6.3.2 配置信息查看.

查看dtb配置信息的方法：

可以通过反编译，从最终的dts生成dtb。

以R6平台为例：

```
./lichee/linux-3.10/scripts/dtc/dtc -I dtb -O dts -o output.dts ./out/sitar-${board} /image
/sunxi.dtb
```

解析出来的output.dts文件包含的信息，跟lichee/linux-3.10/arch/arm/boot/dts/的配置信
息不完全一致，因为有些配置会被sys_config.fex更新。

默认pack的时候，会反编译一份，输出到：

```
out/方案/image/.sunxi.dtb
```

### 6.4 系统启动boot阶段

当firmware下载到target device之后，target device启动到uboot的时候，也可以查看dtb配置信息。

在uboot的控制台输入：

```
fdt --help
```

可以看到uboot提供的可以查看、修改dtb的方法：

```
fdt - flattened device tree utility commands
Usage:
fdt addr [-c] <addr> [<length>] - Set the [control] fdt location to <addr>
fdt move <fdt> <newaddr> <length> - Copy the fdt to <addr> and make it active
fdt resize - Resize fdt to size + padding to 4k addr
fdt print <path> [<prop>] - Recursive print starting at <path>
fdt list <path> [<prop>] - Print one level starting at <path>
fdt get value <var> <path> <prop> - Get <property> and store in <var>
fdt get name <var> <path> <index> - Get name of node <index> and store in <var>
fdt get addr <var> <path> <prop> - Get start address of <property> and store in <var>
fdt get size <var> <path> [<prop>] - Get size of [<property>] or num nodes and store in <
var>
fdt set <path> <prop> [<val>] - Set <property> [to <val>]
fdt mknode <path> <node> - Create a new node after <path>
fdt rm <path> [<prop>] - Delete the node or <property>
fdt header - Display header info
fdt bootcpu <id> - Set boot cpuid
fdt memory <addr> <size> - Add/Update memory node
fdt rsvmem print - Show current mem reserves
fdt rsvmem add <addr> <size> - Add a mem reserve
fdt rsvmem delete <index> - Delete a mem reserves
fdt chosen [<start> <end>] - Add/update the /chosen branch in the tree
<start>/<end> - initrd start/end addr
fdt save - write fdt to flash
```

常用的比如：

```
fdt print --打印整棵设备树。
fdt printf /soc/vdevice --打印“/soc/vdevice”路径下的配置信息。
fdt set /soc/vdevice status "disabled" --设置“/soc/vdevice”下status属性的属性值。
fdt save --fdt set之后需要执行fdt save才能真正写入flash保存。
```

### 6.5 系统启动kernel阶段.

内核配置了CONFIG_PROC_DEVICETREE = y之后，在/proc/device-tree文件夹下的文件节点可以读取到dtb的配置信息。

```
说明
注：该文件节点下配置信息只能读不能写。
```

