## 8 Wayland

### 8.1 Wayland说明.

Wayland是一套display server(Wayland compositor)与client间的通信协议，而Weston是Wayland compositor的参考实现，定位于在Linux上替换X图形系统。

目前Tina中移植了Wayland的核心库以及其组件，下表列出Wayland相关包说明：


![表8-1: Wayland相关包说明]


| 包名              | 作用                                                        |
| :---------------- | :---------------------------------------------------------- |
| glmark2           | 使用Wayland作为运行后端的GPU测试程序，或者使用FBDEV进行显示 |
| wayland           | 编译Weston需要用到的主机端工具                              |
| wayland-protocols | Wayland协议，相当于插件                                     |
| weston            | 核心库                                                      |

### 8.2 Wayland配置.

#### 8.2.1 menuconfig.

Wayland目前可以在R18与R40上运行，其他平台暂未测试，其中在R40只能使用FBDEV
作为运行后端，在R18上可以使用DRM与FBDEV。执行如下命令进行配置：

```
source build/envsetup.sh
lunch XXX平台名称
make menuconfig
```

```
Gui --->
    Wayland --->
        < > glmark2
            [ ] Enabel fbdev support
            [*] Enabel wayland support
        -*- wayland
        -*- wayland-protocols
        <*> weston --->
            [ ] Enabel dbus support
            [ ] Enabel weston-launch linux pam support
            [*] Enabel opengl es support
            [ ] Enabel fbdev compositor support
            [*] Enabel drm compositor support
            [ ] Enabel lcms supports support
            [ ] Enabel junit xml support
            [ ] Enabel demo clients install
```

如下图所示：


![图8-1: Wayland选项](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image17.jpg)


glmark2是使用GPU的跑分测试程序，可以在R18上使用DRM作为Wayland后端的时候使用，除此之外还可以使用FBDEV进行显示并测试GPU性能。wayland，wayland-protocols在编译weston的时候用到，进入到weston的配置界面，可以配置weston支持的功能。如下图所示：


![图8-2: Weston选项](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image18.jpg)


主要关注以下几项配置：


表8-2: Wayland配置说明

| 选项                            | 说明                                                         |
| :------------------------------ | :----------------------------------------------------------- |
| Enabel opengl es support        | 只能在使用DRM 作为weston 后端的时候选上，支持opengl es GPU 加速 |
| Enabel fbdev compositor support | 使用framebuffer 作为显示引擎                                 |
| Enabel drm compositor support   | 使用DRM 作为显示引擎                                         |
| Enabel demo clients install     | 编译出Wayland 测试Demo                                       |

如果使用FBDEV，需要选上kmod-mali-utgard-km与kmod-sunxi-disp：

```
Kernel modules --->
    Video Support --->
        <*> kmod-mali-utgard-km
        <*> kmod-sunxi-disp
        < > kmod-sunxi-drm
```

如果使用DRM与GPU加速，需要选上kmod-mali-utgard-km与kmod-sunxi-drm：


```
Kernel modules --->
    Video Support --->
        <*> kmod-mali-utgard-km
        < > kmod-sunxi-disp
        <*> kmod-sunxi-drm
```

选择DRM与GPU加速的如下图所示：


![图8-3: Kernel modules配置](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image19.jpg)


注意FBDEV与DRM是互斥的，不能同时选择。

如果需要Weston开机自启动，需要修改 tina/package/wayland/weston目录下的Make-file，把$(CP) ./weston $(1)/etc/init.d的注释去除即可。

如果选择了DRM作为显示引擎，还可以把DRM的测试Demo给选上，选项如下：

```
Gui --->
    Libs --->
        libdrm --->
            [*] install libdrm test programs
            [ ] Enable support for vc4's API
```

weston依赖的库cairo也可以配置，其中Enable cairo pdf support与Enable cairo png support是必须选择上的，不然编译的时候会报错，如果编译GTK+的话，需要多选择一些，参考本文档第5.2小节。

```
Gui --->
    Libs --->
        -*- libcairo --->
            [ ] Enable cairo postscript support
            [*] Enable cairo pdf support
            [*] Enable cairo png support
            [ ] Enable script support
            [ ] Enable cairo svg support
            [ ] Enable cairo tee support
            [ ] Enable cairo xml support
```

![图8-4: Cairo选项](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image20.jpg)

#### 8.2.2 kernel_menuconfig

##### 8.2.2.1 FBDEV

如果menuconfig选择的是使用FBDEV作为后端，已经选择kmod-sunxi-disp，R18平台会自动配置下面的选项，不用再执行这一小节的步骤，其他平台暂未实现自动配置，下面的主要是记录使用FBDEV作为后端，需要的内核配置。其他平台内核已经默认配置使用FBDEV。

执行以下命令，以R18的为例：

```
make kernel_menuconfig
```

选上 Framebuffer Console Support(sunxi)、DISP Driver Support(sunxi-disp2)、Framebuffer Console support与Transform Driver Support(sunxi)：

