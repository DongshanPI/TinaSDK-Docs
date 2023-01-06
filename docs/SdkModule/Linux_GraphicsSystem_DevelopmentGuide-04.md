## 4 EFL

### 4.1 EFL说明

Enlightenment Foundation Libraries (EFL)驱动Enlightenment，它们也可以独立使用或者构建在其他库之上以提供有用的功能并创建强大的应用程序。

核心库EFL在速度和大小方面都比其GTK +和Qt等的效率更高，并且具有更小的内存占用量。

目前Tina中移植了EFL 1.20.6的核心库以及其组件，下表列出EFL相关包说明。

表4-1: EFL相关包说明

| 包名        | 说明                  |
| :---------- | :-------------------- |
| efl         | EFL功能函数库         |
| ephoto      | 依赖与EFL的相册应用   |
| terminology | 依赖于EFL的终端仿真器 |


下面是应用截图：


![图4-1: efl-on-wayland](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image10.jpg)


### 4.2 EFL配置

EFL可以使用Framebuffer或者Wayland显示图像，如果使用Wayland，需按照本文档第 8小节配置好Wayland。在Tina系统中，已经默认配置好了Framebuffer。执行如下命令配置EFL：

```
source build/envsetup.sh
lunch XXX平台名称
make menuconfig
```

```
Gui --->
    EFL --->
        -*- efl
        <*> ephoto
        <*> terminology
```

![图4-2:配置EFL选项](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image11.jpg)


efl是核心库，ephoto是一个相册应用，该应用可以选择板子里的图片进行浏览与幻灯片播放，terminology是一个终端仿真器，类似于ubuntu中的终端，进入到efl的配置界面，可以配置efl支持的功能。如下图所示：

![图4-3:配置EFL支持的功能选项](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image12.jpg)


主要关注以下几项配置：


表4-2: EFL配置说明

| 配置                                | 说明                                           |
| :---------------------------------- | :--------------------------------------------- |
| Enable raw Framebuffer access       | 使用framebuffer显示efl的界面                   |
| Enabel wayland display server       | 使用wayland显示efl的界面                       |
| Enabel sunxi-mali opengl es support | 使用opengl es                                  |
| Enable bidirectional text support   | 是否支持双向文本，从左到右，或从右到左显示文字 |
| Enable tslib for touchscreen events | 是否支持触摸                                   |


如果使用framebuffer显示efl的界面，则不需要再做其他配置什么，因为在Tina中默认开启framebuffer的，如果使用wayland，则需要参考本文档第 8 小节配置好wayland。

### 4.3 EFL运行

成功烧写固件后，如果使用Wayland的话，需要保证Weston已经运行，在小机端使用EFL，执行以下命令运行测试程序：

```
elementary_test
```

elementary_test是官方的小程序，包含efl中各种控件的使用示例。其他两个测试程序也是这样执行：

```
ephoto
```

```
terminology
```

还可以执行elementary_config去配置elf，可以配置界面渲染的模式，字体、控件的大小等等。

```
elementary_config
```

也可以手动指定渲染引擎，比如：

```
ECORE_EVAS_ENGINE=wayland_egl elementary_test
//或者
ELM_ACCEL=gl elementary_test
//或者
ELM_DISPLAY=wl elementary_test
```

如果想看efl的调试信息，可以在运行程序前加上：

```
EINA_LOG_LEVEL=4 elementary_test
```


如果是使用wayland显示efl界面的，并且想测试opengl es，则执行：


```
ELM_ACCEL=gl elementary_test
```


然后点击GLView Gears，GLView Many Gears，GLViewSimple查看结果，执行elemen-tary_test的时候界面可能是黑的，移动一下界面，滚动一下界面，或者最大化界面，就可以显示界面了，如果elementary_test没有响应，可以结束进程，再次尝试。使用kill -9 PID命令结束。


## 5 GTK+

### 5.1 GTK+说明

GTK+是用来创造图形界面的库，它可以运行在许多类UNIX系统，Windows和OSX。GTK+按照GNU LGPL许可证发布，这个许可证对程序来说相对宽松。GTK+有一个基于C的面向对象的灵活架构，它有对于许多其他语言的版本,包括C++, Objective-C, Guile/Scheme, Perl,Python, TOM, Ada95, Free Pascal和Eiffel。GTK+依赖于以下库：

