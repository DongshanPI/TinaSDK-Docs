## 3 模块使用范例

### 3.1 外部sysfs 节点

AXP 注册了许多外部sysfs 节点可供调试使用。

#### 3.1.1 Regulator

有关于regulator 调试节点，主要调整各路电压输出的。

在路径/sys/class/regulator/ 下，有关于单路regulator 属性的节点可以读取数值。

| 文件路径                   | 功能                   | 属性 | 设置值                                                |
| -------------------------- | ---------------------- | ---- | ----------------------------------------------------- |
| regulator.X/name           | 各路输出名字           | r    | 对应的输出的名字                                      |
| regulator.X/max_microvolts | 各路输出最大电压       | r    | 对应的最大电压值，单位 uV                             |
| regulator.X/min_microvolts | 各路输出最小电压       | r    | 对应的最小电压值，单位<br/>uV                         |
| regulator.X/state          | 各路输出状态           | r    | 对应的输出状态，<br/>enabled/disabled: 开<br/>启\关闭 |
| regulator.X/num_users      | 各路输出对应的设备个数 | r    | 0\1\2\……                                              |

因为不同AXP 型号regulator 对应的节点不一样，因此下面将分开不同AXP 进行介绍。

• AXP717

电源名称对应表如下：

| 节点名字     | 原理图名称 |
| ------------ | ---------- |
| regulator.0  | dummy      |
| regulator.1  | usb0-vbus  |
| regulator.2  | usb1-vbus  |
| regulator.3  | dcdc1      |
| regulator.4  | dcdc2      |
| regulator.5  | dcdc3      |
| regulator.6  | dcdc4      |
| regulator.7  | aldo1      |
| regulator.8  | aldo2      |
| regulator.9  | aldo3      |
| regulator.10 | aldo4      |
| regulator.11 | bldo1      |
| regulator.12 | bldo2      |
| regulator.13 | bldo3      |
| regulator.14 | bldo4      |
| regulator.15 | cldo1      |
| regulator.16 | cldo2      |
| regulator.17 | cldo3      |
| regulator.18 | cldo4      |
| regulator.19 | rtcldo     |
| regulator.20 | cpusldo    |
| regulator.21 | drivevbus  |

#### 3.1.2 Virtual-consumer

在路径/sys/devices/platform/soc/7081400.s_twi/i2c-6/6-0034/ 下则有修改regulator 输出电压的节点。

| 文件路径                               | 功能             | 属性 | 设置值                        |
| -------------------------------------- | ---------------- | ---- | ----------------------------- |
| reg-virtconsumer. X/max_microvolts     | 设置输出最大电压 | rw   | 对应的最大电压值，单 位uV     |
| reg-virtconsumer.<br/>X/min_microvolts | 设置输出最小电压 | rw   | 对应的最小电压值，单<br/>位uV |
| of_node/name                           | 各路输出名字     | r    | 无                            |

因为不同AXP 型号Virtual-consumer 对应的节点不一样，因此下面将分开不同AXP 进行介绍。

• AXP717

virt-consumer 对应的电路的对应表如下：

| 节点名字             | 原理图名称 |
| -------------------- | ---------- |
| reg-virt-consumer.1  | dcdc1      |
| reg-virt-consumer.2  | dcdc2      |
| reg-virt-consumer.3  | dcdc3      |
| reg-virt-consumer.4  | dcdc4      |
| reg-virt-consumer.5  | aldo1      |
| reg-virt-consumer.6  | aldo2      |
| reg-virt-consumer.7  | aldo3      |
| reg-virt-consumer.8  | aldo4      |
| reg-virt-consumer.9  | bldo1      |
| reg-virt-consumer.10 | bldo2      |
| reg-virt-consumer.11 | bldo3      |
| reg-virt-consumer.12 | bldo4      |
| reg-virt-consumer.13 | cldo1      |
| reg-virt-consumer.14 | cldo2      |
| reg-virt-consumer.15 | cldo3      |
| reg-virt-consumer.16 | cldo4      |
| reg-virt-consumer.17 | rtcldo     |
| reg-virt-consumer.18 | cpusldo    |

#### 3.1.3 Power_supply

在路径/sys/class/power_supply/ 下有power_supply 相关的调试节点，可以读出电池以及供电当前状态和设置属性。

下表是电池相关的调试节点，路径为：/sys/class/power_supply/axp2202-battery/

