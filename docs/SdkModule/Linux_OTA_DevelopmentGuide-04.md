## 4 Tina misc-upgrade 介绍(建议改用swupdate)

### 4.1 方案选择

misc-upgrade 只支持recovery 系统方案。

由于在实际应用中，存储操作系统和持久文件的存储介质（如nand、emmc、spinor）大小各异，在OTA 中需要单独在存储介质上开辟recovery 分区，以防备在

更新中意外断电，造成系统更新失败无法重启的问题。

所以在选择OTA 方案时一定要考虑到recovery 分区大小对分区规划的影响，避免在小容量时recovery 分区太大导致分区规划难题。

综上因素，我们在Tina 上针对大容量和小容量设计了不同的方案。

#### 4.1.1 小容量方案

小容量介质一般指存储介质容量小于32M（一般为spi nor）。

在命令行中进入Tina 根目录，执行命令进入配置主界面：

```
source build/envsetup.sh (见详注1)
lunch (见详注2)
make menuconfig (见详注3)
```

详注：
1 加载环境变量及tina提供的命令
2 输入编号，选择方案
3 进入内核配置主界面(对一个shell而言，前两个命令只需要执行一次)

配置路径：

```
Target Image
└─> *** Image Options ***
[*] For storage less than 32M， enable this when using ota
```

选中该配置项后，rootfs 的/usr 会被分拆出一部分生成usr.squashfs(usr.img)，并建立软链接usr.fex。通过配置分区表将usr.fex 放在extend 分区，开机后自动

挂载到usr 目录。这种设置的目的是可与recovery 镜像（boot_initramfs）复用该分区，以此起到节省存储空间的作用。

因此小容量方案中，并无单独的recovery 分区，而是在OTA 升级时与extend 分区复用。

为了达到启动后自动挂载extend 分区的效果。需要进行配置。

对于busybox-init，默认在pseudo_init 中会尝试挂载usr。如果发现没有正常挂载，请检查/pseudo_init 中的mount_usr。

对于procd-init，可在

```
target/allwinner/<board>/base-files/etc/config/fstab
```

配置文件中，增加：

```
config 'mount'
option target '/usr'
option device '/dev/by-name/extend'
option options 'ro,sync'
option enabled '1'
```



#### 4.1.2 大容量方案

大容量介质一般指存储介质容量大于32M（一般是nand、emmc）。

对于大容量方案，不建议选中小容量方案中所述配置项，即不需要usr.img 和extend 分区，而只需要添加recovery 分区，这样在OTA 升级时会省去很多麻烦。

#### 4.1.3 misc-upgrade

不管是小容量还是大容量，都要在make 之前选中应用包misc-upgrade：

```
make menuconfig
└─> Allwinner
　└─> <*> misc-upgrade...... read and write the misc partition
```

misc-upgrade 包主要功能是从指定服务器下载更新镜像到本地，然后升级相应分区，期间会向misc 分区写入升级阶段的标志，出现意外无法重启时uboot 或内核

（如果能够启动）可以根据misc 分区的状态标志进行下一步的决策。

#### 4.1.4 OTA 的升级流程

##### 4.1.4.1 基本步骤

Tina3.0 及之前版本处理流程：

```
1. 备份busybox等资源到ram中， 使得OTA过程不依赖rootfs
2. 设置开始OTA的标志，upgrade_pre
3. 将recovery系统写入recovery分区
4. 设置标志boot-recovery
5. 更新boot和rootfs分区
6. 设置标志upgrade_post
7. 对于小容量方案，更新usr分区
8. 设置标志upgrade_end
9. 重启，重启后为新系统
```

最新版本处理流程：

```
1. 设置开始OTA的标志，upgrade_pre
2. 将recovery系统写入recovery分区
3. 设置标志boot-recovery
4. 主动重启，重启后进入recovery系统
5. 更新boot和rootfs分区
6. 设置标志upgrade_boot0
7. 如果存在boot0镜像，则更新boot0
8. 设置标志upgrade_uboot
9. 如果存在uboot镜像，则更新uboot
10. 设置标志upgrade_post
11. 对于小容量方案，更新usr分区
12. 设置标志upgrade_end
13. 重启，重启后为新系统
```

