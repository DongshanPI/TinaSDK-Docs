## 3 Tina SWUpdate OTA 介绍

### 3.1 swupdate 介绍

#### 3.1.1 简介

SWUpdate 是一个开源的OTA 框架，提供了一种灵活可靠的方式来更新嵌入式系统上的软件。

官方源码：

https://github.com/sbabic/swupdate

官方文档：

http://sbabic.github.io/swupdate/

非官方翻译的中文文档：

https://zqb-all.github.io/swupdate/

源码自带文档：

解压tina/dl/swupdate-xxx.tar.xz ，解压后的doc 目录下即为此版本源码附带的文档。

社区论坛：

https://groups.google.com/forum/#!forum/swupdate



#### 3.1.2 移植到tina 的改动

移植到tina 主要做了以下修改：

• 位置在package/allwinner/swupdate。

• 仿照busybox，添加了配置项，可通过make menuconfig 直接配置。

• 添加patch，支持了更新boot0,uboot。

• 添加了自启动脚本。

• 默认启动progress 在后台，输出到串口。这样升级时会打印进度条。实际方案不需要的话，可去除。客户应用可参考progress 源码，自行获取进度信息。

• 默认启动一个脚本swupdate_cmd.sh，负责完善参数，最终调用swupdate。脚本介绍详见后续章节。

### 3.2 配置

#### 3.2.1 recovery 系统介绍

若选用主系统+recovery 系统的方式，则需要一个recovery 系统。

recovery 系统是一个带initramfs 的kernel。对应的配置文件是target/allwinner/xxx/defconfig_ota。

如果没有此文件，可以拷贝defconfig 为defconfig_ota，再做配置裁剪。

#### 3.2.2 系统配置命令

对于主系统，使用：

```
make menuconfig
```

配置结果保存在：

```
target/allwinner/xxx/defconfig
```

对于recovery 系统，使用：

```
make ota_menuconfig
```

配置结果保存在：

```
target/allwinner/xxx/defconfig_ota
```

#### 3.2.3 主系统和recovery 都需要的swupdate 包

选上swupdate 包。

```
Allwinner --> [*]swupdate
```

swupdate 中还有很多细分选项，一般用默认配置即可。需要的话可以做一些调整，比如裁剪掉网络部分。

swupdate 会依赖选中uboot-envtools 包，以提供用户空间读写env 分区的功能。

#### 3.2.4 主系统和recovery 都需要的wifimanager daemon

如果想从网络升级，则需要启动系统自动联网。

一种实现方式是，使用wifimanager daemon 。当然，如果用户自己在脚本或应用中去做联网，则不需要此选项。

```
Allwinner
---> <*> wifimanager
---> [*] Enable wifimanager daemon support
---> <*> wifimanager-daemon-demo..................... Tina wifimanager
daemon demo
```



#### 3.2.5 配置主系统

就以上提到的几个包，暂时没有只针对主系统的需要选的包。

#### 3.2.6 编译主系统

正常make 即生成主系统。

```
make
```



#### 3.2.7 配置recovery 系统

对于recovey 系统，需要选上ramdisk，同时建议使用xz 压缩方式以节省flash 空间。

```
make ota_menuconfig
    ---> Target Images
        ---> [*] ramdisk
            ---> Compression (xz)
```

选上recovery 后缀，避免编译recovery 系统时，影响到主系统。

```
make ota_menuconfig
    ---> Target Images
        ---> [*] customize image name
            ---> Boot Image(kernel) name suffix (boot_recovery.img/boot_initramfs_recovery.img)
			---> Rootfs Image name suffix (rootfs_recovery.img)
```

#### 3.2.8 编译recovery 系统

要编译生成recovery 系统，可使用：
swupdate_make_recovery_img
或手工调用：
make -j16 TARGET_CONFIG=./target/allwinner/xxx/defconfig_ota
编译得到：
out/xxx/boot_initramfs_recovery.img

#### 3.2.9 配置env

本方案推荐使用env 来保存信息，不使用misc 分区。

uboot 会从env 分区读取启动命令，并根据启动命令来启动系统。只要我们能在用户空间改动到env，即可控制下次启动的系统。

##### 3.2.9.1 boot_partition 变量

增加一个boot_partition 变量，用于指定要启动的内核所在分区。

配置env 主要是修改boot_normal 命令，将要启动的分区独立成boot_partition 变量。

即从：

```
boot_normal=fatload sunxi_flash boot 40007fc0 uImage;bootm 40007fc0
```

改成：

```
boot_partition=boot
boot_normal=fatload sunxi_flash ${boot_partition} 40007fc0 uImage;bootm 40007fc0
```

这样可以通过控制boot_partition 来直接选择下次要启动的系统，无需uboot 介入。uboot 只需按照boot_normal 启动即可。

对于recovery 方案，可设置boot_partition 为boot 或recovery。OTA 切换系统时，只需要改变此变量即可达到切换主系统和recovery 系统的目的。

对于AB 系统方案，可设置为boot_partition 为bootA 或bootB。OTA 切换系统时，只需要改变此变量即可达到切换kernel 的目的。

##### 3.2.9.2 root_partition 变量

增加一个root_partition 变量，用于指定要启动的rootfs 所在分区。

uboot 会解析分区表，找出此变量指定的分区并在cmdline 中指定root 参数。

例如，在env 中设置：

```
root_partition=rootfs
```

则启动时uboot 会遍历分区表，找到名字为rootfs 的分区，假设找到的分区为/dev/nand0p4，则在cmdline 中增加root=/dev/nand0p4。

kernel 需要挂载rootfs 时，取出root 参数，则得知需要挂载/dev/nand0p4 分区。

对于recovery 方案，就一直设置root_partition 为rootfs 即可。主系统需要从rootfs 分区读取数据，而recovery 系统使用initramfs，无需从rootfs 分区读取数据

即可正常运行OTA 应用等。当然，recovery 系统中要更新rootfs 的话，还是会访问(写入)rootfs 分区的，但这个动作就跟env 的root_partition 无关了。

对于AB 系统方案，可设置root_partition 为rootfsA 或rootfsB，以匹配不同的系统。OTA切换系统时，只需要改变此变量即可达到切换rootfs 的目的。

#### 3.2.10 配置备份env

由于写入env 时断电，可能导致env 的数据被破坏，因此需要支持备份env。

##### 3.2.10.1 方式一：env 分区扩展为存放两份env

此方式是在uboot 中进行定制实现，非社区原生方案。

可在uboot源码中搜索CONFIG_SUNXI_ENV_NOT_BACKUP, 若存在则说明支持此功能。

支持此功能后，只要uboot不配置CONFIG_SUNXI_ENV_NOT_BACKUP，则此功能默认开启。uboot会将env数据在同一分区中进行备份。

启用方法：

1. 修改分区表，将env 分区扩大到128k*2=256k。

工作方式：
uboot 检测到env 分区足够大，则激活env 备份功能，认为0-128k 存放第一份env 数据，128k-256k 存放第二份env 数据。由于env 数据本身带有CRC 校验，所

以可判断一份env 是否完整。启动时，uboot 会对两份env 进行同步，若某一份损坏则取另一份进行覆盖，若两份均完整，则以第一份为准。

对于用户空间的fw_printenv，默认只会更新第一份env，即执行fw_setenv 之后两份env 就有差异了，要到下次启动才由uboot 进行同步。若更新env 的过程中发

生掉电，则第一份env不完整，重新启动时，uboot 会识别到并用第二份env 覆盖第一份。

确认是否生效：

1.直接观察。
用户空间控制台执行：

```
hexdump -C /dev/by-name/env
```

确认是否存在两份。

2.尝试破坏env。

```
dd if=/dev/zero of=/dev/by-name/env bs=1k count=1
```

损坏第一份env，重启看能否正常启动，并尝试

```
fw_printenv
hexdump -C /dev/by-name/env
```

确认env 分区数据是否正常。

##### 3.2.10.2 方式二：增加env-redund 分区

此方式是uboot 原生功能。虽然也需要修改sunxi 的env 读取代码进行适配，但总体读写逻辑是社区原生的。
启用方法：

1.增加env-redund 分区。

将env 分区复制一份，分区名改为env-redund。

注意只是分区名修改为env-redund，其downloadfile 仍然指定为env.fex

2.支持在打包时制作冗余env。

```
make menuconfig --> Global build settings --> [*] sunxi make redundant env data
```

注意事项：
启用上述选项之后，打包时会调用mkenvimage 工具来制作env，对env 的格式有一定要求。

如注释和有效配置不能合并在一行。

若env-x.x.cfg 中存在类似如下配置

```
bootcmd=run setargs_nand boot_normal#default nand boot
```

则需要改成：

```
#default nand boot
bootcmd=run setargs_nand boot_normal
```

3.配置uboot 并重新编译uboot bin。

以r328 spinand 方案为例。

在

```
lichee/brandy-2.0/u-boot-2018/configs/sun8iw18p1_defconfig
```

中增加配置：

```
CONFIG_SUNXI_REDUNDAND_ENVIRONMENT=y
```

重新编译uboot。

4.修改fw_env.config

拷贝

```
package/utils/uboot-envtools/files/fw_env.config
```

到

