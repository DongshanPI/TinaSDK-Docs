## 10 Tina系统定制开发

### 10.1 Tina procd-init与busybox-init切换.

tina默认为procd-init：

```
make menuconfig进行配置：
1.System init 选择procd-init

2.以下一步步选中
Base system --->
    <*>block-mount
    <*>busybox................................ Core utilities for embedded Linux --->
        Init Utilities --->
            [ ] init 此处不选
        Coreutils --->
            [*] head
        Miscellaneous Utilities --->
            [*] strings
    <*> uci
    <*> logd

3.env.cfg修改
init=/sbin/init
```

busybox-init自启动方式配置如下：

```
make menuconfig进行配置：
1.System init 选择busy-init

2.以下一步步选中
Base system --->
    <*>busybox................................ Core utilities for embedded Linux --->
    Init Utilities --->
    [* ] init 此处选上

3.env.cfg修改
    init=/init
    rdinit=/rdinit
```

## 10.2应用移植

在Tina Linux SDK中一个软件包目录下通常包含如下两个目录和一个文件：

```
package/<分类>/<软件包名>/Makefile
package/<分类>/<软件包名>/patches/ [可选]
package/<分类>/<软件包名>/files/ [可选]
其中，
patches 保存补丁文件，在编译前会自动给源码打上所有补丁
files 保存软件包的源码，在编译时会对应源码覆盖源码中的源文件
Makefile 编译规则文件，
```

### 10.2.1 Makefile范例

该Makefile的功能是**软件源码的准备，编译和安装的过程，提供给Tina Linux识别和管理软件包的接口**，软件的编译逻辑是由软件自身的Makefile决定，理论上和该Makefile(该Makefile只执行make命令和相关参数)无实质关系。

![图10-2-1](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_System_software_development_Guide-10-2-1.jpg)

详注：

```
1.如果是开源软件，软件包版本建议与下载软件包的版本一致。
2.以PKG开头的变量主要告诉编译系统去哪里下载软件包。
3.md5sum用于校验下载下来的软件包是否正确，如果正确，在编译该软件的时候，就会在PKG_BUILD_DIR下找到该软件包的源码。
4.Package/<name>: <name>用来指定该Package的名字，该名字会在配置系统中显示。
5.使用依赖包的名字<name>来指定依赖关系，如果是扩展包，前面添加一个”+”号，如果是内核版本依赖使用@LINUX_2_<minor version>。
6.如果该值为 1 ，该包将不会出现在配置菜单中，但会作为固定编译，可选。
7.在开源软件中一般用来生成Makefile，其中参数可以通过CONFIGURE_VARS来传递。
8.在开源软件中一般相当于执行make，其中有两个参数可以使用:MAKE_FLAGS和MAKE_VARS。
9.内置的几个关键字如下：
    INSTALL_DIR相当于install -d m0755
    INSTALL_BIN相当于install -m0755
    INSTALL_DATA相当于install -m0644
    INSTALL_CONF相当于install -m0600
10.该Makefile的所有define部分都是为该宏的参数做的定义.上层Makefile通过调用此宏进行编译。
```

### 10.2.2 自启动设置

在Tina Linux中支持两种格式的初始化脚本，一种是busybox式或者sysV式的初始化脚本，
一种是procd式的初始化脚本。一般我们把**由初始化脚本启动的应用叫做服务**。

初始化脚本以shell脚本的编程语言组织，shell脚本作为基础知识在此不展开说明。一般情况
下，初始化脚本源码保存在软件的files目录，且后缀为“.init”，例如：

```
tina/package/system/fstools/files/fstab.init
```

在Makefile的install中把初始化脚本安装到小机端的/etc/init.d中，例如：

```
define Package/block-mount/install
$(INSTALL_DIR) $(1)/etc/init.d/
$(INSTALL_BIN) ./files/fstab.init $(1)/etc/init.d/fstab
endef
```

#### 10.2.2.1调用自启动脚本.

- 手动调用方式在启动的时候会有太多的log，且log信息已被logd守护进程收集，不利于我们
  调试初始化脚本，此时可通过小机端的命令行手动调用的形式来调试，例如：

```
root@TinaLinux: /# /etc/init.d/fstab start
```

#### 10.2.2.2 sysV格式脚本

sysV式的初始化脚本保存在小机端的/etc/init.d/目录下，实现开机自启动。下例以最小内容的初
始化脚本作示例讲解，核心是实现start/stop函数：


