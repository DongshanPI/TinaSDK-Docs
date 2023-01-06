# 3 打包工具介绍

本文只介绍Tina 打包时特有的工具，其他通用工具如unix2dos 等请自行百度。在tina SDK 中特有的打包工具保存在如下路径：

```
tina/tools/pack-bintools/src
```

## 3.1 update_mbr

| 工具名称 | update_mbr                                                   |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 根据分区配置文件，更新主引导目录文件sunxi_mbr.fex ，sunxi_gpt.fex<br/>及分区的下载文件列表dlinfo.fex 。 |
| 使用方法 | update_mbr <partition_file> (mbr_count)<br/>如果不指定mbr_count，mbr_count = 4;<br/>update_mbr <partition_file> <mbr_countnt> <output_name><br/>使用此用法必须指定mbr_count，本来输出的sunxi_mbr.fex 会改名为output_name。 |
| 参数说明 | partition_file：分区配置文件，如sys_partition.bin<br/>mbr_count：mbr 的备份数量，如果不指定，缺省mbr_count = 4;<br/>output_name：修改sunxi_mbr.fex 的输出名，没有特殊需求不建议使用此用法。 |
| 应用举例 | update_mbr sys_partition.bin 4<br/>update_mbr sys_partition.bin 1 sunxi_mbr_tmp.fex<br/>（没有特殊需求不建议使用此用法） |

## 3.2 merge_full_img

| 工具名称 | merge_full_img                                               |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 指定起始逻辑地址，把boot0，boot1，mbr 及分区文件下载列表里的文件合<br/>并成img 固件包，应用于小容量的nor flash。此时没有分区的概念。 |
| 使用方法 | merge_full_img –out <outfile> <br/>–boot0 <boot0.fex> <br/>–boot1 <boot1.fex> <br/>–mbr <mbr.fex> <br/>–partition <partition.fex> \–logic_start <512\|256>–help |

| 参数说明 | –out <outfile>：指定输出目标文件<br/>–boot0 <boot0.fex>：指定输入的boot0 文件–boot1 <boot1.fex>：指定输入的boot1 文件<br/>–mbr <mbr.fex>：指定输入的mbr 文件–partition <partition.fex>：指定输入的分区配置文件
–logic_start <512\|256>：指定起始逻辑地址
–help：显示使用方法 |
| 应用举例 | merge_full_img –out full_img.fex \<br/>–boot0 boot0_spinor.fex \<br/>–boot1 ${BOOT1_FILE} \<br/>–mbr sunxi_mbr.fex \<br/>–logic_start ${LOGIC_START} \<br/>–partition_file |

## 3.3 script

(1) 注意: 此处讲述的不是Linux 通用的script 工具（Linux 下script 工具用于终端会话录制）
(2) 它是全志实现的一个同名工具，工具功能说明如下：

| 工具名称 | script                                                       |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 解析输入文本文件的所有数据项，生成新的二进制bin 文件，以便程序解析。<br/>生成的目标文件与源文件名字(除后缀) 一样，但后缀为.bin。 |
| 使用方法 | script <source_file>                                         |
| 参数说明 | source_file：输入的文本文件，可多个                          |
| 应用举例 | scriptsys_config.fex<br/>scriptsys_partition.fex             |

## 3.4 dragonsecboot

| 工具名称 | dragonsecboot                                                |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 1) 根据指定的keys 生成toc0 文件。<br/>2) 根据指定的keys 和cnfbase 生成toc1 文件。<br/>3) 根据配置文件生成keys。<br/>4) 按配置文件的配置进行打包生成目的文件。 |
| 使用方法 | dragonsecboot -toc0 <cfg_file> <keypath> <version_file><br/>dragonsecboot -toc1 <cfg_file> <keypath> <cnfbase> <version_file><br/>dragonsecboot -key <cfg_file> <keypath><br/>dragonsecboot -pack <cfg_file> |
| 参数说明 | -toc0：表示要生成toc0 文件<br/>-toc1：表示要生成toc1 文件<br/>-key：表示要生成key<br/>-pack：表示进行打包<br/>cfg_file：配置文件<br/>keypath：key 的路径<br/>cnfbase：输入的cnf_base.cnf 文件<br/>version_file：固件防回滚配置文件 |
| 应用举例 | dragonsecboot -pack boot_package.cfg<br/>dragonsecboot -key dragon_toc.cfg keys<br/>dragonsecboot -toc0 dragon_toc.cfg keys version_base.mk<br/>dragonsecboot -toc1 dragon_toc.cfg keys cnf_base.cnf version_base.mk |

