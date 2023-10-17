## 9 LVGL

### 9.1 LVGL说明.

LVGL是一个免费的开源图形库，提供了创建嵌入式GUI所需的一切，具有易于使用的图形元素，美观的视觉效果和低内存占用，采用MIT许可协议，可以访问LittlevGL获取更多资料。

- 强大的构建块：按钮、图表、列表、滑块、图像等。
- 高级图形引擎：动画、抗锯齿、不透明度、平滑滚动、混合模式等。
- 支持各种输入设备：触摸屏、鼠标、键盘、编码器、按钮等。
- 支持多显示器。
- 独立于硬件，可与任何微控制器和显示器一起使用。
- 可扩展以使用少量内存（64 kB闪存、16 kB RAM）运行。
- 多语言支持，支持UTF-8处理、CJK、双向和阿拉伯语。
- 通过类CSS样式完全可定制的图形元素。
- 受CSS启发的强大布局：Flexbox和Grid。
- 支持操作系统、外部内存和GPU，但不是必需的。
- 使用单个帧缓冲区也能平滑渲染。
- 用C编写并与C++兼容。
- Micropython Binding在Micropython中公开LVGL API。
- 可以在PC上使用模拟器开发。
- 100 多个简单的例子。
- 在线和PDF格式的文档和API参考。

目前Tina中移植了LVGL 8.1.0核心组件与Demo，下表列出LVGL相关库说明：


表9-1: LVGL相关库说明


| 包名        | 说明                                                      |
| :---------- | :-------------------------------------------------------- |
| lv_demos    | lvgl的官方demo                                            |
| lv_drivers  | lvgl的官方设备驱动程序，集成了sunxifb、sunxig2d和sunximem |
| lv_examples | lvgl测试用例，最终调用的是lv_demos中的函数                |
| lvgl        | lvgl核心库                                                |
| lv_g2d_test | g2d测试用例，专门测试已经对接好的g2d接口                  |
| lv_monitor  | 压力测试与状态监测软件                                    |
| sunxifb.mk  | 公共配置文件，写应用Makefile时需要包含进去                |


下面是应用lv_examples截图：


