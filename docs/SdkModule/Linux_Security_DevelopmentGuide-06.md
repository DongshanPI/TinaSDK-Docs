## 6 Secure Storage

数据是最核心资产，存储系统作为数据的保存空间，是数据保护的最后一道防线。当前Tina 上提供了三种Secure Storage 参考实现：
• keybox Secure Storage
• OP-TEE Secure Storage
• dm-crypt Secure Storage

### 6.1 keybox Secure Storage

由于efuse 空间受限，Tina 上支持了keybox Secure Storage 功能，该功能默认开启。keybox是Tina 上实现的一种安全存储技术，它将待烧写的key 传递到secure os，在secure os中使用efuse 中的ssk 或huk 对key 进行加密，然后将加密后的key 保存在flash 上一片特定
预留的区域。该区域未映射到逻辑扇区，通常的数据操作无法访问，正常量产也不会被擦除。

#### 6.1.1 keybox 烧写及读取流程

写keybox 有两种方式，一种是使用DragonSN，写入的数据被efuse 中的ssk 进行加密，所有安全方案均支持；另一种是使用keybox_na，仅MR813/MR813B/R818/R818B/R528 有开发支持，写入的数据会被efuse 中的huk 加密，如果efuse 没有huk 区域，则通过chipid派生出一个key 来进行加密。

**说明:**
**烧写keybox 之前，请注意提前烧写efuse 中的ssk 或huk 区域，烧写方法参考6.1.2 小节。**

##### 6.1.1.1 DragonSN 烧写keybox

使用DragonSN 烧写keybox 流程如下图所示。

![image-20230103103739138](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103103739138.png)

##### 6.1.1.2 keybox_na 烧写keybox

keybox_na 可以在用户空间烧写keybox， 其源码位于tina/package/security/opteekeybox。keybox_na 是一个NA，它发送key data 到optee os 的PTA，PTA 将其加密后，再将加密后的数据返回给NA 端，写入到keybox 中。make menuconfig 选中如下配置，编译生成keybox_na。

```
Tina Configuration
	--> Security
		--> OPTEE
			--> <*> optee-keybox
```

keybox_na 使用方法如下。

```
usage: keybox_na [-rw] [-k key_name] <-f key_file>
[options]:
	-r read key named 'dm_crypt_key'
	-w write key named [key_name] with binary [key_file]
	-k key name
	-f key file, binary
```

##### 6.1.1.3 keybox 读取流程

keybox 读取流程如下图所示。启动过程中uboot 会按照一定的条件（见6.1.1.4 小节）将flash上加密的key 读取到secure os 进行解密，并一直保存在secure os 的内存中，供TA 调用。

![image-20230103103902688](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103103902688.png)

##### 6.1.1.4 keybox 列表

对于非R328/MR813/MR813B/R329/R818/R818B/R528/V853 方案，uboot 会将所有加密的key 加载至secure os 中进行解密。
对于R328/MR813/MR813B/R329/R818/R818B/R528/V853，uboot 会根据环境变量keybox_ist 来选择加载至secure os 中的key。keybox_list 环境变量在env 文件中进行配置，使用逗号分隔各key。比如下面的例子中，名称为rsa_key，ecc_key 与testkey 的key 会被加载至secure os 中进行解密。

```
keybox_list=rsa_key, ecc_key, testkey
```

说明:
对于R328/MR813/MR813B/R329/R818/R818B/R528/V853，使用DragonSN 烧key 到keybox 之前，必须要配置好keybox_list，否则烧写的key 不会经过secure os 加密，只会以明文保存。

#### 6.1.2 DragonSN 烧写efuse 与keybox 的配置

前面已经介绍了烧写rotpk 时的配置，下图给出烧录efuse 中其他key 的配置。

![image-20230103104012598](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103104012598.png)

