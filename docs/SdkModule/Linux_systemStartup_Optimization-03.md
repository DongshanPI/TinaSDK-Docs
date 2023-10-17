## 3 Tina启动速度优化

Tina中启动优化主要依靠宏CONFIG_BOOT_TIME_OPTIMIZATION来完成，该宏会进行如
下工作：

- 调整Linux内核镜像的压缩方式，调整rootfs的压缩方式。具体如何调整需要依据具体方案进
  行预先设定。
- 使boot0、uboot、kernel的打印不会输出到控制台。具体是在scripts/pack_img.sh脚本
  中完成。
- uboot加载内核时不进行校验。具体是在scripts/pack_img.sh脚本中完成。
- 设置内核命令行参数rootfstype，某些情况下会加快根文件系统的加载。具体是在scripts/-
  pack_img.sh脚本中完成。
- 部分方案会调整kernel镜像的加载地址。具体是在scripts/pack_img.sh脚本中完成。

> 注：通过该宏预计可达到70%左右的优化效果，如还需优化，可参考第二章的内容。

### 3.1 开启Tina启动速度优化.

在tina根目录下执行make menuconfig使能CONFIG_BOOT_TIME_OPTIMIZATION，
具体如下所示

```
Tina Configuration
    └─> Target Images
        └─>[*] kernel compression mode setting ----
            └─>Compression (Gzip) --->
                └─> ( ) Gzip
                ( ) LZMA
                ( ) XZ
                (X) LZO
        └─>[*] Boot Time Optimization
```


![图3-1: Tina menuconfig](https://photos.100ask.net/Tina-Sdk/OpenRemoved_Tina_Linux_Optimization_development_Guide-3-1.jpg)


> 注：如果看不到该选项，使用？键搜索，会发现此项有一些依赖选项，使能依赖选项即可看到

Boot Time Optimization

### 3.2 实验结果

在某norflash方案上开启CONFIG_BOOT_TIME_OPTIMIZATION后，启动速度提升效果
如下：

- Linux内核镜像压缩方式从GZIP换成LZO，优化> 0.2s。
- rootfs从squashfs XZ压缩换成squashfs GZIP压缩，优化> 0.15s。
- 屏蔽boot0、uboot、kernel启动阶段控制台打印，优化> 2s。
- 取消内核加载时的校验，优化0.3～0.4s。

> 注：对于不同的方案，由于CPU运算速度、存储器类型、内核压缩及尺寸、根文件系统类型及尺

寸、主应用等的不同，优化结果会有一定差异，请以实际优化结果为准。


## 4 参考资料

- [1] https://elinux.org/Boot_Time
- [2] https://docs.blackfin.uclinux.org/doku.php?id=fast_boot_example
- [3] https://github.com/tbird20d/grabserial
- [4] [http://www.bootchart.org](http://www.bootchart.org)
- [5] A Framework for Optimization of the Boot Time on Embedded Linux Environment
  with Raspberry Pi Platform