```
target/allwinner/<board>/base-files/etc/ （若使用procd-init）
target/allwinner/<board>/busybox-init-base-files/etc/ （若使用busybox-init）
```

修改拷贝后的fw_env.config，增加备份env 的配置。

例如原本最后一行为：

```
/dev/by-name/env 0x0000 0x20000
```

则增加一行：

```
/dev/by-name/env-redund 0x0000 0x20000
```

这样用户空间的fw_printenv 和fw_setenv 即可正确处理两份env。

#### 3.2.11 配置启动脚本

procd-init 是默认配置好的。

busybox-init 需要手工配置下。

参考《Tina System init 使用说明文档》, 拷贝

```
<tina>/package/busybox-init-base-files/files/etc/init.d/load_script.conf
```

到

```
<tina>/target/allwinner/<platform>/busybox-init-base-files/etc/init.d/
```

并在其中添加一行：

```
swupdate_autorun
```



### 3.3 OTA 包

OTA 包中，需要包含sw-description 文件，以及本次升级会用到的各个文件，例如kernel，rootfs。

整个OTA 包是cpio 格式，且要求sw-description 文件在第一个。

#### 3.3.1 OTA 策略描述文件：sw-description

sw-description 文件是swupdate 官方规定的，OTA 策略的描述文件，具体语法可参考swupdate官方文档。

tina 提供了几个示例：

```
target/allwinner/generic/swupdate/sw-description-ab
target/allwinner/generic/swupdate/sw-description
```

也可以自行为具体的方案编写描述文件：

```
target/allwinner/<board>/swupdate/sw-description
```

本文件在SDK 中的存放路径和名字没有限定，只要最终打包进OTA 包中，重命名为swdescription并放在第一个文件即可。

#### 3.3.2 OTA 包配置文件：sw-subimgs.cfg

sw-subimgs.cfg 是tina 提供的，用于指示如何生成OTA 包。

基本格式为

```
swota_file_list=(
#表示把文件xxx拷贝到swupdate目录下，重命名为yyy，并把yyy打包到最终的OTA包中
xxx:yyy
)
swota_copy_file_list=(
#表示把文件xxx拷贝到swupdate目录下，重命名为yyy，但不把yyy打包到最终的OTA包中
xxx:yyy
)
```

swota_copy_file_list 存在的原因是，有一些文件我们只需要其sha256 值，而不需要文件本身。例如使用差分包配合readback handler 时，readback handler 需

要原始镜像的sha256值用于校验。

例子:

```
swota_file_list=(
#将target/allwinner/generic/swupdate/sw-description-ab-sign拷贝成sw-description，后续同理。
target/allwinner/generic/swupdate/sw-description-ab-sign:sw-description
out/${TARGET_BOARD}/uboot.img:uboot
out/${TARGET_BOARD}/boot0.img:boot0
out/${TARGET_BOARD}/image/boot.fex.gz:kernel.gz
out/${TARGET_BOARD}/image/rootfs.fex.gz:rootfs.gz
out/${TARGET_BOARD}/image/rootfs.fex.zst:rootfs.zst
)
```

tina 提供了几个示例：

```
target/allwinner/generic/swupdate/sw-subimgs.cfg # 普通系统，recovery系统，整包升级。这是其余demo的基础版本。
target/allwinner/generic/swupdate/sw-subimgs-ab.cfg # 改为AB系统。
target/allwinner/generic/swupdate/sw-subimgs-secure.cfg # 改为安全系统。
target/allwinner/generic/swupdate/sw-subimgs-ab-rdiff.cfg # 改为AB方案，差分方案。
target/allwinner/generic/swupdate/sw-subimgs-readback.cfg # 增加回读校验。
target/allwinner/generic/swupdate/sw-subimgs-sign.cfg # 增加签名校验。
target/allwinner/generic/swupdate/sw-subimgs-ubi.cfg # 改为ubi方案。
```

也可以自行为具体的方案编写描述文件。

```
target/allwinner/<board>/swupdate/sw-subimgs.cfg
```

本文件在SDK 中的路径需位于target/allwinner/<board>/swupdate 目录下，或target/allwinner/generic/swupdate 目录下。

名字需要命名为sw-subimg.cfg 或sw-subimgsxxx.cfg，其中xxx 可自定义。

这个限定主要是为了方便打包函数处理。在打包时，命令行传入参数xxx，则会使用swsubimgsxxx.cfg 进行打包。

#### 3.3.3 OTA 包生成：swupdate_pack_swu

在build/envsetup.sh 中提供了一个swupdate_pack_swu 函数。
可以参考该函数，自行实现一套打包swupdate 升级包的脚本。也可以直接使用，使用方式如下。

1. 准备好sw-descrition 文件，具体作用和语法请参考swupdate 说明文档。
2. 准备好sw-subimgs.cfg 文件，里面需要每一行列出一个打包需要的子镜像文件，即内核，rootfs 等。可以使用冒号分隔，前面为SDK 中的文件，后面为打包进OTA 包的文件名。若没有冒号则使用原文件名字。使用相对于tina 根目录的相对路径进行描述。其中第一个必须为swdescription。
3. 编译好所需的子镜像，例如主系统的内核和rootfs，recovery 系统等。
4. 执行swupdate_pack_swu 生成swupdate 升级包。不带参数执行，则会在特定路径下寻找sw-subimgs.cfg，解析配置生成OTA 包。带参数-xx 执行，则会在特定路径下寻找swsubimgs-xx.cfg，解析配置生成OTA 包。例如执行swupdate_pack_swu -sign，则会寻找sw-subimgs-sign.cfg，如此方便配置多个不同用途的sw-subimgs-xx.cfg。

注：不同介质使用的boot0/uboot 镜像不同，swupate_pack_swu 需要sys_config.fex 中的storage_type 配置明确指出介质类型，才能取得正确的boot0.img 和

uboot.img 具体可直接查看build/envsetup.sh 中swupdate_pack_swu 的实现。

### 3.4 recovery 系统方案举例

#### 3.4.1 配置分区和env

在分区表中，增加一个recovery 分区，用于保存recovery 系统。

size 根据实际recovery 系统的大小，再加点裕量。

download_file 可以留空，因为OTA 第一步就是写入一个recovery 系统。

当然也可以配置上download_file，并在打包固件之前先编译好recovery 系统，一并打包到固件中，这样出厂就带recovery 系统，后续的OTA 执行过程，可以考

虑不写入recovert 系统，用现成的，直接重启并升级主系统。

在env 中指定：

```
boot_partition=boot
root_partition=rootfs
```

并配置boot_normal 命令，从$boot_partition 变量指定的分区加载系统。

#### 3.4.2 配置主系统

lunch 选择方案后, make menuconfig, 选上swupdate。

#### 3.4.3 配置recovery 系统

假设没有现成的recovery 系统配置，则我们从主系统配置修改得到。lunch 选择方案后, 拷贝配置文件。

```
cdevice
cp defconfig defconfig_ota
```

根据上文介绍，make ota_menuconfig 选上swupdate, ramdisk, recovery 后缀等必要的配置。

recovery 系统整个运行在ram 中，如果系统过大会无法启动，所以需要进行裁剪。make ota_menuconfig, 将不必要的包尽量从recovery 系统中去掉。

#### 3.4.4 准备sw-description

这里我们直接使用：

```
target/allwinner/generic/swupdate/sw-description
```

内容如下，中文部分是注释，原文件中没有。