##### 4.1.4.2 中途掉电

OTA 中途掉电后，下次启动会根据标志，继续完成OTA。

当设置upgrade_pre 标志之后掉电，重启后仍是旧系统，从该标志之后的步骤开始执行。

当设置boot-recovery 标志之后掉电，重启时，uboot 判断到这个标志，并直接启动recovery系统，启动后，从该标志之后的步骤开始执行。

当设置upgrade_post 标志之后掉电，重启时后为新系统，从该标志之后的步骤开始执行。

### 4.2 分区处理

#### 4.2.1 分区定义

<center>表4-1: misc-upgrade 升级分区说明表</center>

| 升级分区       |                                                              |
| -------------- | ------------------------------------------------------------ |
| boot 分区      | 基础系统镜像分区，即/lib，/bin，/etc，/sbin 等非/usr，非挂载其他分<br/>区的路径，wifi 支持环境，alsa 支持环境、OTA 环境。 |
| extend 分区    | 扩展系统镜像分区，即/usr 应用分区，仅小容量方案有。          |
| recovery       | 分区存放恢复系统镜像，仅大容量方案有。                       |
| private 分区   | 存储SN 号分区。                                              |
| misc 分区      | 系统状态、刷机状态分区。                                     |
| UDISK 分区     | 用户数据分区，一般挂载在/mnt/UDISK。                         |
| overlayfs 分区 | 存储overlayfs 覆盖数据。                                     |

#### 4.2.2 分区大小配置

##### 4.2.2.1 配置boot 分区大小

boot 分区镜像的大小依赖内核配置，必须小于等于sys_partition.fex/sys_partition_nor.fex中定义的boot 分区大小。

boot 分区镜像大小设定：

```
make menuconfig
└─> Target Images
└─> *** Image Options ***
(4) Boot (SD Card) filesystem partition size (in MB)
```

boot 分区大小设定：

```
[partition]
name =boot
size =8192
downloadfile ="boot.fex"
user_type =0x8000
```

对于大容量方案，需要在sys_partition.fex 中添加recovery 分区：

```
recovery 分区说明
如果启用了OTA 升级，需要去掉下面recovery 分区的注释以提供恢复分区存储恢复系统镜像，
默认以boot_initramfs.img 作为recovery.fex：
[partition]
name =recovery
size =32768
downloadfile ="recovery.fex"
user_type =0x8000
```

其中recovery.fex 是生成的boot_initramfs.img 软链接而成。

##### 4.2.2.2 rootfs 分区的大小

rootfs 分区不需要通过make menuconfig 去设定，直接根据镜像大小修改分区文件即可。

###### 4.2.2.2.1 小容量对于一些小容量flash 的方案(如16M)，需在/bin 下存放联网逻辑程序、

版本控制程序、下载镜像程序、播报语音程序以及语音文件(这些文件在编译时应该install 到/bin 或者/lib 下)，可以在固件编译完后，查看rootfs.img 的大小再决

定sys_partition.fex 中rootfs 分区的大小。

###### 4.2.2.2.2 大容量对于大容量flash 的方案(如128M 以上，或者有足够的flash 空间存相

关镜像)，不需要小容量中那些OTA 额外的程序，直接查看rootfs.img 的大小设定分区文件即可。

##### 4.2.2.3 extend 分区的大小

extend 分区用于小容量方案下boot_initramfs.img 和usr.img 的复用，其大小需要考虑多个方面：
1 . 编译后usr.img 的大小
2 . make_ota_image 后initramfs 镜像的大小
因此，extend 分区略大于boot_initramfs.img 和usr.img 两个的最大值，并把extend 分区的大小值设置为initramfs 镜像的大小：

```
make menuconfig
└─> Target Images
└─> *** Image Options ***
(8) Boot-Recovery initramfs filesystem partition size (in MB)
```



##### 4.2.2.4 其他分区

如private 、misc 等使用默认的大小即可。

##### 4.2.2.5 UDISK 分区

sys_partition.fex 中不指定UDISK 分区大小，则剩下的空间全部自动分配进入UDISK 分区。

需要注意的是，因为OTA 过程会在里面写一些中间文件，所以一定要留取一定空间给UDISK 分区，至少可以格式化挂载，而小容量flash 的方案，也要保证有256K~512K 的空间。

