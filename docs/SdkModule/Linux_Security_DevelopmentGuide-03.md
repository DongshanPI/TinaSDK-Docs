## 3 Secure Boot

Secure Boot，即安全启动，是一个安全系统必不可少的组成部分，是本文后续安全功能的基础。通常来说，Secure Boot 从brom 执行开始，到Linux 启动结束。Secure Boot 主要设计目的：
• 建立完整的安全信任链，确保启动阶段加载的各种镜像是可信的。
• 相关key 的烧写。
• 安全固件版本管理。
• 设置安全的硬件环境，加载并运行Secure OS 等。

### 3.1 安全启动原理

Tina 安全方案基于私钥签名-公钥验签的业界公认非对称算法实现完整的安全启动方案，具体来说，选择的是RSA2048-SHA256。
先使用私钥给固件进行签名生成安全固件，再将根密钥公钥的SHA256 值即rotpk.bin 烧写至芯
片中efuse 特定区域。启动时，固化在芯片的brom 程序首先会读取efuse 中的rotpk 值，将该值与保存在flash 上的根证书中公钥进行SHA256 运算后的值进行比对，验证根证书中公钥的可信任性。然后会使用flash 上存储的证书链中的一系列公钥来对各个子镜像进行逐级安全校验。

验证顺序为brom->sboot->monitor(仅aarch64)->secure_os->uboot->kernel。efuse 的
不可更改性确保了证书链的可信任，整个流程的设计确保了整个Linux 方案的安全启动。

### 3.2 生成安全固件

Tina SDK 已经将安全固件制作流程中密钥的生成和必要的签名过程集成在打包脚本内部，所以安全固件的编译及打包流程与非安全固件的几乎一致，只是在最后的打包的时候有差异。非安全固件的打包可参考用户《TinaLinux SDK 开发指南》文档，安全固件的打包步骤如下：

```
$ source build/envsetup.sh
==> 设置环境变量。
$ lunch
==> 选择方案。
$ make [-jN]
==> 编译，-jN 参数选择并行编译进程数量。
$ ./scripts/createkeys
==> 生成一组用于签名的密钥，不需要每次执行，详见3.2.2小节。生成的密钥路径位于out/{BOARD}/keys/。
$ pack -s [-d]
==> 打包固件。-s 表示制作安全固件；-d 表示生成的固件包串口信息转到tf卡座输出（可选）。
```

后续几个小节将对安全固件生成过程中一些注意事项进行说明。

#### 3.2.1 安全固件配置

在执行make 进行编译前，请确保包含如下配置。

##### 3.2.1.1 内核镜像格式配置

执行make menuconfig，确保如下选项配置正确。

```
Tina Configuration
└─> Target Images
	└─> [ ] Build filesystem for Boot (SD Card) partition
	└─> Boot (SD Card) Kernel format (boot.img)
	└─> [ ] Build filesystem for Boot-Recovery initramfs partition
	└─> Boot-Recovery initramfs Kernel format (boot.img)
```

3.2.1.2 安全世界内存配置

安全世界使用的内存包含三个部分，分别为:
• Shmem，共享内存，安全与非安全世界通信使用。
• Secure OS 内存，安全世界OS 使用的内存。
• TA 内存，安全应用程序使用的内存。
在Linux 中需要将这一片内存设为保留内存。
(1) 非V853 方案保留内存设置
对于非V853 方案，安全世界使用的这三个部分内存在secure_os 镜像编译时就确定了，但是secure_os 没有开源，因此保留内存具体的地址与大小，请咨询AW 安全接口。

对于非V853 方案，以R328 为例，假设安全世界使用了4M 内存（Shmem 1M，secure os1M，TA 2M），需按照如下补丁修改tina/lichee/linux-4.9/arch/arm/boot/dts/sun8iw18p1.dtsi文件。