```
/* 固定格式，最外层为software = { } */
software =
{
    /* 版本号和描述*/
    version = "0.1.0";
    description = "Firmware update for Tina Project";
    /*
    * 外层tag，stable，
    * 没有特殊含义，就理解为一个字符串标志即可。
    * 可以修改，调用的时候传入匹配的字符串即可
    */
    stable = {
                /*
                * 内层tag，upgrade_recovery,
                * 当调用swupdate xxx -e stable,upgrade_recovery时，就会匹配到这部分，执行｛｝内的动作，
                 * 可以修改，调用的时候传入匹配的字符串即可
                */
                /* upgrade recovery,uboot,boot0 ==> change swu_mode,boot_partition ==> reboot */
                upgrade_recovery = {
                /* 这部分是为了在主系统中，升级recovery系统，升级uboot和boot0 */
                /* upgrade recovery */
                images: ( /* 处理各个image */
                    {
                        filename = "recovery"; /* 源文件是OTA包中的recovery文件*/
                        device = "/dev/by-name/recovery"; /* 要写到/dev/by-name/recovery节点中, 这
                        个节点在tina上就对应recovery分区*/
                        installed-directly = true; /* 流式升级，即从网络升级时边下载边写入, 而不是先完
                        整下载到本地再写入，避免占用额外的RAM或ROM */
                    },
                    {
                        filename = "uboot"; /* 源文件是OTA包中的uboot文件*/
                        type = "awuboot"; /* type为awuboot，则swupdate会调用对应的handler做处理*/
                    },
                    {
                        filename = "boot0"; /* 源文件是OTA包中的boot0文件*/
                        type = "awboot0"; /* type为awuboot，则swupdate会调用对应的handler做处理*/
                    }
                );
                /* image处理完之后，需要设置一些标志，切换状态*/
                /* change swu_mode to upgrade_kernel,boot_partition to recovery & reboot*/
                bootenv: ( /* 处理bootenv，会修改uboot的env分区*/
                    {
                        /* 设置env:swu_mode=upgrade_kernel, 这是为了记录OTA进度*/
                        name = "swu_mode";
                        value = "upgrade_kernel";
                    },
                    {
                        /* 设置env:boot_partition=recovery, 这是为了切换系统，下次uboot就会启动
                        recovery系统(kernel位于recovery分区) */
                        name = "boot_partition";
                        value = "recovery";
                    },
                    {
                        /* 设置env:swu_next=reboot, 这是为了跟外部脚本配合，指示外部脚本做reboot动作*/
                        name = "swu_next";
                        value = "reboot";
                    }
                    /* 实际有什么其他需求，都可以灵活增删标志来解决, 外部脚本和应用可通过fw_setenv/
                    fw_printenv操作env */
                    /* 注意，以上几个env，是一起在ram中修改好再写入的, 不会出现部分写入部分未写入的情况*/
                );
            };
            /*
            * 内层tag，upgrade_kernel,
            * 当调用swupdate xxx -e stable,upgrade_kernel时，就会匹配到这部分，执行｛｝内的动作，
            * 可以修改，调用的时候传入匹配的字符串即可。
            */
            /* upgrade kernel,rootfs ==> change sw_mode */
            upgrade_kernel = {
            /* upgrade kernel,rootfs */
            /* image部分，不赘述*/
      	  images: (
                {
                    filename = "kernel";
                    device = "/dev/by-name/boot";
                    installed-directly = true;
                },
                {
                    filename = "rootfs";
                    device = "/dev/by-name/rootfs";
                    installed-directly = true;
                }
            );
            /* change sw_mode to upgrade_usr,change boot_partition to boot */
            bootenv: (
                {
                    /* 设置env:swu_mode=upgrade_usr, 这是为了记录OTA进度*/
                    name = "swu_mode";
                    value = "upgrade_usr";
                },
                {
                    /* 设置env:boot_partition=boot, 这是为了切换系统，下次uboot就会启动主系统(
                    kernel位于boot分区) */
                    name = "boot_partition";
                    value = "boot";
                }
            );
        };
        /* 内层tag，upgrade_usr,
        当调用swupdate xxx -e stable,upgrade_usr时，就会匹配到这部分，执行｛｝内的动作，
         可以修改，调用的时候传入匹配的字符串即可*/
        /* upgrade usr ==> clean ==> reboot */
        upgrade_usr = {
            /*
            * misc-upgrade的小容量方案，将usr拆成独立分区了。
                  * 这里我们不需要，如果保留的话，不做任何image操作即可。
            * 也可以彻底删除这一部分，并将上面的upgrade_usr改掉。
                  */
            /* upgrade usr */
            /* OTA结束，清空各种标志*/
            /* clean swu_param,swu_software,swu_mode & reboot */
            bootenv: (
                {
                    name = "swu_param";
                    value = "";
                },
                {
                    name = "swu_software";
                    value = "";
                },
                {
                    name = "swu_mode";
                    value = "";
                },
                {
                    name = "swu_next";
                    value = "reboot";
                }
            );
        };
    };
    /* 当没有匹配上面的tag，进入对应的处理流程时，则运行到此处。我们默认清除掉一些状态*/
    /* when not call with -e xxx,xxx just clean */
    bootenv: (
        {
            name = "swu_param";
            value = "";
        },
        {
            name = "swu_software";
            value = "";
        },
        {
            name = "swu_mode";
            value = "";
        },
        {
            name = "swu_version";
            value = "";
        }
    );
}
```

**说明：**

升级过程会进行两次重启。具体的：
（1）升级recovery 分区(recovery)，uboot(uboot)，boot0(boot0) 。设置boot_partition为recovery。
（2）重启，进入recovery 系统。
（3）升级内核(kernel) 和rootfs(rootfs) 。设置boot_partition 为boot。
（4）重启，进入主系统，升级完成。

#### 3.4.5 准备sw-subimgs.cfg

我们直接看下tina 默认的：

```
target/allwinner/generic/swupdate/sw-subimgs.cfg
```

内容如下，中文部分是注释，原文件中没有。

```
swota_file_list=(
#取得sw-description，放到OTA包中。
#注意第一行必须为sw-description。如果源文件不叫sw-description，可在此处加:sw-description做一次重命名
target/allwinner/generic/swupdate/sw-description
#取得boot_initramfs_recovery.img，重命名为recovery，放到OTA包中。以下雷同
out/${TARGET_BOARD}/boot_initramfs_recovery.img:recovery
#uboot.img和boot0.img是执行swupdate_pack_swu时自动拷贝得到的，需配置sys_config.fex中的
storage_type
out/${TARGET_BOARD}/uboot.img:uboot
#注：boot0没有修改的话，以下这行可去除，其他雷同，可按需升级
out/${TARGET_BOARD}/boot0.img:boot0
out/${TARGET_BOARD}/boot.img:kernel
out/${TARGET_BOARD}/rootfs.img:rootfs
#下面这行是给小容量方案预留的，目前注释掉
#out/${TARGET_BOARD}/usr.img:usr
)
```

**说明：**
指明打包swupdate 升级包所需的各个文件的位置。这些文件会被拷贝到out 目录下，再生成swupdate OTA 包。

#### 3.4.6 编译OTA 包所需的子镜像

编译kernel 和rootfs。

```
make
```

编译recovery 系统。

```
swupdate_make_recovery_img
```

编译uboot。

```
muboot
```

打包，若需要升级boot0/uboot，则是必要步骤，打包会将boot0 和uboot 拷贝到out 目录下，并对头部参数等进行修改。生成的固件也可用于测试。注：如果希

望生成的固件的recovery分区是有系统的，则需要先编译recovery 系统，再打包。

```
pack / pack -s
```

生成OTA 包。因为我们使用的就是sw-subimgs.cfg，所以不同带参数。

注意，如果方案目录下存在sw-subimgs.cfg，则优先用方案目录下的。没有方案特定配置才用generic 下的。如果需要升级boot0/uboot，需要配置好

sys_config.fex 中的storage_type参数，swupdate_pack_swu 才能正确拷贝对应的boot0/uboot。

```
swupdate_pack_swu
```

#### 3.4.7 执行OTA

##### 3.4.7.1 准备OTA 包

对于测试来说，直接推入。

```
adb push out/<board>/swupdate/<board>.swu /mnt/UDISK
```

实际应用时, 可从先从网络下载到本地, 再调用swupdate, 也可以直接传入url 给swupdate。

##### 3.4.7.2 调用swupdate

若使用原生的swupdate，则调用：

```
swupdate -i /mnt/UDISK/<board>.swu -e stable,upgrade_recovery
```

但这样不会在自启动的时候帮我们准备好swupdate 所需的-e 参数。

我们可以使用辅助脚本：

```
swupdate_cmd.sh -i /mnt/UDISK/<board>.swu -e stable,upgrade_recovery
```



### 3.5 AB 系统方案举例

#### 3.5.1 配置分区和env

在分区表中，将原有的boot 分区和rootfs 分区，分区名改为bootA 和rootfsA。

将这两个分区配置拷贝一份，即新增两个分区，并把名字改为bootB 和rootfsB。

这样flash 中就存在A 系统(bootA+rootfsA) 和B 系统(bootB+rootfsB)。

一般是一个系统烧录两份。即分区表中的bootA 和bootB 都指定的boot.fex，rootfsA 和rootfsB 都指定的rootfs.fex。

在env 中，指定：

```
boot_partition=bootA
root_partition=rootfsA
```

并配置boot_normal 命令，从$boot_partition 变量指定的分区加载系统。

#### 3.5.2 配置主系统

lunch 选择方案后, make menuconfig, 选上swupdate。

#### 3.5.3 配置recovery 系统

AB 系统方案没有使用recovery 系统，无需配置和生成。

#### 3.5.4 准备sw-description

这里我们直接使用：

```
target/allwinner/generic/swupdate/sw-description-ab
```

内容如下，中文部分是注释，原文件中没有。

