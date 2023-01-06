## 5 TA/CA 开发环境

Tina 上包含了TA/CA 开发环境，便于用户在Tina 上开发TA 与CA 应用程序。
Tina 上TA/CA 开发环境主要涉及如下几个packages:

1. tina/package/security/optee-client-x.x，提供CA 所需的tee-supplicant 以及libteec
   库，其中x.x 为不同的版本。
2. tina/package/security/optee-os-dev-kit，提供TA 端编译环境。
3. tina/package/security/optee-helloworld，关于helloworld 的TA/CA demo 程序。
4. tina/package/security/optee-secure-storage，关于optee Seucre Storage 的TA/CA
   demo 程序。
5. tina/package/security/optee-base64，关于base64 算法的TA/CA demo 程序。
6. tina/package/security/optee-efuse-read， 关于读取efuse 中CHIPID，ROTPK，
   SSK，OEM 或OEM_SEC 等区域的TA/CA demo 程序。
7. tina/package/security/optee-getdmkey，关于从keybox 中读取dm-crypt 加密key 的
   程序。
   上面1 与2 是开发TA/CA 所需的环境，3-7 分别是一些TA/CA demo 程序，这些demo 程序
   需要依赖1、2 这两个包。
   !警告:
   • 要使用TA/CA 开发环境，前提是要支持Secure boot 以及Secure OS。
   • demo 程序仅用于开发测试，实际产品根据需要选中。

### 5.1 TA/CA 开发环境使用

CA 属于Linux 端应用程序，同其他应用程序一样，编译比较简单，只需要依赖optee-client 所提供的库，即可编译完成。
TA 属于安全应用程序，编译需要借助TA dev-kit。如要使用TA/CA 开发环境，执行make menuconfig，开启如下选项：

```
Tina Configuration
└─> Global build settings --->
	└─> [*] OP-TEE Support
	└─> choose OP-TEE version (optee version x.x.0) --->
└─> Security --->
	└─> OPTEE --->
		└─> <*> optee-client-x.x............................................ optee-client
		└─> <*> optee-os-dev-kit........................................ optee-os-dev-kit
```

说明
开启选项时，建议使用默认的OP-TEE 版本。当前仅MR813/MR813B/R329/R818/R818B/R528/V853 使用3.7.0。

编译时，将会把TA 所需的编译环境从tina/package/security/optee-os-dev-kit/dev_kit 复制到tina/out/{BOARD}/staging_dir/target/usr/dev_kit。

### 5.2 TA/CA 开发及编译

TA/CA 的开发需要参考GlobalPlatform 提供的标准接口说明文档。编译TA/CA 的关键点在设置编译环境变量，如CROSS_COMPILE_HOST, CROSS_COMPILE_TA以及TA_DEV_KIT_DIR 等。
TA/CA 开发环境使用可参考Tina 上的optee-helloworld 包tina/package/security/opteehelloworld/src 的实现。相关编译选项设置可参考下图：

![image-20230103103333395](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103103333395.png)

说明
OPTEE 中通过UUID 唯一标识系统中的TA，因此开发TA 时需要在ta/include/user_ta_header_defines.h 文件中设置TA_UUID。UUID 可使用uuidgen 工具生成。

### 5.3 TA 签名

Tina 上支持更换TA 签名key。在编译TA 之前，务必使用openssl genrsa -out default_ta.pem 2048命令重新生成一个key，对默认key 进行替换；默认key 的路径位于package/security/optee-os-dev-kit/dev_kit/arm-plat-{CHIP}/export-ta_arm32/keys/default_ta.pem。
编译TA 的过程中，会使用该key 对TA 进行签名；在打包过程中，会通过scripts/update_optee_pubkey.py 脚本提取该key 的公钥并将其保存到optee_os 的image 中；这样就保证了只有经过该key 签名后的TA 才可以运行在包含该key 公钥的optee_os 上。因此请注意
妥善保存该key。

scripts/update_optee_pubkey.py 脚本使用说明如下：

```
usage: update_optee_pubkey.py [-h] --in_file IN_FILE --out_file OUT_FILE --key KEY
optional arguments:
-h, --help show this help message and exit
--in_file IN_FILE Name of in file
--out_file OUT_FILE Name of out file
--key KEY Name of key file
```

### 5.4 TA 加密

默认情况下，Tina 编译的TA 只进行了签名，不进行加密，TA 二进制文件以明文形式存放在rootfs 中的/lib/optee_armtz 目录下。
当前，Tina 支持在R328/MR813/MR813B/R329/R818/R818B/R528/V853 方案上将TA 加密后再签名，其他方案暂未开发此功能。
执行make menuconfig，开启如下选项使能TA 加密（一旦开启，所有的TA 都会进行加密）。

```
Tina Configuration
└─> Security --->
	└─> OPTEE --->
		└─> -*- optee-os-dev-kit
		└─> [*] whether encrypt ta
			└─> [*] encrypt ta with which key (ssk) --->
```

目前加密密钥来源有两种（加密密钥长度为128bit）：
• 使用ssk 来作为加密秘钥。此方法需要烧写efuse 上的ssk 区域，然后将ssk 的内容复制到tina/package/security/optee-os-dev-kit/dev_kit/arm-plat-{CHIP}/export-ta_arm32/keys/ta_aes_key.
bin中，重新编译TA。有些方案efuse 中ssk 区域的长度为256bit，那么仅取其中前128bit作为ta_aes_key.bin 文件。
• 使用rotpk 派生的key 作为加密密钥。需要借助tina/scripts/generate_ta_key.py 工具来生成。其使用方法如下，将生成的OUT 文件重命名为tina/package/security/optee-os-dev-kit/dev_kit/arm-plat-{CHIP}/export-ta_arm32/keys/ta_aes_key.bin，重新编译系统。

```
usage: generate_ta_key.py [-h] --rotpk ROTPK --out OUT
```

### 5.5 安全应用demo

#### 5.5.1 optee-helloworld 效果

该demo 展示CA 如何调用TA，以及如何通过共享内容向TA 传输数据。

```
root@TinaLinux:/# tee-supplicant &
root@TinaLinux:/# hello_world_na 1234
NA:init context
NA:open session
TA:creatyentry!
TA:open session!
NA:allocate memoryTA:rec cmd 0x210
NA:invoke command: hello 1234
TA:hello 1234
NA:finish with 0
```

#### 5.5.2 optee-efuse-read 效果

该demo 中TA 通过系统调用utee_sunxi_read_efuse 与utee_sunxi_keybox 来获取efuse与keybox 中的内容。

**警告:**
**本demo 只是演示作用，实际使用时不要将获取的内容打印或传递到CA**。

```
root@TinaLinux:/# tee-supplicant &
root@TinaLinux:/# efuse_read_demo_na rotpk
NA:init context
NA:open session
TA:creatyentry!
TA:open session!
NA:allocate memory
NA:invoke command
TA:rec cmd 0x210
read efuse:rotpk
read result:
0x90 0xfa 0x80 0xf1 0x54 0x49 0x51 0x2a
0x8a 0x04 0x23 0x97 0x06 0x6f 0x5f 0x78
0x0b 0x6c 0x8f 0x89 0x21 0x98 0xe8 0xd1
0xba 0xa4 0x2e 0xb6 0xce 0xd1 0x76 0xf3
NA:finish with 0
```

