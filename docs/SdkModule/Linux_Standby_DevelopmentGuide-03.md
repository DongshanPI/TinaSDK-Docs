## 3 FAQ

### 3.1 调试方法

#### 3.1.1 调试节点

*•* pm_test 节点

该节点可用于测试 linux 部分休眠唤醒功能。Eg：echo x > /sys/power/pm_test。

Freezer：表明，任务冻结后，等待 5s，即返回，执行唤醒动作。

Devices: 表明，设备冻结后，等待 5s, 即返回，执行唤醒动作。

Platform：在 a1x, a2x, a3x 上，与 devices 相同；

Processors: 冻结 non-boot cpu 后，等待 5s, 即返回，执行唤醒动作。

Core: 冻结 timer 等系统资源后，等待 5s, 即返回，执行唤醒动作。

None: 表明，整个休眠流程全部走完，等待唤醒源唤醒。

*•* wake_lock 节点

该节点可查看安卓系统 wake lock 状态，安卓系统在持锁时不会进入深度睡眠流程 (Suspend-to-mem)。Eg: cat /sys/power/wake_lock。 

*•* wakeup_sources 节点

该节点可查看系统唤醒源的情况。Eg：cat /sys/kernel/debug/wakeup_sources。



### 3.2 常见问题

#### 3.2.1 系统被错误唤醒

##### 3.2.1.1 系统被定时器唤醒

**问题现象**

休眠后，自动被唤醒，过会自动进入休眠，屏幕黑屏，串口有输出。

**问题分析**

系统休眠后自动被唤醒，原因可能是，某些应用或者后台进程，通过设置闹钟的方式，定时唤醒系统。

当出现如下打印，表示 Linux 已经休眠完成，准备进入 CPUS 休眠阶段：

```
[ 3465.885063] PM: noirq suspend of devices complete after 16.487 msecs
[ 3465.892225] Disabling non-boot CPUs ...
......
```

当出现以上打印后自动唤醒，则查看如下打印：

```
[ 3466.063570] wake up source:0x80000 //H3 linux4.4平台
[21676.174594] [pm]platform wakeup, standby wakesource is:0x100000 //H5/H6 linux-3.10平台
后面的数字代表唤醒源，根据数字定位唤醒源，定位唤醒源后再判断为何被唤醒。
WAKEUP_SRC is as follow:
CPUS_WAKEUP_LOWBATT bit 0x1000
CPUS_WAKEUP_USB bit 0x2000
CPUS_WAKEUP_AC bit 0x4000
CPUS_WAKEUP_ASCEND bit 0x8000
CPUS_WAKEUP_DESCEND bit 0x10000
CPUS_WAKEUP_IR bit 0x80000
CPUS_WAKEUP_ALM0 bit 0x100000
CPUS_WAKEUP_HDMI_CEC bit 0x100000
```

**使用范围**适用于非 psci1.0 版本，详见 dts 文件 psci 节点配置。

常见场景：android 某些应用或者后台进程，会通过设置闹钟的方式，定时唤醒系统，当判断唤醒源为 0x100000 时，大多数为该原因导致。

**问题解决**

确认是某些应用或者后台进程设置闹钟定时唤醒系统，方案开发人员可以自行解决。



##### 3.2.1.2 系统被其他唤醒源唤醒

**问题现象**

休眠后，被异常唤醒。

**问题分析**

系统休眠后被异常唤醒，原因可能是，被其他非预期的唤醒源唤醒。

查看唤醒源。对应的代码路径在：lichee/linux4.9/include/linux/power/aw_pm.h

```
/* the wakeup source of assistant cpu: cpus */
#define CPUS_WAKEUP_HDMI_CEC (1<<11)
#define CPUS_WAKEUP_LOWBATT (1<<12)
#define CPUS_WAKEUP_USB (1<<13)
#define CPUS_WAKEUP_AC (1<<14)
#define CPUS_WAKEUP_ASCEND (1<<15)
#define CPUS_WAKEUP_DESCEND (1<<16)
#define CPUS_WAKEUP_SHORT_KEY (1<<17)
#define CPUS_WAKEUP_LONG_KEY (1<<18)
#define CPUS_WAKEUP_IR (1<<19)
#define CPUS_WAKEUP_ALM0 (1<<20)
#define CPUS_WAKEUP_ALM1 (1<<21)
#define CPUS_WAKEUP_TIMEOUT (1<<22)
#define CPUS_WAKEUP_GPIO (1<<23)
#define CPUS_WAKEUP_USBMOUSE (1<<24)
#define CPUS_WAKEUP_LRADC (1<<25)
#define CPUS_WAKEUP_WLAN (1<<26)
#define CPUS_WAKEUP_CODEC (1<<27)
#define CPUS_WAKEUP_BAT_TEMP (1<<28)
#define CPUS_WAKEUP_FULLBATT (1<<29)
#define CPUS_WAKEUP_HMIC (1<<30)
#define CPUS_WAKEUP_POWER_EXP (1<<31)
#define CPUS_WAKEUP_KEY (CPUS_WAKEUP_SHORT_KEY | CPUS_WAKEUP_LONG_KEY)
```

