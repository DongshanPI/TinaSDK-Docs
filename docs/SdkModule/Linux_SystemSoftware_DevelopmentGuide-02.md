## 4 Tina开发环境

### 4.1 概述

嵌入式产品开发流程中，通常有两个关键的步骤，编译源码与烧写固件。源码编译需要先准备好

编译环境，而固件烧写则需要厂家提供专用烧写工具。本章主要讲述这如何搭建环境来实现Tinasdk的编译、烧写。

### 4.2 编译环境搭建.

一个典型的嵌入式开发环境包括本地开发主机和目标硬件板。

- 本地开发主机作为编译服务器，需要提供Linux操作环境，建立交叉编译环境，为软件开发提供代码更新下载，代码交叉编译服务。
- 本地开发主机通过串口或USB与目标硬件板连接，可将编译后的镜像文件烧写到目标硬件板，并调试系统或应用程序。

#### 4.2.1 开发主机配置.

Tina Linux SDK是在ubuntu14.04开发测试的，因此我们 **推荐使用Ubuntu 14.04** 主机环境进行源码编译，其他版本没有具体测试，可能需要对软件包做相应调整。

#### 4.2.2 软件包配置.

编译Tina Linux SDK 之前，需要先确定编译服务器安装了gcc，binutils，bzip2，flex，python，perl，make，ia32-libs，find，grep，diff，unzip，gawk，getopt，subver-sion，libz-dev，libc headers。

ubuntu可直接执行以下命令安装：

```
sudo apt-get install build-essential subversion git-core libncurses5-dev zlib1g-dev gawkflex quilt libssl-dev xsltproc libxml-parser-perl mercurial bzr ecj cvs unzip ia32-libslib32z1 lib32z1-dev lib32stdc++6 libstdc++6 -y
```

ubuntu 16.04及以上版本，执行下面命令安装软件包：

```
sudo apt-get install build-essential subversion git-core libncurses5-dev zlib1g-dev gawk flex quilt libssl-dev xsltproc libxml-parser-perl mercurial bzr ecj cvs unzip lib32z1 lib32z1-dev lib32stdc++ libstdc++6 -y libc6:i386 libstdc++6:i386 lib32ncurses5 lib32z
```



## 5 Tina系统获取

### 5.1 概述

### 5.2 SDK获取

Allwinner Tina Linux SDK通过全志代码服务器对外发布。客户需要向业务/技术支持窗口申请SDK下载权限。申请需同步提供SSH公钥进行服务器认证授权，获得授权后即可同步代码。

### 5.3 SDK结构

Tina Linux SDK主要由构建系统、配置工具、工具链、host工具包、目标设备应用程序、文档、脚本、linux内核、bootloader部分组成，下文按照目录顺序介绍相关的组成组件。

```
Tina-SDK/
    ├── build
    ├── config
    ├── Config.in
    ├── device
    ├── dl
    ├── docs
    ├── lichee
    ├── Makefile
    ├── out
    ├── package
    ├── prebuilt
    ├── rules.mk
    ├── scripts
    ├── target
    ├── tmp
    ├── toolchain
    └── tools
```

#### 5.3.1 build目录

build目录存放Tina Linux的构建系统文件，此目录结构下主要是一系列基于Makefile规格编
写的mk文件。主要的功能是：

1. 检测当前的编译环境是否满足Tina Linux的构建需求。
2. 生成host包编译规则。
3. 生成工具链的编译规则。
4. 生成target包的编译规则。
5. 生成linux kernel的编译规则。
6. 生成系统固件的生成规则。

```
build/
    ├── autotools.mk
    ├── aw-upgrade.mk
    ├── board.mk
    ├── cmake.mk
    ├── config.mk
    ├── debug.mk
    ├── depends.mk
    ├── device.mk
    ├── device_table.txt
    ├── download.mk
    ├── dumpvar.mk
    ├── envsetup.sh
```

#### 5.3.2 config目录

config目录主要存放Tina Linux中配置菜单的界面以及一些固定的配置项，该配置菜单基于内
核的mconf规格书写。

```
config/
    ├── Config-build.in
    ├── Config-devel.in
    ├── Config-images.in
    ├── Config-kernel.in
    └── top_config.in
```

#### 5.3.3 devices目录

devices目录用于存放方案的配置文件，包括内核配置，env配置，分区表配置，sys_config.fex，
board.dts等。

这些配置在旧版本上是保存于target目录下，新版本挪到device目录。

注意defconfig仍保存在target目录。

```
device/
    └── config
        ├── chips
        ├── common
        └── rootfs_tar
```

快捷跳转命令：cconfigs

#### 5.3.4 docs目录

docs目录主要存放用于开发的文档，以markdown格式书写。

本目录不再更新，请以全志客户服务平台系统文档为准。

```
docs/
    ├── build.md
    ├── config.md
    ├── init-scripts.md
    ├── Makefile
    ├── network.md
    ├── tina.md
    ├── wireless.md
    └── working.md
```

#### 5.3.5 lichee目录

lichee目录主要存放bootloader，内核，arisc，dsp等代码。