1. GLib是一个多方面用途的库,不仅仅针对图形界面。GLib提供了有用的数据类型、宏、类型转换，字符串工具，文件工具，主循环抽象等等。
2. GObject是一个提供了类型系统、包括一个元类型的基础类型集合、信号系统的库。
3. GIO是一个包括文件、设备、声音、输入输出流、网络编程和DBus通信的现代的易于使用的VFS应用程序编程接口。
4. cairo Cairo是一个支持复杂设备输出的2D图形库。
5. Pango Pango是一个国际化正文布局库。它围绕一个表现正文段落的PangoLayout ob-ject。Pango提供GtkTextView、GtkLabel、GtkEntry和其他表现正文的引擎。
6. ATK是一个友好的工具箱。它提供了一个允许技术和图形用户界面交互的界面的集合。例如，一个屏幕阅读程序用ATK去发现界面上的文字并为盲人用户阅读。GTK＋部件已经被制作方便支持ATK框架。
7. GdkPixbuf是一个允许你从图像数据或图像文件创建GdkPixbuf(“pixel buffer”)的小的库。用一个GdkPixbuf与显示图像的GtkImage结合。
8. GDK是一个允许 GTK＋支持复杂图形系统的抽象层。GDK支持X11、wayland、Win-dows和OS X的图形系统工具。
9. GTK+是GTK+库本身包含的部件，确切的说是GUI零件，比如GtkButton或者Gtk-TextView。

更多GTK应用编程可参考：[示例](https://yjhenan.gitbooks.io/gtk-3-chinese-reference-manual/content/content/basic.html)

Gtk+(GIMP Tool Kit，GIMP工具包)是一个用于创造图形用户接口的图形库，下面是GIMP on GNU/Linux的截图：


![图5-1: GTK+GIMP运行截图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image13.jpg)

Tina系统移植了GTK+3的库及其组件，对应GTK包及依赖说明如下：

gtk+-3.22.10.tar.xz：Gtk+3对应源代码。

Pkgconf、gettext-full、atk、glib2、libcairo、pango、gdk-pixbuf、libepoxy、libxkb-common、libpixman、libinput、wayland、wayland-protocols、udev、libdrm、sunxi-mali：Openwrt系统Gtk+3依赖包名称；对应Makefile位于package/libs/libgtk3/。

### 5.2 GTK+配置

GTK仅基于R18系统平台验证过，其它平台暂未验证；默认GTK配置成wayland port，理论上GTK可以运行于所有支持Wayland的平台；其中R40使用Wayland+FBDEV作为显示后端，R18使用Wayland+DRM。

```
source build/envsetup.sh
lunch XXX平台名称
make menuconfig
```

以R18平台为例，主要配置项如下：

```
Gui --->
    Libs --->
        -*- libcairo --->
            [*] Enable cairo postscript support
            [*] Enable cairo pdf support
            [*] Enable cairo png support
            [ ] Enable script support
            [*] Enable cairo svg support
            [ ] Enable cairo tee support
            [ ] Enable cairo xml support
    Gtk --->
        <*> libgtk3 --->
            [*] Broadway GDK backend
            [*] Wayland GDK backend
            [*] Install libgtk3 demo program
```

因为Gtk+3依赖于Wayland，Wayland依赖于Weston合成器，配置时需要选上Weston和Wayland，需按照本文档第 8 小节配置好Wayland。

### 5.3 GTK+运行

成功烧写固件后，如果使用Wayland的话，需要保证Weston已经运行，然后在小机终端运行：

```
/usr/bin/gdk-pixbuf-query-loaders --update-cache
gdk-pixbuf-query-loaders > /usr/lib/gdk-pixbuf-2.0/2.10.0/loaders.cache
```

然后运行gtk3-demo：

```
gtk3-demo
```


![图5-2: GTK+Demo运行截图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image14.jpg)


### 5.4 GTK+示例

```
#include <gtk/gtk.h>

int main( int argc, char *argv[] ){
    GtkWidget *window;
    gtk_init (&argc, &argv);
    window = gtk_window_new (GTK_WINDOW_TOPLEVEL);
    gtk_widget_show(window);
    gtk_main ();
    return(0);
}
```

