
# 编译烧写Kernel
简述：东山Pi壹号开发板使用的内核是基于LinuxKernel主线4.9版本进行适配支持，支持了一些自有的控制器等专有的处理单元等驱动模块。

## 编译Kernel
首先进入kernel根目录，在编译之前还是需要确认清楚交叉编译工具链等环境是否配置成功，如果不确定 可以参考 [配置交叉编译工具链](https://dongshanpi.com/DongshanPi-One/05-GetSourceCode/#_4) 章节。

> make infinity2m_spinand_ssc011a_s01a_minigui_defconfig;

执行make clean;make -j8（根据服务器配置，可以支持多线程编译）生成image，如需要打包需要Relase到project对应目录。

Nand Flash	 
> arch/arm/boot/uImage.xz	

如果需要将编译出来的镜像最终打包放到一个完整的镜像内需要拷贝到project目录：
> project\release\nvr\i2m\011A\glibc\8.2.1\bin\kernel\spinand	 


### 检查编译环境

### 指定配置文件

### 编译系统

## 烧写Kernel

### uboot内烧写
``` shell

fatload  mmc 0:1  0x21000000 uImage.xz
nand erase.part KERNEL
nand write.e 0x21000000 KERNEL ${filesize}
nand erase.part RECOVERY
nand write.e 0x21000000 RECOVERY ${filesize}

```