# LVGL界面开发

## 必备硬件
* 一个7寸的RGB显示屏
* 一个可以运行ubuntu系统的PC电脑
* ubuntu电脑可以和开发板之间网络互通

## 获取配套资源
> 获取lvgl开发配套的交叉编译器以及系统镜像。

* 自行获取：打开 https://github.com/DongshanPI/buildroot-external-dongshanpiseven/releases/ 页面点击最新的版本，找到名为 `dongshanpiseven_standard-lvgl_defconfig.tar.gz` 的压缩包，点击下载。
* 备用站点:  https://github.91chi.fun/https://github.com//DongshanPI/buildroot-external-dongshanpiseven/releases/download/v0.3.1-beta/dongshanpiseven_standard-lvgl_defconfig.tar.gz

下载完成后使用解压缩工具进行解压缩，之后使用烧写工具烧写系统镜像至开发板内，最后并将配套的`arm-buildroot-linux-gnueabihf_sdk-buildroot.tar.gz` 交叉编译工具链拷贝至 Ubuntu系统内，并配置开发环境。

## 烧写系统镜像
### 烧写镜像至EMMC
参考左侧 烧写&恢复系统 章节将下载好的镜像文件烧写至 EMMC内。
等待烧写完成后启动即可。

### 烧写镜像至SDcard
使用wind32diskimage 工具讲 sdcard.img烧录进tf卡内，开发板设置为tf卡启动模式，启动开发板。

## 配置LVGL开发环境
进入ubuntu系统内，先将 arm-buildroot-linux-musleabihf_sdk-buildroot.tar.gz  拷贝到系统内，之后执行`tar -xvf arm-buildroot-linux-musleabihf_sdk-buildroot.tar.gz -C ~`进行解压缩，之后进入 `arm-buildroot-linux-musleabihf_sdk-buildroot.tar.gz/bin`使用   `pwd`命令，将获取到的绝对路径增加到系统 PATH 环境变量里，我们推荐设置到 `~/.bashrc` 。
* 假如我这里到了 `/home/book `目录下，那么完整的路径是 ` /home/book/arm-buildroot-linux-musleabihf_sdk-buildroot/bin` 之后就可以在 ~/.bashrc 行尾增加 
`export PATH=$PATH:/home/book/arm-buildroot-linux-musleabihf_sdk-buildroot/bin` 设置完成后 就可以使用 source ~/.bashrc 命令使其生效。

**注意：一定不要解压缩到windows挂载的虚拟网络目录内，由于文件类型等问题，会导致后续出现很多不可预测的问题。**
设置成功后 可以执行 `arm-buildroot-linux-musleabihf-gcc -v ` 有版本输出 即表示配置成功。

### 运行第一个LVGL程序


## 问题反馈与建议
* 请在这个页面下面留言 https://github.com/DongshanPI/buildroot-external-dongshanpiseven/discussions/2