```
diff --git a/arch/arm/boot/dts/sun8iw18p1.dtsi b/arch/arm/boot/dts/sun8iw18p1.dtsi
index 589466f..90c4131 100644
--- a/arch/arm/boot/dts/sun8iw18p1.dtsi+++ b/arch/arm/boot/dts/sun8iw18p1.dtsi
@@ -5,7 +5,7 @@
*/
/* optee used */
-/memreserve/ 0x41a00000 0x00100000; /* optee range : [0x41a00000~0x41b00000], size = 1M
*/
+/memreserve/ 0x41900000 0x00400000; /* optee range : [0x41900000~0x41D00000], size = 4M
*/
#include <dt-bindings/interrupt-controller/arm-gic.h>
#include <dt-bindings/gpio/gpio.h>
```

(2) V853 方案保留内存设置
对于V853 方案，只有Secure OS 内存在secure_os 镜像编译时确定，Shmem 与TA 内存用户可以进行动态配置起始位置与大小。
对于V853 方案， 其中Seucre OS 内存在linux 内核dts 文件(tina/lichee/linux-4.9/arch/arm/boot/dts/sun8iw21.dtsi) 中的optee_reserve 节点中设置，如下所示，默认配置起始地址为0x41980000，大小为0x19000。

```
optee_reserve {
reg = <0 0x41980000 0 0x00019000>;
status = "okay";
};
```

对于V853 方案，Shmem 与TA 内存需要在tina/device/config/chips/v853/configs//ubootboard.dts 中的optee 节点中进行配置，当前默认的配置如下。

```
optee {
shm_base = <0x418E0000>;
shm_size = <0x00020000>;
ta_ram_base = <0x41a00000>;
ta_ram_size = <0x00100000>;
};
```

对于V853 方案：

1. 如果打开安全启动功能，需要将Secure OS 内存预留的大小修改为0x80000。
2. 只有安全方案才会使用Shmem 与TA，启动过程中uboot 会解析uboot-board.dts，获取Shmem 与TA 的内存
   信息，并传递给内核，因此不需要额外在内核dts 中配置Shmem 与TA 的预留内存。

#### 3.2.2 签名密钥

注意：客户在首次进行安全固件打包之前，必须运行一次./scripts/createkeys 创建自己的签名密钥，并将创建的秘钥妥善保存。每次执行createkeys 后都会生成新的密钥，因此不用每次都执行，除非需要更换密钥。

createkeys 脚本会根据dragon_toc*.cfg生成一组用于签名的密钥，生成的密钥保存在out/{BOARD}/keys/目录下。执行pack -s 时，会使用这些密钥分别对相应的镜像进行签名并生成证书。

![image-20230103101032932](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103101032932.png)

以R328 为例，其dragon_toc*.cfg文件内容如上图所示。createkeys 依据[key_rsa] 下的keyvalue生成密钥对。打包过程中会将sboot.bin 封装成toc0.fex，将optee/uboot/dts 等封装成toc1.fex。
请将生成的密钥保存到自己的私密目录，其中Trustkey.bin，Trustkey.pem 与rotpk.bin 三个文件（有的方案为RootKey_Level_0.bin, RootKey_Level_0.pem 与rotpk.bin）为根密钥相关文件，要重点保护。
Trustkey.bin 与Trustkey.pem（RootKey_Level_0.bin 与RootKey_Level_0.pem）是根密钥私钥，不能泄漏和丢失。丢失与泄露会导致一系列问题，比如：生成的安全固件无法在芯片上启动、失去防刷机功能等。

#### 3.2.3 安全固件版本管理

注：pack -s 打包完成后，生成的安全镜像位于out/{BOARD}/keys/目录下，文件名为tina_{BOARD}_<uart0/card0>_secure_v[NUM].img。其中NUM 为固件版本号，由version_base.mk文件决定。
Tina 提供了一种安全固件防回退机制，具体实现是：在设备启动过程中会比较当前flash 上固件版本与efuse 中版本信息，如果efuse 中版本信息更高，启动失败；如果flash 上固件的版本更高，将此版本信息写入efuse 中，继续启动；如果版本信息一致，正常启动。
说明注意：最多支持更新32 个版本。另外此功能需要烧写efuse，所以务必保证efuse 供电充足。对于部分方案(R328)，由于考虑硬
件成本，出厂的设备efuse 供电不足，此功能不可用。