烧录efuse 时配置“烧写模式” 为“安全key”。
其中“显示名称” 只是显示在DragonSN 工具上的名字，不会影响设备端。其中的“Key 名称” 只能是特定的字符串。对于R328 来说包括chipid、oem、rotpk、ssk、oem_secure 五种，其他方案有一些差异，通常chipid、rotpk 等都是可行的。
烧录efuse 时，“key type” 需要选成efuse。
烧写keybox key 时，DragonSN 的关键配置如下图所示。

![image-20230103104027342](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103104027342.png)

烧录keybox 时配置“烧写模式” 为“安全key”。其中“显示名称” 只是显示在DragonSN 工具上的名字，不会影响设备端。
其中的“Key 名称” 对于不同的IC 有不同的配置。对于R328、MR813、MR813B、R329、R818、R818B、R528、V853，可以自己定义。对于其他方案，必须是widevine、ec_key、rsa_key、ec_cert1、ec_cert2、ec_cert3、rsa_cert1、rsa_cert2、rsa_cert3 这些特定的
字符串，如果希望自定义名字，则需要修改uboot、monitor/secure os 等。烧录keybox 时，“key type” 需要选成flash。
**说明**
**如果“类型” 选择为“二进制文件”，那么待烧写的key 文件名必须要以.bin为后缀。**

### 6.2 OP-TEE Secure Storage

OP-TEE Secure Storage 是根据GP TEE Internal API 规范实现的安全存储技术。它借助Secure OS 将数据进行加密，然后保存到文件系统（/data/tee）或RPMB 中。此功能可以与具体的设备绑定，充分保证了数据的私密性与完整性。
根据数据存储位置的不同，Tina 上支持两种OP-TEE Secure Storage：• REE FS Secure Storage。加密后的数据保存在linux 文件系统中（/data/tee）。
• RPMB Secure Storage。加密后的数据保存在eMMC 设备的RPMB（Replay ProtectedMemory Block）分区中。
**说明**
**• Secure Storage 依赖Secure OS，因此只有安全固件中才包含OP-TEE Secure Storage 功能。**
**• RPMB 是eMMC 中的一个具有安全特性的分区，因此只有eMMC 才支持。**
**• 当前仅R18、R328、MR813/MR813B、R329、R818/R818B、R528、V853 支持OP-TEE REE FS Secure**
**Storage 功能，具体原因见6.2.1.3 小节。仅MR813/MR813B/R818/R818B/R528 支持OP-TEE RPMB Secure**
**Storage 功能。**



#### 6.2.1 OP-TEE REE FS Secure Storage

##### 6.2.1.1 REE FS Secure Storage 功能框架

OP-TEE REE FS Secure Storage 的软件架构如下图所示。

![image-20230103104202781](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103104202781.png)

##### 6.2.1.2 REE FS Secure Storage 文件操作流程

当要写入数据时，TA 调用GP Trusted Storage API 提供的写接口，此接口会调用TEETrusted Storage Service 中的相关syscall 实现陷入到OP-TEE 的kernel space 中，该syscall 会调用一系列的TEE File Operation Interface 接口来存储写入的数据。TEE 文件系统会将写入的数据进行加密，然后通过一系列的RPC 消息向TEE supplicant 发送REE 文件操作命令以及已加密的数据。TEE Supplicant 对这些消息进行解析，按照参数的定义将加密的数据存放到对应的Linux 文件系统中（默认是/data/tee 目录）。以上是对写数据的处理，对读数
据的处理类似。

##### 6.2.1.3 REE FS Secure Storage 密钥管理Key Manager

Key Manager 是TEE file system 中的一个组件，它主要是用来处理数据加解密，并对敏感的key 进行管理。在Key Manager 中会使用三种类型的key：Secure Storage Key(SSK)、TAStorage Key(TSK)、File Encryption Key(FEK)。
（1）Secure Storage Key - SSK
SSK 是一个per-device key，当OP-TEE 启动时，会生成此key，并保存在安全内存中。SSK用来生成TSK。SSK 由如下公式计算得出：
SSK = HMACSHA256 (HUK, Chip ID || "static string")其中HUK 为Hardware Unique Key.