```
/* 固定格式，最外层为software = { } */
software =
{
/* 版本号和描述*/
version = "0.1.0";
description = "Firmware update for Tina Project";
/*

* 外层tag，stable，
* 没有特殊含义，就理解为一个字符串标志即可。
* 可以修改，调用的时候传入匹配的字符串即可。
  */
  stable = {
  /*
* 内层tag，now_A_next_B,
* 当调用swupdate xxx -e stable,now_A_next_B时，就会匹配到这部分，执行｛｝内的动作，
 * 可以修改，调用的时候传入匹配的字符串即可。
   */
   /* now in systemA, we need to upgrade systemB(bootB, rootfsB) */
   now_A_next_B = {
   /* 这部分是描述，当前处于A系统，需要更新B系统，该执行的动作。执行完后下次启动为B系统*/
   images: ( /* 处理各个image */
   {
   filename = "kernel"; /* 源文件是OTA包中的kernel文件*/
   device = "/dev/by-name/bootB"; /* 要写到/dev/by-name/bootB节点中, 这个节点
   在tina上就对应bootB分区*/
   installed-directly = true; /* 流式升级，即从网络升级时边下载边写入, 而不是先完
   整下载到本地再写入，避免占用额外的RAM或ROM */
   },
   {
   filename = "rootfs"; /* 同上，但处理rootfs，不赘述*/
   device = "/dev/by-name/rootfsB";
   installed-directly = true;
   },
   {
filename = "uboot"; /* 源文件是OTA包中的uboot文件*/
type = "awuboot"; /* type为awuboot，则swupdate会调用对应的handler做处理*/
},
{
filename = "boot0"; /* 源文件是OTA包中的boot0文件*/
type = "awboot0"; /* type为awuboot，则swupdate会调用对应的handler做处理*/
}
);
/* image处理完之后，需要设置一些标志，切换状态*/
bootenv: ( /* 处理bootenv，会修改uboot的env分区*/
{
/* 设置env:swu_mode=upgrade_kernel, 这是为了记录OTA进度, 对于AB系统来说，此时
已经升级完成，置空*/
name = "swu_mode";
value = "";
},
{
/* 设置env:boot_partition=bootB, 这是为了切换系统，下次uboot就会启动B系统(
kernel位于bootB分区) */
name = "boot_partition";
value = "bootB";
},
{
/* 设置env:root_partition=rootfsB, 这是为了切换系统，下次uboot就会通过cmdline
指示挂载B系统的rootfs */
name = "root_partition";
value = "rootfsB";
},
{
/* 兼容另外的切换方式，可以先不管*/
name = "systemAB_next";
value = "B";
},
{
/* 设置env:swu_next=reboot, 这是为了跟外部脚本配合，指示外部脚本做reboot动作*/
name = "swu_next";
value = "reboot";
}
);
};
/*

* 内层tag，now_B_next_A,
* 当调用swupdate xxx -e stable,now_B_next_A时，就会匹配到这部分，执行｛｝内的动作，
 * 可以修改，调用的时候传入匹配的字符串即可
   */
   /* now in systemB, we need to upgrade systemA(bootA, rootfsA) */
   now_B_next_A = {
   /* 这里面就不赘述了, 跟上面基本一致，只是AB互换了*/
   images: (
   {
   filename = "kernel";
   device = "/dev/by-name/bootA";
   installed-directly = true;
   },
   {
   filename = "rootfs";
   device = "/dev/by-name/rootfsA";
   installed-directly = true;
   },
{
filename = "uboot";
type = "awuboot";
},
{
filename = "boot0";
type = "awboot0";
}
);
bootenv: (
{
name = "swu_mode";
value = "";
},
{
name = "boot_partition";
value = "bootA";
},
{
name = "root_partition";
value = "rootfsA";
},
{
name = "systemAB_next";
value = "A";
},
{
name = "swu_next";
value = "reboot";
}
);
};
};
/* 当没有匹配上面的tag，进入对应的处理流程时，则运行到此处。我们默认清除掉一些状态*/
/* when not call with -e xxx,xxx just clean */
bootenv: (
{
name = "swu_param";
value = "";
},
{
name = "swu_software";
value = "";
},
{
name = "swu_mode";
value = "";
},
{
name = "swu_version";
value = "";
}
);
}
```

**说明：**

升级过程会进行一次重启。具体的：
（1）升级kernel 和rootfs 到另一个系统所在分区，升级uboot(uboot)，boot0(boot0) 。设置boot_partition 为切换系统。
（2）重启，进入新系统。

#### 3.5.5 准备sw-subimgs.cfg

我们直接看下tina 默认的：

```
target/allwinner/generic/swupdate/sw-subimgs-ab.cfg
```

内容如下，中文部分是注释，原文件中没有。

```
swota_file_list=(
#取得sw-description-ab, 重命名成sw-description, 放到OTA包中。
#注意第一行必须为sw-description
target/allwinner/generic/swupdate/sw-description-ab:sw-description
#取得uboot.img，重命名为uboot，放到OTA包中。以下雷同
#uboot.img和boot0.img是执行swupdate_pack_swu时自动拷贝得到的，需配置sys_config.fex中的
storage_type
out/${TARGET_BOARD}/uboot.img:uboot
#注：boot0没有修改的话，以下这行可去除，其他雷同，可按需升级
out/${TARGET_BOARD}/boot0.img:boot0
out/${TARGET_BOARD}/boot.img:kernel
out/${TARGET_BOARD}/rootfs.img:rootfs
)
```

说明：
指明打包swupdate 升级包所需的各个文件的位置。这些文件会被拷贝到out 目录下，再生成
swupdate OTA 包。

#### 3.5.6 编译OTA 包所需的子镜像

编译kernel 和rootfs。

```
make
```

编译uboot。

```
muboot
```

打包，若需要升级boot0/uboot，则是必要步骤，打包会将boot0 和uboot 拷贝到out 目录下，并对头部参数等进行修改。生成的固件也可用于测试。

```
pack / pack -s
```

生成OTA 包。因为我们使用的是sw-subimgs-ab.cfg，所以调用时带参数-ab。

注意，如果方案目录下存在sw-subimgs-ab.cfg，则优先用方案目录下的。没有方案特定配置才用generic 下的。

```
swupdate_pack_swu -ab
```

#### 3.5.7 执行OTA

##### 3.5.7.1 准备OTA 包

对于测试来说，直接推入。

```
adb push out/<board>/swupdate/<board>.swu /mnt/UDISK
```

实际应用时, 可从先从网络下载到本地, 再调用swupdate, 也可以直接传入url 给swupdate。

##### 3.5.7.2 判断AB 系统

对于AB 系统方案来说，必须判断当前所处系统，才能知道需要升级哪个分区的数据。

判断当前是处于A 系统还是B 系统。

方式一：直接使用fw_printenv 读取判断当前的boot_partition 和root_partition 的值。

##### 3.5.7.3 调用swupdate

若使用原生的swupdate，则调用：

```
当前处于A系统：

swupdate -i /mnt/UDISK/<board>.swu -e stable,now_A_next_B

当前处于B系统：

swupdate -i /mnt/UDISK/<board>.swu -e stable,now_B_next_A
```

但这样不会在自启动的时候帮我们准备好swupdate 所需的-e 参数。

我们可以使用辅助脚本：

```
当前处于A系统：
swupdate_cmd.sh -i /mnt/UDISK/<board>.swu -e stable,now_A_next_B

当前处于B系统：
swupdate_cmd.sh -i /mnt/UDISK/<board>.swu -e stable,now_B_next_A
```

### 3.6 辅助脚本swupdate_cmd.sh

为什么需要辅助脚本？

因为我们需要启动时能自动调用swupdate，自动传递合适的-e 参数给swupdate, 需要在合适的时候调用重启。

具体可直接看下脚本内容。

其基本思路是，当带参数调用时，脚本从传入的参数中，取出”-e xxx,yyy” 部分，将其余参数原样保存为env 的swu_param 变量。

取出的”-e xxx,yyy” 中的xxx 保存到env 的swu_software 变量, yyy 保存为env 的swu_mode变量。

然后就取出变量，循环调用。

```
swupdate $swu_param -e "$swu_software,$swu_mode"
```

sw-description 中可以通过改变env 的swu_software 和swu_mode 变量，来影响下次的调用参数。

实际应用时，可不使用此脚本，直接在主应用中，调用swupdate 即可。但要自行做好-e 参数的处理。

### 3.7 版本号

#### 3.7.1 使用方式

在sw-descriptionwen 文件中，会配置一个版本号字符串，如：

```
software =
{
version = "1.0.0";
...
}
```

如果需要在升级时检查版本号，则可使用-N 参数，传入的参数代表小机端当前的版本号。如果不需要，则不传递-N 参数，忽略版本号即可。

swupdate 会进行比较，如果OTA 包中sw-descriptionwen 文件配置的版本号小于当前版本号，则不允许升级。

如何在小机端保存，获取，更新版本号，需要自定义, swupdate 没有规定具体的方式。

#### 3.7.2 实现例子

应用可以按自己的逻辑维护版本号，不依赖系统env 等，只需按照swupate 要求传递参数即可。

此处提供一种依赖系统env 的实现方式供参考。

1.初始化设备端版本号。

首先需要定义设备端的版本号存放在哪，如何获取。

本方法定义设备端的版本号保存于env 之中，用swu_version 记录。则在SDK 中，需在env-x.x.cfg 中添加一行：

```
swu_version=1.0.0
```

表示此时版本为1.0.0，烧录固件后可执行fw_printenv 查看。

此步骤如果不做，则第一次烧录固件后env 中不存在swu_version，调用swupdate 时也无法传入获得并版本号，则第一次升级时不会检查版本。

注：这是tina 自定义的，可修改。只要读写这个版本号的地方均配套修改即可。实际应用时版本号可以存在任意分区中，或者存放在文件系统的文件中，或者硬编

码在系统和应用的二进制中，swupdate 未做限制。

2.在sw-description 中，设置OTA 包版本号。

升级时如果检查到OTA 包的sw-description 中的version，小于通过-N 参数传入的版本号，则不允许升级。

```
software =
{
version = "2.0.0";
...
```

例如当设备端的env 中设置了swu_version=2.0.0, 则调用swupdate_cmd.sh 时，会自动获取此参数并在调用swupdate 时传入-N 2.0.0。

