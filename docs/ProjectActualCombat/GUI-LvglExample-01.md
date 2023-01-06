# LVGL 开发实战

## 移植基于 LVGL 的 2048 小游戏

这一节将以一个已经编写好的 `lvgl` 小游戏 `2048` 描述如何将已经编写完成的 `lvgl` 程序移植到开发板上。

这里使用的 `2048` 小游戏由百问网提供，开源地址：[lv_lib_100ask](https://gitee.com/weidongshan/lv_lib_100ask)

### 准备脚手架

在这之前，我们先准备基础的 LVGL 脚手架。可以直接从 `lv_g2d_test` 里复制过来进行修改即可。

首先我们复制源码，在 `platform/thirdparty/gui/lvgl-8` 源码文件夹里，把 红色箭头 所指的 `lv_g2d_test` 的源码作为模板复制到 黄色箭头指向的 `lv_2048` 文件夹里。

如下图所示，并清理下 `res` 资源文件夹，

[![img](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-13-53-08-1658123584%281%29.png)](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-13-53-08-1658123584(1).png)

同样的，复制一份引索文件，找到 `openwrt/package/thirdparty/gui/lvgl-8` 并把 `lv_g2d_test` 复制一份重命名为 `lv_2048` 作为我们 `2048` 小游戏使用的引索。

[![img](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-13-53-55-1658123630%281%29.png)](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-13-53-55-1658123630(1).png)

并编辑 `Makefile`，修改文件名称，把 `lv_g2d_test` 修改为这里的 `lv_2048`

```makefile
include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/package.mk
include ../sunxifb.mk

PKG_NAME:=lv_2048
PKG_VERSION:=8.1.0
PKG_RELEASE:=1

PKG_BUILD_DIR := $(BUILD_DIR)/$(PKG_NAME)
SRC_CODE_DIR := $(LICHEE_PLATFORM_DIR)/thirdparty/gui/lvgl-8/$(PKG_NAME)
define Package/$(PKG_NAME)
  SECTION:=gui
  SUBMENU:=Littlevgl
  CATEGORY:=Gui
  DEPENDS:=+LVGL8_USE_SUNXIFB_G2D:libuapi +LVGL8_USE_SUNXIFB_G2D:kmod-sunxi-g2d \
           +LVGL8_USE_FREETYPE:libfreetype
  TITLE:=lvgl 2048 
endef

PKG_CONFIG_DEPENDS := \
    CONFIG_LVGL8_USE_SUNXIFB_DOUBLE_BUFFER \
    CONFIG_LVGL8_USE_SUNXIFB_CACHE \
    CONFIG_LVGL8_USE_SUNXIFB_G2D \
    CONFIG_LVGL8_USE_SUNXIFB_G2D_ROTATE

define Package/$(PKG_NAME)/config
endef

define Package/$(PKG_NAME)/Default
endef

define Package/$(PKG_NAME)/description
  a lvgl 2048 v8.1.0
endef

define Build/Prepare
    $(INSTALL_DIR) $(PKG_BUILD_DIR)/
    $(CP) -r $(SRC_CODE_DIR)/src $(PKG_BUILD_DIR)/
    $(CP) -r $(SRC_CODE_DIR)/../lvgl $(PKG_BUILD_DIR)/src/
    $(CP) -r $(SRC_CODE_DIR)/../lv_drivers $(PKG_BUILD_DIR)/src/
endef

define Build/Configure
endef

TARGET_CFLAGS+=-I$(PKG_BUILD_DIR)/src

ifeq ($(CONFIG_LVGL8_USE_SUNXIFB_G2D),y)
TARGET_CFLAGS+=-DLV_USE_SUNXIFB_G2D_FILL \
                -DLV_USE_SUNXIFB_G2D_BLEND \
                -DLV_USE_SUNXIFB_G2D_BLIT \
                -DLV_USE_SUNXIFB_G2D_SCALE
endif

define Build/Compile
    $(MAKE) -C $(PKG_BUILD_DIR)/src\
        ARCH="$(TARGET_ARCH)" \
        AR="$(TARGET_AR)" \
        CC="$(TARGET_CC)" \
        CXX="$(TARGET_CXX)" \
        CFLAGS="$(TARGET_CFLAGS)" \
        LDFLAGS="$(TARGET_LDFLAGS)" \
        INSTALL_PREFIX="$(PKG_INSTALL_DIR)" \
        all
endef

define Package/$(PKG_NAME)/install
    $(INSTALL_DIR) $(1)/usr/bin/
    $(INSTALL_DIR) $(1)/usr/share/lv_2048
    $(INSTALL_BIN) $(PKG_BUILD_DIR)/src/$(PKG_NAME) $(1)/usr/bin/
endef

$(eval $(call BuildPackage,$(PKG_NAME)))
```

完成脚手架的搭建后，可以 `make menuconfig` 里查看是否出现了 `lv_2048` 这个选项，选中它。

[![img](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-13-56-31-image.png)](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-13-56-31-image.png)

### 修改源码

第二步是修改源码。编辑之前复制的 `main.c` 文件，把不需要的 `lv_g2d_test` 的部分删去。保留最基础的部分。

```c
#include "lvgl/lvgl.h"
#include "lv_drivers/display/sunxifb.h"
#include "lv_drivers/indev/evdev.h"
#include <unistd.h>
#include <pthread.h>
#include <time.h>
#include <sys/time.h>
#include <stdlib.h>
#include <stdio.h>

static lv_style_t rect_style;
static lv_obj_t *rect_obj;
static lv_obj_t *canvas;

int main(int argc, char *argv[]) {
    lv_disp_drv_t disp_drv;
    lv_disp_draw_buf_t disp_buf;
    lv_indev_drv_t indev_drv;
    uint32_t rotated = LV_DISP_ROT_NONE;

    lv_disp_drv_init(&disp_drv);

    /*LittlevGL init*/
    lv_init();

    /*Linux frame buffer device init*/
    sunxifb_init(rotated);

    /*A buffer for LittlevGL to draw the screen's content*/
    static uint32_t width, height;
    sunxifb_get_sizes(&width, &height);

    static lv_color_t *buf;
    buf = (lv_color_t*) sunxifb_alloc(width * height * sizeof(lv_color_t), "lv_2048");

    if (buf == NULL) {
        sunxifb_exit();
        printf("malloc draw buffer fail\n");
        return 0;
    }

    /*Initialize a descriptor for the buffer*/
    lv_disp_draw_buf_init(&disp_buf, buf, NULL, width * height);

    /*Initialize and register a display driver*/
    disp_drv.draw_buf = &disp_buf;
    disp_drv.flush_cb = sunxifb_flush;
    disp_drv.hor_res = width;
    disp_drv.ver_res = height;
    disp_drv.rotated = rotated;
    disp_drv.screen_transp = 0;
    lv_disp_drv_register(&disp_drv);

    evdev_init();
    lv_indev_drv_init(&indev_drv); /*Basic initialization*/
    indev_drv.type = LV_INDEV_TYPE_POINTER; /*See below.*/
    indev_drv.read_cb = evdev_read; /*See below.*/
    /*Register the driver in LVGL and save the created input device object*/
    lv_indev_t *evdev_indev = lv_indev_drv_register(&indev_drv);

    /*Handle LitlevGL tasks (tickless mode)*/
    while (1) {
        lv_task_handler();
        usleep(1000);
    }

    return 0;
}

/*Set in lv_conf.h as `LV_TICK_CUSTOM_SYS_TIME_EXPR`*/
uint32_t custom_tick_get(void) {
    static uint64_t start_ms = 0;
    if (start_ms == 0) {
        struct timeval tv_start;
        gettimeofday(&tv_start, NULL);
        start_ms = (tv_start.tv_sec * 1000000 + tv_start.tv_usec) / 1000;
    }

    struct timeval tv_now;
    gettimeofday(&tv_now, NULL);
    uint64_t now_ms;
    now_ms = (tv_now.tv_sec * 1000000 + tv_now.tv_usec) / 1000;

    uint32_t time_ms = now_ms - start_ms;
    return time_ms;
}
```

接下来则是对接 `lv_lib_100ask` 与 `2048` 小游戏，我们先下载 `lv_lib_100ask` 的源码，放置到 `platform/thirdparty/gui/lvgl-8/lv_2048` 的 `src` 文件夹里。并按照 `lv_lib_100ask` 的说明，复制一份 `lv_lib_100ask_conf_template.h` 到 `src` 目录，并改名为 `lv_lib_100ask_conf.h`

[![img](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-14-55-44-image.png)](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-14-55-44-image.png)

编辑 `lv_lib_100ask_conf.h`，开启整个库的引用，并配置启用 `LV_USE_100ASK_2048` 。为了简洁，这里删除了不需要的配置项。

```c
/**
 * @file lv_lib_100ask_conf.h
 * Configuration file for v8.2.0
 *
 */
/*
 * COPY THIS FILE AS lv_lib_100ask_conf.h
 */

/* clang-format off */
#if 1 /*Set it to "1" to enable the content*/ 

#ifndef LV_LIB_100ASK_CONF_H
#define LV_LIB_100ASK_CONF_H

#include "lv_conf.h"

/*******************
 * GENERAL SETTING
 *******************/

/*********************
 * USAGE
 *********************

/*2048 game*/
#define LV_USE_100ASK_2048                               1
#if LV_USE_100ASK_2048
    /* Matrix size*/
    /*Do not modify*/
    #define  LV_100ASK_2048_MATRIX_SIZE          4

    /*test*/
    #define  LV_100ASK_2048_SIMPLE_TEST          1
#endif  

#endif /*LV_LIB_100ASK_H*/

#endif /*End of "Content enable"*/
```

再编辑 `platform/thirdparty/gui/lvgl-8/lv_2048/src/lv_lib_100ask/lv_lib_100ask.h` 中的版本号，修改为 `(8,1,0)`

[![img](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-15-48-46-image.png)](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-15-48-46-image.png)

之后在 `main.c` 里修改，对接 `lv_100ask_2048_simple_test`，具体如下。

（1）头文件加入 `lv_lib_100ask/lv_lib_100ask.h`

```c
#include <lv_lib_100ask/lv_lib_100ask.h>
```

（2）在 `main` 函数里添加接口调用

```c
lv_100ask_2048_simple_test();
```

完整的 `main.c` 如下

```c
#include <unistd.h>
#include <pthread.h>
#include <time.h>
#include <sys/time.h>
#include <stdlib.h>
#include <stdio.h>

#include "lvgl/lvgl.h"
#include "lv_drivers/display/sunxifb.h"
#include "lv_drivers/indev/evdev.h"

#include "lv_lib_100ask/lv_lib_100ask.h"  // 引用头文件

static lv_style_t rect_style;
static lv_obj_t *rect_obj;
static lv_obj_t *canvas;

int main(int argc, char *argv[]) {
    lv_disp_drv_t disp_drv;
    lv_disp_draw_buf_t disp_buf;
    lv_indev_drv_t indev_drv;
    uint32_t rotated = LV_DISP_ROT_NONE;

    lv_disp_drv_init(&disp_drv);

    /*LittlevGL init*/
    lv_init();

    /*Linux frame buffer device init*/
    sunxifb_init(rotated);

    /*A buffer for LittlevGL to draw the screen's content*/
    static uint32_t width, height;
    sunxifb_get_sizes(&width, &height);

    static lv_color_t *buf;
    buf = (lv_color_t*) sunxifb_alloc(width * height * sizeof(lv_color_t), "lv_nes");

    if (buf == NULL) {
        sunxifb_exit();
        printf("malloc draw buffer fail\n");
        return 0;
    }

    /*Initialize a descriptor for the buffer*/
    lv_disp_draw_buf_init(&disp_buf, buf, NULL, width * height);

    /*Initialize and register a display driver*/
    disp_drv.draw_buf = &disp_buf;
    disp_drv.flush_cb = sunxifb_flush;
    disp_drv.hor_res = width;
    disp_drv.ver_res = height;
    disp_drv.rotated = rotated;
    disp_drv.screen_transp = 0;
    lv_disp_drv_register(&disp_drv);

    evdev_init();
    lv_indev_drv_init(&indev_drv); /*Basic initialization*/
    indev_drv.type = LV_INDEV_TYPE_POINTER; /*See below.*/
    indev_drv.read_cb = evdev_read; /*See below.*/
    /*Register the driver in LVGL and save the created input device object*/
    lv_indev_t *evdev_indev = lv_indev_drv_register(&indev_drv);

    lv_100ask_2048_simple_test();  // 调用 2048 小游戏

    /*Handle LitlevGL tasks (tickless mode)*/
    while (1) {
        lv_task_handler();
        usleep(1000);
    }

    return 0;
}

/*Set in lv_conf.h as `LV_TICK_CUSTOM_SYS_TIME_EXPR`*/
uint32_t custom_tick_get(void) {
    static uint64_t start_ms = 0;
    if (start_ms == 0) {
        struct timeval tv_start;
        gettimeofday(&tv_start, NULL);
        start_ms = (tv_start.tv_sec * 1000000 + tv_start.tv_usec) / 1000;
    }

    struct timeval tv_now;
    gettimeofday(&tv_now, NULL);
    uint64_t now_ms;
    now_ms = (tv_now.tv_sec * 1000000 + tv_now.tv_usec) / 1000;

    uint32_t time_ms = now_ms - start_ms;
    return time_ms;
}
```

然后就是 `Makefile` 修改，增加一个 `lv_lib_100ask` 的 SRC 引用。

```
include lv_lib_100ask/lv_lib_100ask.mk
```

顺便也把 `BIN` 改为 `lv_2048` ，完整的 `Makefile` 如下

```makefile
#
# Makefile
#
CC ?= gcc
LVGL_DIR_NAME ?= lvgl
LVGL_DIR ?= ${shell pwd}
CFLAGS ?= -O3 -g0 -I$(LVGL_DIR)/ -Wall -Wshadow -Wundef -Wmissing-prototypes -Wno-discarded-qualifiers -Wall -Wextra -Wno-unused-function -Wno-error=strict-prototypes -Wpointer-arith -fno-strict-aliasing -Wno-error=cpp -Wuninitialized -Wmaybe-uninitialized -Wno-unused-parameter -Wno-missing-field-initializers -Wtype-limits -Wsizeof-pointer-memaccess -Wno-format-nonliteral -Wno-cast-qual -Wunreachable-code -Wno-switch-default -Wreturn-type -Wmultichar -Wformat-security -Wno-ignored-qualifiers -Wno-error=pedantic -Wno-sign-compare -Wno-error=missing-prototypes -Wdouble-promotion -Wclobbered -Wdeprecated -Wempty-body -Wtype-limits -Wshift-negative-value -Wstack-usage=2048 -Wno-unused-value -Wno-unused-parameter -Wno-missing-field-initializers -Wuninitialized -Wmaybe-uninitialized -Wall -Wextra -Wno-unused-parameter -Wno-missing-field-initializers -Wtype-limits -Wsizeof-pointer-memaccess -Wno-format-nonliteral -Wpointer-arith -Wno-cast-qual -Wmissing-prototypes -Wunreachable-code -Wno-switch-default -Wreturn-type -Wmultichar -Wno-discarded-qualifiers -Wformat-security -Wno-ignored-qualifiers -Wno-sign-compare
LDFLAGS ?= -lm
BIN = lv_2048


#Collect the files to compile
SRCDIRS   =  $(shell find . -maxdepth 1 -type d)
MAINSRC = $(foreach dir,$(SRCDIRS),$(wildcard $(dir)/*.c))

include $(LVGL_DIR)/lvgl/lvgl.mk
include $(LVGL_DIR)/lv_drivers/lv_drivers.mk
include lv_lib_100ask/lv_lib_100ask.mk

OBJEXT ?= .o

AOBJS = $(ASRCS:.S=$(OBJEXT))
COBJS = $(CSRCS:.c=$(OBJEXT))

MAINOBJ = $(MAINSRC:.c=$(OBJEXT))

SRCS = $(ASRCS) $(CSRCS) $(MAINSRC)
OBJS = $(AOBJS) $(COBJS)

## MAINOBJ -> OBJFILES

all: default

%.o: %.c
    @$(CC)  $(CFLAGS) -c $< -o $@
    @echo "CC $<"

default: $(AOBJS) $(COBJS) $(MAINOBJ)
    $(CC) -o $(BIN) $(MAINOBJ) $(AOBJS) $(COBJS) $(LDFLAGS)

clean: 
    rm -f $(BIN) $(AOBJS) $(COBJS) $(MAINOBJ)
```

### 对接触摸

做了以上操作，可能会发现触摸没有反应，这是因为触摸绑定的 `event` 事件号不对，默认的绑定是 `event3` 而查阅启动 `log` 可知，开发板的触摸屏对接的是 `event0`

[![img](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-15-17-48-image.png)](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-15-17-48-image.png)

这时需要修改绑定的 `event` 事件号，其配置文件在 `lv_drv_conf.h` 内：

[![img](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-15-19-12-image.png)](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-15-19-12-image.png)

这里将 `event3` 改为 `event0` 即可

```c
#  define EVDEV_NAME   "/dev/input/event0"
```

当然除了这样的方法，另外也可以用命令生成软连接`touchscreen`，就会直接以 `touchscreen` 为触摸节点，方便调试:

```shell
ln -s /dev/input/eventX /dev/input/touchscreen
```

### 测试编译

修改好了，希望单独编译这个包测试下而不编译完整的 SDK。可以这样做：

（1）确保已经 `source build/envsetup.sh` 并已经 `lunch`

（2）在任意文件夹下执行命令 `mmo lv_2048 -B`

[![img](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-15-13-39-image.png)](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-15-13-39-image.png)

其中 `mmo` 的意思是 单独编译一个 `openWrt` 软件包，后面的 `lv_2048` 是软件包名。`-B` 参数是先 `clean` 再编译，不加这个参数就是直接编译了。

### 测试运行

编译打包后，到开发板上使用 `lv_2048` 即可运行

[![img](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-14-47-19-lQDPJxaBvZoKbvPNC9DND8CwaCpI77wfq4cC1aIjrECqAA_4032_3024.jpg)](https://v853.docs.aw-ol.com/assets/img/lvgl_demo_2048/2022-07-18-14-47-19-lQDPJxaBvZoKbvPNC9DND8CwaCpI77wfq4cC1aIjrECqAA_4032_3024.jpg)