### 3.3 开启安全启动

完全开启安全启动共需三个前提：

1. 烧写efuse 中的secure enable bit。
2. 烧写rotpk.bin 到efuse 中rotpk 区域。
3. 烧写安全固件到flash 中。

警告
• 不同的IC，efuse 大小不同。efuse 的硬件特性决定了efuse 中每个bit 仅能烧写一次。此外，efuse 中会划分出很多区域，大部分区域也只能烧写一次。详细请参考芯片SID 规范。
• 烧写secure enable bit 后，会让设备变成安全设备，此操作是不可逆的。后续将只能启动安全固件，启动不了非安全固件。
• 默认情况下，通过LiveSuit/PhoenixSuit 烧写安全固件完成时会自动烧写secure enable bit。
• 如果既烧写了secure enable bit，又烧写了rotpk.bin，设备就只能启动与rotpk.bin 对应密钥签名的安全固件；如果只烧写secure enable bit，没有烧写rotpk.bin，此设备上烧写的任何安全固件都可以启动。调试时可只烧写secure enable bit，但是设备出厂前必须要烧写rotpk.bin。
• 为节省成本，某些硬件(R328) 方案上efuse 供电不足，导致不能写入。因此，在所有需要写efuse操作的时候，请注意给efuse 供电。通常在烧写安全固件、升级sboot/uboot、DragonSN 烧key 到efuse 等场景会写efuse。

如何判断secure enable bit 是否烧写？
• 因为只有secure enable bit 烧写后才能启动安全固件，所以如果是安全启动，secure enablebit 就一定烧写了。安全启动过程中有一些特有的打印，如“SBOOT is starting!”、“sbootcommit...”、“OLD version:...”、“NEW version: ...”、“secure enable bit: 1”等等，可用来进
行判断。
• 执行cat /sys/class/sunxi_info/sys_info，如果输出的结果中sunxi_secure 为secure，则表明secure enable bit 已经烧写。



如何判断rotpk.bin 是否烧写？
• 执行cat /proc/cmdline，查看输出结果中的rotpk_status 值，如果为1 表明已经烧写。注：当前仅R328/MR813/MR813B/R818/R818B 有开发支持。需要uboot 配置中打开SID_ROTPK_CTRL，同时env 文件中setargs_nor，setargs_nand，setargs_mmc的定义中包含rotpk_status=${rotpk_status}。
• 反证法。烧录使用其他key 签名的安全固件（安全版本号一致），如果不能启动，则表明已经烧写rotpk。
• 执行cat /sys/class/sunxi_info/sys_info，如果输出的结果中sunxi_rotpk 为1，则表明rotpk.bin 已经烧写。仅R329/R818/R818B/MR813/MR813B/R528/V853 支持。



### 3.4 烧写rotpk.bin 与secure enable bit

#### 3.4.1 方法一

方法一为通用方法，所有IC 都支持，主要包含两个步骤：

1. 使用LiveSuit/PhoenixSuit 烧写安全固件，安全固件烧写完毕时自动烧写efuse 中的secure
   enable bit 位。

2. 使用DragonSN 工具将rotpk.bin 烧写到设备的efuse 中。

  

  DragonSN 是AW 开发的PC 端烧key（SN 号、MAC 地址、rotpk 等）工具，可以将key 烧
  录到private 分区、efuse 或keybox 中，当前仅支持在windows 上运行。DragonSN 与设备
  之间通过USB 通信，控制设备烧录配置好的key 信息。