说明
**• 这里的HUK 是通过tee_otp_get_hw_unique_key 函数获取的。对于**
**R18/MR813/MR813B/R818/R818B/R528 来说，该函数会获取efuse 中HUK 内容的前128bit；对于**
**R328/R329 来说，由于efuse 中不存在HUK 区域，该函数会读取efuse 中chipid 的内容并进行派生；对于其他方**
**案，此函数没有实现，即该函数获取的内容全部为0。**
**• 这里的SSK 是由HUK 与Chip ID 等运算得到，与efuse 中的ssk 区域不是同一个意思，要注意区分。**
**• 对于MR813/MR813B/R818/R818B/R528，固件第一次启动时，会由CE 模块的TRNG 生成192bit 的随机数，写**
**入到efuse 的HUK 中。对于其他平台，默认efuse 中的HUK 区域（如果efuse 中存在HUK 区域）为全0，需要借**
**助DragonSN 工具来进行烧写。具体烧写说明详见6.1.2 小节。**

（2）TA Storage Key - TSK
TSK 是一个per-Trusted Application key，用来对FEK 进行加解密。TSK 公式计算如下：TSK = HMACSHA256 (SSK, TA_UUID)
（3）File Encryption Key - FEK
当创建一个TEE 文件时，Key Manager 会通过PRNG 为此文件生成一个新的FEK。并将加密之后的FEK 存放在meta file 中。而FEK 本身用来对TEE 文件进行加解密。

##### 6.2.1.4 REE FS Secure Storage Meta Data 加密流程

![image-20230103104355519](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103104355519.png)

##### 6.2.1.5 REE FS Secure Storage Block data 加密流程

![image-20230103104412370](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103104412370.png)

#### 6.2.2 OP-TEE RPMB Secure Storage

##### 6.2.2.1 RPMB Secure Storage 功能框架

RPMB Secure Storage 软件框架如下图所示。

![image-20230103104442759](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103104442759.png)

OP-TEE OS 中并不包含eMMC 驱动，因此会借助Linux 端的tee-supplicant 通过ioctl 来对RPMB 分区进行访问。

##### 6.2.2.2 RPMB Secure Storage 密钥管理与加解密

RPMB Secure Storage 文件加解密过程如下：

```
FEK = AES-Decrypt(TSK, encrypted FEK);
k = SHA256(FEK);
IV = AES-Encrypt(128 bits of k, block index padded to 16 bytes)
Encrypted block = AES-CBC-Encrypt(FEK, IV, block data);
Decrypted block = AES-CBC-Decrypt(FEK, IV, encrypted block data);
```

其中SSK、TSK 与FEK 的处理与REE FS Secure Storage 一致，最终的加解密算法有差异，RPMB 用的是AES-CBC:ESSIV，REE FS 用的是AES-GCM。

##### 6.2.2.3 RPMB Secure Storage 功能启用

首先需要生成rpmb key 并写入到eMMC 的OTP 区域，同时设备端也需要保留该key（当前Tina 上将此key 写入到keybox 中）。整个配置步骤如下：
（1）uboot rpmb 支持
uboot 中，默认没有开启rpmb 的支持，需要手动开启。在平台头文件tina/lichee/brandy-2.0/uboot-2018/include/configs/{IC}.h加入如下宏来使能。

```
#define CONFIG_SUPPORT_EMMC_RPMB
```

（2）uboot 加载rpmb_key 到optee
在env 文件中的keybox_list 项中加入rpmb_key，uboot 在启动过程中，就会自动读取rpmb_key的安全key，送到optee os 中。
（3）烧写rpmb_key
rpmb_key 的烧录，请参考本文档6.1 小节关于keybox 的说明，这里需要注意的是，名字必须是rpmb_key。
烧写过程中，如果uboot 或optee 检测到名字为rpmb_key 的安全key，首先将此key 烧录到emmc 的OTP 中，然后再保存到keybox。