![图9-1: lv_demo_widgets主页截图](https://photos.100ask.net/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image25.jpg)


![图9-2: lv_demo_music主页截图](https://photos.100ask.net/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image26.jpg)


下面是应用lv_monitor截图：



![图9-3: lv_monitor主页截图](https://photos.100ask.net/Tina-Sdk/OpenRemoved_Tina_Linux_Graphics_system_development_Guide-image27.jpg)


### 9.2 LVGL配置.

```
source build/envsetup.sh
lunch XXX平台名称
make menuconfig
```

```
Gui --->
    Littlevgl --->
        < > lv_demo
        <*> lv_examples （lvgl官方demo）
        -*- lvgl-8.1.0 use sunxifb double buffer （使能双缓冲，解决撕裂问题）
        [*] lvgl-8.1.0 use sunxifb cache （使能fb cache）
        [ ] lvgl-8.1.0 use sunxifb g2d （使能G2D硬件加速）
        [ ] lvgl-8.1.0 use sunxifb g2d rotate （使能G2D硬件旋转）
        [ ] lvgl-8.1.0 use freetype （自动链接freetype）
        <*> lv_g2d_test （g2d接口测试用例）
        <*> lv_monitor （压力测试与数据监测软件）
        < > smartva
        < > smartva_ota
```

### 9.3 LVGL使用.

lvgl路径：


```
tina/package/gui/littlevgl-8
```

#### 9.3.1 sunxifb

在sunxifb中，我们提供了一组接口，如下：


表9-2: sunxifb相关接口说明


| 接口              | 说明                                                         |
| :---------------- | :----------------------------------------------------------- |
| sunxifb_init      | 该函数主要功能是初始化显示引擎。带一个旋转参数，使能g2d旋转的话，就用这个参数指定旋转方向 |
| sunxifb_exit      | 该函数比较简单，实现关闭cache，关闭g2d，释放旋转buffer，关闭fb0 |
| sunxifb_flush     | 该函数比较重要，负责把draw buffer拷贝到back buffer中，并且绘制最后一帧后，交换frontback buffer。应用不要调用该函数 |
| sunxifb_get_sizes | 该函数获取屏幕分辨率，这样应用程序就可以不用写死初始化时的分辨率了 |
| sunxifb_alloc     | 该函数主要用来申请系统绘图内存，使能部分G2D功能后，会申请连续物理内存 |
| sunxifb_free      | 该函数用来释放sunxifb_alloc申请的内存                        |


代码位置如下：

```
tina/package/gui/littlevgl-8/lv_drivers/display/sunxifb.c
```

在sunxifb_init(rotated)，中rotated的值为LV_DISP_ROT_NONE，LV_DISP_ROT_90，LV_DISP_ROT_180，LV_DISP_ROT_270。

最后还有赋值disp_drv.rotated=rotated。如果没有g2d旋转，也可以指定disp_drv.sw_rotate = 1使用软件旋转。

#### 9.3.2 sunxig2d

在sunxig2d中，实现了对g2d ioctl的封装，这些函数都不需要应用调用，如下：


表9-3: sunxig2d相关接口说明

| 接口               | 说明                                                         |
| :----------------- | :----------------------------------------------------------- |
| sunxifb_g2d_init   | g2d模块初始化函数，打开/dev/g2d节点，设置g_format。初始化时，根据使能的宏，打印相应的log |
| sunxifb_g2d_deinit | 该函数关闭g2d设备                                            |


| 接口                   | 说明                                                         |
| :--------------------- | :----------------------------------------------------------- |
| sunxifb_g2d_get_limit  | 该函数获取g2d使用阈值                                        |
| sunxifb_g2d_blit_to_fb | 该函数用来拷贝fb0的front和back buffer这两块buffer，也可以把rotate buffer旋转到back buffer |
| sunxifb_g2d_fill       | 该函数使用g2d填充一个颜色矩形，颜色可以带透明度              |
| sunxifb_g2d_blit       | 该函数用来拷贝图像，不能blend图像                            |
| sunxifb_g2d_blend      | 该函数可以进行图像blend                                      |
| sunxifb_g2d_scale      | 该函数用来缩放图像                                           |


代码位置如下：

```
tina/package/gui/littlevgl-8/lv_drivers/display/sunxig2d.c
```

以上g2d函数，都已经对接lvgl绘图框架，使用lvgl的lv_draw_map、lv_img_set_zoom和lv_canvas_draw_img函数就可以使用起来。

lv_g2d_test应用中有完整的使用示例。


#### 9.3.3 sunximem.

在sunximem中，实现了管理物理内存的封装，这些函数都不需要应用调用，如下：

表9-4: sunximem相关接口说明

| 接口                    | 说明                                                         |
| :---------------------- | :----------------------------------------------------------- |
| sunxifb_mem_init        | 该函数会在sunxifb_init中调用，初始化物理内存申请接口，使用的是libuapi中间件 |
| sunxifb_mem_deinit      | 该函数通过调用SunxiMemClose，释放申请的接口资源              |
| sunxifb_mem_alloc       | 该函数比较重要，许多地方都会用到，需要传入申请的字节数和使用说明 |
| sunxifb_mem_free        | 该函数用来释放调用sunxifb_mem_alloc申请的内存                |
| sunxifb_mem_get_phyaddr | 该函数把sunxifb_mem_alloc申请内存的虚拟地址转换为物理地址，g2d驱动只接受buffer的物理地址或者fd |
| sunxifb_mem_flush_cache | 该函数用来刷sunxifb_mem_alloc申请buffer的cache               |


代码位置如下：

```
tina/package/gui/littlevgl-8/lv_drivers/display/sunxigmem.c
```

因为g2d驱动只能使用物理连续内存，因此解码图片时，必须要通过sunxifb_mem_alloc来申
请内存。


> 说明：当前只实现了 bmp 、 png 和 gif 图片的内存申请， jpeg 图片暂未实现。


当使用lv_canvas_set_buffer时，传入的buffer需要是sunxifb_alloc申请的buffer，sunx-
ifb_alloc中会判断是否需要申请物理连续内存。


> 说明：自定义画布 lv_canvas 暂未对接 g2d 缩放功能。


#### 9.3.4 evdev

触摸我们用的是lvgl官方的evdev。

代码位置如下：

```
tina/package/gui/littlevgl-8/lv_drivers/indev/evdev.c
```

在应用 lv_drv_conf.h 中修改 EVDEV_NAME 为触摸屏对应生成的 event 节点，例如lv_examples的配置文件：

```
tina/package/gui/littlevgl-8/lv_examples/src/lv_drv_conf.h
```

另外也可以用命令生成软连接touchscreen，就会直接以touchscreen为触摸节点，方便调试。命令如下：

```
ln -s /dev/input/eventX /dev/input/touchscreen
```

如果disp_drv.rotated指定了旋转 90 或者 180 度，lvgl内部会自行旋转触摸坐标，不用触摸驱动内部去旋转触摸坐标。

### 9.4 LVGL新建应用

推荐以lv_g2d_test为模板，复制一个新项目：

```
tina/package/gui/littlevgl-8/lv_g2d_test
```

在Makefile中，需要包含sunxifb.mk公共配置，在编译应用时会把宏传递下去。方式如下：

```
tina/package/gui/littlevgl-8/lv_g2d_test/Makefile

include ../sunxifb.mk
```

另外可以注意到有以下配置，这些配置需要按需开启，在部分芯片上是不支持G2D_BLEND等

操作的，只支持简单的旋转功能：


```
ifeq ($(CONFIG_LVGL8_USE_SUNXIFB_G2D),y)
TARGET_CFLAGS+=-DLV_USE_SUNXIFB_G2D_FILL \
                -DLV_USE_SUNXIFB_G2D_BLEND \
                -DLV_USE_SUNXIFB_G2D_BLIT \
                -DLV_USE_SUNXIFB_G2D_SCALE
endif
```

在应用编译的实际Makefile中，可以只编译需要的文件，缩减可执行文件的大小，像下面的示例

就是不编译examples文件夹：

```
tina/package/gui/littlevgl-8/lv_g2d_test/src/Makefile
```

```
include $(LVGL_DIR)/lvgl/lvgl.mk
include $(LVGL_DIR)/lv_drivers/lv_drivers.mk

#Do not compile the example
EXCSRCS += $(shell find -L $(LVGL_DIR)/$(LVGL_DIR_NAME)/examples -name \*.c)
CSRCS := $(filter-out $(EXCSRCS),$(CSRCS))
```

关于lvgl的配置文件，也是建议用lv_g2d_test中的，可以对比原始未修改过的配置，然后再根
据实际场景开关相应配置。配置文件如下：

```
tina/package/gui/littlevgl-8/lv_g2d_test/src/lv_conf.h
tina/package/gui/littlevgl-8/lv_g2d_test/src/lv_drv_conf.h
```


```
tina/package/gui/littlevgl-8/lvgl/lv_conf_template.h
tina/package/gui/littlevgl-8/lv_drivers/lv_drv_conf_template.h.h
```


最后就是应用的初始化了，在lv_g2d_test中，有比较清晰的调用流程了，需要注意的是sunx-ifb_init需要传入旋转参数和sunxifb_alloc申请内存即可。

### 9.5 LVGL运行.

我们提供了几个测试用例，执行命令如下：

```
lv_examples 0
```

```
lv_examples 0, is lv_demo_widgets
lv_examples 1, is lv_demo_music
lv_examples 2, is lv_demo_benchmark
lv_examples 3, is lv_demo_keypad_encoder
lv_examples 4, is lv_demo_stress
```

```
lv_g2d_test
```

```
lv_g2d_test 0 5 0 1
one num is rotate, range is 0~3
tow num is gif, range is 0~11, 11 is no show gif
three num is bmp, range is 0~2, 2 is no show bmp
four num is png, range is 0~3, 3 is no show png
```

```
lv_monitor
```

在初始化时，会有如下打印，根据配置的不同会有差异，表示打开了某项配置：

```
wh=1280x800, vwh=1280x1600, bpp=32, rotated=0
Turn on double buffering.
Turn on 2d hardware acceleration.
Turn on 2d hardware acceleration fill.
Turn on 2d hardware acceleration blit.
Turn on 2d hardware acceleration blend.
Turn on 2d hardware acceleration scale.
Turn on 2d hardware acceleration rotate.
```

