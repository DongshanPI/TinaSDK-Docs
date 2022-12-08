# MPP_Sample_使用说明
## 4 编译

本章节主要介绍 MPP sample 的编译方法。 

以下以 tina 为例。 

编译命令

```
cleanmpp && mkmpp
```

编译后 MPP sample 测试程序和配置文件存放位置

```
有两个位置：
（1）每个 MPP sample 的源码目录下
tina\external\eyesee-mpp\middleware\sun8iw21\sample\sample_xxx\
（2）统一存放到bin目录下
tina\external\eyesee-mpp\middleware\sun8iw21\sample\bin\
```

PS：由于 MPP sample 测试程序个数较多，且每个测试程序大小约 10M 左右，故不方便打包 到文件系统中，测试时需要接 SD 卡，同时把测试程序和配置文件放到 SD 卡中进行测试。