此时若OTA 包中定义了version = “1.0.0” , 则此时升级会降低版本号，拒绝升级。

此时若OTA 包中定义了version = “2.0.0” , 则此次升级不会降低版本号，可以升级。

此时若OTA 包中定义了version = “3.0.0” , 则此次升级不会降低版本号，可以升级。

注：这是swupdate 原生的OTA 包版本号规则，不是tina 自定义的。

3.更新设备端版本号。

本方式版本号定义在env 中，则升级kernel 和rootfs 分区不会自动更新版本号，需要主动修改env。

若版本号是记录于rootfs 的某个文件，则不必在sw_description 中添加这种操作，因为更新rootfs 时版本号就自然更新了。但缺点是版本号跟rootfs 绑定了，每

次OTA 必须升级rootfs 才能更新版本号。

添加一个设置version 代表swu_version 的env 操作, 在OTA 时自动更新版本号。

```
software =
{
    #表示这个OTA包的版本号，给swupdate读取检查的。原生规定的。
    version = "2.0.0";
    ...
    bootenv: (
        ...
        {
            #表示这个OTA包的版本号，OTA时会写入env分区，用于在下次OTA时读出作为-N参数的值。
            Tina自定义的。
            name = "swu_version";
            value = "2.0.0";
        }
        	...
        );
    ...
}
```

注意，这么做的话，更新版本时需要修改env 中的版本号，以使得新的固件包拥有新的版本号，以及更新sw-description 的两个位置，一处是最上面的version = 

xxx 的版本号，一处是bootenv 操作中的版本号，以使得OTA 包拥有新的版本号，以及能在OTA 时写入新版本号。

4. 读取设备端版本号传给swupdate。

假如小机端是用脚本调用，则可用如下方式读取并传给swupdate：

```
swu_version=$(fw_printenv -n swu_version)
swupdate ... -N $swu_version
```

更好的方式是判断非空才传入，如此可支持不在env 中提前配置好swu_version。

```
check_version_para=""
[ x"$swu_version" != x"" ] && {
echo "now version is $swu_version"
check_version_para="-N $swu_version"
}
swupdate ... $check_version_para
```

注：
如果不使用版本号，则不在env 中设置swu_version，也不在bootenv 中写swu_version 即可。

如果sw_description 中的版本号一直保持v1.0.0，也总是能升级。

### 3.8 签名校验

#### 3.8.1 检验原理

OTA 包中包含了sw-decsription 文件和各个具体的镜像，如kernel，rootfs。

如果对整个OTA 包进行完整校验，则会对流式升级造成影响，要求必须把整个OTA 包下载下来，才能判断出校验是否通过。

为了避免上述问题，swupdate 的校验是分镜像的，首先从OTA 包最前面取出两个文件，即swdescription和sw-description.sig，使用传入的公钥校验sw-

description，校验通过则认为sw-description 可信，则说明其中描述的image 和sha256 也是可信的。

后续无需再使用公钥，直接校验每个镜像的sha256 即可。因此可以逐个镜像处理，无需全部下载完毕再处理。

#### 3.8.2 配置

swupdate 支持使用签名校验功能，需要在编译时选中对应功能。

出于安全考虑，一旦使能了校验，则swupdate 不再支持不使用签名的更新调用。

```
make menuconfig --->
    Allwinner --->
        <*> swupdate --->
            [*] Enable verification of signed images
                Signature verification algorithm (RSA PKCS#1.5) --->(选择校验算法，此处以RSA为例)
```

注意，recovery 系统也需要对应进行配置，即：

```
make ota_menuconfig ---> ...(重复以上配置)
```

#### 3.8.3 使用方法

在PC 端使用私钥签名OTA 包。

在小机端调用swupdate 时，使用-k 参数传入公钥。

#### 3.8.4 初始化key

Tina 封装了一条命令，生成默认的密钥对。执行：

```
swupdate_init_key
```

执行后会使用默认密码生成密钥对并拷贝到指定目录：

密码/私钥/公钥：

```
password:tina/target/allwinner/方案名/swupdate/swupdate_priv.password
private key:tina/target/allwinner/方案名/swupdate/swupdate_priv.pem
public key:tina/target/allwinner/方案名/swupdate/swupdate_public.pem

公钥拷贝到base-files中，供使用procd-init的方案使用
public key:tina/target/allwinner/方案名/base-files/swupdate_public.pem
公钥拷贝到busybox-init-base-files中，供使用busybox-init的方案使用
public key:tina/target/allwinner/方案名/busybox-init-base-files/swupdate_public.pem
```

此步骤仅为方便调试使用，只需要做一次。

用户也可使用自己的密码自行生成密钥，生成密钥的具体命令可参考build/envsetup.sh 中swupdate_init_key 的实现：

```
local password="swupdate";
echo "$password" > swupdate_priv.password;
echo "-------------------- init priv key --------------------";
openssl genrsa -aes256 -passout file:swupdate_priv.password -out swupdate_priv.pem;
echo "-------------------- init public key --------------------";
openssl rsa -in swupdate_priv.pem -passin file:swupdate_priv.password -out
swupdate_public.pem -outform PEM -pubout;
```

生成的密钥如swupdate_init_key 一般放到tina/target/allwinner/方案名/swupdate/ 中，即可在打包OTA 包时自动使用。

主要就是调用openssl 生成，私钥拷贝到SDK 指定目录，供生成OTA 包时使用。公钥放到设备端，供设备端执行OTA 时使用。

密钥的作用是校验OTA 包，意味着拿到密钥的人即可生成可通过校验的OTA 包，因此正式产品中一般密钥只掌握在少数人手中，并采取适当措施避免泄漏或丢失。

一种可参考的实践方式是，正式密钥做好备份，并仅部署在有权限管控的服务器上，只能代码入库后通过自动构建生成OTA 包，普通工程师无法拿到密钥自行本地

生成用于正式产品的OTA包。

#### 3.8.5 修改sw-description

如上文所述，每个image 在使用时会校验sha256，因此需要在为每个更新文件在swdesctiption中添加sha256 属性，指定sha256 的值供更新过程校验。

有独立镜像的文件才需要sha256 属性，例如images 中配置的文件。而bootenv 等直接写在sw-description 中的，则无需sha256 属性。

目前脚本支持自动在生成OTA 包时，更新sha256 的值。但需要在sw-description 中，手工添加：

```
sha256 = @文件名
```

如：

```
$ git diff sw-description
diff --git a/allwinner/cowbell-perf1/configs/sw-description b/allwinner/cowbell-perf1/
configs/sw-description
index ed04b64..467ac3b 100644
--- a/allwinner/cowbell-perf1/configs/sw-description
+++ b/allwinner/cowbell-perf1/configs/sw-description
@@ -9,14 +9,17 @@ software =
{
filename = "boot_initramfs_recovery.img"
device = "/dev/by-name/recovery";

+ sha256 = "@boot_initramfs_recovery.img"
  },
  {
  filename = "boot_package.fex"
  type = "awuboot";
+ sha256 = "@boot_package.fex"
  },
  {
  filename = "boot0_nand.fex"
  type = "awboot0";
+ sha256 = "@boot0_nand.fex"
  }
  );
  @@ -34,10 +37,12 @@ software =
  {
  filename = "boot.img";
  device = "/dev/by-name/boot";
+ sha256 = "@boot.img"
  },
  {
  filename = "rootfs.img";
  device = "/dev/by-name/rootfs";
+ sha256 = "@rootfs.img"
  }
  );
  bootenv: (
```

在打包OTA 包时，脚本自动算出sha256 的值，并替换到上述位置，再完成OTA 包的生成。

可参考：

```
target/allwinner/generic/swupdate/sw-description-sign
target/allwinner/generic/swupdate/sw-subimgs-sign.cfg
注：
调用swupdate_pack_swu 则会使用sw-subimgs.cfg，其中默认指定了使用sw-description做为最终的sw-description。
调用swupdate_pack_swu -sign则会使用sw-subimgs-sign.cfg，其中默认指定了使用sw-description-sign做为
最终的sw-description。
即关键还是看使用哪份sw-subimgs.cfg，以及sw-subimgs.cfg中如何指定。
```

#### 3.8.6 添加sw-description.sig

签名的OTA 包，需要生成签名文件sw-description.sig，并使其在OTA 包中，紧随在swdescription后面。

目前脚本中自动处理。

#### 3.8.7 生成OTA 包

方法不变，脚本中会检测defconfig 的配置，并自动完成签名等动作。

#### 3.8.8 将公钥放置到小机端

目前脚本中生成key 的时候，自动拷贝了。如需手工处理，可参考如下方式。

对于procd-init：

```
cdevice
mkdir -p ./base-files
cp swupdate_public.pem ./base-files/etc/
```

对于busybox-init

```
cdevice
mkdir -p ./busybox-init-base-files/
cp swupdate_public.pem ./busybox-init-base-files/etc/
```



#### 3.8.9 在小机端调用

在原本的命令基础上，加上-k /etc/swupdate_public.pem 即可, 如：

```
swupdate_cmd.sh -v -i /mnt/UDISK/tina-cowbell-perf1.swu -k /etc/swupdate_public.pem
```

### 3.9 压缩

