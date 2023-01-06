## 10 Flutter

### 10.1 Flutter说明

Flutter为应用开发带来了革新：只要一套代码库，即可构建、测试和发布适用于移动、Web、桌面和嵌入式平台的精美应用。Flutter特性如下：

- 快速：Flutter代码可以编译为ARM 32、ARM 64、x86和JavaScript代码，确保了有原生平台的性能。
- 高效：使用热重载(HotReload)快速构建和迭代你的产品，更新代码之后可以立即看到变化，且不会丢失应用状态。
- 灵活：屏幕的每一个像素皆可由你创作，创建高定制性、自适应的设计，在所有屏幕上都有优雅的体验。
- 多平台：部署到多种设备，只需要一份代码库，支持移动、网页、桌面和嵌入式设备。
- 开发体验：在工程中可以使用插件、自动化测试、开发者工具以及任何可以用来帮助构建高质量应用的工具。
- 稳定可依赖：Flutter由Google支持并广泛使用，全球性的开发者社区广泛参与和维护，并得到众多世界知名品牌的信任。
- 编程语言：Flutter由Dart强力驱动，为全平台优化，构建快速应用。
- 本地迭代：部署到设备之前，你可以在本地调试代码，并在Web或移动平台运行产品原型。
- 灵活扩展：任何嵌入式设备，Flutter灵活且轻量级的UI引擎都能轻松扩展以满足你的需求。
- 蓬勃发展的生态：通过Flutter成熟的package生态，你可以为众多嵌入式设备创造新的可能。

目前Tina中移植了Flutter 2.10.4与Demo，注意Flutter应用只能在glibc编译工具链下运行。下表列出Flutter相关库说明：


表10-1: Flutter相关库说明

| 包名                       | 说明                                                       |
| :------------------------- | :--------------------------------------------------------- |
| complex_layout             | 滑动列表测试app应用                                        |
| gallery flutter            | 的官方大型app应用，集成了各种控件效果和常见应用场景        |
| video_player               | 视频播放测试app应用                                        |
| flutter_eglfs              | 预编译加载flutter app的应用，用gpu渲染，支持旋转           |
| flutter_fbdev              | 预编译加载flutter app的应用，用cpu渲染，暂时不支持旋转     |
| flutter-client             | 预编译加载flutter app的应用，用gpu渲染，支持旋转与视频播放 |
| libvideo_player_plugin.so  | 视频播放插件，目前仅供测试使用，后续会替换视频播放接口     |
| libflutter_elinux_eglfs.so | 如果需要自定义插件，需要链接该库                           |
| libflutter_engine.so       | flutter核心库                                              |
| gen_snapshot flutter       | app编译AOT所需要的工具                                     |

下面是应用complex_layout截图：


![图10-1: complex_layout主页截图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image28.jpg)

下面是应用gallery截图：


![图10-2: gallery主页截图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image29.jpg)


![图10-3: gallery_1主页截图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image30.jpg)


### 10.2 Flutter配置

```
source build/envsetup.sh
lunch XXX平台名称
make menuconfig
```

```
Gui --->
    Flutter --->
        flutter-sunxi --->
            --- flutter-sunxi
            -*- flutter use fbdev
            [*] flutter use eglfs
            [ ] flutter use client
            [ ] flutter use elinux so
            [*] flutter demo complex layout
            [*] flutter demo gallery
            [ ] flutter demo video player
```

### 10.3 Flutter运行

flutter路径如下：

```
tina/package/gui/flutter/flutter-sunxi
tina/dl/flutter-sunxi-1.0.7.tar.gz
```

当配置上flutter之后，会把flutter_fbdev，complex_layout等放到/usr/bin目录下，libflut-ter_engine.so等放到/usr/lib目录下，执行如下命令：

```
flutter_eglfs /usr/bin/bundle_complex_layout/
flutter_eglfs /usr/bin/usr/bin/bundle_gallery/
```

```
flutter_fbdev /usr/bin/bundle_complex_layout/
flutter_fbdev /usr/bin/usr/bin/bundle_gallery/
```

```
flutter-client -b /usr/bin/bundle_complex_layout/
flutter-client -b /usr/bin/bundle_gallery/
```


初始化时会打印一些信息和探测触摸节点，log如下：

```
root@TinaLinux:/# flutter_eglfs /usr/bin/bundle_gallery/
flutter: egl version: 1.4 (1.4 build 1.11@5516664)
flutter: egl vendor: Imagination Technologies
flutter: red OK: 8
flutter: green OK: 8
flutter: blue OK: 8
flutter: alpha OK: 8
flutter: found input device <sunxi-keyboard>
flutter: input props: <none>
flutter: found input device <axp806-pek>
flutter: input props: <none>
```

```
flutter: found input device <gt82x>
flutter: input props: <INPUT_PROP_DIRECT>
```


如果没有识别到INPUT_PROP_DIRECT，那么需要在触摸驱动中加上如下代码：

```
set_bit(INPUT_PROP_DIRECT, ts->input_dev->propbit);
```


另外也可以用命令生成软连接touchscreen，就会直接以touchscreen为触摸节点，方便调
试。命令如下:

```
ln -s /dev/input/eventX /dev/input/touchscreen
```


还可以看更详细的信息，增加旋转参数，命令如下：


```
root@TinaLinux:/# flutter_eglfs -h
    flutter_eglfs - run flutter apps on your device.

USAGE:
    flutter_eglfs [options] <bundle path>

OPTIONS:
    -f, --fps-print Print frame rates.
    -p, --touch-print Print touch points.
    -r, --rotate-screen Rotate the screen, the values are 0, 90, 180, 270.
    -v, --version Show flutter_eglfs version and exit.
    -h, --help Show this help and exit.

BUNDLE PATH TREE:
    ./app_bundle/data/flutter_assets
    ./app_bundle/data/icudtl.dat
    ./app_bundle/lib/libapp.so

EXAMPLES:
    flutter_eglfs ./app_bundle
    flutter_eglfs -r 90 ./app_bundle
    LD_LIBRARY_PATH=./ flutter_eglfs ./app_bundle
    LD_LIBRARY_PATH can ensure that libflutter_engine.so is found.

OTHER:
    Some applications may require system information.
    export LANG="en_US.UTF-8"
```


关于如何编译flutter应用，可以看readme.txt中的说明，路径如下：

```
tina/out/方案名称/compile_dir/target/flutter-sunxi-1.0.7/readme.txt
```

