## 5 开机音乐功能

这里开机音乐功能，指的是在uboot阶段开始启动dma将音频数据搬运到AudioCodec的FIFO中进行播歌，同时CPU继续开机流程进入内核。进入系统后，在合适的启动脚本中加载音频驱动模块(如果builtin，那么音乐会提前中止),这样开机音乐可以大大的提前，给用户一种迅速开机的错觉。

目前SDK代码中支持uboot开机音乐功能的平台有:R6,R7s,R11,R16,R328,R311,MR133

配置使用方法都比较类似，下面以R328为例进行说明。

### 5.1 配置方法

1. 修改uboot配置文件
   - 可修改默认配置configs/sun8iw18p1_defconfig增加下面几项
     CONFIG_SOUND_SUNXI_SOC_RWFUNC=y
     CONFIG_SOUND_SUNXI_SUN8IW18_CODEC=y
     CONFIG_SOUND_SUNXI_BOOT_TONE=y
   - 也可以通过kconfig进行配置lichee/brandy-2.0/u-boot-2018目录下执行makemenu-
     config ARCH=arm，选上对应功能项:选中sun8iw18的codec驱动以及开机音乐功能

![5-1-1](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-5-1-1.jpg)

2. 修改sys_config文件

```
配置boottone_used为1,表示使用开机音乐功能
;----------------------------------------------------------------------------------
;boot tone configuration
;
;boottone_used ---boot tone enable
;len_limit ---set size in bytes, normally do not need to set it and driver will use
file size
;----------------------------------------------------------------------------------
[boottone]
boottone_used = 1
```

```
另外codec节点也需要配置正确,下面是部分重要配置
[codec]
codec_used = 0x1
lineout_vol =0x1a
gpio-spk = port:PH9<1><1><1><1>
```

3. 创建boottone分区

```
开机音乐的音频文件需要放置在单独一个分区中，例如boottone分区，修改sys_partition.fex
文件：
[partition]
name = boottone
size = 2048
downloadfile = "boottone.fex"
user_type = 0x8000
```

```
上述 2048 表示1M的空间,注意根据实际音频文件大小填写合适的size
然后将音频文件重命名为boottone.fex,并放置到方案配置目录下，以cowbell-perf1方案
为例：
mv kaiji.wav target/allwinner/cowbell-perf1/configs/boottone.fex
```

4. 修改env配置文件

```
主要增下面几行：
uboot_tone_addr=0x4327ffd4
boottone_partition=boottone
load_boottone=sunxi_flash read ${uboot_tone_addr} ${boottone_partition}
```

```
uboot_tone_addr用于指定音频文件加载到dram的地址
boottone_partition用于指定音频文件所在分区名
load_boottone用于加载音频文件到dram的命令
```

5. 将内核AudioCodec驱动编译成模块

```
make kernel_menuconfig去除下面几个配置：
@@ -702,14 +705,9 @@ CONFIG_SND_SOC_I2C_AND_SPI=y
# CONFIG_SND_SOC_WM8960 is not set
# CONFIG_SND_SOC_WM8974 is not set
# CONFIG_SND_SOC_WM8985 is not set
-CONFIG_SND_SUN8IW18_CODEC=y
# CONFIG_SND_SUNXI_MAD is not set
-CONFIG_SND_SUNXI_SOC=y
-CONFIG_SND_SUNXI_SOC_CPUDAI=y
-CONFIG_SND_SUNXI_SOC_DAUDIO=y
-CONFIG_SND_SUNXI_SOC_RWFUNC=y
-CONFIG_SND_SUNXI_SOC_SUN8IW18_CODEC=y
-CONFIG_SND_SUNXI_SOC_SUNXI_DAUDIO=y
+# CONFIG_SND_SUNXI_SOC_SUN8IW18_CODEC is not set
+# CONFIG_SND_SUNXI_SOC_SUNXI_DAUDIO is not set
# CONFIG_SND_SUNXI_SOC_SUNXI_DMIC is not set
# CONFIG_SND_SUNXI_SOC_SUNXI_SPDIF is not set
CONFIG_SND_SUPPORT_OLD_API=y
```

make menuconfig增加驱动模块配置

```
@@ -2923,7 +2929,7 @@ CONFIG_PACKAGE_kmod-sound-core=y

# CONFIG_PACKAGE_kmod-sound-soc-ac97 is not set

# CONFIG_PACKAGE_kmod-sound-soc-core is not set

# CONFIG_PACKAGE_kmod-sound-via82xx is not set

-# CONFIG_PACKAGE_kmod-sunxi-sound is not set
+CONFIG_PACKAGE_kmod-sunxi-sound=y
```



### 5.2 注意事项

1. uboot退出时，不能关掉dma。如果发生在跳转kenrel时声音变化或消失，请确认跳转到内核前是否调用了sunxi_dma_exit();
2. 内核并不知道uboot将音频加载到dram的位置。理论上,kernel可能会在初始化过程中，用到那片内存，导致音乐播放不正常。
   - 可以在dts中，将该片内存改为reserve属性，可以确保kernel不会使用到。
   - 选用kernel初始化不会用到的内存块
3. 可以在uboot命令行下执行boottone命令进行调试