swupdate 支持对镜像先解压，再写入目标位置，当前支持gzip 和zstd 两种压缩算法。

#### 3.9.1 配置

使用gzip 压缩无需配置，使用zstd 则需选上

```
make menuconfig --> Allwinner ---> <*> swupdate --> [*] Zstd compression support
```



#### 3.9.2 生成压缩镜像

如果希望每次打包固件自动生成，则可修改scripts/pack_img.sh, 在function do_pack_tina()函数的最后加上压缩的动作。

```
生成gz镜像:
gzip -k -f boot.fex
gzip -k -f rootfs.fex
gzip -k -f recovery.fex #如果使用AB方案，则无需recovery
生成lzma镜像:
zstd -k -f boot.fex -T0
zstd -k -f rootfs.fex -T0
zstd -k -f recovery.fex -T0
```

对于lzma，若需要调整压缩率，可指定0-19 的数字(数字越大，压缩率越高，耗时越长)，如

```
zstd -19 -k -f boot.fex -T0
zstd -19 -k -f rootfs.fex -T0
zstd -19 -k -f recovery.fex -T0
```

如果不希望每次打包固件多耗时间，则需自行在生成OTA 包之前，使用上述命令制作好压缩镜像。原始的boot.fex，rootfs.fex, recovery.fex 在out/方案/image/目录下。

#### 3.9.3 sw-subimgs.cfg 配置压缩镜像

以rootfs 为例，将原本未压缩的版本

```
out/${TARGET_BOARD}/rootfs.img:rootfs
```

改成压缩的

```
out/${TARGET_BOARD}/image/rootfs.fex.gz:rootfs.gz
```

或

```
out/${TARGET_BOARD}/image/rootfs.fex.zst:rootfs.zst
```

注：为了方便差分包的处理，此处约定压缩镜像需以.gz 或.zst 结尾，生成差分项的脚本会检查后缀名，并自动解压。

#### 3.9.4 sw-description 配置压缩镜像

以rootfs 为例，将原本未压缩的版本

```
{
    filename = "rootfs";
    device = "/dev/by-name/rootfs";
    installed-directly = true;
    sha256 = "@rootfs";
},
```

换成

```
{
    filename = "rootfs.gz";
    device = "/dev/by-name/rootfs";
    installed-directly = true;
    sha256 = "@rootfs.gz";
    compressed = "zlib";
},
```

或

```
{
    filename = "rootfs.zst";
    device = "/dev/by-name/rootfsB";
    installed-directly = true;
    sha256 = "@rootfs.zst";
    compressed = "zstd";
},
```



### 3.10 调用OTA

swupdate_cmd.sh , 用于给swupdate 传入相关参数，切换更新状态，以及不断重试。

#### 3.10.1 进度条

swupdate 提供了progress 程序，该程序会在后台运行，从socket 获取进度信息，打印进度条到串口。

具体方案可参考其实现(在swupdate 源码中搜索progress)，自行在应用中获取进度，通过屏
幕等其他方式进行指示。

#### 3.10.2 重启

1.调用swupdate 的时候加上-p reboot , 则swupdate 更新完毕后，会执行reboot。

2.swupdate_cmd.sh 支持检测env 中的swu_next 变量，如果为reboot，则脚本中执行reboot。可在sw-description 中设置此变量。

3.如果调用progress 的时候加上-r 参数，则progress 会在检测到更新完成后，执行reboot。

#### 3.10.3 本地升级示例

将生成的OTA 包推送到小机端，如放在/mnt/UDISK 目录下。

PC 端执行：

```
adb push out/cowbell-perf1/swupdate/tina-cowbell-perf1.swu /mnt/UDISK
```

小机端执行(不带签名校验版本)：

```
swupdate_cmd.sh -i /mnt/UDISK/tina-cowbell-perf1.swu -e stable,upgrade_recovery
```

小机端执行(带签名校验版本)：

```
swupdate_cmd.sh -i /mnt/UDISK/tina-cowbell-perf1.swu -k /etc/swupdate_public.pem -e stable,upgrade_recovery
```



#### 3.10.4 网络升级示例

启动服务器：

```
cd out/cowbell-perf1/swupdate/
sudo python -m SimpleHTTPServer 80 #启动一个服务器
```

小机端命令，使用-d -uxxx，xxx 为url。

例如(不带签名校验版本)：

```
swupdate_cmd.sh -d -uhttp://192.168.35.112/tina-cowbell-perf1.swu -e stable,upgrade_recovery
```

例如(带签名校验版本)：

```
swupdate_cmd.sh -d -uhttp://192.168.35.112/tina-cowbell-perf1.swu -k /etc/swupdate_public.pem -e stable,upgrade_recovery
```

注：需依赖外部程序，提供自动联网支持。OTA 本身不处理联网。

#### 3.10.5 错误处理

如何判断swupdate 升级出错？

1.调用swupdate 时获得并判断返回值是否为0。

2.读取env 变量recovery_status。根据swupdate 官方文档，swupdate 开始执行时，会设置recovery_status=“progress”，升级完成会清除这个变量，升级失败则设置recovery_status=“failed”。

### 3.11 裁剪

swupdate 本身是可配置的, 不需要某些功能时，可将其裁剪掉。

```
make menuconfig --->
    Allwinner --->
    	<*> swupdate --->

```

例如，不需要使用swupdate 来从网络下载OTA 包的话，则可将

```
[*] Enable image downloading
```

取消掉。
不需要更新boot0/uboot 的话，则将

```
Image Handlers --->
	[*] allwinner boot0/uboot
```

取消掉

### 3.12 调试

#### 3.12.1 直接调用swupdate

目前swupdate_cmd.sh 主要有两个作用：

1.自启动，无限重试。

2.在主系统和recovery 系统中，传入不同的-e 参数给swupdate。

出问题时，可以不使用swupdate_cmd.sh，手工直接调用swupdate，在后面加上合适的-e 参数，观察输出log。如：

```
swupdate -v -i xxx.swu -e stable,boot
swupdate -v -i xxx.swu -e stable,recovery
```



#### 3.12.2 手工切换系统

按上述env 的配置，启动的系统，是由boot_partition 变量控制的。

注意，需要/var/lock 目录存在且可写。

切换到主系统：

```
fw_setenv boot_partition boot
reboot
```

切换到recovery 系统：

```
fw_setenv boot_partition recovery
reboot
```

观察当前变量：

```
fw_printenv
```



#### 3.12.3 更新boot0/uboot

目前更新boot0，uboot 实际功能是由另一个软件包ota-burnboot 完成的，swupdate 只是准备数据，并调用ota-burnboot 提供的动态库。

如果更新失败，先尝试手工使用ota-burnboot0 xxx 和ota-burnuboot xxx 能否正常更新。以确定是ota-burnboot 的问题，还是swupdate 的问题。

#### 3.12.4 解压OTA 包

swupdate 的OTA 包，本质上是一个cpio 格式的包，直接使用通用的cpio 解包命令即可。

```
cpio -idv < xxx.swu
```



#### 3.12.5 校验OTA 包

当使能了签名校验，会对sw-description 签名生成sw-description.sig，如果校验失败，可以在PC 端手工验证下：

使用RSA 时, build/envsetup.sh 中调用的命令是：

```
openssl dgst -sha256 -sign "$priv_key_file" $password_para "$SWU_DIR/sw-description" > "
$SWU_DIR/sw-description.sig"
```

则对应的密钥验证签名命令为：

```
openssl dgst -prverify swupdate_priv.pem -sha256 -signature sw-description.sig swdescription
```

公钥验证签名的命令为：

```
openssl dgst -verify swupdate_public.pem -sha256 -signature sw-description.sig swdescription
```



### 3.13 测试固件示例

#### 3.13.1 生成方式

一般而言，测试需要两个有差异的OTA 包，如，uboot 和kernel 的log 有差异，rootfs 的文件有差异。

这样方法测试人员根据log 判断是否升级成功。

##### 3.13.1.1 准备工作

如果需要网络更新，OTA 不负责联网，所以需要选上wifimanager-daemon。

```
Allwinner
---> <*> wifimanager
---> [*] Enable wifimanager daemon support
---> <*> wifimanager-daemon-demo..................... Tina wifimanager daemon demo
```



##### 3.13.1.2 生成固件1 和OTA 包1

重新编译boot，使得编译时间更新：

```
mboot
```

创建文件以标记rootfs：

```
cd target/allwinner/cowbell-ailabs_c1c/base-files
rm -f OTA1 OTA2
echo OTA1 > OTA1
```

重新编译recovery 系统：

```
swupdate_make_recovery_img
```

重新编译打包，使得编译时间更新：

```
mp -j32
```

生成OTA 包1：

```
swupdate_pack_swu
```

得到产物：

```
cp out/cowbell-perf1/tina_cowbell-perf1_uart0.img tina_cowbell-perf1_uart0_OTA1.img
cp out/cowbell-perf1/swupdate/tina-cowbell-perf1.swu tina-cowbell-perf1_OTA1.swu
```



##### 3.13.1.3 生成固件2 和OTA 包2

重新编译uboot，使得编译时间更新：

```
muboot
```

创建文件以标记rootfs：

```
cd target/allwinner/cowbell-ailabs_c1c/base-files
rm -f OTA1 OTA2
echo OTA2 > OTA2
```