```
lichee/
    ├── bootloader
    │ ├── uboot_2011_sunxi_spl
    │ └── uboot_2014_sunxi_spl
    ├── brandy
    │ ├── u-boot-2011.09
    │ └── u-boot-2014.07
    ├── brandy-2.0
    │ ├── spl
    │ ├── tools
    │ └── u-boot-2018
    ├── linux-3.4
    ├── linux-3.10
    ├── linux-4.4
    ├── linux-4.9
    ├── arisc
```

快捷跳转命令：ckernel，cboot，cboot0，carisc。

#### 5.3.6 package目录

package目录存放target机器上的软件包源码和编译规则，目录按照目标软件包的功能进行分
类。

```
package/
    ├── allwinner
    ├── base-files
    ├── devel
    ├── dragonst
    ├── firmware
    ├── kernel
    ├── ......
    └── utils
```

#### 5.3.7 prebuilt目录

prebuild目录存放预编译交叉编译器，目录结构如下。

```
prebuilt/
    └── gcc
        └── linux-x86
            ├── aarch64
            │   ├── toolchain-sunxi-musl
            │   └── toolchain-sunxi-glibc
            ├── arm
            │   ├── toolchain-sunxi-arm9-glibc
            │   ├── toolchain-sunxi-arm9-musl
            │   ├── toolchain-sunxi-glibc
            │   ├── toolchain-sunxi-musl
            └── host
            └── host-toolchain.txt
```

#### 5.3.8 scripts目录

scripts目录用于存放一些构建编译相关的脚本。

```
scripts/
    ├── arm-magic.sh
    ├── brcmImage.pl
    ├── bundle-libraries.sh
    ├── checkpatch.pl
    ├── clang-gcc-wrapper
    ├── cleanfile
    ├── clean-package.sh
    ├── cleanpatch
    ├── ......
```

#### 5.3.9 target目录

target目录用于存放目标板相关的配置以及sdk和toolchain生成的规格。


```
target/
    ├── allwinner
    ├── Config.in
    ├── imagebuilder
    ├── Makefile
    ├── sdk
    └── toolchain
```

快捷跳转命令：cdevice。

#### 5.3.10 toolchain目录.

toolchain目录包含交叉工具链构建配置、规则。

```
toolchain/
    ├── binutils
    ├── Config.in
    ├── fortify-headers
    ├── gcc
    ├── gdb
    ├── glibc
    ├── info.mk
    ├── insight
    ├── kernel-headers
    ├── Makefile
    ├── musl
    └── wrapper
```

#### 5.3.11 tools目录.

tools目录用于存放host端工具的编译规则。

```
tools/
    ├── autoconf
    ├── automake
    ├── aw_tools
    ├── b43-tools
    ├── ......
```

#### 5.3.12 out目录.

out目录用于保存编译相关的临时文件和最终镜像文件，编译后自动生成此目录，例如编译方案
r328s2-perf1。

```
out/
    ├── r328s2-perf
    └── host
```

其中host目录用于存放host端的工具以及一些开发相关的文件。

r328s2-perf1目录为方案对应的目录。方案目录下的结构如下：

```
out/r328s2-perf
    ├── boot.img
    ├── compile_dir
    ├── image
    ├── md5sums
    ├── packages
    ├── r328s2-perf1-boot.img
    ├── r328s2-perf1-uImage
    ├── r328s2-perf1-zImage
    ├── rootfs.img
    ├── sha256sums
    ├── staging_dir
    └── tina_r328s2-perf1_uart0.img
```

- boot.img为最终烧写到系统boot分区的数据，可能为boot.img格式也可能为uImage格式。
- rootfs.img为最终烧写到系统rootfs分区的数据，该分区默认为squashfs格式。
- r328s2-perf1-zImage为内核的zImage格式镜像，用于进一步生成uImage。
- r328s2-perf1-uImage为内核的uImage格式镜像，若配置为uImage格式，则会拷贝成boot.img。
- r328s2-perf1-boot.img为内核的boot.img格式镜像，若配置为boot.img格式，则会拷贝成boot.img
- compile_dir为sdk编译host，target和toolchain的临时文件目录，存有各个软件包的源码。
- staging_dir为sdk编译过程中保存各个目录结果的目录。
- packages目录保存的是最终生成的ipk软件包。
- tina_r328s2-perf1_uart0.img为最终固件包(系统镜像)，串口信息通过串口输出
- 若使用pack -d，则生成的固件包为xxx_card0.img，串口信息转递到tf卡座输出。

快捷跳转命令：cout。

### 5.4 SDK更新

SDK更新分为两类：一类是以补丁的形式发布到一号通，发布后系统以邮件形式通知开发者；另一类是定期（半年或季度）的小版本迭代升级，将过往的补丁合入到SDK中，发布后以系统邮件通知开发者，可基于干净的SDK包通过repo sync命令更新。

### 5.5 问题反馈

Allwinner提供全志客户服务平台（https://open.allwinnertech.com），用来登记客户遇到的问题以及解决状态。方便双方追踪，使问题处理更加高效。后续SDK

问题、技术问题、技术咨询等都可以提交到此系统上，Allwinner技术服务会及时将问题进行分发、处理和跟踪。

注：系统登录帐号需要与Allwinner开通确认。