##### 4.2.2.6 其他说明

在OTA 升级过程中并不能修改上述分区的大小，因此应在满足分区大小条件限制( 如3.2.1-3.2.3) 的情况下尽可能留有足够的空余空间，以满足OTA 升级添加内容

的需求。

修改分区大小时，尽量对齐到所用存储介质的块大小。

对于大容量，使用recovery 分区，对应的，其env 定义中，boot_recovery 命令需定义为从recovery 分区启动系统。对于小容量，使用extend 分区，对应的，其

env 定义中，boot_recovery 命令需定义为从extend 分区启动系统。

### 4.3 misc-upgrade 升级

#### 4.3.1 misc-upgrade 构成

misc-upgrade 是Tina 下的一个应用，其路径为：

```
tina/package/allwinner//misc-upgrade
```

Makefile 符合tina 安装包的书写规范。

misc-upgrade 目录结构如下：

```
├── aw_fstab.init #小容量方案挂载用。
├── aw_reboot.sh
├── aw_upgrade_autorun.init #自启动脚本。
├── aw_upgrade_image.sh
├── aw_upgrade_lite.sh
├── aw_upgrade_log.sh
├── aw_upgrade_normal.sh #大容量方案。
├── aw_upgrade_plus.sh
├── aw_upgrade_process.sh #小容量方案。
├── aw_upgrade_utils.sh
├── aw_upgrade_vendor_default.sh
├── Makefile
├── readme.txt
└── tools #编译出write_misc和read_misc应用。
    ├── Makefile
    ├── misc_message.c
    ├── misc_message.h
    ├── read_misc.c
    └── write_misc.c
```

#### 4.3.2 OTA 镜像包编译

在命令行中进入Tina 根目录，执行命令进入配置主界面(环境配置)：

```
source build/envsetup.sh ( 详见1 )
lunch ( 详见2 )
make ota_menuconfig (可选) ( 详见3 )
详注：
1 加载环境变量及tina 提供的命令。
2 输入编号，选择方案。
3 进入OTA config 配置界面。直接保存退出即可。
此步骤可选，目的是解决开发过程中defconfig_ota 未能及时更新而可能引发的编译问题。
```

编译命令：

```
make_ota_image ( 详见1 )
make_ota_image --force ( 详见2 )
详注：
1 在新版本代码已经成功编译出烧录固件的环境的基础上，打包OTA 镜像。
2 重新编译新版本代码，然后再打包OTA 镜像。
注：不同介质使用的boot0/uboot镜像不同，
make_ota_image需要sys_config.fex中的storage_type配置明确指出介质类型，
才能取得正确的boot0.img和uboot.img。
具体可直接查看build/envsetup.sh中make_ota_image的实现。
```

执行make_ota_image 之前，可通过make ota_menuconfig 对ota 的恢复系统镜像

boot_initramfs.img 进行配置，可根据实际情况，配置ota 恢复系统包含的功能。

如以下配置支持ramdisk 并选用xz 压缩cpio ：

```
make ota_menuconfig
    └─> target Images
        └─> [*] ramdisk
            └─> --- ramdisk
            	└─> Compression (xz) --->
```

OTA 镜像包路径为：tina/out/xxx/ota/

目录结构如下：

```
├── boot0_sys #boot0升级文件。
│ ├── boot0.img
│ └── boot0.img.md5
├── boot0_sys.tar.gz #boot0升级文件的压缩包。
├── package_sys #某定制版OTA脚本使用，不在本文档描述范围。
│ ├── boot0.img
│ ├── boot0.img.md5
│ ├── boot.img
│ ├── boot.img.md5
│ ├── ota.tar
│ ├── recovery.img
│ ├── recovery.img.md5
│ ├── rootfs.img
│ ├── rootfs.img.md5
│ ├── uboot.img
│ └── uboot.img.md5
├── ramdisk_sys #recovery系统升级文件。
│ ├── boot_initramfs.img
│ └── boot_initramfs.img.md5
├── ramdisk_sys.tar.gz #recovery系统升级压缩包。
├── target_sys #kernel和rootfs升级文件。
│ ├── boot.img
│ ├── boot.img.md5
│ ├── rootfs.img
│ └── rootfs.img.md5
├── target_sys.tar.gz #kernel和rootfs升级压缩包。
├── uboot_sys #uboot升级文件。
│ ├── uboot.img
│ └── uboot.img.md5
└── uboot_sys.tar.gz #uboot升级压缩包。
```