| 文件路径                | 功能             | 属性 | 设置值                                                       |
| ----------------------- | ---------------- | ---- | ------------------------------------------------------------ |
| capacity                | 电池剩余电量     | r    | 百分比，0\1\2\……\100                                         |
| capacity_alert_min      | 低电量警告阈值   | r    | 百分比                                                       |
| capacity_level          | 当前充电等级     | r    | “UNKNOWN” 未知，<br/>“Critical” 接近没电，“LOW” 低<br/>电量，“NORMAL” 正常电量，<br/>“HIGH” 高电量，“FULL” 满电 |
| charge_counter          | 当前电池容量     | r    | 单位mAh                                                      |
| charge_full             | 充满电的电池容量 | r    | 单位mAh                                                      |
| constant_charge_current | 恒定充电电流     | r    | 单位mA                                                       |
| energy_full_design      | 充满电的电池容量 | r    | 单位mAh                                                      |
| health                  | 电池状况         | r    | “Unknown” 未知, “Good” 好,<br/>“Overheat” 过温, “Dead” 坏掉,<br/>“Over voltage” 过<br/>压,“Unspecified failure” 错误,<br/>“Cold” 冷 |
| present                 | 电池存在         | r    | 0\1：存在\不存在                                             |
| serial_number           | pmu 型号         | r    | string                                                       |
| status                  | 电池当前状态     | r    | “Unknown” 未知, “Charging”<br/>正在充电, “Discharging” 放电,<br/>“Not charging” 未在充电,<br/>“Full” 满 |
| temp                    | 电池温度         | r    | 单位°C                                                       |
| temp_alert_min          | 电池高温预警阈值 | r    | 单位°C                                                       |
| time_to_empty_now       | 放电剩余时间     | r    | 单位min                                                      |
| time_to_full_now        | 充电剩余时间     | r    | 单位min                                                      |
| type                    | 设备类别         | r    | “battery” 电池,“Mains” 火<br/>牛,“USB”USB                    |
| voltage_now             | 当前电压         | r    | 单位uA                                                       |

下表则是供电相关的节点，路径为：/sys/class/power_supply/axp2202-usb

| 文件路径            | 功能                | 属性  | 设置值                                         |
| ------------------- | ------------------- | ----- | ---------------------------------------------- |
| input_current_limit | 输入电流限流值      | r     | 单位mA                                         |
| online USB          | 是否在使用          | r     | 0\1：未使用\正在使用                           |
| present USB         | 是否接上            | r     | 0\1：插上\没插上                               |
| serial_number       | pmu                 | 型号r | string                                         |
| type                | 设备类别            | r     | “battery” 电<br/>池,“Mains” 火<br/>牛,“USB”USB |
| voltage_min_design  | DC 供电最小设计电压 | r     | 单位uV                                         |
| voltage_now         | DC 供电时的电压大小 | r     | 单位uV                                         |

**说明**

AXP717 没有使用acin，因此接入usb 或者适配器都是使用usb-power-supply 的驱动，节点也是公用的。



### 3.2 Regulator 使用方法

#### 3.2.1 内核代码调用regulator 示例

以DCDC1 为例，需要设置DCDC1 最大输出电压值为3.4V，需要设置目标电压值为3V。

```
#include <linux/regulator/consumer.h>
struct regulator *regu= NULL;
    int ret = 0;
    regu= regulator_get(NULL, "axp2202_dcdc1");
    if (IS_ERR(regu)) {
        pr_err("%s: some error happen, fail to get regulator \n", __func__);
        goto exit;
    }
    //set output voltage to 3V
    ret = regulator_set_voltage(regu, 3000000, 3400000);
    if (0 != ret) {
        pr_err("%s: some error happen, fail to set regulator voltage!\n", __func__);
        goto exit;
    }
    //enalbe regulator
    ret = regulator_enable(regu);
    if (0 != ret) {
        pr_err("%s: some error happen, fail to enable regulator!\n", __func__);
        goto exit;
    }
    //disalbe regulator
    ret = regulator_disable(regu);
    if (0 != ret) {
        pr_err("%s: some error happen, fail to disable regulator!\n", __func__);
        goto exit;
    }
    //put regulater, when module exit
    regulator_put(regu);
```

**技巧**
regulator_get 调用有两种方法：
	第一种就是直接获取dts 中regulator 的句柄，比如说axp2202_dcdc1。调用regulator_get 时第一个参数不需要配置设备，直接写NULL。
	第二种是在dts 的设备里配置regulator 的节点，比如说dts 某个设备里配置regulator0 = "axp2202\_dcdc1";，该设备调用时写法为：regulator_get(dev, "regulator0");。

#### 3.2.2 Regulator shell 命令使用示例

AXP regulator 可以通过shell 命令控制和设置其开关以及输出电压，各路文件节点建立在在/sys/devices/platform 目录下

• AXP717

AXP717 可控的节点可参考上面virtual 章节Virtual-consumer

以设置DCDC1 输出最大电压为3.3V，设置目标电压为3.0V 为例做说明。

```
cd /sys/devices/platform/soc/7081400.s_twi/i2c-6/6-0034/reg-virt-consumer.1
cat of_node/name //确认是否为DCDC1
//设置输出电压为3.0V
echo 3300000 > max_microvolts
echo 3000000 > min_microvolts
//关闭输出
echo 3300000 > max_microvolts
echo 3000000 > min_microvolts
echo 0 > min_microvolts
```

#### 3.2.3 usb_count 查看

根据上面Regulator 章节Regulator 找到对应的regulator 节点，这里以dcdc1 为例，其节点名称为regulator.3，则在/sys/class/regulator 目录下就有个

regulator.3 目录，regulator.3目录有个num_users 节点，cat 此节点就可以获得当前use_count 值。

```
cat /sys/class/regulator/regulator.3/num_users
```

num_uesrs 代表当前有多少设备使用了regulator 节点用来控制输出电压。