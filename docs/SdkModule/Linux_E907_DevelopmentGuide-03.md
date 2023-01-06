## 6 AMP 环境搭建

AMP 环境用于Linux 和E907 间通信，Linux 依赖于2 个驱动，melis 依赖于openamp 驱
动。

1. remoteproc 驱动：主要用来管理E907 固件的加载器的
2. rpmsg：在virtio 框架上实现的消息传送框架

### 6.1 Linux 配置

**注意：需要前面的启动环境配置好后，再执行以下操作。**
需要打开的配置有：

1. remoteproc 驱动
2. rpmsg 驱动

#### 6.1.1 remoteproc 驱动

```
ckernel
m kernel_menuconfig
```

选中

![image-20221121115519043](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121115519043.png)

<center>图6-1: rproc config</center>

#### 6.1.2 rpmsg 驱动

```
ckernel
m kernel_menuconfig
# 红框必选,蓝色框为sdk提供的rpmsg demo,视情况而选择
# 建议选上sunxi rpmsg ctrl driver 方便后面测试rpmsg通信功能
```

选中

![image-20221121115554767](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121115554767.png)

<center>图6-2: rpmsg config</center>

### 6.2 melis 配置

主要进行2 个配置：

1. msgbox 配置
2. openamp 配置

#### 6.2.1 msgbox 配置

```
mmelis menuconfig #选择下面2项
```


选中

![image-20221121115636186](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121115636186.png)

<center>图6-3: msgbox-melis config</center>

#### 6.2.2 openamp 配置

```
mmelis menuconfig

# 红框是必选,蓝框是可选的rpmsg demo
```

刚刚Linux 端选择了rpmsg hearbeat demo 和ctrl driver，我们这里也选上对应的驱动hearbeatdriver 和client driver。
选中

![image-20221121115718036](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121115718036.png)

<center>图6-4: openamp config</center>

为了方便在控制台测试rpmsg 通信，rpmsg client driver 还需开启下面2 个选项

![image-20221121115739276](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121115739276.png)

<center>图6-5: rpmsg client config</center>

### 6.3 打包

```
make -j16 # 编译tina
mmelis # 编译melis
p
烧录
```

### 6.4 测试

本章节介绍一些AMP 提供的控制台命令，用于测试AMP 环境

#### 6.4.1 E907 控制

1.在linux 控制台执行：echo stop > /sys/kernel/debug/remoteproc/remoteproc0/state
（停止e907）

2.在linux 控制台执行：echo start > /sys/kernel/debug/remoteproc/remoteproc0/state
（启动e907）

若控制台出现remoteproc0: remote processor e907_rproc is now up，表明启动e907 成功。
如果使能了rpmsg_heartbeat 和rpmsg_ctrl 驱动，可以在Linux 控制台start 之后会看到如下输出：

![image-20221121115904466](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121115904466.png)

<center>图6-6: rproc test</center>

红框里面表示有2 个设备成功创建，代表rpmsg 正常。

#### 6.4.2 rpmsg 通信测试

借助rpmsg_ctrl 驱动帮助我们进行测试

##### 6.4.2.1 名字监听.

平台：melis 控制台
输入如下图命令：
eptdev_bind 命令：监听name=test 的链接，最大连接数5 个

![image-20221121115954891](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121115954891.png)

<center>图6-7: rproc test</center>

##### 6.4.2.2 节点创建

平台：Linux 控制台
输入如下图的命令，进行节点创建

![image-20221121120018359](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121120018359.png)

<center>图6-8: rpmsg test</center>

![image-20221121120038291](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121120038291.png)

<center>图6-9: rpmsg test</center>

根据log 可以看出，创建了一个rpmsg0-4 5 个设备，因为melis 只监听的5 个，故最多只能创建5 个。



##### 6.4.2.3 节点通信

rpmsg 节点支持标准的文件操作，直接读写即可。
Linux 向e907 发数据：

![image-20221121120107001](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121120107001.png)

<center>图6-10: rpmsg test</center>

![image-20221121120127394](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121120127394.png)

<center>图6-11: rpmsg test</center>

e907 向Linux 发数据：

![image-20221121120151355](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121120151355.png)

<center>图6-12: rpmsg test</center>

![image-20221121120206688](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121120206688.png)

<center>图6-13: test</center>

##### 6.4.2.4 节点关闭

Linux 主动释放：

![image-20221121120236147](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121120236147.png)

<center>图6-14: rpmsg test</center>

![image-20221121120250818](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121120250818.png)

<center>图6-15: rpmsg test</center>

e907 主动释放：

![image-20221121120354607](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121120354607.png)

<center>图6-16: rpmsg test</center>

![image-20221121120410691](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121120410691.png)

<center>图6-17: rpmsg test</center>

e907 端接触监听，会释放所有的链接：

![image-20221121120433076](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121120433076.png)

<center>图6-18: rpmsg test</center>

![image-20221121120447261](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121120447261.png)

<center>图6-19: rpmsg test</center>