其中” .img.md5 “是” .img “的校验值文件。

升级脚本不带-n 参数，则使用压缩包，带-n 则直接使用不压缩的升级文件。

#### 4.3.3 小机端OTA 升级命令

必选参数：-f -p 二选一。

aw_upgrade_process.sh -f 升级完整系统(内核分区、rootfs 分区、extend 分区)。

aw_upgrade_process.sh -p 升级应用分区( extend 分区)。

可选参数：-l , -d -u, -n。

注：对于大容量，用aw_upgrade_normal.sh 替代aw_upgrade_process.sh ，且一般用-f参数而不用-p。

##### 4.3.3.1 大容量flash 方案

可以使用本地镜像测试，如主程序下载校验好镜像后，存在/mnt/UDISK/misc-upgrade 中，调用如下命令。

OTA 升级期间掉电，重启后升级程序也能自动完成烧写，不需要依赖联网重新下载镜像。

###### 4.3.3.1.1 -l 选项-l < 路径>

如：

```
aw_upgrade_normal.sh -f -l /mnt/UDISK/misc-upgrade
```

( 注：mnt 前的根目录“/” 最好带上，misc-upgrade 后不要带“/”) ( -l 参数，默认使用压缩镜像包)。

不使用-n 参数的方案需要部署上服务器上的镜像是：

• recovery 系统和主系统（必选）：ramdisk_sys.tar.gz 、target_sys.tar.gz。
• uboot 和boot0（可选， 调用脚本时镜像不存在则自动跳过）：uboot_sys.tar.gz，boot0_sys.tar.gz。

###### 4.3.3.1.2 -n 选项如：

```
aw_upgrade_normal.sh -f -n -l /mnt/UDISK/misc-upgrade
```

一般情况下不使用-n 选项，而是下载ramdisk_sys.tar.gz 、target_sys.tar.gz 及可选的uboot_sys.tar.gz、boot0_sys.tar.gz。

但对于某些ram 较小的平台，如R6 spinand 的情况，flash 的容量足够放下大容量的OTA 包，但升级过程可能因为ram 不足而失败。

对于这种情况，可以选择下载未压缩的数据，即上述xxx.tar.gz 解压出来的所有内容。

将多个分区数据文件下载到/mnt/UDISK/misc-upgrade 中，调用上述命令。

使用-n 参数的方案需要部署上服务器上的镜像是：
• recovery 系统和主系统（必选）：boot_initramfs.img 、boot.img 、rootfs.img 及其对应的md5 后缀的校验文件。
• uboot 和boot0（可选，调用脚本时镜像不存在则自动跳过）：uboot.img、boot0.img 及其对应的md5 后缀的校验文件。

##### 4.3.3.2 小容量flash 方案

###### 4.3.3.2.1 -l 选项原始的设计是用于网络升级，不能使用-l 参数，升级区间出错重启后，需

要联网下载程序获取镜像。
后续考虑存在小容量设备插SD 卡升级的情况，支持了-l 参数，命令类似上述的大容量方案，不赘述。

###### 4.3.3.2.2 -d -u -n 选项

```
-d arg1 -u arg2
```

同时使用，-d 参数为可以ping 通的OTA 服务器的地址、-u 参数为镜像的下载地址。

```
-n
```

-n 一些小ddr 的方案(如剩余可使用内存在20m 以下的方案)，可以使用这个参数，shell 会直接下载不压缩的4 个img 文件，这样子设备下载后不需要tar 解压，减少内存使用。

如：

```
aw_upgrade_process -f -d 192.168.1.140 -u http://192.168.1.140/
```

升级shell 会先ping -d 参数( ping 192.168.1.140 )，ping 通过后，会根据升级命令和系统当前场景请求下载：
无-n 参数：

```
http://192.168.1.140/ramdisk_sys.tar.gz
http://192.168.1.140/target_sys.tar.gz
http://192.168.1.140/usr_sys.tar.gz
```