方法一的优缺点：
• 优点：所有IC 都支持；方便调试；
• 缺点：需要使用Windows 端工具；量产时通常需要两个工位。

##### 3.4.1.1 DragonSN 烧写efuse 流程

DragonSN 烧写efuse 流程如下图所示。
uboot 获取到DragonSN 下发的key 数据，将其传送到ATF（aarch64）或者Secure OS（arm32），ATF 或者Secure OS 调用efuse 驱动将key 数据写入到efuse 中。

![image-20230103101453785](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103101453785.png)

##### 3.4.1.2 DragonSN 烧写rotpk.bin 步骤

DragonSN 烧rotpk.bin 具体步骤如下：
• 设置burn_key 属性为1。只有burn_key 的值为1，设备才会接收DragonSN 通过usb传过来的信息，进行烧录动作。该属性位于uboot-board.dts或者sys_config.fex文件中[target] 项下。如果未显式配置，按照burn_key=0 来处理。
• 打包安全固件，烧写到flash 中。

![image-20230103101524737](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103101524737.png)

• 在PC 端对DragonSN 工具进行配置。打开DragonSNConfig.exe，如上图所示，点击“添加”，在“类型” 一栏下拉菜单中选择rotpk，点击“保存”、“确定”。点击“全局配置”, 设置“烧写模式” 为“安全key”。配置完成后，关闭配置工具。
• 运行DragonSN.exe 工具，配置rotpk.bin 所在的路径。然后将设备通过usb 与PC 连接，重启设备。当DragonSN 提示框显示设备已连接后，开始烧录。为了保证不会烧录错误的rotpk.bin，在烧录过程中，会将PC 端下发的rotpk.bin 与当前flash 上安全固件中根证书公钥的SHA256 值进行对比，匹配后才烧录该rotpk.bin。

#### 3.4.2 方法二

方法二在烧写安全固件完毕时，解析安全固件获取rotpk.bin 并写入efuse，然后再将efuse 中的secure enable bit 置1。当前仅R328/MR813/MR813B/R329/R818/R818B/R528/V853有开发支持。

说明：

• 要支持此功能，需要在uboot 中configs/{CHIP}_defconfig或者configs/{CHIP}_tina_defconfig文件中打开如下宏：CONFIG_SUNXI_BURN_ROTPK_ON_SPRITE=y
• 此功能仅在首次烧写安全固件时生效。

方法二的优缺点：
• 优点：量产时比较方便。
• 缺点：烧写固件时默认就烧写了rotpk.bin。

#### 3.4.3 方法三

方法三是在Linux 用户空间烧写rotpk.bin 与secure enable bit。由于rotpk.bin 与secureenable bit 只能在安全环境下读写，而Linux 环境属于非安全环境，因此在用户空间的程序会发送相关命令至安全环境下的TA，TA 收到命令后，在安全环境下对efuse 中的rotpk.bin 和
secure enable bit 进行读写。当前仅R328 有开发支持。

方法三的优缺点：
• 优点：量产比较快，比较适合固件离线烧录（即固件事先已经保存在flash 上，需要时直接组
装到设备）。
• 缺点：需要支持secure os、TA 等，增大内存消耗。

##### 3.4.3.1 API 说明

相关源码位于tina/package/security/optee-rotpk 目录下。

![image-20230103101720738](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103101720738.png)

其中librotpk.c 会被编译成库文件，该库提供如下三个API。