**警告**
**• rpmb_key 是安全key，长度是256bit，请注意保密。**
**• RPMB Secure Storage 需要特定的optee.bin，Tina SDK 默认没有包含此optee.bin。如需支持**
**此功能，请联系AW 安全接口人。**

##### 6.2.2.4 RPMB 调试工具

Tina 上集成了mmc-utils 工具包，用于调试RPMB。执行"mmc -h"查看使用说明。
mmc 工具使用时，在没有传入rpmb_key 的情况下，可以读取RPMB 数据，但是并不能保证这个数据没有被修改过。传入rpmb_key 才能保证读取的数据没有被修改。RPMB 的写入需要传入rpmb_key。如果传入的rpmb_key 不匹配，读写都会报错。

#### 6.2.3 Tina OP-TEE Secure Storage demo

Tina OP-TEE Secure Storage demo 是基于第三方开源库optee-test 中的测试样例修改而来，客户也可以参考optee-test 自行编写代码。
相关文件保存在tina/package/security/optee-secure-storage 目录中，其内容如下：

```
.
├── Makefile
└── src
	├── Makefile
	├── na
	│ 	├── demo.c
	│ 	├── libstorage.c
	│ 	├── libstorage.h
	│	├── Makefile
	│ 	├── tee_api_defines_extensions.h
	│ 	└── tee_api_defines.h
└── ta
	├── include
	│ 	├── storage.h
	│	├── ta_storage.h
	│   └── user_ta_header_defines.h
	├── Makefile
	├── storage.c
	├── sub.mk
	├── ta_common.mk
	└── ta_entry.c
```

##### 6.2.3.1 Tina OP-TEE Secure Storage TA

我们在Secure World 端实现了一个TA demo，用来调用Secure OS 中的TEE File System对数据进行加解密等操作。TA 的源码位于optee-secure-storage/src/ta 下。当Normal World 中有应用程序发起请求时，此TA 会被加载到Secure World 并运行。
Ta 目录下还包含了ta_storage.h 头文件，此文件中包含了TA 的UUID 以及相关的command编号。

##### 6.2.3.2 Tina OP-TEE Secure Storage Library

我们将Normal World 中同Secure Storage TA 交互的接口进行了封装，具体实现在opteesecure-storage/src/na/libstorage.c 文件中，默认编译成库文件。Linux 端应用程序可以直接调用封装好的接口，便于开发。包含如下五个API。

(1) 创建文件

```
TEEC_Result OP-TEE_fs_create(TEEC_Context ctx, TEEC_Session *sess, void *file_name,
uint32_t file_size, uint32_t flags, uint32_t *obj, uint32_t storage_id);
```

函数功能：创建一个文件。
参数说明：

```
TEEC_Context ctx：NA 端打开TA 前创建初始化的一个TEE context，主要用于申请共享
内存。
• TEEC_Session *sess：NA 端创建一个TA 连接的一个session 结构体。
• void *file_name：创建文件的索引指针。
• uint32_t file_size：创建文件的大小。
• uint32_t flags：打开文件的权限，一般配置如下三种：TEE_DATA_FLAG_ACCESS_WRITE
| TEE_DATA_FLAG_ACCESS_READ | TEE_DATA_FLAG_ACCESS_META 其中分别对
应对文件的写、读、擦除权限。
• uint32_t *obj：文件描述符指针，成功创建文件时，会赋予obj 打开文件的文件描述符，供后
面读写擦除等操作使用。
• uint32_t storage_id：配置存储属性。默认有三种：
1. TEE_STORAGE_PRIVATE
2. TEE_STORAGE_PRIVATE_RE_REE
3. TEE_STORAGE_PRIVATE_RPMB
前面两种支持文件加密存储在REE 端/data/tee 目录，最后一种表示存储在eMMC 的RPMB
分区。
```