## 3.5 update_boot0

| 工具名称 | update_boot0                                                 |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 根据配置脚本内容，修正boot0 头部的参数。修正参数：debug_mode、<br/>dram_para 参数、uart 参数、bootcpu、jtag 参数、NAND 参数等。 |
| 使用方法 | update_boot0 <boot0> <sys_config_file> <storage_type>        |
| 参数说明 | boot0：boot0 文件<br/>sys_config_file：系统配置文件storage_type：存储介质类型 |
| 应用举例 | update_boot0 boot0_nand.fex sys_config.bin NAND<br/>update_boot0 boot0_sdcard.fex sys_config.bin SDMMC_CARD<br/>update_boot0 boot0_spinor.fex sys_config.bin SDMMC_CARD |

## 3.6 update_dtb

| 工具名称 | update_dtb                                                   |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 把Linux 设备树二进制dtb 文件进行512 字节对齐后再预留空间。   |
| 使用方法 | update_dtb <dtb_file> <reserve_size>                         |
| 参数说明 | dtb_file：输入的Linux 设备树二进制dtb 文件reserve_size：输出目标文件预留多少字节 |
| 应用举例 | update_dtb sunxi.fex 4096                                    |

## 3.7 update_fes1

| 工具名称 | update_fes1                                                  |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 从系统配置文件中取出数据对fes1 头部相关参数进行修正。<br/>修正参数包括：DRAM 参数、UART 参数、JTAG 参数等。 |
| 使用方法 | update_fes1 <fes1_file> <config_file>                        |
| 参数说明 | fes1_file：更修正的FES1 文件<br/>config_file：输入的系统配置文件 |
| 应用举例 | update_fes1 fes1.fex sys_config.bin                          |

## 3.8 signature

| 工具名称 | signature                                                    |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 对MBR 指定要进行签名的分区文件进行签名。                     |
| 使用方法 | signature <sunxi_mbr_file> <dlinfo_file>                     |
| 参数说明 | sunxi_mbr_file：输入的MBR 文件<br/>dlinfo_file：输入的分区下载列表文件 |
| 应用举例 | signature sunxi_mbr.fex dlinfo.fex                           |

## 3.9 update_toc0

| 工具名称 | update_toc0                                                  |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 从系统配置文件中取出相关参数对toc0 配置参数进行修正，修正参数包括：<br/>DRAM、UART、JTAG、NAND、卡0 、卡2 、secure 参数等。 |
| 使用方法 | update_toc0 <toc0_file> <config_file>                        |
| 参数说明 | toc0_file：输入的toc0 文件<br/>config_file：输入的系统配置文件 |
| 应用举例 | update_toc0 toc0.fex sys_config.bin                          |

## 3.10 update_uboot

| 工具名称 | update_uboot                                                 |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 从系统配置文件中取出相关参数对uboot 头部参数进行修正。<br/>修正参数包括：UART 参数、TWI 参数、target 参数、SDCARD 参数等。 |
| 使用方法 | update_uboot <uboot_file> <config_file><br/>update_uboot -merge <uboot_file> <config_file><br/>update_uboot -no_merge <uboot_file> <config_file> |
| 参数说明 | uboot_file：要更新的uboot 文件<br/>config_file：系统配置文件<br/>-merge：系统配置文件会拼接在uboot 文件尾部<br/>-no_merge：系统配置文件不会拼接在uboot 文件尾部注意：没有显式指明-no_merge 参数默认会把系统配置文件拼接在uboot 文件尾部 |
| 应用举例 | update_uboot u-boot.fex sys_config.bin<br/>update_uboot -merge u-boot.fex sys_config.bin<br/>update_uboot -no_merge u-boot.fex sys_config.bin |



