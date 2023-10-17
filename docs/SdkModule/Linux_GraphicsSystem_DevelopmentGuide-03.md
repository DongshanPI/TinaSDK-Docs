## 3 QT

### 3.1 QT5配置

目前Tina中移植了QT5.10.1版本，Qt配置可以参考如下说明：

```
source build/envsetup.sh
lunch XXX平台名称
make menuconfig
```

```
Global build settings
    Binary stripping method (strip) ---> strip
Gui --->
    Qt --->
        -*- qt5-core
        <*> qt5-examples
```

这个将原本的库的制表符信息裁剪，来减小库的大小，Qt的某些库需要用到库的头信息strtab这个符号表，因此选择strip这种轻度的裁剪，留下strtab这个符号表，默认的选择是sstrip。

为了加快编译速度，提供了不同编译工具链预编译的QT包：

```
tina/dl/qt-everywhere-opensource-src-5.12.9-prebuilt_glibc_32bit.tar.gz
tina/dl/qt-everywhere-opensource-src-5.12.9-prebuilt_glibc_64bit.tar.gz
tina/dl/qt-everywhere-opensource-src-5.12.9-prebuilt_musl_32bit.tar.gz
tina/dl/qt-everywhere-opensource-src-5.12.9-prebuilt_musl_64bit.tar.gz
tina/dl/qt-everywhere-opensource-src-5.12.9.tar.xz
```

如果源码编译有问题，查看alsa-lib配置是否选上。

通过在make menuconfig中选择Qt–> qt5 use prebuilt来判断使用哪种编译方法。

```
make menuconfig
Libraries --->
    -*- alsa-lib
Gui --->
    Qt --->
        [*] qt5 use prebuilt
```

qt5 use prebuilt的值会在tina/package/qt/qt5/Makefile文件中使用。

```
ifeq ($(CONFIG_QT5_USE_PREBUILT),y)
ifeq ($(CONFIG_USE_GLIBC),y)
ifeq ($(TARGET_ARCH),aarch64)
    PKG_MD5SUM:=b96ae8d2d55983911b7bf46896d516bc
    PKG_SOURCE:=qt-everywhere-opensource-src-$(PKG_VERSION)-prebuilt_glibc_64bit.tar.gz

    PKG_BUILD_DIR=$(COMPILE_DIR)/qt-everywhere-opensource-src-$(PKG_VERSION)-
    prebuilt_glibc_64bit
else
    PKG_MD5SUM:=6fc40f289dd51ad2bf2403ad2da85bf
    PKG_SOURCE:=qt-everywhere-opensource-src-$(PKG_VERSION)-prebuilt_glibc_32bit.tar.gz
    PKG_BUILD_DIR=$(COMPILE_DIR)/qt-everywhere-opensource-src-$(PKG_VERSION)-
    prebuilt_glibc_32bit
endif
else ifeq ($(CONFIG_USE_MUSL),y)
    ifeq ($(TARGET_ARCH),aarch64)
    PKG_MD5SUM:=b7859b3fc75a28f10047cc63f8bb
    PKG_SOURCE:=qt-everywhere-opensource-src-$(PKG_VERSION)-prebuilt_musl_64bit.tar.gz
    PKG_BUILD_DIR=$(COMPILE_DIR)/qt-everywhere-opensource-src-$(PKG_VERSION)-
    prebuilt_musl_64bit
else
    PKG_MD5SUM:=9d1e2d3b5673976b3277142f047d2c
    PKG_SOURCE:=qt-everywhere-opensource-src-$(PKG_VERSION)-prebuilt_musl_32bit.tar.gz
    PKG_BUILD_DIR=$(COMPILE_DIR)/qt-everywhere-opensource-src-$(PKG_VERSION)-
    prebuilt_musl_32bit
endif
endif
else
    PKG_MD5SUM:=f177284b4d3d572aa46a34ac8f5a7f
    PKG_SOURCE:=qt-everywhere-opensource-src-$(PKG_VERSION).tar.xz
    PKG_BUILD_DIR=$(COMPILE_DIR)/qt-everywhere-opensource-src-$(PKG_VERSION)
endif
```

### 3.2 QT5 platforms选择

1. eglfs，在绘图的时候会使用GPU渲染UI，如果平台有GPU，尽量使用eglfs。
2. libqlinuxfb，linux标准的显示框架，会打开/dev/fb0节点进行绘图和显示。

平台插件的参数配置在package/qt/qt5/files/qt-env.sh 这个文件，如下所示，默认的plat-forms是eglfs，其中MALI_NOCLEAR环境变量的作用是调用eglInitialize函数时不清屏，不然在显示开机logo之后，会有一段黑屏时间，用户体验不好。

```
#!/bin/sh

export QT_QPA_PLATFORM=eglfs:size=800x
export QT_QPA_PLATFORM_PLUGIN_PATH=/usr/lib/qt5/plugins
export QT_QPA_FONTDIR=/usr/lib/fonts
export QT_QPA_GENERIC_PLUGINS=tslib
export QT_QPA_GENERIC_PLUGINS=evdevmouse:/dev/input/event
export QT_QPA_GENERIC_PLUGINS=evdevkeyboard:/dev/input/event
export MALI_NOCLEAR=
```

通常生成的平台插件在小机端的：