查看关键打印：

```
platform wakeup, standby wakesource is:0x800000
```

此时对应的是 gpio 唤醒。

**问题解决**

确认是其他非预期的唤醒源唤醒系统，方案开发人员可以自行解决。



#### 3.2.2 系统不能被唤醒

##### 3.2.2.1 休眠后无法唤醒

**问题现象**

系统休眠后无法唤醒。



**问题分析**

系统休眠后无法唤醒，原因可能有：

*•* 模块休眠失败。

查看是否模块休眠失败，输入以下命令，确认是否在内核休眠唤醒模块出异常。

```
echo N > /sys/module/printk/parameters/console_suspend
echo 1 > /sys/power/pm_print_times
```

*•* 若上述无异常打印，则认为是 Linux 后的阶段出现异常。

不断电重启系统，将启动时候的 RTC 寄存器的信息发给休眠模块负责人，根据 RTC 寄存器信息判断。

```
[2341]HELLO! pmu_init stub called!
[2645]set pll start
[2648]set pll end
[2649]try to probe rtc region
[2652]rtc[0] value = 0x00000000
[2655]rtc[1] value = 0x000000e0
[2658]rtc[2] value = 0xf1f18000
[2661]rtc[3] value = 0x0000000f
[2663]rtc[4] value = 0x00000000
[2666]rtc[5] value = 0x00000000
```

**问题解决**

*•* 模块休眠失败。确认是模块休眠失败，方案开发人员可以自行解决。

*•* Linux 后的阶段出现异常。将复位重启时的 RTC 寄存器信息发给相关负责人。



##### 3.2.2.2 唤醒源不支持唤醒

**问题现象**

休眠后，唤醒源无法唤醒系统，串口没有输出。

**问题分析**

休眠后，唤醒源无法唤醒，可能是唤醒源不支持。

*•* cpus 休眠后异常。

当出现如下打印时表示 Linux 休眠已经完毕，此时唤醒不了，则可能是 cpus 退出休眠失败，或者唤醒源不对。

```
[ 3465.885063] PM: noirq suspend of devices complete after 16.487 msecs
[ 3465.892225] Disabling non-boot CPUs ...
......
```

通过以下手段可以判断 cpus 休眠后是否正常运行，以下命令表示休眠后 cpus 过一定时间软件自动唤醒。

如果串口能正常打印，wake up source 为 0x400000，则表示 cpus 是正常运行的，这时应该排查一下系统是否支持相应的唤醒源。

*•* 唤醒源不支持。

确认唤醒源不支持的情况。

**问题解决**

*•* cpus 休眠后异常。

将复位重启时的 RTC 寄存器信息发给相关负责人。

*•* 唤醒源不支持。

将唤醒源的情况发给相关负责人。



##### 3.2.2.3 红外遥控器不能唤醒系统

**问题现象**

红外遥控不能唤醒系统。

**问题分析**

红外遥控唤醒需要配置唤醒源。

**问题解决**

红外遥控器默认支持 NEC 红外协议遥控器唤醒，也支持 RC5 红外协议遥控器唤醒，但是需要在sys_config.fex 进行配置，配置如下：

```
;--------------------------------------------------------------------------------
;ir --- infra remote configuration
;ir_protocol_used : 0 = NEC / 1 = RC5
;--------------------------------------------------------------------------------
[s_cir0]
s_cir0_used = 1
ir_used = 1
ir_protocol_used = 0 //置为0 支持NEC 红外协议遥控唤醒，配置为0 支持RC5 红外协议遥控唤醒
```



对于同一协议的红外遥控器，能支持唤醒的个数也是有限的，具体在 sys_config.fex 配置 s_cir0 节点。

```
ir_power_key_code0 = 0x57 //遥控POWER 值
ir_addr_code0 = 0x9f00    //遥控设备码
ir_power_key_code1 = 0x1a
ir_addr_code1 = 0xfb04
```



##### 3.2.2.4 USB设备不能唤醒系统

**问题现象**

USB 不能唤醒系统。

**问题分析**

USB 需要设置为唤醒源。

**问题解决**

USB 设备唤醒需要系统支持 USB_STANDBY，需要在 sys_config.fex 配置 usbc* 节点。需要

