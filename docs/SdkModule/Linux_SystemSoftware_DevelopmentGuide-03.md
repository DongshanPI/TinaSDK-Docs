## 6 Tina编译打包

### 6.1 概述

### 6.2 编译系统

```
(1) source build/envsetup.sh
(2) lunch
(3) make [-jN]
(4) pack [-d]

其中，
步骤(1)建立编译环境，导出编译变量。
步骤(2)提示需要选择你想要编译的方案。
步骤(3)参数N为并行编译进程数量，依赖编译服务器CPU核心数，如 4 核PC，可"make -j4"
步骤(4)打包固件，-d参数使生成固件包串口信息转到tf卡座输出。
编译完成后系统镜像会打包在out/<board>/目录下
```

### 6.3 编译boot

| 命令   | 命令有效目录   | 作用             |
| :----- | :------------- | :--------------- |
| mboot  | tina下任意目录 | 编译boot0和uboot |
| mboot0 | tina下任意目录 | 编译boot0        |
| muboot | tina下任意目录 | 编译uboot        |

### 6.4 编译内核

| 命令    | 命令有效目录   | 作用     |
| :------ | :------------- | :------- |
| mkernel | tina下任意目录 | 编译内核 |

### 6.5 编译arisc

arisc是AW平台对cpus代码环境的代称，主要功能是负责休眠，关机等底层操作。它包含
cpus运行所需的驱动，库及工具链等，编译产生scp.bin，然后打包在Tina镜像中，有boot-
loader在启动时加载到cpus域运行。

Tina中提供了如下与arisc相关的跳转、编译命令


| 命令    | 命令有效目录   | 作用                   |
| :------ | :------------- | :--------------------- |
| carisc  | tina下任意目录 | 跳转到cpus代码工程目录 |
| mkarisc | tina下任意目录 | 编译cpus代码           |

在Tina中，编译arisc代码有两种方式，如下：

第一种方式，跳转到arisc代码路径下，使用make编译

arisc代码库具有独立的工具链和构建体系，因此对arisc代码的编译，也可以cd到对应的路径
下，执行make命令。但需注意，此时需要手动将生成的scp.bin文件拷贝到SDK的bin文件
路径下，覆盖对应的文件（一般路径是tina/device/config/chips/${平台名}/bin）。

操作如下，

```
step1
使用carisc或cd命令跳转到arisc代码环境下，一般为tina/lichee/arisc/ar100s。

step2
生成必要的配置。在arch/configs/下，存在许多defconfig默认配置文件，可根据AXP型号选择使用，例如make sun50iw10p1_axp803_defconfig。此操作将会在arisc根目录生成.config。

step3
修改默认配置，可使用make menuconfig。此操作会生成配置菜单，按需选择配置即可，该操作会修改.config。如需重新使用默认配置，重新执行step2即可。

step4
使用make命令编译

step5
将scp.bin拷贝到Tina的bin目录，即cbin可跳转的目录即可。如/tina/device/config/chips/r818/binstep6

若需清理工程，可使用make clean

step7
如需提交defconfig配置修改，可使用make savedefconfig命令。此命令会根据.config在arisc根目录生成 defconfig。将defconfig拷贝到arch/configs/下，覆盖对应配置文件即可。
```

注意：

一般来说，arisc执行的功能较为底层，多与休眠，关机等操作相关，对稳定性要求较高。我们不
建议客户自己修改任何相关配置和代码，如必须，请与我司联系或执行足够的稳定性测试。

第二种方式，使用mkarisc命令

如上述方法所述，Tina为了解决arisc编译后还需手动拷贝的问题，Tina提供了这个快捷命令
mkarisc。需要指出的是：目前只支持R818、MR813的arisc代码编译。


操作如下，较为简单

```
step1
配置tina环境，如soure build/envsetup.sh, lunch等操作。

step2
使用mkarisc编译即可或直接编译Tina,在编译Tina时，也会自动调用mkarisc命令。
```

注意：

使用mkarisc命令编译时，会自动读取board.dts获取AXP型号，然后使用对应的默认配置文
件编译。如board.dts中指定使用“x-powers,axp803”时，mkarsic命令会使用arisc工程环
境下的sun50iw10p1_axp803_defconfig配置文件编译。此时，若您需要修改该arisc配置，
只能先修改该配置文件后编译，除此外不接受其他配置修改的方式。

### 6.6 编译E907固件(V85x平台异构AMP核)

E907是V85x平台AMP CPU的代称，其主要功能是提供通用算力补充、辅助Linux实现快起
等。它包含E907运行所需的驱动、库及工具链等，编译产生riscv.fex，打包到Tina镜像中，
由bootloader在启动时加载到RISC-V核上运行。

在Tina SDK中，编译E907 RISCV代码如下：

E907 RISCV 代码库具有独立的工具链和构建体系，因此对 E907 固件的编译，也可以跳
转到对应的路径下，执行命令。但需注意，此时需要手动将生成的 melis30.elf 文件拷贝
到 SDK 的 bin 文件路径下，覆盖对应的文件（一般路径是 device/config/chips/${平台
名}/configs/default/riscv.fex）。

操作如下，

