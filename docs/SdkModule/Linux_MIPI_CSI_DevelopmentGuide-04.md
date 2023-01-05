## 4 模块使用范例

### 4.1 测试 demo

模块使用的 demo 的代码位于 drivers/media/platform/sunxi-vin/vin_test/mplane_image；此目录下可以直接 make 生成 demo；把 demo 推到机器里面执行便可以获取指定 video 节点的图像。推荐在 pc 上创建 bat 批处理文件，使用 adb 命令完成一系列抓图的动作，bat 内容参考如下，不同机器请注意修改 push 进去的路径：

```
del .\result\*.bin
adb root
adb remount
adb shell "mkdir /vendor/extsd/"
adb shell "mkdir /vendor/extsd/result"
adb shell rm /vendor/extsd/result/*.bin
adb push demo路径\csi_test_mplane /vendor/extsd/csi_test1
adb shell chmod 777 /vendor/extsd/csi_test1
adb shell "cd /vendor/extsd/ && ./csi_test1 0 0 1920 1080 ./result 1 20000 60 0"
adb shell ls /vendor/extsd/result
adb pull /vendor/extsd/result
pause
```

最后会在 bat 指令的文件夹生成 result 文件夹里面保存二进制的图像数据 *.bin 文件；可用 RawViewer 等软件查看图像数据。demo 参数说明：0 0 1920 1080 ./result 1 20000 60 0，分别表示 video0，set_input index0，目标分辨率宽，目标分辨率高，bin 文件保存路径、图像格式（如 NV21，具体含义可以看 demo 代码的 s_fmt 参数）、采集帧数（帧数大于 10000 即为常开节点）、目标帧率、和是否开启 wdr。



### 4.2 调用流程

![](http://photos.100ask.net/tina-docs/LinuxMIPICSIDevelopmentGuide_005.png)

​														  		图 4-1: CSI 调用流程	