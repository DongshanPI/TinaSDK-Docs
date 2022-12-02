# MQTT物联网开发

## 必备硬件
* 有Internet网络
* 一个可以运行ubuntu系统的PC电脑
* ubuntu电脑可以和开发板之间网络互通

## 获取配套资源
> 获取Mqtt开发配套的交叉编译器以及系统镜像。

* 自行获取：打开 https://github.com/DongshanPI/buildroot-external-dongshanpiseven/releases/ 页面点击最新的版本，找到名为 `dongshanpiseven_standard-aiot_defconfig.tar.gz` 的压缩包，点击下载。
* 备用站点:  https://github.91chi.fun//https://github.com//DongshanPI/buildroot-external-dongshanpiseven/releases/download/v0.1.1-beta.1/dongshanpiseven_standard-aiot_defconfig.tar.gz

下载完成后使用解压缩工具进行解压缩，之后使用烧写工具烧写系统镜像至开发板内，最后并将配套的`arm-buildroot-linux-gnueabihf_sdk-buildroot.tar.gz` 交叉编译工具链拷贝至 Ubuntu系统内，并配置开发环境。

## 烧写系统镜像


### 烧写镜像至EMMC


### 烧写镜像至SDcard


## 配置MQTT开发环境


### 运行第一个MQTT程序


## 问题反馈与建议
* 请在这个页面下面留言 https://github.com/DongshanPI/buildroot-external-dongshanpiseven/discussions/2