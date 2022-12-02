# 常见问题
## 烧写默认系统
* 参考操作演示视频教程
<iframe width="800" height="600"
  src="//player.bilibili.com/player.html?aid=677495111&bvid=BV1Rm4y1X7GZ&cid=462849541&page=1">
</iframe>

### 准备工作
烧写系统是在uboot命令行下进行操作，在操作之前您需要确认以下几个问题：

 1. 开发板可以启动进入到uboot命令行，如果不行请点击页面 [编译烧写Boot](DongshanPi-One/06-BuildFlashBoot/) 章节进行烧写更新。
 2. 由于东山Pi壹号 开发板没有 网卡接口，所以烧录是通过SD卡进行烧录，
    1. 所以您需要提前准备一个分区格式为 `FAT32` `32GB`及以下容量的TF卡，以及`读卡器`。
 3. 烧写默认系统是烧写了所有的分区，所以它会擦除并重新写入文件到所有的分区，请提前将重要文件拷贝出来。

### 烧写更新
* 下载默认系统镜像
 * 点击下载链接：https://cowtransfer.com/s/c82c42aabbf041 
下载后会得到一个名为 `images.zip` 的压缩包，需要先用压缩工具进行解压缩，解压缩后的文件列表如下。
``` shell
appconfigs.ubifs
auto_update.txt
auto_update_bin.txt
boot/
cis.bin
customer.ubifs
ipl_cust_s.bin
ipl_s.bin
kernel
logo
miservice.ubifs
rootfs.ubifs
scripts/
scripts_bin/
uboot_s.bin
```

* 拷贝到tf卡内
首先将已经准备好的分区格式为FAT32的TF卡连接至电脑，将上述解压出来的文件 全部拷贝至tf卡内。
拷贝完成后,即可从电脑弹出tf卡，将卡插入 东山Pi开发板的 tf卡卡座内，完全按压进去(注意tf卡不可超过32G)。

* 进入uboot命令行执行烧写命令
  此时确保开发板已经可以通过串口工具进行交互，按下复位键，紧接着在有启动信息打印时一直长按键盘 `enter` 回车键三秒左右松开，即可进入uboot命令行界面内。
  进入uboot命令行以后 可以直接输入 `dstar` 命令，就会自动进行更新，等待更新 结束 自动重启系统即可。
![SomeQustion-01](https://cdn.staticaly.com/gh/codebug8/DongshanPi-Photos@master/SomeQustion-01.png)  

更新系统完整的log 输出，仅限参考：  

## 其它常见问题

### 无串口输出信息
* 检查是否连接成功 电脑是否安装串口驱动
* 检查串口波特率是否设置正确
* 检查串口是否被其它工具拦截
  
### 无法烧写系统
* 检查开发板是否进入uboot模式
* 是否可以加载SD卡内镜像到系统内
* 检查是否可以烧写
  
###  屏幕无任何显示
* 检查连接是否稳定。
* 查看屏幕背光是否亮起。
* 检查是否有操作屏幕进行显示。