* ```
  /**
  
  * write_rotpk_hash() - write rotpk hash to efuse.
  * @buf: input c-style string, should be 32byte hash, with a nul terminated.
    *
  * return value: zero, write success; non-zero, write failed.
    */
    int write_rotpk_hash(const char *buf);
    /**
  * read_rotpk_hash() - read rotpk hash from efuse.
  * @buf: buf used to contain the rotpk hash value.
    *
  * return value: size of hash length.
    */
    int read_rotpk_hash(char *buf);
  ```

  其中optee-rotpk-demo.c 是一个调用上述API 的demo 程序， 被编译成可执行文件rotpk_na，使用说明如下：

  ```
  usage: rotpk_na [options] [hex-string]
  [options]:
  r read rotpk from efuse.
  w write rotpk to efuse.
  hex-string: input hex-string to burn to efuse.
  ```

  常见的四种使用方法：

  ```
  "rotpk_na w"，烧写90fa80f15449512a8a042397066f5f780b6c8f892198e8d1baa42eb6ced176f3 到efuse
  的rotpk 区域。
  "rotpk_na w 1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef", 烧写自定义
  的字符串1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef 到efuse的
  rotpk 区域。
  "rotpk_na r"，读取efuse 中的rotpk 内容。
  ```

  注：客户可依据自身需要修改应用程序，库，TA。



##### 3.4.3.2 开启方法

首先，需要开启secure os 与TA/CA 开发环境支持，具体参考第4、5 章。
其次，执行make menuconfig，开启如下选项。

```
Tina Configuration
Global build settings --->
[*] OP-TEE Support
choose OP-TEE version (optee version x.x.0) --->
Security --->
OPTEE --->
-*- optee-client-x.x
-*- optee-os-dev-kit
<*> optee-rotpk
```

然后重新打包安全固件并烧写。

##### 3.4.3.3 使用例子

```
root@TinaLinux:/# tee-supplicant &
root@TinaLinux:/# rotpk_na w
buf_in: 90fa80f15449512a8a042397066f5f780b6c8f892198e8d1baa42eb6ced176f3, size: 64
NA: write efuse hash
NA: init context
NA: open session
TA: create entry!
TA: open session!
NA: allocate memory
NA: invoke command
TA: rec cmd 0x221
TA: keyname:rotpk,key len:32,keydata:
0x90 0xfa 0x80 0xf1 0x54 0x49 0x51 0x2a
0x8a 0x04 0x23 0x97 0x06 0x6f 0x5f 0x78
0x0b 0x6c 0x8f 0x89 0x21 0x98 0xe8 0xd1
0xba 0xa4 0x2e 0xb6 0xce 0xd1 0x76 0xf3
NA: finish with 0
```

### 3.5 校验rootfs

3.1 节中提到，Secure Boot 从brom 执行开始，到Linux 启动结束。但是rootfs 没有进行校验，为了校验rootfs 的完整性，将Secure Boot 延展至rootfs，Tina 引入两种方法：uboot校验rootfs 与dm-verity。

警告：

• rootfs 必须为只读才能进行校验。
• rootfs 类型必须是squashfs。

#### 3.5.1 uboot 校验rootfs

由于rootfs 通常来说较大，从flash 中读取以及校验时间都比较长。Tina 上提供了一种在uboot阶段校验rootfs 的方法，可以提取部分rootfs 的数据来进行校验，有效减少校验时间。
注：仅R328/R329/MR813/MR813B/R818/R818B/R528/V853 有开发支持该功能。

##### 3.5.1.1 uboot 校验squashfs rootfs 功能实现

主要思路是：
• 使用extract_suqashfs 工具对squashfs rootfs 进行采样， 具体为每1M 取前面rootfs_per_MB 字节的数据，最后不足1M 的不采样。rootfs_per_MB 在env 中设置，必须为4096 的倍数或者full，其中full 表示对整个rootfs 进行校验；如未设置，默认
取4096 字节。
• 将所有采集的数据组合成新的文件，对该文件进行签名，生成证书。
• 使用update_squashfs 工具将证书附着在squashfs rootfs 的结尾处。
具体来说，使用extract_squashfs 将out/{BOARD}/image/rootfs.fex 进行采样，获取文件out/{BOARD}/image/rootfs-extract.fex。使用秘钥SCPFirmwareContentCertPK 对该rootfs-extract.fex 进行签名，生成证书out/{BOARD}/image/toc1/cert/rootfs.der。然后
使用工具update_squashfs 将该rootfs.der 证书附着在out/{BOARD}/image/rootfs.fex 的结尾处。启动过程，在uboot 中按照相反的步骤对rootfs 进行校验。
以上操作都是在打包脚本scripts/pack_img.sh 中实现。