(2) 打开文件

```
TEEC_Result OP-TEE_fs_open(TEEC_Context ctx, TEEC_Session *sess, void *file_name, uint32_t
file_size, uint32_t flags, uint32_t *obj, uint32_t storage_id);
```

函数功能：打开一个文件，如果文件不存在，返回错误。
参数说明：

```
• TEEC_Context ctx：NA 端打开TA 前创建初始化的一个TEE context，主要用于申请共享
内存。
• TEEC_Session *sess：NA 端创建一个TA 连接的一个session 结构体。
• void *file_name：打开文件的索引指针。
• uint32_t file_size：打开文件名的大小。
• uint32_t flags：打开文件的权限，一般配置如下三种：TEE_DATA_FLAG_ACCESS_WRITE
| TEE_DATA_FLAG_ACCESS_READ | TEE_DATA_FLAG_ACCESS_META 其中分别对
应对文件的写、读、擦除权限。
• uint32_t *obj：文件描述符指针，成功打开或者创建文件时，会赋予obj 打开文件的文件描述
符，供后面读写擦除等操作使用。
• uint32_t storage_id：配置存储属性。默认有三种：
1. TEE_STORAGE_PRIVATE
2. TEE_STORAGE_PRIVATE_RE_REE
3. TEE_STORAGE_PRIVATE_RPMB
```

(3) 读取文件

```
TEEC_Result OP-TEE_fs_read(TEEC_Context ctx, TEEC_Session *sess, uint32_t obj, void *data,
uint32_t data_size, uint32_t *count);
```

函数功能：读取一个文件指定长度。
参数说明：

```
• TEEC_Context ctx：NA 端打开TA 前创建初始化的一个TEE context，主要用于申请共享
内存。
• TEEC_Session *sess：NA 端创建一个TA 连接的一个session 结构体。
• uint32_t obj：文件描述符。
• void *data：承载读取文件数据的buffer 地址。
• uint32_t data_size：读取文件数据长度。
• uint32_t *count：实际读取文件的长度。
```

(4) 写文件

```
TEEC_Result OP-TEE_fs_write(TEEC_Context ctx, TEEC_Session *sess, uint32_t obj, void *data,
uint32_t data_size);
```

函数功能：向文件写入指定长度数据。
参数说明：

```
• TEEC_Context ctx：NA 端打开TA 前创建初始化的一个TEE context，主要用于申请共享
内存。
• TEEC_Session *sess：NA 端创建一个TA 连接的一个session 结构体。
• uint32_t obj：文件描述符。
• void *data：写入文件数据的buffer 地址。
• uint32_t data_size：写入文件数据长度。
```

(5) 删除文件

```
TEEC_Result OP-TEE_fs_unlink(TEEC_Session *sess, uint32_t obj);
```

函数功能：关闭并删除文件
参数说明：
• TEEC_Session *sess：NA 端创建一个TA 连接的一个session 结构体。
• uint32_t obj：文件描述符。

##### 6.2.3.3 Tina OP-TEE Secure Storage Demo

此为Linux 端的demo 程序，源文件为demo.c，默认编译成ss_demo。使用方法如下：

```
usage: ss_demo [type] [options] [file name]
[type]: 'ree_fs' or 'rpmb_fs'
[options]:
-c create a file named [file name] to secure storage
-r read a file named [file name] from secure storage
-w write a file named [file name] to secure storage
content is 256 bytes random number
-d delete a file named [file name] from secure storage
[file name]: file name
```

比如，当运行"ss_demo ree_fs -w 1.file"，会随机生成256 个字节的数据，保存到Secure Storage中的1.file 文件中。

#### 6.2.4 Tina OP-TEE Secure Storage 开启

##### 6.2.4.1 OP-TEE Secure Storage 配置

