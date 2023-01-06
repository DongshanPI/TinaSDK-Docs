## 3 FAQ

### 3.1 常见问题

#### 3.1.1 USB 基本功能异常排查

##### 3.1.1.1 USB Host 基本功能异常排查步骤

*•* 多找几个 USB 设备试试，排除个别 USB 设备本身的问题。

*•* 多更换几根 USB 线缆试试，排除个别 USB 线缆的问题。

*•* 多找几个 PC 主机做相同的实验，作为参考对比。若在 PC 有相同现象，则认为正常。

*•* 若硬件有多个 USB 口，尝试同样条件下测试其他 USB 口的主机功能是否正常。

*•* 样机设备 USB 口外接独立供电的 USB-HUB 设备，再将 USB 设备连接到 USB-HUB 上，确认主机功能是否正常。

*•* 确认主机驱动是否加载成功。

（1）若为 USB0 口，则可通过如下方式确认:

```
cat /sys/devices/platform/soc/usbc0/otg_role
```

（2）若为 USB1 口，可通过如下方式确认:

```
cat sys/devices/platform/soc/5200000.ehci1-controller/ehci_enable
cat sys/devices/platform/soc/5200000.ohci1-controller/ohci_enable
```

若为0，则没有加载Host驱动。



*•* 重新加载 Host 驱动，确认此时功能是否正常。

（1）若为 USB0 口，则可通过如下方式:

```
方式1：重新插拔OTG线。
方式2：手动切换到Host模式。
```

（2）若为 USB1 口，则可通过卸载驱动、再加载驱动。

*•* 对比 SDK 代码与最新发布的代码或者补丁, 确认代码是否更新到最新。

*•* 同样条件下，分别打印出功能异常板子和功能正常板子的相关寄存器，并进行对比，确认是否有不同之处。

*•* 出现异常时，测试 USB 高速眼图是否正常。

*•* 若眼图测试未通过，可尝试调节眼图参数。



##### 3.1.1.2 USB Device 基本功能异常排查步骤

*•* 多换几个 PC 主机做相同的测试，排除个别 PC 的问题。

*•* 多更换几根 USB 线缆做相同的测试，排除个别 USB 线缆的问题。

*•* 确认 Device 驱动是否加载成功，可通过如下方式: 

（1）通过 Log。

```
[ 104.732695]  insmod_device_driver
[ 104.732695] 
device_chose finished!
```

（2）通过节点查看当前 Role。 

*•* 重新加载 Device 驱动，确认此时功能是否恢复正常。

（1）重新插拔 USB 线。

（2）手动切换到 Device 模式。

*•* 对比 SDK 代码与最新发布的代码或者补丁, 确认代码是否更新到最新。

*•* 同样条件下，分别打印出功能异常板子和功能正常板子的相关寄存器，并进行对比，确认是否有异常。

*•* 出现异常时，确认 USB 高速眼图是否正常。



#### 3.1.2 配置其他 gadget 功能前关闭 adb 功能时却报异常的解决办法

问题产生的原因是：仅执行./etc/adb_conf.sh stop 只是强制杀死 adb 守护进程，但 adb 功能链接仍存，当配置其他 gadget 功能时，便会复合 adb 链接导致异常，故在需要配置其他 gadget 功能时，除了强制杀死 adb 守护进程还须移除 adb 功能链接，在小机中操作步骤如下：

```
1、./etc/adb_conf.sh stop
2、umount /sys/kernel/config
3、rm -fr /sys/kernel/config/usb_gadget/g1/configs/c.1/ffs.adb
```

执行以上操作，正常关闭 adb 后，根据需要的 gadget 功能，参考【附录】章节进行配置即可。