##### 3.5.1.2 uboot 校验squashfs rootfs 开启

首先，执行make menuconfig，打开CONFIG_USE_UBOOT_VERIFY_SQUASHFS 选项。

```
Tina Configuration
└─> Global build settings --->
	└─> [*] Verify squashfs rootfs in uboot
```

其次，确保uboot 文件lichee/brandy-2.0/u-boot-2018*/configs/{CHIP}_defconfig中开启了CONFIG_SUNXI_PART_VERIFY=y 的配置。
使能该功能后，在启动过程中，uboot 会出现类似如下的log。

```
pubkey rootfs valid
partition rootfs verify pass
```

#### 3.5.2 dm-verity 机制

Tina dm-verity 是为了在启动过程中验证特定分区（通常是rootfs 分区）的完整性而设计的一套解决方案。dm-verity 从启动开始，在整个设备运行过程中，提供对特定分区数据的验证。
dm-verity 在开机过程中，依靠内核提供的device mapper 机制，验证特定分区hash tree 数据。验证通过后，在设备节点上添加dm-verity 设备。以后任何对该特定分区上数据的操作，都会映射到dm-verity 设备节点上，首先对待操作数据所在的block 计算一次hash，将此hash值与该block 在初始hash tree 中对应的hash 进行对比，一旦对比失败，dm-verity 就会返回失败给此次操作的调用者。

![image-20230103102201476](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103102201476.png)

Tina dm-verity 主要用在安全平台，是Secure Boot 最后一个环节，目的是校验根文件系统分区的完整性，确保根文件系统的数据没有被篡改。
dm-verity 验证根文件系统分区的流程如上图所示。

##### 3.5.2.1 Initramfs 构建

Tina 启动时，在initramfs 中验证dm-verity table 签名完整性，并挂载dm-verity 分区，因此必须要使用initramfs，同时initramfs 中需要包含有相关的工具，如openssl、veritysetup等。
Tina 3.0 版本之后提供了一个initramfs 生成办法。下面以cowbell-perf1 为例给出构建步骤，其他方案类似。

```
1. source build/envsetup.sh
2. lunch cowbell_perf1-tina
3. make ramfs_menuconfig
```

make ramfs_menuconfig 命令对target/allwinner/cowbell-perf1/defconfig_ramfs 进行配置修改，并基于该配置生成initramfs。如果该方案没有defconfig_ramfs，请复制defconfig 为defconfig_ramfs，为节省空间，对于不需要的配置请尽可能关闭。

请查看如下选项是否配置正确：

```
Tina Configuration
└─> Target Images
└─> [*] customize image name
└─> --- customize image name
└─>Boot Image(kernel) name suffix (boot_ramfs.img/boot_initramfs_ramfs.img)
└─> Rootfs Image name suffix (rootfs_ramfs.img)
└─> System init (busybox-init)
└─> Base system
└─> -*- busybox-init-base-files
└─> [*] Customize busybox init base files options
└─> (busybox-init-base-files_ramfs) PATH for busybox base files
└─> Security --->
└─> Device Mapper --->
└─> <*> cryptsetup
└─> use crypt lib (use libopenssl) --->
└─> <*> dm-verity
└─> Utilities --->
└─> <*> openssl-util
```

```
4. make_ramfs [-jN]
```

警告：
此处必须是make_ramfs，不能是make。

以上步骤执行完后，生成的initramfs 位于out/cowbell-perf1/compile_dir/target/rootfs_ramfs目录下。
通常来说，initramfs 只需要构建一次，可以将构建好的initramfs 保存在一个地方，以免以后重新生成。如需更改initramfs 的内容，才需要重新构建。