```
step1
使用cd命令跳转到e907代码环境下，一般为rtos-dev/lichee/melis-v3.0/source

step2
source melis-env.sh

step3
lunch v853-e907-ver1-board

step4
make menuconfig

step5
make -j

step6
如需减少存储空间，可裁剪调试信息，在melis source目录执行：riscv64-unknown-elf-strip ekernel/melis30.elf

step7
拷贝并重命名成riscv.fex替换掉tina/device/config/chips/v853/configs/default目录下的riscv.fex

cp ekernel/melis30.elf /home/xxx/tina/device/config/chips/v853/configs/default/riscv.fex

step8
Tina系统目录执行打包命令pack
```

### 6.7 重编应用

请确保进行过一次固件的编译，确保SDK基础已经编译，才能单独重编应用包。重编应用包应用场景一般为： 只修改了应用，不想重新烧写固件，只需要安装应用安装包即可 。请确保在编译前已加载tina环境：

```
$ source build/envsetup.sh
$ lunch
```

#### 6.7.1 方法一

当在应用包的目录（包括其子目录）中，可执行

```
$ mm [-B]
=> B参数则先clean此应用临时文件再编译
```

示例：假设软件包路径为：tina/package/utils/rwcheck，则：

```
$ cd tina/package/utils/rwcheck
$ mm -B
```

编译出应用安装包保存路径为：

```
tina/out/<方案>/packages/base
```

#### 6.7.2 方法二

当在tina的根目录，可执行：

```
$ make <应用包的路径>/clean，==>清空应用包临时文件
$ make <应用包的路径>/install，==>编译软件包
或者
$ make <应用包的路径>/{clean，install}，==>先清空临时文件再编译
```

示例：假设软件包的路径为：tina/package/utils/rwcheck，则：

```
$ cd tina
$ make package/utils/rwcheck/{clean，install}
```

### 6.8 其他命令


| 命令         | 命令有效目录                                               | 作用 |
| :----------- | :--------------------------------------------------------- | :--- |
| make         | tina根目录 编译整个sdk                                     |      |
| make         | menuconfig tina根目录 启动软件包配置界面                   |      |
| make         | kernel_menuconfig tina根目录 启动内核配置界面              |      |
| mkarisc      | tina下任意目录 编译cpus源码，根据AXP型号选择对应的默认配置 |      |
| printfconfig | tina下任意目录 打印当前SDK的配置                           |      |
| croot        | tina下任意目录 快速切换到tina根目录                        |      |
| cconfigs     | tina下任意目录 快速切换到方案的bsp配置目录                 |      |
| cdevice      | tina下任意目录 快速切换到方案配置目录                      |      |
| carisc       | tina下任意目录 快速切换到cpus代码目录                      |      |
| cgeneric     | tina下任意目录 快速切换到方案generic目录                   |      |
| cout         | tina下任意目录 快速切换到方案的输出目录                    |      |
| cboot        | tina下任意目录 快速切换到bootloader目录                    |      |
| cgrep        | tina下任意目录 在c／c++／h文件中查找字符串                 |      |
| minstall     | path/to/package/ tina根目录 编译并安装软件包               |      |
| mclean       | path/to/package/ tina根目录 clean软件包                    |      |
| mm [-B]      | 软件包目录 编译软件包，-B指编译前先clean                   |      |
| pack         | tina根目录 打包固件                                        |      |
| m            | tina下任意目录 make的快捷命令，编译整个sdk                 |      |
| p            | tina下任意目录 pack的快捷命令，打包固件                    |      |


## 7 Tina系统烧写

### 7.1 概述

本章节主要介绍如何将构建完成的镜像文件(image)烧写并运行在硬件设备上的流程。

SDK中的烧录工具不再更新，后续会删除，请优先选择从全志客户服务平台下载最新烧录工具。

windows工具均集成在APST中，下载安装APST即可，APST的工具均自带文档。

### 7.2 烧录工具

Tina提供的几种镜像烧写工具介绍如表所示，用户可以选择合适的烧写方式进行烧写。


| 工具          | 运行系统 | 描述                             |
| :------------ | :------- | :------------------------------- |
| PhoenixSuit   | windows  | 分分区升级及整个固件升级工具     |
| PhoenixCard   | windows  | 卡固件制作工具                   |
| PhoenixUSBpro | windows  | 量产升级工具，支持USB一拖 8 烧录 |
| LiveSuit      | ubuntu   | 固件升级工具                     |

对于ubuntu：

- 64bit主机使用LiveSuitV306_For_Linux64.zip。
- 32bit主机使用LiveSuitV306_For_Linux32.zip。

具体烧录工具和使用说明，请到全志客户服务平台下载。

### 7.3 进入烧录模式.

设备需进入烧录模式，以下几种情况会进入烧录模式：

1. BROM无法读取到boot0，例如新换的flash不包含数据，或者上电时短路flash阻断通信。
2. 在串口中按 2 进入烧录。即，在串口工具输出框中，按住键盘的’2’，不停输出字符’2’，上电启动。boot0检测到此字符，会跳到烧录模式。
3. 在uboot控制台，执行efex。
4. 在linux控制台，执行reboot efex。
5. adb可用的情况下，可使用adb shell reboot efex，或点击烧录工具上的“立即烧录”按钮。
6. 当完整配置[fel_key]下fel_key_max和fel_key_min时，按下键值在范围内按键，之后上电。
7. 当板子有FEL按键时，按住FEL按键上电。
8. 制作特殊的启动卡，从卡启动再进入烧录模式。