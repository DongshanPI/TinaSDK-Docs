## 6 其他功能

### 6.1 在用户空间操作env

Tina 中支持了uboot-envtools 软件包。

```
make menuconfig --> Utilities ---> <*> uboot-envtools
```

选上后即可在用户空间使用fw_printenv 和fw_setenv 来读写env 分区的变量。

### 6.2 AB 系统切换

#### 6.2.1 uboot 原生启动计数机制

uboot 原生支持了启动计数功能。该功能主要涉及三个变量。

```
upgrade_available=0/1 #总开关，设置1才会进行bootcount++及检查切换，设为0则表示关闭此功能
bootcount=N #启动计数，若upgrade_available=1，则每次启动bootcount++，并用于跟bootlimit比较
bootlimit=5 #配置判定启动失败的阈值
```

即编译支持该功能后，当upgrade_available=1，则uboot 会维护一个bootcount 计数值，每次启动自动加一，并判断bootcount 是否超过bootlimit。如果未超

过，则正常启动，执行env中配置的bootcmd 命令。如果超过，则执行env 中的altbootcmd 命令。

支持此功能后，启动脚本或主应用需要在合适的时机清除bootcount 计数值，表示已经正常启动。

配置：

```
使能：配置CONFIG_BOOTCOUNT_LIMIT
选择bootcount存放位置，
如存放在env分区(掉电不丢失)：配置CONFIG_BOOTCOUNT_ENV
如存放在RTC寄存器(掉电丢失)：配置CONFIG_BOOTCOUNT_RTC
如需存放在其他位置可自行拓展
```

在用户空间可配置upgrade_available 的值对此功能进行动态开关。例如可在OTA 之前打开，确认OTA 成功后关闭。也可一直保持打开。在用户空间，需要清空

bootcount，否则多次重启就会导致bootcount 超过bootlimit。

#### 6.2.2 全志定制系统切换

全志自己添加了CONFIG_SUNXI_SWITCH_SYSTEM 功能进行系统切换。目前尚未大量使用，供参考。

对上接口：

在flash 的env 分区中设置了2 个标志变量systemAB_now,systemAB_next。systemAB_now：由uboot 阶段进行设置, 该标志位系统应用只读不能写（通过工具

fw_printenv，或其他工具）。systemAB_next：无论是uboot 还是系统应用都可读可写（一般uboot 不会主动修改，除非系统已经损坏需要切换）初始值一般设

置为

```
systemAB_next=A #表示下次启动A系统
systemAB_now=A #表示当前为A系统，不设置也可以，uboot会自动设置
```

当前系统应用可以通过systemAB_now 标志识别当前系统是A 系统还是B 系统, 系统应用可以把标志变量systemAB_next=A/B 写到env 分区里, 然后重启. 重启后

uboot 会去检查systemAB_next 这个标志, 如果标志是systemAB_next=A,uboot 会启动A 系统, 如果是systemAB_next=B,uboot 会启动B 系统。同时uboot 会根

据本次systemAB_next 的值设置systemAB_now 供系统读取。

底层设置：

对上接口定义的是systemA/B，以系统为单位。目前的系统组成定义为包含kernel 和rootfs 分区。考虑不同方案，分区名可能不同，因此支持用环境变量指定具体

分区名，而不是hardcode为某个分区名。

```
systemA=boot A系统kernel分区名
rootfsA=rootfs A系统rootfs分区名
systemB=boot_B B系统kernel分区名
rootfsB=rootfs_B B系统rootfs分区名
```

底层实现：
此机制会动态修改boot_partition 和root_partition 的值。具体boot_partition 和root_partition 变量的作用，请参考本文档其他章节的介绍。