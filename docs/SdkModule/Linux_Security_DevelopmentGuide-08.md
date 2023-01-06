## 8 量产工具

从整个安全系统的角度看，需要一整套工具来配合完成对应的工作。

### 8.1 RSA 密钥对生成工具

目前，有公开的密钥对生成工具openssl，可以生成足够长度的密钥对。Tina 开发平台scripts 下提供了一个生成密钥对的脚本createkeys，该脚本调用dragonsecboot工具，解析dragon_toc*.cfg 中[key_rsa] 字段，并基于字段的内容生成对应名字的密钥
对。关于dm-verity 所需要的keys 是由tina/scripts/dm-verity-key.sh 生成。

### 8.2 安全固件版本管理

安全固件打包时会解析version_base.mk文件决定。在efuse 中会有一块区域用来记录固件版本。当设备启动时，会将efuse 中记录的版本号同固件中的版本号比较，如果固件中的版本较低，则不能继续启动；如果固件中的版本比较高，将固件中的版本写入efuse，继续启动；如果版本相同，正常启动。可防止固件版本回退。

### 8.3 数据封包工具

Tina 开发平台中提供固件封包工具dragonsecboot，在安全固件打包过程中会对相关的镜像文件（sboot、uboot、kernel 等）进行签名，并生成证书以及相关信息，以便启动时对这些镜像文件进行校验，验证完整性。

### 8.4 烧key 工具

烧key 工具用来将rotpk.bin 烧写到设备的efuse 中，efuse 位于IC 内部，由于efuse 中内容一旦写入便不可更改，所以从根源上保证了根证书公钥hash 的安全性。可用的烧key 工具包含DragonKey 或者DragonSN，工具的使用说明位于工具包中。

### 8.5 关闭jtag

将sys_config.fex 中jtag_para 节下的jtag_enable 设置为0 即可关闭jtag 调试功能。

### 8.6 密钥说明

#### 8.6.1 固件签名密钥

