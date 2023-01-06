## 1 概述

编写目的：本文档作为Allwinner Tina Linux系统平台开发指南，旨在帮助软件开发工程师、技术支持工程师快速上手，熟悉Tina Linux系统的开发及调试流程。

适用范围：Tina Linux v3.5及以上版本。


## 2 Tina系统资料

### 2.1 概述

Tina SDK发布的文档旨在帮助开发者快速上手开发及调试，文档中涉及的内容并不能涵盖所有的开发知识和问题。文档列表也正在不断更新。


Tina SDK提供丰富的文档资料，包括硬件参考设计文档、Flash等基础器件支持列表、量产工具使用说明、软件开发与制定介绍文档、芯片研发手册等资料。

### 2.2 文档列表

请以全志科技全志客户服务平台最新列表为准。

## 3 Tina系统概述

### 3.1 概述

Tina Linux系统是基于openwrt-14.07的版本的软件开发包，包含了Linux系统开发用到的内核源码、驱动、工具、系统中间件与应用程序包。openwrt是一个开源的嵌入式Linux系统自动构建框架，是由Makefile脚本和Kconfig配置文件构成的。使得用户可以通过menuconfig配置，编译出一个完整的可以直接烧写到机器上运行的Linux系统软件。

### 3.2 系统框图


![图3-1: Tina Linux系统框图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_System_software_development_Guide-3-1.jpg)

Tina系统软件框图如图所示，从下至上分为Kernel && Driver、Libraries、System Ser-vices、Applications四个层次。各层次内容如下：


1. Kernel&&Driver主要提供Linux Kernel的标准实现。Tina平台的Linux Kernel 采用Linux3.4、linux3.10、linux4.4、linux4.9等内核(不同硬件平台可能使用不同内核版本)。提供安全性，内存管理，进程管理，网络协议栈等基础支持；主要是通过Linux内核管理设备硬件资源，如CPU调度、缓存、内存、I/O等。
2. Libraries层对应一般嵌入式系统，相当于中间件层次。包含了各种系统基础库，及第三方开源程序库支持，对应用层提供API接口，系统定制者和应用开发者可以基于Libraries层的API开发新的应用。
3. System Services层对应系统服务层，包含系统启动管理、配置管理、热插拔管理、存储管理、多媒体中间件等。
4. Applications层主要是实现具体的产品功能及交互逻辑，需要一些系统基础库及第三方程序库支持，开发者可以开发实现自己的应用程序，提供系统各种能力给到最终用户。

### 3.3 开发流程

Tina Linux 系统是基于 Linux Kernel，针对多种不同产品形态开发的 SDK。可以基于本
SDK，有效地实现系统定制和应用移植开发。


![图3-2: Tina Linux系统开发流程](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_System_software_development_Guide-3-2.jpg)

如上图所示，开发者可以遵循上述开发流程，在本地快速构建Tina Linux系统的开发环境和编译
代码。下面将简单介绍下该流程：

1. 检查系统需求：在下载代码和编译前，需确保本地的开发设备能够满足需求，包括机器的硬件能力，软件系统，工具链等。目前Tina Linux系统只支持Ubuntu操作系统环境下编译，并仅提供Linux环境下的工具链支持，其他如MacOS，Windows等系统暂不支持。
2. 搭建编译环境：开发机器需要安装的各种软件包和工具，详见开发环境章节，获知TinaLinux已经验证过的操作系统版本，编译时依赖的库文件等。
3. 选择设备：在编译源码前，开发者需要先导出预定义环境变量，然后根据开发者根据的需求，选择对应的硬件板型，详见编译章节。
4. 系统定制：开发者可以根据使用的硬件板子、产品定义，定制U-Boot、Kernel及Open-wrt，请参考后续章节中相关开发指南和配置的描述。
5. 编译与打包：完成设备选择、系统定制之后执行编译命令，包括整体或模块编译以及编译清理等工作，进一步的，将生成的boot/内核二进制文件、根文件系统、按照一定格式打包成固件。详见编译打包章节。
6. 烧录并运行：继生成镜像文件后，将介绍如何烧录镜像并运行在硬件设备，进一步内容详见系统烧写章节。