(1) 开启Tina 相关配置
在Tina 环境下，执行"make menuconfig"，确保如下选项已经开启。

```
Tina Configuration
	Global build settings --->
		[*] OP-TEE Support
			choose OP-TEE version (optee version x.x.0) --->
Security --->
	OPTEE --->
		-*- optee-os-dev-kit
		-*- optee-client-x.x
		<*> optee-secure-storage
```

(2) 开启内核相关配置版在Tina 环境下，执行"make kernel_menuconfig"，确保如下选项已经开启。

```
Linux/arm 4.9.118 Kernel Configuration
Device Drivers --->
<*> Trusted Execution Environment support
TEE drivers --->
<*> OP-TEE
```

(3) 设置dts
在Tina 环境下，确保tina/lichee/linux-<kernel_version>/arch/arm*/boot/dts/sunxi/{CHIP}.dtsi文件中的firmware 下包含如下内容：

```
optee {
compatible = "linaro,optee-tz";
method = "smc";
};
```

(4) 确保huk 烧写
对于MR813/MR813B/R818/R818B/R528，安全固件第一次启动时自动使用CE 产生的随机数对efuse 中huk 进行烧写。对于其他方案，如果efuse 中有huk 区域的，需要通过DragonSN烧写efuse 中的huk 区域；如果efuse 中没有huk 区域的，optee 中会基于chipid 派
生出一个密钥。

##### 6.2.4.2 编译安全固件

在Tina 环境下，按照第3 章说明来编译安全固件。

#### 6.2.5 OP-TEE Secure Storage 使用

```
root@tulip-mozart:/# tee-supplicant &
root@TinaLinux:/# ss_demo rpmb_fs -c test.file
root@TinaLinux:/# ss_demo rpmb_fs -r test.file
---- Read file:test.file 0 Bytes data: ----
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
---- Read file:test.file end! ----
root@TinaLinux:/# ss_demo rpmb_fs -w test.file
---- Write file:test.file with 256 Bytes data: ----
0d 84 14 34 76 19 a9 c2 98 76 86 f9 2f c7 07 29
77 3b 9b 98 cb dd 57 f4 5f d5 b3 f6 d1 01 f4 5e
05 88 12 fa 22 3c be 3a b2 c4 34 61 8d ba 8b 84
76 27 9d c1 84 f4 b7 e4 4a 6b db 1c ec 51 f9 f1
d9 0c ed 7b c7 2c b5 7b f0 6a 5c 7e 25 e7 83 9b
8e 21 5e 14 16 95 f8 60 01 54 fb ed 25 75 60 7f
01 cd fa c9 f9 b1 c4 ea 9b 21 e9 40 89 6d dc 18
8e ba 2c 24 cf a4 84 d0 79 00 3f 9e 75 9f 1e f5
6d 1a bf e6 4b 04 d1 66 26 bb a6 af a8 03 47 37
bd 74 5b 0d 98 5f de 12 de 9d b1 d3 bc 4f ca a9
e8 0a 90 b3 0f e1 1a 35 9e c1 64 c6 c4 ab fe 03
9f d9 10 39 b9 6e ca 18 8b fb ec 48 4c b7 f1 b4
41 82 69 50 65 03 05 83 44 e8 4a 89 95 c8 8c b4
23 1c ed 5c 0b b9 74 96 b5 61 df 81 98 51 37 da
d4 20 aa b9 23 b0 bc e7 99 86 71 ae cf 7d 64 72
99 d1 ce 24 0b c2 bb 41 a4 1b c2 3d 6c 79 97 c0
---- Write file:test.file end! ----
root@TinaLinux:/# ss_demo rpmb_fs -r test.file
---- Read file:test.file 256 Bytes data: ----
0d 84 14 34 76 19 a9 c2 98 76 86 f9 2f c7 07 29
77 3b 9b 98 cb dd 57 f4 5f d5 b3 f6 d1 01 f4 5e
05 88 12 fa 22 3c be 3a b2 c4 34 61 8d ba 8b 84
76 27 9d c1 84 f4 b7 e4 4a 6b db 1c ec 51 f9 f1
d9 0c ed 7b c7 2c b5 7b f0 6a 5c 7e 25 e7 83 9b
8e 21 5e 14 16 95 f8 60 01 54 fb ed 25 75 60 7f
01 cd fa c9 f9 b1 c4 ea 9b 21 e9 40 89 6d dc 18
8e ba 2c 24 cf a4 84 d0 79 00 3f 9e 75 9f 1e f5
6d 1a bf e6 4b 04 d1 66 26 bb a6 af a8 03 47 37
bd 74 5b 0d 98 5f de 12 de 9d b1 d3 bc 4f ca a9
e8 0a 90 b3 0f e1 1a 35 9e c1 64 c6 c4 ab fe 03
9f d9 10 39 b9 6e ca 18 8b fb ec 48 4c b7 f1 b4
41 82 69 50 65 03 05 83 44 e8 4a 89 95 c8 8c b4
23 1c ed 5c 0b b9 74 96 b5 61 df 81 98 51 37 da
d4 20 aa b9 23 b0 bc e7 99 86 71 ae cf 7d 64 72
99 d1 ce 24 0b c2 bb 41 a4 1b c2 3d 6c 79 97 c0
---- Read file:test.file end! ----
root@TinaLinux:/# ss_demo rpmb_fs -d test.file
Delete file:test.file !
root@TinaLinux:/# ss_demo rpmb_fs -r test.file
Failed to optee_fs_open: test.file, ret = 0xffff0008
```

