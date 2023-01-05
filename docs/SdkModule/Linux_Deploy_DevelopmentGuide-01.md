## 1 概述

编写目的: 介绍TinaLinux的配置文件，配置方法。



## 2 menuconfig

Tina采用Kconfig机制，对SDK和内核进行配置。

具体用法，可以参考Kconfig机制的相关介绍。

### 2.1 tina menuconfig.

Tina Linux SDK的根目录下，执行make menuconfig命令可进入Tina Linux的配置界面。

对于具体软件包：

```
<*> (按y)： 表示该软件包将包含在固件中。
<M>(按m)： 表示该软件将会被编译，但不会包含在固件中。
< >(按n)： 表示该软件不会被编译。
```

配置文件保存在：

```
target/allwinner/${borad}/defconfig
```

make menuconfig修改后的文件，会保存回上述配置文件。

### 2.2 kernel menuconfig

Tina Linux SDK的根目录下，执行make kernel_menuconfig命令可进入对应内核的配置界
面。

每个方案有对应的内核版本，如3.4，3.10，4.4，4.9等，记为x.y。

对于Tina3.5.0及之前版本，配置后文件会保存在：

```
target/allwinner/${borad}/config-x.y
```

对于Tina3.5.1及之后版本，配置后文件会保存在：

```
device/config/chips/${chip}/configs/${borad}/linux
```