```
#!/bin/sh /etc/rc.common
# Example script
# Copyright (C) 2007 OpenWrt.org
START=10
STOP=15
DEPEND=xxxx
start() {
#commands to launch application
}
stop() {
#commands to kill application
}
注意：
START=10， 指明开机启动优先级(序列) [数值越小， 越先启动]，取值范围0-99。
STOP=15， 指明关机停止优先级(序列) [数值越小， 越先关闭]，取值范围0-99。
DEPEND=xxxx， 指明初始化脚本会并行执行，通过此项配置确保执行的依赖。
```

| 名称    | 属性     | 功能         |
| :------ | :------- | :----------- |
| start   | 必须实现 | 启动一个服务 |
| stop    | 必须实现 | 停止一个服务 |
| reload  | 可选实现 | 重启一个服务 |
| enable  | 可选实现 | 重新加载服务 |
| disable | 可选实现 | 禁用服务     |


在shell里面可以使用如下的命令来操作相关的服务。

```
$ root@TinaLinux:/# /etc/init.d/exmple restart|start|stop|reload|enable|disable
```

#### 10.2.2.3 procd格式脚本

以下例的初始化脚本作示例讲解，主要是实现函数start_service：

```
#!/bin/sh /etc/rc.common
USE_PROCD=1
PROG=xxxx
START=10
STOP=15
DEPEND=xxxx
start_service() {
procd_open_instance
procd_set_param command $PROG -f
......
procd_close_instance
}
```

详细的介绍可以参考：https://wiki.openwrt.org/inbox/procd-init-scripts。

## 10.3应用调试

新添加的软件默认配置为不使能，此时需要手动配置使能软件包。通过在tina的根目录执行
make menuconfig进入软件包的配置界面：

![图10-1:应用配置主界面](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_System_software_development_Guide-10-1.jpg)

软件包的所在路径与软件包的Makefile中的定义有关，以fstools为例，在Makefile中定义
为：

```
define Package/fstools
    SECTION:=base
    CATEGORY:=Base system
    DEPENDS:=+ubox +USE_GLIBC:librt +NAND_SUPPORT:ubi-utils
    TITLE:=OpenWrt filesystem tools
    MENU:=1
endef
```

此时，只需要在menuconfig界面中进入Basy system即可找到fstools的软件包。

![图10-2:软件包所在界面](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_System_software_development_Guide-10-2.jpg)

前缀符号含义：

```
[*] 或<*> ： 编译进入SDK
[ ] 或< > ： 不包含
```

支持操作：

```
Y或y：选择包含
N或n：取消选择
```

## 10.4应用编译

详见重编应用章节。

## 10.5应用安装

1. 获取安装包

安装包一般位于目录：


```
tina/out/<方案>/packages/base
```

安装包命名格式为：

```
<应用名>_<应用版本>-<应用释放版本>_sunxi.ipk
```

2. 安装应用包

通过adb推送安装包到小机：

```
$ adb push <安装包路径> <推送到小机路径>
```

安装应用包：

```
$ opkg install <安装包路径>
```

## 10.6分区与挂载

- 升级分区

| 分区         | 功能                                                         |
| :----------- | :----------------------------------------------------------- |
| boot分区     | 存内核镜像                                                   |
| rootfs分区   | 基础系统镜像分区，包含/lib，/bin，/etc等                     |
| recovery分区 | 存放恢复系统镜像，仅大容量方案有，详见OTA文档                |
| extend分区   | 存放恢复系统镜像及rootfs的usr部分，仅小容量方案有，详见OTA文档 |


- 不升级分区

| 分区          | 功能                               |
| :------------ | :--------------------------------- |
| private分区   | 存储SN号分区                       |
| misc分区      | 系统状态、刷机状态分区             |
| UDISK分区     | 用户数据分区，一般挂载在/mnt/UDISK |
| overlayfs分区 | 存储overlayfs覆盖数据              |

- 默认挂载点

| 分区                         | 挂载点      | 备注                  |
| :--------------------------- | :---------- | :-------------------- |
| /dev/by-name/boot            | /boot       |                       |
| /dev/by-name/boot-res        | /boot-res   |                       |
| /dev/by-name/UDISK           | /mnt/UDISK  | 用户数据分区          |
| /dev/mmcblk0或/dev/mmcblk0p1 | /mnt/SDCARD | Tf卡挂载点            |
| /dev/by-name/rootfs_data     | /overlay    | 存储overlayfs覆盖数据 |