重新编译recovery 系统：

```
swupdate_make_recovery_img
```

重新编译打包，使得编译时间更新：

```
mp -j32
```

生成OTA 包2：

```
swupdate_pack_swu
```

得到产物：

```
cp out/cowbell-perf1/tina_cowbell-perf1_uart0.img tina_cowbell-perf1_uart0_OTA2.img
cp out/cowbell-perf1/swupdate/tina-cowbell-perf1.swu tina-cowbell-perf1_OTA2.swu
```



#### 3.13.2 使用方式

任意选择一个OTA 固件烧录后，可在此基础上进行本地升级或网络升级。

##### 3.13.2.1 本地升级方式

PC 端执行：

```
adb push out/astar-parrot/swupdate/tina-cowbell-perf1_OTA1.swu /mnt/UDISK
```

小机端执行：

```
swupdate_cmd.sh -i /mnt/UDISK/tina-cowbell-perf1_OTA1.swu
```

##### 3.13.2.2 网络升级方式

PC 端搭建服务器：

```
sudo python -m SimpleHTTPServer 80
```

小机端联网：

```
wifi_connect_ap_test SSID 密码
```

小机端执行：

```
swupdate_cmd.sh -d -uhttp://192.168.xxx.xxx/tina-cowbell-perf1_OTA1.swu
```

注明：启动后会自动联网，连网后等待OTA 后台脚本尝试更新。中途掉电重启后，正常会在启动后几十秒内，成功联网并开始继续更新。

##### 3.13.2.3 升级过程

升级过程会进行两次重启。具体的：
（1）升级recovery 分区(boot_initramfs_recovery.img)，uboot(boot_package.fex)，boot0(boot0_nand.fex)。
（2）重启，进入recovery 系统。
（3）升级内核(boot，img) 和rootfs(rootfs.img)。
（4）重启，进入主系统，升级完成。

##### 3.13.2.4 判断升级

从log 中的时间和rootfs 文件可以判断当前运行的版本。

例如：

```
OTA_1：
boot0：[66]HELLO! BOOT0 is starting Dec 29 2018 16:15:59!
uboot：U-Boot 2018.05-00015-g5068c23-dirty (Dec 29 2018 - 16:15:41 +0800) Allwinner Technology
kernel：[ 0.000000] Linux version 4.9.118 (zhuangqiubin@Exdroid5) (gcc version 6.4.1 (
OpenWrt/Linaro GCC 6.4-2017.11 2017-11) ) #189 SMP Sat Dec 29 08:13:38 UTC 2018 （注，进入控制台cat /proc/version 也可看到）
rootfs：ls查看下，在根目录下有一个文件OTA1
OTA_2：
boot0: [66]HELLO! BOOT0 is starting Dec 29 2018 16:17:35!
uboot: U-Boot 2018.05-00015-g5068c23-dirty (Dec 29 2018 - 16:17:17 +0800) Allwinner Technology
kernel:[ 0.000000] Linux version 4.9.118 (zhuangqiubin@Exdroid5) (gcc version 6.4.1 (
OpenWrt/Linaro GCC 6.4-2017.11 2017-11) ) #190 SMP Sat Dec 29 08:18:33 UTC 2018（注，进入控制台cat /proc/version 也可看到）
rootfs：ls查看下，在根目录下有一个文件OTA2
```

### 3.14 升级定制分区

如果定制了一个分区，并需要对此分区进行OTA，则需要：

1.确认需不需要备份。

2.将分区文件加入OTA 包。

3.确定升级策略，在sw-description 中增加对此分区的处理。

#### 3.14.1 备份

在OTA 过程随时可能掉电，如果掉电时正在升级分区mypart ，则重启后mypart 的数据是不完整的。

是否需要备份主要取决于mypart 所保存的文件是否影响继续进行OTA。

例如mypart 中保存的是开机音乐，被损坏的结果只是开机无声音，但开机后能正常继续OTA，则无需备份。

例如mypart 中保存的是应用，且OTA 中途掉电后重启，需要应用负责联网从网络重新下载OTA 包，则需要备份，否则无法继续OTA，设备就无法恢复了。

例如mypart 中保存的是dsp，启动过程没有有效的dsp 镜像会无法启动，则需要备份，否则无法启动也就无法继续OTA，设备就无法恢复了。

#### 3.14.2 无需备份

假设分区表中定义了：

```
[partition]
name = mypart
size = 512
downloadfile = "mypart.fex"
user_type = 0x8000
```

文件放在：

```
out/${TARGET_BOARD}/image/mypart.fex
```

则在sw-subimg.cfg 中增加一行（注意要放到sw-description 之后，因为sw-description 必须是第一个文件），将mypart.fex 打包到OTA 包中，重命名为mypart：

```
out/${TARGET_BOARD}/image/mypart.fex:mypart
```

在sw-description 中增加升级动作的定义。例如可以在升级rootfs 之后，升级该分区：

```
software =
{
...
stable = {
...
upgrade_kernel = {
images: (
...
{
filename = "rootfs";
device = "/dev/by-name/rootfs";
installed-directly = true;
},
{
filename = "mypart";
device = "/dev/by-name/mypart";
installed-directly = true;
}
...
);
...
```



### 3.14.3 需要备份

在分区表中定义好两个分区，这样升级过程掉电，总有一份是完整的。

```
[partition]
name = mypart
size = 512
downloadfile = "mypart.fex"
user_type = 0x8000
[partition]
name = mypart-r
size = 512
downloadfile = "mypart.fex"
user_type = 0x8000
```

文件放在：

```
out/${TARGET_BOARD}/image/mypart.fex
```

则在sw-subimg.cfg 中增加一行（注意要放到sw-description 之后，因为sw-description 必

须是第一个文件），将mypart.fex 打包到OTA 包中，重命名为mypart：

```
out/${TARGET_BOARD}/image/mypart.fex:mypart
```

有两个分区，则需要条件决定使用哪一个，可考虑在env 中定义一个变量：

```
mypart_partition=mypart
```

使用mypart 时，要先读取env 的mypart_partition 的值来决定要使用哪个分区。

在sw-description 中，定义好要升级的分区和bootenv，保证每次升级那个未在使用的分区。

这样即使掉电也无妨。

```
software =
{
...
stable = {
upgrade_recovery = {
images:
{
filename = "recovery";
device = "/dev/by-name/recovery";
installed-directly = true;
},
/* 更新mypart-r， 此时掉电，mypart分区是完整的*/
{
filename = "mypart-r";
device = "/dev/by-name/mypart-r";
installed-directly = true;
}
);
bootenv: (
{
name = "swu_mode";
value = "upgrade_kernel";
},
{
name = "boot_partition";
value = "recovery";
},
/* 设置这个env，指示下次启动，即启动到recovery分区时，配套使用mypart-r */
{
name = "mypart_partition";
value = "mypart-r";
},
{
name = "swu_next";
value = "reboot";
}
);
};
upgrade_kernel = {
images: (
{
filename = "kernel";
device = "/dev/by-name/boot";
installed-directly = true;
},
{
filename = "rootfs";
device = "/dev/by-name/rootfs";
installed-directly = true;
},
/* 更新mypart， 此时掉电，mypart分区还是完整的*/
{
filename = "mypart-r";
device = "/dev/by-name/mypart-r";
installed-directly = true;
}
);
bootenv: (
{
name = "swu_mode";
value = "upgrade_usr";
},
/* 设置这个env，指示下次启动，即启动到正常系统时，使用mypart */
{
name = "mypart_partition";
value = "mypart";
},
{
name = "boot_partition";
value = "boot";
}
);
};
...
```

### 3.15 handler 说明

此处只对一些handler 做简介。

具体的handler 的用法，请参考swupdate 官方文档说明。

#### 3.15.1 awboot

##### 3.15.1.1 nand/emmc

全志拓展的handler，用于支持升级全志boot0 和uboot，实质上会调用外部的ota-burnboot包完成升级。

目前只支持nand/emmc，详见ota-burnboot 章节。

使用方式：

选上handler 支持：

```
make menuconfig --->
Allwinner --->
<*> swupdate --->
Image Handlers -->
[*] allwinner boot0/uboot
```

注意，recovery 系统也需要对应进行配置，即：

```
make ota_menuconfig ---> ...(重复以上配置)
```

在sw-description 中指定type 即可：

```
{
filename = "uboot"; /* 源文件是OTA包中的uboot文件*/
type = "awuboot"; /* type为awuboot，则swupdate会调用对应的handler做处理*/
},
{
filename = "boot0"; /* 源文件是OTA包中的boot0文件*/
type = "awboot0"; /* type为awuboot，则swupdate会调用对应的handler做处理*/
}
```

##### 3.15.1.2 nor

对于nor 方案，升级boot0/uboot 有掉电风险。如确需升级，可直接配置合适偏移即可，不使用awboot handler。

例如已知boot0 存放在偏移为0 处，uboot 存放在偏移为24k 处，则配置：

```
{
filename = "boot0";
device = "/dev/mtdblock0";
installed-directly = true;
},
{
filename = "uboot";
device = "/dev/mtdblock0";
offset = "24k"
installed-directly = true;
}
```