有-n 参数：

```
http://192.168.1.140/boot_initramfs.img
http://192.168.1.140/boot.img
http://192.168.1.140/rootfs.img
http://192.168.1.140/usr.img
```

使用-n 参数的方案需要部署上服务器上的镜像是：boot_initramfs.img 、boot.img 、rootfs.img 、usr.img 及其对应的md5 后缀的校验文件。

不使用-n 参数的方案需要部署上服务器上的镜像是：ramdisk_sys.tar.gz 、target_sys.tar.gz、usr_sys.tar.gz。

注：若由misc-upgrade 自行下载镜像，当前实现暂不支持可选的boot0/uboot 镜像，即不会自动从服务器下载升级boot0/uboot。

### 4.4 脚本接口说明

对于小容量flash 的方案，没有空间存储镜像，相关镜像只会存在ram 中，断电就会丢失。假如升级过程断电，需要在重启后重新下载镜像。

aw_upgrade_vendor.sh 设计为各个厂家实现的钩子，SDK 上只是个demo 可以随意修改。

#### 4.4.1 实现联网逻辑

```
check_network_vendor(){
    return 0 联网成功(如：可以ping 通OTA 镜像服务器)。
    return 1 联网失败。
}
```

#### 4.4.2 请求下载目标镜像

$1 : ramdisk_sys.tar.gz $2 : /tmp

```
download_image_vendor(){

# $1 image name $2 DIR $@ others

rm -rf $2/$1
echo "wget $ADDR/$1"
wget $ADDR/$1 -P $2
}
```



#### 4.4.3 开始烧写分区状态

```
aw_upgrade_process.sh -p 主动升级应用分区的模式下，返回0 开始写分区1 不写分区。
aw_upgrade_process.sh -f 不理会这个返回值。
upgrade_start_vendor(){

# $1 mode: upgrade_pre,boot-recovery,upgrade_post

#return 0 -> start upgrade; 1 -> no upgrade
#reutrn value only work in nornal mode
#nornal mode: $NORMAL_MODE
echo upgrade_start_vendor $1
return 0
}
```



#### 4.4.4 写分区完成

```
upgrade_finish_vendor(){
#set version or others
reboot -f
}
```



#### 4.4.5 -f (-n) 调用顺序

```
1 . check_network_vendor ->
2 . upgrade_start_vendor ->
3 . download_image_vendor (ramdisk_sys.tar.gz, -n 为boot_initramfs.img)->
4 .内部烧写、清除镜像逻辑(不让已经使用镜像占用内存) ->
5 . download_image_vendor(target_sys.tar.gz, -n 为boot.img rootfs.img) ->
6 .内部烧写、清除镜像逻辑(不让已经使用镜像占用内存) ->
7 . download_image_vendor(usr_sys.tar.gz, -n 为usr.img) ->
8 .内部烧写、清除镜像逻辑(不让已经使用镜像占用内存) ->
9 . upgrade_finish_vendor
```

#### 4.4.6 -p 调用顺序

```
1 . check_network_vendor ->
2 . download_image_vendor (usr_sys.tar.gz) ->
3 . upgrade_start_vendor ->
4 . 检测返回值，烧写->
5 . upgrade_finish_vendor
```



### 4.5 相关系统状态读写

相关的信息存储在misc 分区，OTA 升级不会清除这个分区(重新烧写镜像会擦除)。

读：

```
read_misc [command] [status] [version]
```

其中：

```
command 表示升级的系统状态( shell 脚本处理使用)。
status 自由使用，表示用户自定义状态。
version 自由使用，表示用户自定义状态。
```

写：

```
write_misc [ -c command ] [ -s status ] [ -v version ]
```

其中：

```
-c 不能随意修改，只能由aw-upgrade shell 修改。
-s -v 自定义使用。
```



### 4.6 OTA 配置

#### 4.6.1 recovery 系统生成

一般来说，默认方案目录下会有一份defconfig_ota 配置文件，该文件用于编译生成一个带ramdisk 的kernel ，即boot_initramfs.img ，作为OTA 期间烧写到

recovery 分区或extend 分区的备份系统。

