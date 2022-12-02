# 编译烧写boot
简述：SSD202D芯片的boot阶段也是我们常说的bootLoader，是基于uboot2015进行开发适配，里面增加了大量的芯片原厂相关代码，和社区版本存在很大差异，由于SSD202只支持从NANDFLASH / NorFLASH启动，所以如果您的开发板上没有boot分区或者不小心破坏了boot分区，那么系统就无法正常启动，需要通过专门的烧录工具才可以进行烧录更新。

> 烧录工具受芯片市场供货影响，价格非常高，所以我们不建议大家操作这部分。

如果您发现您跟着此文档操作出现问题，可以直接查看我们提前录制好的手把手教学视频。


## 编译bootloader

首先进入boot源码根目录下，在编译之前需要确认清楚交叉编译工具链等环境是否配置成功，如果不确定 可以参考 [配置交叉编译工具链](/DongshanPi-One/05-GetSourceCode/#_4) 章节。

> make infinity2m_spinand_defconfig

指定完成配置文件后，继续执行如下命令开始编译（根据服务器配置，可以支持多线程编译）
> make -j8

* 参考执行示例
``` shell
book@100ask:~/DongshanPiOne-TAKOYAKI/boot$ make infinity2m_spinand_defconfig
arch/../configs/.tmp_defconfig:326:warning: override: reassigning to symbol CMD_FASTBOOT
#
# configuration written to .config
#
book@100ask:~/DongshanPiOne-TAKOYAKI/boot$ make -j16
```

最后生成的镜像文件为：
> u-boot_spinand.xz.img.bin

注意：如果需要打包 编译出来的镜像到最终的完整系统内则 需要讲生成的文件拷贝到project目录：
> cp u-boot_spinand.xz.img.bin project/board/i2m/boot/spinand/uboot/

* 参考执行示例
``` shell

book@100ask:~/DongshanPiOne-TAKOYAKI/boot$  cp u-boot_spinand.xz.img.bin  ../project/board/i2m/boot/spinand/uboot/

```

## 烧写bootloader

### 进入uboot内烧写
注意：**这种烧写方式是在您的开发板还可以正常启动，可以进入到uboot命令行内。**
首先参考上述操作将编译好的镜像拷贝到您的sd卡内，
之后开发板上电，在串口有输出时一直长按键盘 Enter 键3秒左右 之后放开就可以进入到uboot命令行内，
最后执行如下命令更新。

> fatload  mmc 0:1 0x21000000 u-boot_spinand.xz.img.bin
> 
> nand erase.part UBOOT0
> 
> nand write.e 0x21000000 UBOOT0 ${filesize}
> 
> nand erase.part UBOOT1
> 
> nand write.e 0x21000000 UBOOT1 ${filesize}



### 使用开源社区工具烧写
* 开源社区工具更详细的说明请看；[](https://dongshanpi.com/DongshanPi-One/19.OpensourceMstarTools/)
* 社区版烧写工具 连线介绍 https://item.taobao.com/item.htm?id=665728385852
* 将烧写工具连接好烧录线后，将烧录器和开发板烧写接口连接好，拨码开关切换为烧写模式(拨码开发白色拨码切换到非NO方向)。

![Mstar-I2C-Flash-tools](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/DongshanPI-ONE/Mstar-I2C-Flash-tools.png)<br>

之后我们按下 主板上`Reset`键，同时执行如下烧写命令，其中第一句话`sudo SNANDer -p mstarddc -c /dev/i2c-0:49 -e`是整个 擦除 flash 命令，建议执行。

注意:烧写操作是在ubuntu-18烧写，需要将烧写工具挂载到ubuntu系统内，挂载成功后系统会自动装在驱动模块，此时设备管理器会多出一个`/dev/i2c-0`的设备节点，其中烧写命令中 ` -l `参数指定的是烧写文件的长度(大小)，如果你修改了`u-boot_spinand.xz.img.bin`文件，请确认大小是否在这个区间内。

```shell
sudo SNANDer -p mstarddc -c /dev/i2c-0:49 -e

sudo SNANDer -p mstarddc -c /dev/i2c-0:49 -w GCIS.bin

sudo SNANDer -p mstarddc -c /dev/i2c-0:49 -a 0x140000 -l 0x6000 -w IPL.bin

sudo SNANDer -p mstarddc -c /dev/i2c-0:49 -a 0x200000 -l 0x5800 -w IPL_CUST.bin

sudo SNANDer -p mstarddc -c /dev/i2c-0:49 -a 0x2C0000 -l 0x3B800 -w u-boot_spinand.xz.img.bin
```



### 使用专门的烧写器烧写
注意：**本方式适用于空片烧录或者板子无法进入Uboot控制台使用其它方式烧录的情况**。

首先关闭串口调试工具，接下来打开烧写工具：烧写工具下载 [点击跳转](https://dongshanpi.com/DongshanPi-One/10-SupportTools/)  选择合适的下载方式下载。
其中烧录器购买链接地址：

同时将烧录器和开发板烧写接口连接好，拨码开关切换为烧写模式(拨码开发白色拨码切换到非NO方向)。
![FlashTools-01](https://cdn.staticaly.com/gh/codebug8/DongshanPi-Photos@master/FlashTools-01.png)

电脑解压缩下载后的 **ISP_5.0.18.zip** 烧写工具压缩包，打开 `Flash_Tool_5.0.18.exe`，点击Connect，建立连接状态（Connect必须确保关闭串口工具，否则会出现争抢串口资源问题）选择Flash Type为Nand Flash。

![FlashTools-02](https://cdn.staticaly.com/gh/codebug8/DongshanPi-Photos@master/FlashTools-02.png)

接下来需要根据根据以下的分区以及分区起始地址，按照以下方法依次烧录分区。
其中 `GCIS.bin IPL.bin IPL_CUST.bin u-boot_spinand.xz.img.bin `这些文件可以从我们提供提供默认系统镜像中获取 [恢复默认系统]()。

|分区文件 |	分区起始地址 |
|-------- | ---------- |
| GCIS.bin |	0x000000 |
| IPL.bin | 	0x140000 |
| IPL_CUST.bin | 	0x200000 |
| u-boot_spinand.xz.img.bin | 	0x2C0000 |

* 烧录  GCIS.bin :参考下图 **1** 红框选择文件所在位置，(注意这些镜像文件来源于 前面 常见问题章节  更新默认系统部分)。
选择完成后，参考下图 **2** 红框设置烧写地址，配置完成后可以点击 **箭头3** `Run`按钮来开始烧录。等待运行结束，直至提示Pass状态。
![FlashTools-03](https://cdn.staticaly.com/gh/codebug8/DongshanPi-Photos@master/FlashTools-03.png)

* 烧录  IPL.bin :参考下图**1 **红框选择文件所在位置，(注意这些镜像文件来源于 前面 常见问题章节  更新默认系统部分)。
选择完成后，参考下图 **2** 红框设置烧写地址，配置完成后可以点击 **箭头3** `Run`按钮来开始烧录。等待运行结束，直至提示Pass状态。
![FlashTools-04](https://cdn.staticaly.com/gh/codebug8/DongshanPi-Photos@master/FlashTools-04.png)

* 烧录  IPL_CUST.bin :参考下图**1 **红框选择文件所在位置，(注意这些镜像文件来源于 前面 常见问题章节  更新默认系统部分)。
选择完成后，参考下图 **2** 红框设置烧写地址，配置完成后可以点击 **箭头3** `Run`按钮来开始烧录。等待运行结束，直至提示Pass状态。
![FlashTools-05](https://cdn.staticaly.com/gh/codebug8/DongshanPi-Photos@master/FlashTools-05.png)

* 烧录 u-boot_spinand.xz.img.bin :参考下图**1 **红框选择文件所在位置，(注意这些镜像文件来源于 前面 常见问题章节  更新默认系统部分)。
选择完成后，参考下图 **2** 红框设置烧写地址，配置完成后可以点击 **箭头3** `Run`按钮来开始烧录。
![FlashTools-06](https://cdn.staticaly.com/gh/codebug8/DongshanPi-Photos@master/FlashTools-06.png)
点击Run，等待运行结束，直至提示Pass状态。烧录完之后执行reset重启即可正常启动。