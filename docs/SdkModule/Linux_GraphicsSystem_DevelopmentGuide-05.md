## 6 WebKitGtk

### 6.1 WebkitGtk说明.

Tina系统移植了WebKitGtk的库及其组件，对应WebKitGtk包及依赖说明如下：

webkitgtk-2.18.6.tar.xz、midori_0.5.11_all_.tar.bz2、package/libs/webkitgtk、pack-age/utils/midori：WebKitGtk和Midori浏览器对应源代码及Makefile。

ruby/host、flex/host、bison/host、gperf/host、enchant、harfbuzz、icu、libjpeg、libgtk3、libsecret、libsoup、libxml2、libxslt、libsqlite3、libegl、libgles、libwebp libgles、lcms2、libtasn1、gstreamer1、gst1-libav 、gst1-plugins-bas、gst1-plugins-good、gst1-plugins-ugly、gst1-plugins-bad：Openwrt系统WebkitGtk依赖包名称。

下面是WebKitGtk的截图：

![图6-1: WebKitGtk运行截图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image15.jpg)


### 6.2 WebKitGtk配置

WebKitGtk仅基于R18系统平台验证过，其它平台暂未验证；默认WebKitGtk配置成way-land port，R18使用Wayland+DRM。

```
source build/envsetup.sh
lunch XXX平台名称
make menuconfig
```

```
Gui --->
    Gtk --->
        <*> webkitgtk
        <*> midori
```

因为WebKitGtk依赖于Gtk+3和Wayland，Wayland依赖于Weston合成器，配置时需要选上Gtk+3、Weston和Wayland，需按照本文档第 5 和 8 小节配置好Gtk+3和Wayland。

### 6.3 WebKitGtk运行

成功烧写固件后，如果使用Wayland的话，需要保证Weston已经运行，然后在小机终端运行：

```
/usr/bin/gdk-pixbuf-query-loaders --update-cache
gdk-pixbuf-query-loaders > /usr/lib/gdk-pixbuf-2.0/2.10.0/loaders.cache
```

然后运行Midori或minibrowser：

```
midori
```

或者：

```
minibrowser
```

### 6.4 WebKitGtk问题锦集

报错：

```
error: Package `gee-0.8' not found in specified Vala API directories or GObject-Introspection GIR directories
```

原因是主机环境安装了Vala这个工具，但是需要的是tina中编译出的这个工具，卸载主机Vala工具即可。


## 7 DirectFB

### 7.1 DirectFB说明

DirectFB（直接帧缓冲区）是在Linux帧缓冲区（fbdev）抽象层之上实现的一组图形API。

- 最大化硬件加速的实用程序。

- 支持高级图形操作，例如多种alpha混合模式。
- 没有内核修改没有库依赖项，libc除外。
- 符合MHP规范的要求。

目前在Tina中，还没有对接过GPU。

目前Tina中移植了DirectFB的核心库以及其Demo，下表列出DirectFB相关包说明：



表7-1: DirectFB相关包说明



| 包名              | 说明           |
| :---------------- | :------------- |
| directfb          | directfb核心库 |
| directfb-examples | directfb demo  |


### 7.2 DirectFB配置

```
source build/envsetup.sh
lunch XXX平台名称
make menuconfig
```

```
Gui --->
    Directfb --->
        -*- directfb
        <*> directfb-examples
```

### 7.3 DirectFB运行

在小机端可以执行一些df_开头的测试用例，比如df_andi，df_dok：


df_andi


![图7-1: DirectFB运行截图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image16.jpg)