| 密钥     | 安全固件签名私钥                                             |
| -------- | ------------------------------------------------------------ |
| 功能     | RSA2048 类型私钥。对sboot、monitor、scp、optee、uboot、boot、<br/>rootfs 等分区进行签名 |
| SDK 路径 | tina/out/${BOARD}/keys/*.pem与tina/out/${BOARD}/keys/*.bin，除开rotpk.bin |
| 设备位置 | 设备上不保存                                                 |
| 烧写方式 | 不烧写                                                       |
| 保密     | 是                                                           |

| 密钥     | 安全固件签名私钥                                             |
| -------- | ------------------------------------------------------------ |
| 功能     | RSA2048 类型私钥。对sboot、monitor、scp、optee、uboot、boot、<br/>rootfs 等分区进行签名 |
| SDK 路径 | 位于tina/out/${BOARD}/image/toc0以及tina/out/${BOARD}/image/toc1目录下的证书中 |
| 设备位置 | flash 上TOC0、TOC1、boot、rootfs 等分区中                    |
| 烧写方式 | 随固件一起烧写                                               |
| 保密     | 否                                                           |

#### 8.6.2 efuse 中密钥

| 密钥     | rotpk                                                        |
| -------- | ------------------------------------------------------------ |
| 功能     | 签名根密钥公钥的sha256 值，用于安全启动中根证书的校验。长度256bit。 |
| SDK 路径 | tina/out/${BOARD}/keys/rotpk.bin                             |
| 设备位置 | IC 中efuse 中的rotpk 区域                                    |
| 烧写方式 | DragonSN 等，参考3.4 小节                                    |
| 保密     | 否                                                           |

| 密钥     | ssk                                                          |
| -------- | ------------------------------------------------------------ |
| 功能     | 对称密钥。可用于DragonSN 烧写keybox 时对待烧写的内容进行加密，可用<br/>于TA 加密 |
| SDK 路径 | SDK 中没有                                                   |
| 设备位置 | IC 中efuse 中的rotpk 区域                                    |
| 烧写方式 | DragonSN 等，参考3.4 小节                                    |
| 保密     | 是                                                           |

| 密钥     | huk                                                          |
| -------- | ------------------------------------------------------------ |
| 功能     | 对称密钥。可用于rotpk_na 烧写keybox 时对待烧写的内容进行加密，可用于<br/>OPTEE Secure Storage 对数据进行加密。 |
| SDK 路径 | SDK 中没有                                                   |
| 设备位置 | IC 中efuse 中的huk 区域                                      |
| 烧写方式 | 对于MR813/MR813B/R818/R818B/R528，安全固件第一次启动时自动使用<br/>CE 产生的随机数进行烧写。其他方案通过DragonSN 烧写 |
| 保密     | 是                                                           |

#### 8.6.3 dm-verity 密钥

| 密钥     | dm-verity 私钥                                               |
| -------- | ------------------------------------------------------------ |
| 功能     | RSA2048 类型私钥。用于对rootfs 的hash table 进行签名         |
| SDK 路径 | 位于tina/out/${BOARD}/verity/keys/dm-verity-pri.pem与tina/package/security/dmverity/<br/>files/dm-verity-pri.pem |
| 设备位置 | 设备上不保存                                                 |
| 烧写方式 | 不烧写                                                       |
| 保密     | 是                                                           |

| 密钥     | dm-verity 私钥                                               |
| -------- | ------------------------------------------------------------ |
| 功能     | RSA2048 类型公钥。用于在initramfs 启动脚本中对rootfs 的hash table 进<br/>行验签 |
| SDK 路径 | 位于tina/out/${BOARD}/verity/keys/dm-verity-pub.pem、<br/>tina/package/security/dm-verity/files/dm-verity-pub.pem以<br/>及tina/out/${BOARD}/compile_dir/target/rootfs_ramfs/verity_key |
| 设备位置 | flash 上boot 分区中的initramfs 文件系统中                    |
| 烧写方式 | 随固件boot 分区一起烧写                                      |
| 保密     | 否                                                           |

#### 8.6.4 TA 签名密钥

| 密钥     | dm-verity 私钥                                               |
| -------- | ------------------------------------------------------------ |
| 功能     | RSA2048 类型私钥。用于对TA 进行签名。没有签名或签名错误的TA 将不会<br/>运行 |
| SDK 路径 | 位于tina/package/security/optee-os-dev-kit/dev_kit/arm-plat-{CHIP}/export-ta_arm32<br/>/keys/default_ta.pem与tina/out/{BOARD}/staging_dir/target/usr/dev_kit/arm-plat-{<br/>CHIP}/export-ta_arm32/keys/default_ta.pem |
| 设备位置 | 设备上不保存                                                 |
| 烧写方式 | 不烧写                                                       |
| 保密     | 是                                                           |

| 密钥     | TA 签名公钥                                                  |
| -------- | ------------------------------------------------------------ |
| 功能     | RSA2048 类型公钥。用于OPTEE 对TA 进行验签。验签失败的TA 将不会运<br/>行。 |
| SDK 路径 | tina/device/config/chips/${CHIP}/bin/optee_${IC}.bin与tina/out/${BOARD}/image/<br/>optee.fex二进制文件中包含TA 签名公<br/>钥 |
| 设备位置 | flash 上TOC1 分区中的optee 内                                |
| 烧写方式 | 随固件TOC1 一起烧写                                          |
| 保密     | 否                                                           |

#### 8.6.5 TA 加密密钥

| 密钥     | TA 签名公钥                                                  |
| -------- | ------------------------------------------------------------ |
| 功能     | 对称密钥。用于对TA 进行加密。密钥来源可以是ssk 或由rotpk 派生而来。长<br/>度128bit。 |
| SDK 路径 | tina/package/security/optee-os-dev-kit/dev_kit/arm-plat-{CHIP}/export-ta_arm32/<br/>keys/ta_aes_key.bin与tina/out/{BOARD}/staging_dir/target/usr/dev_kit/arm-plat-{<br/>CHIP}/export-ta_arm32/keys/ta_aes_key.bin |
| 设备位置 | IC 中efuse 中的ssk 或rotpk 区域                              |
| 烧写方式 | DragonSN                                                     |
| 保密     | 是                                                           |

#### 8.6.6 dm-crypt 密钥

| 密钥     | dm-crypt 密钥                                                |
| -------- | ------------------------------------------------------------ |
| 功能     | 对称密钥。用于对dm-crypt 分区文件系统数据进行加密。dm-crypt.sh 脚本中<br/>可使用三种类型的key：keyfile、pass 与optee-pass，建议使用<br/>optee-pass。 |
| SDK 路径 | SDK 中没有该密钥                                             |
| 设备位置 | 对于keyfile 类型，位于根文件系统/encrypt-key-file，此文件经过加密，使用<br/>时需要输入passphrase 进行解密，keyfile 最大8192kiB；对于pass 类型，<br/>不保存，使用时实时输入passphrase，passphrase 最大长度512B；对于<br/>optee-pass 类型，保存在flash 上的keybox 中，长度256bit。 |
| 烧写方式 | 对于keyfile 类型，执行dm-crypt.sh 时，写到根文件系统根目录；对于pass<br/>类型，不需要烧写；对于optee-pass 类型，通过DragonSN 或keybox_na<br/>进行烧写。 |
| 保密     | 是                                                           |

#### 8.6.7 rpmb 密钥

| 密钥     | rpmb 密钥                                                  |
| -------- | ---------------------------------------------------------- |
| 功能     | 对称密钥。用于访问RPMB 时身份认证。长度256bit。            |
| SDK 路径 | SDK 中没有该密钥                                           |
| 设备位置 | 保证在flash 上的keybox 中，同时保存在eMMC 中的OTP 区域中。 |
| 烧写方式 | 通过DragonSN 或keybox_na 进行烧写。                        |
| 保密     | 是                                                         |