##### 3.5.2.2 dm-verity 启用

前提是参考3.5.2.1 小节的说明构建好initramfs。
下面以cowbell-perf1 为例给出构建步骤，其他方案类似。

```
1. source build/envsetup.sh
2. lunch cowbell_perf1-tina
3. make menuconfig
```

```
Tina Configuration
└─> target Images
	└─> [*] ramdisk
		└─> --- ramdisk
			└─> Compression (gzip)
			└─> (../../out/cowbell-perf1/compile_dir/target/rootfs_ramfs) Use external cpio
└─> Global build settings --->
	└─> [*] Device Mapper Verity
```

4. make kernel_menuconfig

选中如下配置。

```
Linux/arm 4.9.118 Kernel Configuration
└─> General setup --->
	└─> [*] Initial RAM filesystem and RAM disk (initramfs/initrd) support
	└─> [*] Support initial ramdisks compressed using gzip
└─> Device Drivers --->
	└─> [*] Multiple devices driver support (RAID and LVM) --->
		└─> <*> Device mapper support
            └─> <*> Verity target support
```

5. ./scripts/dm-verity-key.sh

注意：

执行该脚本，将在out/cowbell-perf1/verity/keys 下生成一组key，为dm-verity-pri.pem 与dm-veritypub.pem，并复制到package/security/dm-verity/files/目录下，同时将公钥dm-verity-pub.pem 复制到out/cowbell-perf1/compile_dir/target/rootfs_ramfs 下，重命名为verity_key，设备启动时需要使用此公钥来验证rootfs。

警告：

• dm-verity-pri.pem 是私钥，非常重要的隐私数据，用以对dm-verity table 进行签名，请妥善保存。
• 如果不执行此脚本，将使用package/security/dm-verity/files/下默认的key，因此请务必执行一次来替换此目录下的key。每运行一次脚本，就会更新一次key。

6. make -j
7. pack -s [-d]

经过以上步骤，可以生成一个支持dm-verity 的安全固件。

##### 3.5.2.3 dm-verity 测试

方法一：查看设备节点

```
root@TinaLinux:/# ls -l /dev/dm-0 /dev/mapper/rootfs
brw-r--r-- 1 root root 254, 0 Jan 1 08:21 /dev/dm-0
brw------- 1 root root 254, 0 Jan 1 08:21 /dev/mapper/rootfs
```

如果没有/dev/mapper/rootfs 文件，执行mount -t devtmpfs devtmpfs /dev后即可看到。
• 方法二：查看挂载

```
root@TinaLinux:/# mount
/dev/mapper/rootfs on /rom type squashfs (ro,relatime)
```

##### 3.5.2.4 dm-verity 影响

占用flash 空间需要新增dm-verity 相关信息。同时会用到initramfs，导致内核镜像增大。
• 系统性能影响
dm-verity 功能可以提高Tina 系统安全性能，但是从其实现机制来讲，会延长启动时间，降低rootfs 分区的读取速度。

### 3.6 安全启动代价

#### 3.6.1 启动时间增加

安全启动过程中会逐级对下一阶段运行的镜像进行校验，会增加启动时间。相对于非安全启动，整体会增长500ms 左右（不包括rootfs 的校验）。实际增加时间会因存储介质、硬件CE 版本、cpu/dram 频率等因素的影响而不同。

#### 3.6.2 ota 升级的变化

由3.2 小节可知，安全固件封包与非安全固件有一定的差异，因此在ota 升级时，请确保使用正确的文件。
• 升级optee/uboot/dts/sys_config，需要使用tina/out/{BOARD}/image/toc1.fex 文件；

• 升级sboot，需要使用tina/out/{BOARD}/image/toc0.fex 文件；
• 升级linux kernel，需要使用tina/out/{BOARD}/image/boot.fex 文件；
• 升级rootfs，需要使用tina/out/{BOARD}/image/rootfs.fex 文件。