用户可以基于原有的defconfig_ota 进行配置，也可以自行拷贝defconfig 为新的defconfig_ota ，然后进行适当修改和配置。

执行make ota_menuconfig 可进行OTA 配置。

选上recovery 后缀，避免编译recovery 系统时，影响到主系统。

```
make ota_menuconfig
    ---> Target Images
        ---> [*] customize image name
            ---> Boot Image(kernel) name suffix (boot_recovery.img/boot_initramfs_recovery.img)
            ---> Rootfs Image name suffix (rootfs_recovery.img)
```

如果方案进行了较多的修改，建议删除原本的defconfig_ota ，然后拷贝defconfig 为新的defconfig_ota ，再进行配置。

如上文所述，必须选上misc-upgrade 包，以及ramdisk 选项，保留wifi 功能。其他选项可以关掉，以对生成的boot_initramfs.img 进行裁剪。

#### 4.6.2 recovery 系统切换

##### 4.6.2.1 切换方式1：基于misc 分区

对于存在misc 分区的方案，可以在misc 分区中设置boot-recovery 标志。启动时，uboot 会检测misc 分区，如果为boot-recovery，则执行env 中配置的

boot_recovery 命令启动内核。否则执行boot_normal 命令启动内核。

因此，env 中需要正确配置boot_normal 和boot_recovery。类似：

```
boot_normal=sunxi_flash read 40007800 boot;bootm 40007800
boot_recovery=sunxi_flash read 40007800 extend;bootm 40007800
```

对于大容量方案boot_recovery 配置为recovey 分区启动，对于小容量方案，boot_recovery配置为从extend 分区启动。

OTA 过程需要正确设置misc 分区的值。

##### 4.6.2.2 切换方式2：基于env 分区

可以去除misc 分区，直接基于env 进行系统切换。OTA 过程需修改env 分区中的值。

一般是将env 的boot_normal 配置为从$boot_partition 分区启动，即将要使用的分区抽离成一个变量。

类似：

```
boot_partition=boot
boot_normal=sunxi_flash read 40007800 ${boot_partition};bootm 40007800
```

env 中默认boot_partition=boot。需要切换到recovery 系统时，在用户空间直接设置env，设置boot_partition=recovery。需要切换回主系统时，设置

boot_recovery=boot。

OTA 过程需要正确设置env 中的值。

### 4.7 对overlayfs 的处理

Tina 默认使用overlayfs ，则用户对原rootfs 的修改会记录在rootfs_data 分区中。而OTA更新的是rootfs 分区，默认不会修改rootfs 分区。则用户对rootfs 文件

的增加，删除，修改操作都会保留，假如原本的rootfs 中有文件A ，用户将其修改为A1 ，而OTA 更新将该文件修改为B ，则最终看到的仍然是A1 。这是由

overlayfs 的特性决定的，上层目录的文件会屏蔽下层目录。

如果希望OTA 之后，以OTA 更新的文件为准，移除所有用户的修改。则可以在OTA 之后，重新格式化rootfs_data 分区。

### 4.8 对busybox-init 的处理

#### 4.8.1 upgrade_etc 标志（不再使用）

有些平台使用了busybox-init ，以R6 为例子：
R6 方案将启动方式从procd 修改为busybox-init ，不再使用procd 和overlayfs ，由此带来一系列变化。

OTA 脚本通过1 号进程的名字来判断启动方式。并根据结果。在后续脚本中做区别处理。

对于busybox-init ，在第一次启动之后，会将etc 的文件拷贝到rootfs_data 分区中，并在后续挂载该分区作为etc 目录。OTA 的过程不会更新rootfs_data 分区。

为了支持更新etc 目录，增加了一个系统状态，upgrade_etc ，并在原本设置upgrade_end 的地方，改为设置成upgrade_etc 。而busybox-init 的启动脚本会判断

此标志，如果启动是标志为upgrade_etc，则进行etc 分区文件的更新，更新后设置系统状态为upgrade_end 。

主系统指定了启动脚本init=/pseudo_init. 对于OTA 使用的recovery 系统也需要指定启动脚本，若所使用的方案未配置，可自行修改env ，在cmdline 中传递