## 3.11 update_scp

| 工具名称 | update_scp                                                   |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 从系统配置文件中取出相关参数对scp （小cpu 运行代码只有带有小cpu 方案的芯片<br/>会用到）头部参数进行修正。修正参数包括：UART 参数、dram_para 参数等。 |
| 使用方法 | update_scp <scp_file> <config_file>                          |
| 参数说明 | uboot_file：要更新的scp 文件<br/>config_file：系统配置文件   |
| 应用举例 | update_scp scp.fex sunxi.fex                                 |

## 3.12 u_boot_env_gen

| 工具名称 | u_boot_env_gen                                               |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 解析env 文件生成uboot 能识别的env 二进制数据文件，功能与标准的mkenvimage<br/>工具类似。 |
| 使用方法 | u_boot_env_gen <env_file> <env_bin_file>                     |
| 参数说明 | env_file：输入的evn 文件<br/>env_bin_file：输出的env 二进制文件 |
| 应用举例 | u_boot_env_gen env.cfg env.fex                               |

## 3.13 fsbuild

| 工具名称 | fsbuild                                                      |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 根据boot-resource.ini 生成fat 格式文件。                     |
| 使用方法 | fsbuild <rootfs_config_file> <magic_file>                    |
| 参数说明 | rootfs_config_file：fat 系统配置文件<br/>magic：用于fat 文件系统校验 |
| 应用举例 | fsbuild boot-resource.ini split_xxxx.fex                     |

## 3.14 update_toc1（旧版本工具，后面会弃用，可以不理会

| 工具名称 | update_toc1                                                  |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 从系统配置文件中取出相关参数对toc1 配置参数进行修正，修正参数包括：<br/>board_id_simple_gpio 参数（需配置启用，目前已没有方案使用）。 |
| 使用方法 | update_toc1 <toc1_file> <config_file>                        |
| 参数说明 | toc1_file：输入的toc1 文件<br/>config_file：输入的系统配置文件 |
| 应用举例 | update_toc1 toc1.fex sys_config.bin                          |

## 3.15 programmer_img

| 工具名称 | programmer_img                                               |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 生成mmc 介质的烧录固件。                                     |
| 使用方法 | programmer_img <boot0_file> <uboot_file> <out_img><br/>programmer_img <partition file> <mbr_file> <out_img> <in_img> |
| 参数说明 | in_img：输入的文件或镜像<br/>boot0_file：boot0 文件<br/>uboot_file：uboot 文件<br/>partition_file：分区配置文件<br/>mbr_file：sunxi_mbr 文件output_img：输出的文件或镜像 |
| 应用举例 | programmer_img boot0_sdcard.fex boot_package.fex ${out_img}<br/>programmer_img sys_partition.bin sunxi_mbr.fex ${out_img} ${in_img} |

## 3.16 dragon

| 工具名称 | dragon                                                       |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 根据img 配置文件和分区配置文件生成固件。                     |
| 使用方法 | dragon <img_config> <partition_file>                         |
| 参数说明 | img_config：配置文件，描述img 文件格式和包含其他文件列表<br/>partition_file：分区配置文件 |
| 应用举例 | dragon image.cfg sys_partition.fex                           |

## 3.17 sigbootimg

| 工具名称 | sigbootimg                                                   |
| -------- | ------------------------------------------------------------ |
| 功能说明 | 在输入文件的后面添加一个证书，并输出一个带证书的文件。       |
| 使用方法 | sigbootimg –image <input_img> –cert <cert_file> –output <output_img> |
| 参数说明 | input_img：输入的文件或镜像<br/>cert_file：文件或镜像对应的证书<br/>output_img：带证书的文件或镜像 |
| 应用举例 | sigbootimg –image boot.fex –cert boot.fex –output boot_sig.fex |