```
/usr/lib/qt5/plugins/platforms/libqeglfs.so
```

linuxfb平台插件动态库为libqlinuxfb.so。


如需更改为linuxfb，需要修改tina/package/qt/qt5/files/qt-env.sh文件内容，还需要make menuconfig选上qt5-drivers-linuxfb，如下所示：

```
Gui --->
    Qt --->
        -*- qt5-core
        <*> qt5-drivers-linuxfb
        <*> qt5-examples
```

linuxfb可以通过以下环境变量进行配置：

```
#!/bin/sh

export QT_QPA_PLATFORM=linuxfb:fb=/dev/fb0:size=800x480:mmSize=800x480:offset=0x0:tty=/dev/tty1
export QT_QPA_PLATFORM_PLUGIN_PATH=/usr/lib/qt5/plugins
export QT_QPA_FONTDIR=/usr/lib/fonts
export QT_QPA_GENERIC_PLUGINS=tslib
export QT_QPA_GENERIC_PLUGINS=evdevmouse:/dev/input/event1
export QT_QPA_GENERIC_PLUGINS=evdevkeyboard:/dev/input/event2
```

```
fb=/dev/fbN //指定帧缓冲设备;
size=<width>x<height> //指定屏幕大小，以像素为单位;
mmsize=<width>x<height> //物理宽度和高度;
offset=<width>x<height> //屏幕左上角像素偏移量
nographicsmodeswitch- //不要将虚拟终端切换到图形模式;
tty=/dev/ttyN //覆盖虚拟控制台，仅在nographicsmodeswitch未设置时使用;
```

eglfs可以通过以下环境变量进行配置:

```
export QT_QPA_EGLFS_WIDTH=800 //包含屏幕宽度（以像素为单位）
export QT_QPA_EGLFS_HEIGHT=480 //包含屏幕高度（以像素为单位）
export QT_QPA_EGLFS_FB=/dev/fb0 //覆盖帧缓冲设备，默认是/dev/fb0
export QT_QPA_EGLFS_DEPTH=32 //覆盖屏幕的颜色深度，默认值为 32
```


### 3.3 QT5鼠标触摸屏配置

Qt中使用鼠标，需要启动udev，将鼠标设备标记为输入设备，然后Qt的libinput来处理输入事件，才能够识别鼠标。设置udev为自启动，默认已经将udev设置为自启动。

屏幕为触摸屏，因此需要make menuconfig选上Qt触摸模块qt5-drivers-touchscreen，如下所示：

```
Gui --->
    Qt --->
        -*- qt5-core
        <*> qt5-drivers-touchscreen
        <*> qt5-examples
```

触摸屏驱动在小机端的：


```
/usr/lib/qt5/plugins/generic/libqtslibplugin.so
```

如果触摸没效果，执行如下环境变量：

```
export QT_QPA_GENERIC_PLUGINS=tslib
```

### 3.4 QT5示例运行

成功烧写固件后，在小机端使用QT，如果使用的是电阻触摸屏，需要进行触摸屏校准，请参考本文档2.3.1小节。

QT的应用示例在小机端的如下路径：

```
usr/share/qt5/examples /这是QT自带的测试应用/
```

```
make menuconfig
Gui --->
    Qt --->
        <*> qt-easing.
        <*> qt-textures
        <*> qt-washing-machine

//这是新添加的三个QT应用,如果运行此应用有问题，请参照package/qt/qt-washing-machine/src/doc文档
```


运行qt应用需要指定插件平台，目前QT支持的插件平台有eglfs或者linuxfb，运行示例如下所示：

```
./application -platform eglfs
./application -platform linuxfb
```

或者先执行下面的命令，导入QT的环境变量，再执行程序。

```
./etc/qt-env.sh
```

### 3.5 QT5问题锦集

#### 3.5.1 strip

运行QT的应用程序会出现如下问题，需要将libqeglfs.so库重新推到/usr/lib/qt5/plugins/platforms路径下。这里如果多个插件平台库都出现这个问题，可能是由于，Tina系统中将编译生成的库进行裁剪，使其更小，Qt在进行动态加载的时候，需要找到库头信息中的strtab制表符，因此在make menuconfig中选择轻度裁剪模式-strip。

![图3-1: QT5问题锦集strip](https://photos.100ask.net/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image7.jpg)

#### 3.5.2 eglfs

出现下面错误，申请不上native window有可能是缺少libqeglfs-mali-integration.so这个库，需要将其adb push到小机端的/usr/lib/qt5/plugins/egldeviceintegrations路径下。


![图3-2: QT5问题锦集eglfs](https://photos.100ask.net/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image8.jpg)


#### 3.5.3 runtime

出现下面错误，传入环境变量：

```
export QT_QPA_EGLFS_INTEGRATION=none
export XDG_RUNTIME_DIR=/dev/shm
```


![图3-3: QT5问题锦集runtime](https://photos.100ask.net/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image9.jpg)



#### 3.5.4 触摸使用不了.

出现这个原因有可能是下面步骤导致：

1. 触摸屏没有适配校准。

参考《2.3.1触摸屏校准》/etc/ts_calibrate进行校准。


2. qt没有配置触摸屏的节点。

参考《3.2 QT5 platforms选择》。

3. 如果还是不行单独执行。

```
export QT_QPA_GENERIC_PLUGINS=tslib
```