#### 3.15.2 readback

用于支持在sw-description 中配置sha256，在升级后读出数据进行校验。

一种应用场景是，在AB 系统差分升级时，应用差分包后读出校验，以确认差分得到的结果是对的，再切换系统。

需选上对应handler。

```
make menuconfig/make ota_menuconfig
--> Allwinner
--> swupdate
--> <*> Allow to add sha256 hash to each image
make menuconfig/make ota_menuconfig
--> Allwinner
--> swupdate
--> Image Handlers
--> [*] readback
```



##### 3.15.2.1 示例

```
target/allwinner/generic/swupdate/sw-description-readback
target/allwinner/generic/swupdate/sw-subimgs-readback.cfg
```

#### 3.15.3 ubi

用于ubi 方案。
需先选上MTD 支持：

```
make menuconfig/make ota_menuconfig
--> swupdate
--> Swupdate Settings
---> General Configuration
---> [*] MTD support
```

再选上对应handler：

```
make menuconfig/make ota_menuconfig
--> swupdate
--> Image Handlers
--> [*] ubivol
```



##### 3.15.3.1 示例

请参考：

```
target/allwinner/generic/swupdate/sw-subimgs-ubi.cfg
target/allwinner/generic/swupdate/sw-description-ubi
```

#### 3.15.4 rdiff

rdiff handle 用于差分包升级。

需选上对应handler：

```
make menuconfig/make ota_menuconfig
--> swupdate
--> Image Handlers
--> [*] rdiff
#若使用AB系统方案，则无recovery系统，则make ota_menuconfig的配置可不做。
```



##### 3.15.4.1 特性

在https://librsync.github.io/page_rdiff.html 中指出

```
rdiff cannot update files in place: the output file must not be the same as the input file.
```

即不支持原地更新，即应用差分包将A0 更新成A1，需要有足够空间存储A0 和A1，不能直接对A0 进行改动。

```
rdiff does not currently check that the delta is being applied to the correct file. If a delta is applied to the wrong basis file, the results will be garbage.
```

不校验原文件，如果将差分包应用于错误的文件，则会得到无效的输出文件，但不会报错。

```
The basis file must allow random access. This means it must be a regular file rather than apipe or socket.
```

原文件必须支持随机访问，因此不能从管道或socket 中获取原文件。
更多介绍请参考：https://librsync.github.io/

##### 3.15.4.2 示例

首先需要将方案修改为AB 系统的方案，请参考上文的“AB 系统方案举例”。

swupdate 的配置文件请参考：

```
target/allwinner/generic/swupdate/sw-description-ab-rdiff
target/allwinner/generic/swupdate/sw-subimgs-ab-rdiff.cfg
```

差分文件的生成使用rdiff：

```
$ rdiff -h
Usage: rdiff [OPTIONS] signature [BASIS [SIGNATURE]]
[OPTIONS] delta SIGNATURE [NEWFILE [DELTA]]
[OPTIONS] patch BASIS [DELTA [NEWFILE]]
```

tina 封装了一条命令，用于解出两个swu 包并生成各个子镜像的差分文件。

一种参考的差分包生成方式是，先按普通的升级方式生成整包。

假设旧固件对应V1.swu，新固件对应V2.swu，则可使用：

```
swupdate_make_delta V1.swu V2.swu
```

生成差分文件。

再将生成的差分文件，用于生成差分的OTA 包。

例如：

```
make && pack #生成V1固件
swupdate_pack_swu -ab #得到out/r328s2-perf1/swupdate/tina-r328s2-perf1-ab.swu
cp out/r328s2-perf1/swupdate/tina-r328s2-perf1-ab.swu V1.swu #保存V1的OTA包
#进行一些修改
make && pack #生成V2固件
swupdate_pack_swu -ab #得到out/r328s2-perf1/swupdate/tina-r328s2-perf1-ab.swu
cp out/r328s2-perf1/swupdate/tina-r328s2-perf1-ab.swu V2.swu #保存V2的OTA包
swupdate_make_delta V1.swu V2.swu #生成差分镜像
swupdate_pack_swu -ab-rdiff #生成差分OTA包，这一步用到了刚刚生成的差分镜像
```



##### 3.15.4.3 开销问题

差分升级的主要好处在于节省传输的文件大小。而不是节省ram 和rom 的占用。

设备端在进行差分升级时，需要使用版本匹配的旧版本镜像，加上差分包，生成新版本镜像。

rsync 不支持原地更新，必须有额外的空间保存新生成的镜像。

从掉电安全的角度考虑，在新版本镜像完整保存到flash 之前，旧版本镜像不能破坏，否则一旦中途掉电，将无法再次使用旧镜像+ 差分包生成新镜像，只能联网

下载完整的OTA 包。

以上限制，导致flash 必须在旧镜像之外，有足够flash 空间用于存放新的镜像。

对于recovery 方案，原本的：

```
从OTA包获取新recovery写入recovery分区-->
reboot -->
从OTA包获取新kernel写入boot分区-->
从OTA包获取新rootfs升级rootfs分区-->
reboot
```

就需要变成：

```
从OTA包获取recovery差分包，读recovery分区，生成新recovery暂存到文件系统中-->
从文件系统获取新recovery写入recovery分区-->
reboot -->
...
```

总体较为麻烦，且需要文件系统足够大。
对于AB 方案，原本的：

```
从OTA包获取新kernel写入bootB分区-->
从OTA包获取新rootfs写入rootfsB分区-->
reboot
```

就需要变成：

```
从OTA包获取kernel差分包，读出bootA分区，合并生成新kernel写入bootB分区-->
从OTA包获取rootfs差分包，读出rootfsA分区，合并生成新rootfs写入bootB分区-->
reboot
```

不需要依赖额外的文件系统空间。

因此，若希望节省ram/rom 占用，差分包并非解决的办法。若希望使用差分包，建议配合AB 系统使用。

##### 3.15.4.4 管理问题

差分包的一个麻烦问题在于，差分包必须跟设备端的版本匹配。而出厂之后的设备，可能存在多种版本。

例如出厂为V1，当前最新为V4，则设备可能处于V1，V2，V3。此时若使用整包升级，则无需区分。

若使用差分升级，一种策略是为每个旧版本生成一个差分包，则需要制作三个差分包V1_4，V2_4，V3_4，并在OTA 时先判断设备端和云端版本，再使用对应的差

分包。

另一种策略是，只为上一个版本生成差分包V3_4，并额外准备一个整包。在OTA 时先判断设备端版本和云端版本，若可相差一个版本则使用差分包，若跨版本则

使用整包。不管哪一种，都需要应用做出额外的判断。这一点需要主应用和云端服务器做好处理。

##### 3.15.4.5 校验问题

目前社区支持的rdiff 本身并不包含较好的校验机制，即应用一个版本不对的差分包，也能跑完升级流程。这样一旦出错，就会导致机器变砖。

一种可考虑的方式是搭配readback 使用，即在应用差分包，写入目标分区之后，将更新后的目标分区数据读出，校验其sha256 是否符合预期，校验成功才切换系

统，校验失败则报错。

可参考

```
target/allwinner/generic/swupdate/sw-description-ab-rdiff-sign
target/allwinner/generic/swupdate/sw-subimgs-ab-rdiff-sign.cfg
```

其中增加了readback 的处理。

具体的，sw-subimgs-xxx.cfg 中可以配置swota_copy_file_list，指定一些文件只拷贝到swupdate 目录，不打包到最终的OTA 包（swu 文件）中。因为在这个场

景下，我们需要原始的kernel, rootfs 等文件来计算sha256，但并不需要将其加入最终的OTA 包中。

##### 3.15.4.6 跟ubi 的配合问题

swupdate 官方目前没有支持rdiff 用于ubi 卷。

目前采用增加预处理脚本的方式来兼容，即在执行rdiff handler 之前，先调用脚本使用ubiupdatevol创建一个可用于升级目标ubi 卷的fifo。随后rdiff handler 即

可将此fifo 当作目标裸设备，无需特殊处理ubi 卷。在后处理脚本中，再通过填0 的方式，结束所有的ubiupdatevol。

具体可参考如下文件夹中的配置和脚本：

```
target/allwinner/r329-evb5/swupdate
```

前提仍然是配置好AB 系统，使用方式：

```
# 编译系统，得到要烧录的固件，记为V1

make && pack

# 生成OTA包，此OTA包的各个分区镜像跟V1固件的镜像一致

swupdate_pack_swu -ab
cp out/r329-evb5/swupdate/tina-r329-evb5-ab.swu tina-r329-evb5-ab_V1.swu

# 进行修改，编译出V2的系统

make && pack

# 生成OTA包，此OTA包的各个分区镜像跟V2固件的镜像一致

swupdate_pack_swu -ab
cp out/r329-evb5/swupdate/tina-r329-evb5-ab.swu tina-r329-evb5-ab_V2.swu

# 此时tina-r329-evb5-ab_V2.swu 可用于正常的OTA升级

# 生成各个镜像的差分文件

swupdate_make_delta ./tina-r329-evb5-ab_V1.swu ./tina-r329-evb5-ab_V2.swu

# 基于上一步得到的差分文件，生成差分OTA包

swupdate_pack_swu -ab-rdiff
```

