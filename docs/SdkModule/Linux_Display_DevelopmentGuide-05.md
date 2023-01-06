## 6 输出设备介绍

平台支持屏以及HDMI 输出，及二者同时显示。

### 6.1 屏

屏的接口很多，平台支持RGB/CPU/LVDS/DSI 接口。

### 6.2 HDMI

HDMI 全名是：High-Definition Multimedia Interface。可以提供DVD,audio device, settop boxes，television sets, and other video displays 之间的高清互

联。可以承载音，视频数据，以及其他的控制，数据信息。支持热插拔，内容保护，模式是否支持的查询。

### 6.3 同显

驱动支持双路显示。屏（主）+ HDMI（辅）。

同显或异显，差别只在于显示内容，如果显示内容一样，则为同显；反之，则为异显。

1. 如果是android 系统，4.2 版本以上版本，原生框架已经支持多显（同显，异显，虚拟显示设备），实现同显则比较简单，在android hal 与上层对接好即可。

2. 如果是android 4.1 以下版本，同显需要自行实现，参考做法为主屏内容由android 原生提供，辅屏需要android hal 在合适的时机（比如HDMI 插入时）打开

  辅屏，并且将主屏的内容（存放于FB0 中），拷贝至辅屏的显示后端buffer 中，然后将辅屏的后端buffer 切换到前端buffer。注意问题为，两路显示的显示

  buffer 的同步，如果同步不好，会产生图像撕裂，错位的现象。

3. 如果是Linux 系统，做法与上一个做法类似。