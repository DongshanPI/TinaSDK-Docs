## 7 注意事项

### 7.1 Q & A

Q：系统的哪些部分是可以升级的？
A：kernel 和rootfs 是可以升级的，但为了掉电安全，需要搭配一个recovery 系统或者做AB 系统。对于nand 和emmc 来说，boot0/uboot 存在备份，可以升

级。对于nor来说，boot0/uboot 没有备份，不能升级。或者说升级有风险，中途掉电会导致无法启动。boot0/uboot 的升级具体可参考本文档中的ota-burnboot 

部分。对于swupdate 升级方案，可以自行在sw-description 中配置策略，升级自己的定制分区和文件，但务必考虑升级中途掉电的情况，必要的话需要做备份和

恢复机制。



Q：系统的哪些部分是不能升级的？
A：分区表是不可升级的，因为改动分区表后，具体分区对应的数据也要迁移。建议在量产前规划好分区，为每个可能升级的分区预留部分空间，防止后续升级空

间不足。nor 方案的boot0/uboot 是不可升级的，因为没有备份。其余不确定是否能够升级的，请向开发人员确认。



Q：dts/sys_config 如何升级？
A：默认dts 和sys_config，会跟uboot 绑定生成一个bin 文件。因此升级uboot 实质上是升级了uboot+dts/sys_config。



Q：能否单独升级dts？
A：目前默认跟uboot 绑定，需要跟开发人员确认如何将dts 独立出来，放到独立分区或者跟kernele 绑定到一起。　如果是dts 位于独立分区，那么就需要修改配

置，将dts 放置到OTA 包中，OTA 时写入到对应分区。

Q：升级过程掉电重启后，是从断点继续升级还是从头升级？
A：从头开始升级，例如定义了在recovery 系统中升级boot 分区和rootfs 分区，则在升级boot 或rootfs 过程中断电，重启后均是从boot 重新开始升级。



## 8 升级失败问题排查

凡是遇到升级失败问题先看串口log，如果不行再看/mnt/UDISK/swupdate.log 文件

### 8.1 分区比镜像文件小引起的失败

log 大概如下。找error 的地方，可以看到recovery 分区比其镜像小，所以报错。

```
[ERROR] : SWUPDATE failed [0] ERROR handlers/ubivol_handler.c : update_volume : 171 : "
recovery" will not fit volume "recovery"
```

解决方法： 增加对应方案tina/device/config/chips/< 芯片编号>/configs/< 方案>/sys_partition.fex 文件的对应分区的大小

### 8.2 校验失败

差分有严格的版本控制，当出现checksum 有问题时，基本可以归类为这种问题。

解决方法：重新烧录固件，制作差分包