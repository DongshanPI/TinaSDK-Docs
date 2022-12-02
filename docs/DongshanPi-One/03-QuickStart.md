# 快速开始使用
如果您感觉跟着文档不知道怎么操作，可以直接查看我们提前录制好的手把手教学视频。

## 上电启动系统
### 1. 连接串口线
将配套的TypeC线一段连接至开发板的串口/供电接口，另一端连接至电脑USB接口，连接成功后板载的红色电源灯会亮起。
默认情况下系统会自动安装串口设备驱动，如果没有自动安装，可以使用驱动精灵来自动安装。

* 对于Windows系统
此时Windows设备管理器 在 端口(COM和LPT) 处会多出一个串口设备，一般是以 `USB-Enhanced-SERIAL CH9102`开头，您需要留意一下后面的具体COM编号，用于后续连接使用。

![](https://cdn.staticaly.com/gh/codebug8/DongshanPi-Photos@master/QuickStart-01.png)
如上图，COM号是96，我们接下来连接所使用的串口号就是96。


* 对于Linux系统
可以查看是否多出一个/dev/tty<> 设备,一般情况设备节点为 /dev/ttyACM0  。
![](https://cdn.staticaly.com/gh/codebug8/DongshanPi-Photos@master/QuickStart-02.png)

### 2. 打开串口控制台
#### 获取串口工具
使用Putty或者MobaXterm等串口工具来开发板设备。

* 其中putty工具可以访问页面  https://www.chiark.greenend.org.uk/~sgtatham/putty/  来获取。
* MobaXterm可以通过访问页面 https://mobaxterm.mobatek.net/ 获取 (推荐使用)。


#### 使用putty登录串口
![登录截图](https://cdn.staticaly.com/gh/codebug8/DongshanPi-Photos@master/QuickStart-04.png)

#### 使用Mobaxterm登录串口
打开MobaXterm，点击左上角的“Session”，在弹出的界面选中“Serial”，如下图所示选择端口号（前面设备管理器显示的端口号COM21）、波特率（Speed 115200）、流控（Flow Control: none）,最后点击“OK”即可。步骤如下图所示。
**注意：流控（Flow Control）一定要选择none，否则你将无法在MobaXterm中向串口输入数据**

![连接](https://cdn.staticaly.com/gh/codebug8/DongshanPi-Photos@master/QuickStart-05.png)

### 3. 进入系统shell
使用串口工具成功打开串口后，可以直接按下 Enter 键 进入shell，当然您也可以按下板子上的 `Reset`复位键，来查看完整的系统信息。
![](https://cdn.staticaly.com/gh/codebug8/DongshanPi-Photos@master/QuickStart-06.png)

注意：星辰原厂系统没有登录密码，直接启动进入文件系统内shell终端。


## 扩展硬件

### 连接模块

#### 连接7寸屏模块
* 连接视频教程


* 模块介绍 

### 连接底板
* 连接视频教程

* 底板介绍

## 简述开发流程

### 调试环境概述 