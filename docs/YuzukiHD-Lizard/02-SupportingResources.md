
## 软件资源简述
### BWW提供的SDK
* 百度网盘获取地址 链接：https://pan.baidu.com/s/1f1K82iA0kNibuMlZu34i0g?pwd=9rds  提取码：9rds 

#### Buildoot For Dongshan NezhaSTU D1-H

**有视频 有文档 有手册**

- 此套构建系统基于全志RISCV-64 Linux D1-H 芯片，适配了buildroot 2022lts主线版本，兼容了百问网的项目课程以及相关组件，真正做到了低耦合，高可用，使用不同的buildroot external tree规格，讲不同的项目 不同的组件分别管理，来实现更容易上手 也更容易学习理解。
  * 仓库地址  https://github.com/DongshanPI/buildroot_dongshannezhastu
  
    

#### elinuxCore 最小东山哪吒STU D1 嵌入式Linux系统

**有视频 有文档 有手册**

* 此套系统是专门用于了解D1-H RISC-V Linux组件以及各个部分构成的源码示例，适合喜欢不依赖任何构建工具的前提下自行研究。

  * https://github.com/DongshanPI/eLinuxCore_dongshannezhastu

    

#### 东山哪吒STU自动构建Debian ubuntu发行版系统

**有文档 有手册**

* 此套系统是基于 `debootstrap `配合GUN/linux社区版本实现的自动构建系统，主要是一些列脚本等。此套系统并非是一步编译所有，需要分多次执行，因为脚本框架等问题，所以不是特别完善。目前构建系统使用的社区大佬 https://github.com/smaeul 提供的源码，硬件支持上并不是特别完整。但是最小系统没有问题。
  * https://github.com/DongshanPI/NezhaSTU-ReleaseLinux



###  全志原厂Tina-SDK V2.0

**有文档 有手册**

* Tina Linux是全志科技基于Linux内核开发的针对智能硬件类产品的嵌入式软件系统。Tina Linux基于openwrt-14.07 版本的软件开发包，包含了 Linux 系统开发用到的内核源码、驱动、工具、系统中间件与应用程序包。openwrt 是知名的开源嵌入式 Linux 系统自动构建框架，是由 Makefile 脚本和 Kconfig 配置文件构成的。使得用户可以通过 menuconfig配置，编译出一个完整的可以直接烧写到机器上运行的 Linux 系统软件。

  * 在线文档 https://d1.docs.aw-ol.com/

    

### Linux社区版Buildroot-LTS

**有简单文档**

* https://bbs.aw-ol.com/topic/1208/buildroot-2022