rdinit=/pseudo_init 进行指定。若文件系统中已经有rdinit 文件(例如是一个到pesudo_init 的软链接），则可使用rdinit=/rdinit。

#### 4.8.2 etc_need_update 文件

rootfs_data 有两种可能的用法

1.保存rootfs /etc 的副本，挂载到/etc，以支持/etc 可写

2.用作overlayfs，以支持全目录可写，类似procd 方案
对于早期版本，仅支持1。对于当前最新版本，/pseudo_init 的最上方可以配置，当配置MOUNT_ETC=1, MOUNT_OVERLAY=0 时，即为1，跟早期版本相同，此处只讨论上述的1。
其行为如下:

1. 烧录后第一次启动，rootfs_data 分区为空，则/pesudo_init 会格式化rootfs_data，将/etc文件拷贝一份到rootfs_data 的文件系统，将rootfs_data 挂载为/etc，对用户来说看到的/etc 就是rootfs_data 中的
2. 第二次启动，rootfs_data 中存在有效的文件系统，直接挂载为/etc，对用户来说看到的/etc就是rootfs_data 中的
3. 此时若直接更新rootfs 分区中的rootfs，重启，用户看到的/etc 仍为rootfs_data 中的，不会改变
4. 若创建/etc/etc_need_update 文件(OTA 之后应该创建它)，则下次启动/pseudo_init 脚本会将rootfs 分区中的/etc 进行一次同步
5. 同步/etc 的具体行为:
   5.1. 将rootfs_data 挂载到/mnt
   5.2. 将rootfs 分区的/etc 拷贝到/tmp/etc
   5.3. 将rootfs_data 中不希望被覆盖的文件，拷贝到/tmp/etc。例如wpa_supplocant.conf。如果有其他特殊文件不希望被OTA 覆盖，可参考/pseudo_init 中的wpa_supplicant 的处理方法，修改/pseudo_init。
   5.4. 将/tmp/etc 的内容，拷贝回/mnt/etc(即rootfs_data 分区)
   5.5. 创建标志文件etc_complete，删除标志文件etc_need_update。若在此步骤完成前掉电，则下次启动会从步骤1 重新同步etc
   5.6 将rootfs_data 的挂载点从/mnt 改为/etc, 继续启动

### 4.9 常见问题

#### 4.9.1 OTA 时出现SQUASHFS ERROR

此问题是由于Tina3.0 及更早版本的OTA ，在更新rootfs 分区时，只是将busybox 等备份到ram 中，rootfs，分区尚处于挂载状态，此时如果有进程访问rootfs ，

则会出现错误。

解决方式：

1. OTA 之前，关闭其他进程，避免有进程访问rootfs 。如果确有进程需要保留，将其依赖的资源提前备份到ram 中(相关代码可参考OTA 脚本中的prepare_env 函数)。
2. 参照最新版本的做法，在更新完recovery 分区后，主动reboot ，进入recovery 系统，在recovery 系统中更新boot 和rootfs 分区。
3. 参考原生openwrt 的做法，在内存中构建ram 文件系统后，执行切换根文件系统的操作，再进行更新。

#### 4.9.2 编译OTA 包之后，正常编译出错

出现编译问题，可能是由于defconfig 被替换了。

请检查下target 仓库中方案对应目录的defconfig 和defconfig_ota 的状况。当前的OTA 包编译过程，实质上是备份defconfig ，使用defconfig_ota 替换defconfig 

，执行make ，最终还原defconfig。

如果此过程中断，则defconfig 未被还原，会导致下次正常编译出问题。

解决方式：

还原方案目录下的defconfig 文件。

建议make menuconfig 和make ota_menuconfig 之后，及时在target 仓库下将修改提交入git 仓库中，避免修改丢失，方便在必要时进行还原。

（此问题为Tina3.0 之前的问题，最新版本没有此问题。）

#### 4.9.3 是否可更新boot0/uboot

misc-upgrade 原设计流程中未包含此功能

当前版本中，本地升级支持可选地更新boot0/uboot(存在镜像则更新，不存在则跳过)，网络升级暂未支持。

具体升级功能由ota-burnboot 软件包支持，细节请参考相关章节。

判断是否支持的方式：搜索脚本中是否有地方调用了ota-burnboot0 和ota-burnuboot。