### 6.3 dm-crypt Seucre Storage

为防止未授权用户通过对设备进行物理攻击（如直接读取Flash）来获取敏感信息，造成用户数据泄露，Tina 引入dm-crypt 机制，对用户文件系统的数据提供加密保护。

![image-20230103105451159](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103105451159.png)

dm-crypt 是使用linux 内核加密API 框架和设备映射（device mapper）子系统的磁盘加密技术。Device mapper 在内核中作为一个块设备驱动被注册的，它包含三个重要的对象概念：mapped device、映射表、target device。Mapped device 是一个逻辑抽象，可以理解成为
内核向外提供的逻辑设备，它通过映射表描述的映射关系和target device 建立映射。这里的映射关系可以是verity（完整性校验），也可以是crypt（加密）。上图示例中，将/dev/mmcblk0p1 通过device mapper 映射称/dev/dm-0 设备，对/dev/dm-0进行文件系统格式化后可将/dev/dm-0 挂载至/data 目录。

#### 6.3.1 Tina dm-crypt

dm-crypt 中的加解密可使用内核原生的软件加解密实现，也可以使用AW SOC 自带的硬件加密引擎（CE Crypto Engine）来实现。
当前Tina dm-crypt 分区的初始化、挂载与卸载借助package/security/dm-crypt/dmcrypt.sh 脚本来实现。该脚本默认将映射后的分区格式化为ext4。

**说明**：
**该脚本是一个demo，客户可依据需求自行开发。**

##### 6.3.1.1 dm-crypt 配置

使用Tina dm-crypt 需要三个先决条件：
(1) 配置Linux 内核。
执行make kernel_menuconfig，开启内核dm-crypt 相关功能以及加解密API：

```
Device Drivers --->
	[*] Multiple devices driver support (RAID and LVM) --->
		<*> Device mapper support
		<*> Crypt target support
File systems --->
    <*> The Extended 4 (ext4) filesystem
	[*] 	Use ext4 for ext2 file systems
-*- Cryptographic API --->
	<*> XTS support
	<*> SHA224 and SHA256 digest algorithm
	<*> AES cipher algorithms
	<*> User-space interface for hash algorithms
	<*> User-space interface for symmetric key cipher algorithms
```

如果希望使用硬件加密引擎，开启如下配置。

