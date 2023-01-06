# 6 rootfs_data 及UDISK

## 6.1 overlayfs 简介

Tina 默认根文件系统格式使用squashfs 格式，这是一种只读压缩的文件系统。很多应用则需要文件系统可写，特别是/etc 等存放较多配置文件的目录，为了满足

可写的需求，Tina 默认使用overlayfs 技术。overlayfs 是一种堆叠文件系统，可以将底层文件系统和顶层文件系统的目录进行合并呈现。

## 6.2 使用rootfs_data 作为overlayfs

Tina 常用的方式是专门划分一个rootfs_data 分区， 先格式化成可写的文件系统（如ext4/ubifs）， 再进一步挂载为overyfs， 成为新的根， 让上层应用认为

rootfs 是可写的。

rootfs_data 分区的大小就决定了应用能修改多少文件。具体原理和细节请参考网上公开资料，此处仅举简单例子辅助理解。

1. 底层（即rootfs 分区的文件系统）不存在文件A，应用创建A，则A 只存在于上层（即
   rootfs_data 分区）。
2. 底层存在文件B，应用删除B，则B 仍然存在于底层，但上层会创建一个特殊文件屏蔽掉，导
   致对应用来说B 就看不到了，起到删除的效果。
3. 底层存在文件C，上层修改C，则C 会先被整个拷贝到上层，C 本身多大就需占用多大的上
   层空间，在这个基础上应用对上层的C 进行修改。
   基于以上理解，可按需配置rootfs_data 的大小。一般开发期间会配置得较大，量产时可减小
   （考虑实际只会修改少量配置文件）甚至去除（需要确认所有应用均不依赖rootfs 可写）。

## 6.3 使用UDISK 作为overlayfs

如果希望overlayfs 的空间尽可能较大，也可考虑直接使用UDISK 分区作为上层文件系统空间。

可将target/allwinner/xxx/base-files/etc/config/fstab 中的rootfs_data 分区及UDISK 分区的挂载配置disable 掉。

```
config 'mount'
	option target '/overlay'
	option device '/dev/by-name/rootfs_data'
	option options 'rw,sync,noatime'
	option enabled '0' #此行改成0
config 'mount'
	option target '/mnt/UDISK'
	option device '/dev/by-name/UDISK'
	option options 'rw,async,noatime'
	option enabled '0'
```

再新增一个配置，将UDISK 直接挂载到/overlay 目录。

```
config 'mount'
	option target '/overlay'
	option device '/dev/by-name/UDISK'
	option options 'rw,sync,noatime'
	option enabled '1'
```

## 6.4 如何清空rootfs_data 和UDISK

出于恢复出厂设置或其他需要，有时需要清空rootfs_data 和UDISK。

一般不建议在文件系统仍处于挂载状态时直接操作对应的底层分区，因此建议需要清空时，不要直接操作对应块设备，而是先设置标志并重启，再在挂载对应分区

前的启动脚本中检测到对应标志后，对分区进行重新格式化。

当前79_format_partition 中实现了一个clean_parts 功能， 会检测env 分区中的parts_clean 变量并清空对应分区的头部。清空后，rootfs_data 和UDISK 分区会

自动重新触发格式化。

```
# to clean rootfs_data and UDISK, please run
fw_setenv parts_clean rootfs_data:UDISK
reboot
```

具体实现请查看

```
package/base-files/files/lib/preinit/79_format_partition
```

