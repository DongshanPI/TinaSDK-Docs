## 5 Tina upgrade app 介绍(建议改用swupdate)

### 5.1 功能简介

以前Tina 只有misc-upgrade，为了特殊需求，创建了本脚本。现在建议直接使用swupdate。

有些客户，需要单独更新应用程序。一种解决方式是，将应用程序单独放到一个分区中，并在启动时挂载该分区。为了保证更新过程掉电重启，仍有可用的应用程

序，可设置两个应用分区，并配合环境变量等选择挂载。

### 5.2 应用源码

需修改应用源码的Makefile，将应用涉及文件，全部安装到/mnt/app 路径。

如果应用是二进制形式，则请放到：

```
target/allwinner/xxx/busybox-init-base-files/mnt/app
```



### 5.3 menuconfig 设置

选中：

```
make menuconfig ---> Target Images ---> [*] Separate /mnt/app from rootfs
```

使得/mnt/app 目录， 从rootfs 中分离出来。打包的时候， 会被制作成一个单独的文件app.fex。
选中：

```
make menuconfig ---> Allwinner ---> <*> tina-app-upgrade
```

选中后，可使用

```
/sbin/aw_upgrade_dual_app.sh
```

进行OTA。

### 5.4 分区设置

增加两个分区，名字固定为app 和app_sub，downloadfile 固定为app.fex，size 根据实际情况调整。

```
[partition]
name = app
size = 51200
downloadfile = "app.fex"
user_type = 0x8000
[partition]
name = app_sub
size = 51200
downloadfile = "app.fex"
user_type = 0x8000
```



### 5.5 env 设置

对于tina3.5.0 及之前版本，位于：

```
target/allwinner/${board}/configs/env-xxx.cfg
```

对于tina3.5.1 及之后版本，位于：

```
device/config/chips/${chip}/configs/${board}/linux/env-xxx.cfg
```

增加配置：

```
appAB=A
#set applimit=0 to disable appcount check
applimit=0
appcount=0
```

其中appAB 指定要启动时要挂载哪个分区：

```
appAB=A 挂载/dev/by-name/app到/mnt/app
appAB=B 挂载/dev/by-name/app到/mnt/app
```

applimit 和appcount 用于支持应用的自动回退功能。

### 5.6 自动回退

applimit=0 时，没有任何作用。

applimit 非0 时，会在每次启动，递增appcount 值，并检测appcount 是否大于applimit，若大于，则切换挂载另一个app 分区。

例如，当前挂载/dev/by-name/app，设置applimit=2，appcount=0。

则每次启动，appcount 加一，两次启动后，appcount ＝２，再次重启，appcount+1=3>applimit，超过限制，自动改成挂载/dev/by-name/app_sub。

这个功能主要是要解决，更新了一个有问题的应用，导致无法正常启动应用的问题。例如：

当前处于app，OTA 更新了一个有问题的新应用到app_sub 分区，重启，重启后无法正常启动app_sub 中的新应用。则applimit 次重启后，会自动改回使用app 

分区中，旧的可用的应用。
使用此功能，需要应用在正常启动后，主动使用
fw_setenv appcount 0
清空计数值，避免累积到applimit 切换分区。

### 5.7 OTA 步骤

#### 5.7.1 生成OTA 包

tina 目录下，执行

```
make_ota_package_for_dual_app
```

生成OTA 包

```
out/xxx/ota_dual_app/app_ota.tar
```



#### 5.7.2 下载OTA 包

通过网络或ADB 等方式，将app_ota.tar 放到小机端/mnt/UDISK/app_ota.tar。

#### 5.7.3 执行OTA

执行脚本：

```
aw_upgrade_dual_app.sh
```

### 5.8 调试

生成OTA 包的函数，位于：

```
build/envsetup.sh
function make_ota_package_for_dual_app()
```

可从生成的tar 文件中，将app.fex 解出来：

```
tar -xf out/xxx/ota_dual_app/app_ota.tar
```

解压得到的app.fex，是一个ext4 文件系统，可在linux 主机，或推送到小机端，挂载，查看文件系统中的内容：

```
mkdir app
mount app.fex app
ls aaa
```