```
Device Drivers --->
    Graphics support --->
        Frame buffer Devices --->
            <*> Support for frame buffer devices --->
            Video support for sunxi --->
                [*] Framebuffer Console Support(sunxi)
                <*> DISP Driver Support(sunxi-disp2)
        Console display driver support --->
            <*> Framebuffer Console support
    Character devices --->
        <*> Transform Driver Support(sunxi)
```

如下图所示：


![图8-5: Video support for sunxi选项](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image21.jpg)


![图8-6: Console display driver support选项](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image22.jpg)


![图8-7: Character devices选项](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image23.jpg)


##### 8.2.2.2 DRM.

如果menuconfig选择的是使用DRM作为后端，由于内核中默认使用FBDEV，所以先要取消原本的配置，再选择上DRM的配置，在menuconfig的配置中取消kmod-sunxi-disp，选上kmod-sunxi-drm，R18平台会自动配置下面的选项，不用在执行这一小节的步骤，其他平台暂未实现自动配置。

执行以下命令，以R18的为例。现阶段只有R18支持DRM：

```
make kernel_menuconfig
```

取消选择Framebuffer Console Support(sunxi)、DISP Driver Support(sunxi-disp2)、Framebuffer Console support与Transform Driver Support(sunxi)：

```
Device Drivers --->
    Graphics support --->
        Frame buffer Devices --->
            < > Support for frame buffer devices --->
                Video support for sunxi --->
                    [ ] Framebuffer Console Support(sunxi)
                    < > DISP Driver Support(sunxi-disp2)
        Console display driver support --->
            < > Framebuffer Console support
    Character devices --->
        < > Transform Driver Support(sunxi)
```

选上DRM配置：


```
Device Drivers --->
    Graphics support --->
        <*> Direct Rendering Manager (XFree86 4.1.0 and higher DRI support)
        <*> DRM Support for Allwinnertech SoC A and R Series
```

如下图所示：


![图8-8: Graphics support选项](http://photos.100ask.net/tina-docs/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image24.jpg)

DRM配置好之后，以前可能编译过不使用DRM的固件，需要先把out删除掉，并且需要清理
之前内核编译的文件，不然可能会遇到一些编译问题，在内核目录下执行：

```
make clean
```

### 8.3 Wayland使用.

#### 8.3.1 weston运行.

成功烧写固件后，在小机端使用Wayland，需要执行以下命令：

```
chmod 0700 /dev/shm/
export XDG_RUNTIME_DIR=/dev/shm
export XDG_CONFIG_HOME=/etc/xdg
weston --backend=drm-backend.so --tty=1 --idle-time=0 &
//或者
weston --backend=fbdev-backend.so --tty=1 --idle-time=0 &
```

如果没有/dev/shm/文件夹，手动创建即可：

```
mkdir /dev/shm/
```

需要开启调试的话，运行weston前执行以下命令：

```
export MESA_DEBUG=1
export EGL_LOG_LEVEL=debug
export LIBGL_DEBUG=verbose
export WAYLAND_DEBUG=1
```

如果有编译Wayland Demo的话，运行weston之后，可以运行Demo，在/usr/bin下：

wayland-scanner、weston-calibrator、weston-clickdot、weston-cliptest、weston-confine、weston-dnd、weston-eventdemo、weston-flower、weston-fullscreen、weston-image、weston-info、weston-multi-resource、weston-presentation-shm、weston-resizor、weston-scaler、weston-simple-damage、weston-simple-dmabuf-intel、weston-simple-dmabuf-v4l、weston-simple-egl、weston-simple-shm、weston-simple-touch、weston-smoke、weston-stacking、weston-subsurfaces、weston-terminal、weston-transformed。

GPU跑分测试程序可以执行以下命令，前提是编译了glmark2：

```
glmark2-es2-wayland
```

鼠标、键盘等输入设备，插上就可以使用。如果没有反应的话，确定是否编译了鼠标，键盘的驱动。

#### 8.3.2 weston.ini.

weston.ini 是 Wayland 的桌面配置文件，比如说想要去掉背景与状态栏，则可以修改以下的参数值。注释掉 background-image，background-color 改成黑色 0xff000000，panel-position改成none：

```
vi /etc/xdg/weston.ini
```

```
[shell]
# background-image=/usr/share/weston/background.png
background-color=0xff000000
panel-position=none
```

如果需要旋转屏幕的话：

```
# [output]
[output]
# name=LVDS1，mipi屏DSI-1
name=DSI-1
# mode=1680x1050，修改成对应的分辨率
```

```
mode=480*800
# transform=90，旋转的角度
transform=90
```

更多具体参数，请参考[weston.ini（ 5 ）- Arch手册页](http://jlk.fjfi.cvut.cz/arch/manpages/man/weston.ini.5)。

### 8.4 Wayland问题锦集

报错：

```
no "wayland-egl" found
```

原因可能是在之前已经编译过了没有 wayland 的图形系统，GPU 库被编译成不支持 wayland 的库，在配置 weston 的时候一定要把 Enabel opengl es support 选择上，在tina/package/libs/gpu-um/目录下执行mm -B重新编译GPU的库，如果还报no “wayland-egl” found，可以删除tina/out/目录再重新编译。