```
-*- Cryptographic API --->
[*] Hardware crypto devices --->
<*> Support for Allwinner Sunxi CryptoEngine
```

如果方案使用了UBI，即Linux 内核中开启了UBI 相关选项，还需开启如下配置。

```
Device Drivers --->
<*> Memory Technology Device (MTD) support --->
<*> Caching block device access to MTD devices
-*- Enable UBI - Unsorted block images --->
<*> MTD devices emulation driver (gluebi)
```

(2) 配置rootfs。
执行make menuconfig，开启如下选项。

```
Tina Configuration
	Security --->
		Device Mapper --->
			<*> dm-crypt
```

(3) 配置分区表，新增一个需要加密的分区。
修改sys_partition*.fex 文件，新增secret 分区，用来对其进行加密，分区size 可以自定义。

```
[partition]
	name = secret
	size = 40960
	user_type = 0x8000
```

6.3.1.2 dm-crypt 使用
开启如上配置之后，rootfs 中或包含相关工具及脚本。其中dm-crypt.sh 脚本会借助cryptsetup
与openssl 相关工具来执行映射、格式化、打开、挂载dm-crypt 分区等操作，其使用说明如下（当前加密算法为aes-xts-plain64）：

```
dm-crypt.sh - this script helps you to use the secret partition.
Usage: /usr/bin/dm-crypt.sh <op_flag> <type> <keyfile>
<op_flag>:
'c' - create & format secret partition;
'm' - mount secret partition;
'u' - unmount secret partition and close mapper device
<type>: Device type, can be 'plain' or 'luks'.
<keyfile>: Key, can be 'keyfile' or 'pass' or 'optee-pass'
```

cryptsetup 支持使用keyfile、pass 或optee-pass。
• keyfile，这可以是任何文件，但建议使用具有适当保护的随机数据的文件（考虑到访问此密钥文件将意味着访问加密数据）。
• pass, 此模式下需要手动输入key。
• optee-pass，此模式下会调用getdmkey_na 程序从optee 中获取一个256bit 的key，此部分将在下一小结详细说明。cryptsetup 还支持多次加密操作模式，如luks，plain，loopaes 等，当前dm-crypt.sh 支持luks 与plain 模式。
(1) 格式化dm-crypt 分区
执行dm-crypt.sh c luks pass，创建并格式化dm-crypt 分区。

```
root@TinaLinux:/# dm-crypt.sh c luks pass
Enter passphrase:
Enter same passphrase again:
Creating Filesystems...
mke2fs 1.42.12 (29-Aug-2014)
```

(2) 挂载dm-crypt 分区
执行dm-crypt.sh m luks pass。

```
root@TinaLinux:/# dm-crypt.sh m luks pass
Enter passphrase:
Enter same passphrase again:
[ 412.744846] EXT4-fs (dm-0): mounted filesystem with ordered data mode. Opts: (null)
mount /dev/mapper/secret to /mnt/secret
```

查看secret 分区是否挂载成功，是否可以读写。

```
root@TinaLinux:/# mount | grep secret
/dev/mapper/secret on /mnt/secret type ext4 (rw,relatime,data=ordered)
root@TinaLinux:/# ls /mnt/secret/
lost+found
```

##### 6.3.1.3 dm-crypt key

dm-crypt 的key 可以是两种模式，一种是passphrase，最大长度是512B；另一种是keyfile，文件的最大长度是8192KB。
为增加key 的安全性，Tina 上支持从optee os 中获取用于dm-crypt 的key。该key 需要预先烧录到keybox 中，具体烧录方法请参考本文档6.1 小节关于keybox 的说明，这里需要注意的是，名字必须是dm_crypt_key，key 长度为256bit。
Linux 端对应的应用程序是getdmkey_na， 源码位于tina/package/security/opteegetdmkey/目录下，具体使用方法参考第5 章关于TA/CA 开发环境的说明。