注意 [usbc0] usb_wakeup_suspend = 1 //1 表示支持 USB0 唤醒，0 表示屏蔽该唤醒源。



##### 3.2.2.5 hdmi_cec 不能唤醒系统

**问题现象**

HDMI 不能唤醒系统。

**问题分析**

HDMI 需要设置为唤醒源。

**问题解决**

需要在 sys_config.fex 配置 hdmi 节点。

```
[hdmi]
hdmi_cec_support = 1
hdmi_cec_super_standby = 1
```



##### 3.2.2.6 cpus 退出休眠失败

**问题现象**

休眠后，无法唤醒，串口没有输出。



**问题分析**

可能是 cpus 退出休眠失败。

如果通过写 time_to_wakeup 命令，系统没法正常唤醒，则考虑是 cpus 退出休眠失败的了，这时需要短接 reset 脚重启系统（注意不是完全断电，完全断电将无法保留 RTC 值），然后将 boot 阶段打印的 RTC 码值发送给休眠唤醒负责人，定位问题。

```
[2341]HELLO! pmu_init stub called!
[2645]set pll start
[2648]set pll end
[2649]try to probe rtc region
[2652]rtc[0] value = 0x00000000
[2655]rtc[1] value = 0x000000e0
[2658]rtc[2] value = 0xf1f18000
[2661]rtc[3] value = 0x0000000f
[2663]rtc[4] value = 0x00000000
[2666]rtc[5] value = 0x00000000
```

**问题解决**

*•* 确认是否 dram 错误。

*•* 确认是否上下电时序错误。

*•* 将复位重启时的 RTC 寄存器信息发给相关负责人。



#### 3.2.3 系统无法休眠

##### 3.2.3.1 系统持锁无法休眠

**问题现象**

系统持锁，suspend 失败。

**问题分析**

suspend 失败，可能是系统持锁阻止休眠。

**问题解决**

*•* 安卓查看是否有持锁相关信息：

dumpsys power | grep PART

*•* 内核中是否有相关持锁信息：

cat /sys/kernel/debug/wakeup_sources

查看 active_since 项，若对应模块不为 0，则该模块一直阻止系统进入休眠，查看该模块是否异常；

cat /sys/power/wake_lock

查看是否有安卓申请的锁。

*•* Linux devices driver suspend 失败：



##### 3.2.3.2 Android 系统持锁无法休眠

**问题现象**

定时休眠到时后，屏幕亮屏，串口可以输入，系统无法休眠。

**问题分析**

系统无法休眠，确认 Android 是否支持，是否禁止定时休眠。

串口输入 dumpsys power，查看如下打印。如果发现 mStayOn=true，则系统是不支持定时休眠功能，可以将该属性设置成 false；另外 screen off timeout 表示休眠时间，可通过该值判断你设置的定时时间是否正确；如果是 androidN ，screen off timeout 还取决于 Sleep timeout，Sleep timeout 的值比 Screen off timeout 小时，系统取 Sleep timeout 当做定时休眠的时间，当 Sleep timeout 为-1 时，也表示系统无法定时休眠。如果是以上情况可咨询 android 系统的同事，修改相应的属性。

```
Power Manager State:
......
mStayOn=true //true 保持常亮，false 可以支持定时休眠
Sleep timeout: -1 ms //只有androidN 平台无该该值，定时休眠时间
Screen off timeout: 1800000 ms //定时灭屏时间
Screen dim duration: 7000 ms //屏保时间
```

常见场景：eng 固件开发阶段为了测试长时间老化的测试项，会禁止系统定时进入休眠。

**问题解决**

确认是 Android 系统持锁阻止休眠，方案开发人员可以自行解决。



#### 3.2.4 休眠唤醒过程中挂掉

##### 3.2.4.1 分阶段过程挂掉

**问题现象**

在 Linux 某个阶段出现的休眠或者唤醒失败。

**问题分析**

打开内核选项

```
CONFIG_PM_DEBUG=y
```



查看具体失败在哪个阶段：

1. echo freezer > /sys/power/pm_test 查看 freezer 阶段是否正常；

2. echo devices > /sys/power/pm_test 查看 freezer/devices 阶段是否正常；

3. echo platform > /sys/power/pm_test 查看 freezer/devices/platform 阶段是否正常；

4. echo processors > /sys/power/pm_test 查看 freezer/devices/platform/processors 阶段是否正常；

5. echo core > /sys/power/pm_test 查看 freezer/devices/platform/processors/core 阶段是否正常；

6. echo mem > /sys/power/state 测试分阶段休眠唤醒。

**问题解决**

确认是哪个阶段出现的休眠或者唤醒失败，方案开发人员可以自行解决。



