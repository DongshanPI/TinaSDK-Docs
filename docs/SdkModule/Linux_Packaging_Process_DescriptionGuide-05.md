# 6 打包常见问题FAQ

• Q1: 镜像文件的大小超过规划的分区大小

```
...
ERROR: dl file rootfs.fex size too large
ERROR: filename = rootfs.fex
ERROR: dl_file_size = 5888 sector
ERROR: part_size = 4864 sector
ERROR: update mbr file fail
ERROR: update_mbr failed
```

A：这里表明rootfs 分区所关联的rootfs.fex 镜像文件(dl_file_size: 5888 x 512 byte) 大于分区规划的大小(part_size = 4864 x 512 byte)
需要将sys_partition.fex (NOR 案：sys_partition_nor.fex) 里面的分区大小改大，以便可以装下分区镜像的大小。
**说明**

```
sys_partition.fex 里面的分区都需要按照flash的擦除块大小来对齐。例如SPINOR Flash, 应该按照64K
来对齐。
```

• Q2: 找不到指定的分区镜像

```
...
ERROR: unable to open file usr.fex
update_for_part_info -1
ERROR: update mbr file fail
ERROR: update_mbr failed
```

A: 这个错误是说明你设定的分区指定了分区镜像文件，但在打包的时候没有找到。像这个usr.fex 是需要开启[CONFIG_SUNXI_SMALL_STORAGE_OTA] 才会主动生成的。其他自建的分区镜像文件，改后缀名xx.fex 并放以下目录即可自动找到：

```
device/config/chips/${TARGET_PLATFORM}/configs/${TARGET_PLAN}/linux
```

或者在sys_partition.fex(NOR 方案：sys_partition_nor.fex) 里面写入绝对路径

```
...
downloadfile = "/home/aw1315/img/DIY.fex"
```

• Q3: NOR 方案分区设置太大

```
...
load file: env_nor.fex ok
load file: bootlogo.fex ok
error:offset(67252224) is too large MAX_IMAGE_SIZE:67108864!
merge_package fail
```

```
...
scripts/pack_img.sh: line 1441: 73617 Segmentation fault (core dumped) merge_full_img
--out full_img.fex --boot0 boot0_spinor.fex --boot1 ${BOOT1_FILE} --mbr ${mbr_file} --
logic_start ${LOGIC_START} --uboot_start ${UBOOT_START} --partition sys_partition_nor.
bin
ERROR: merge_full_img failed
```

A：以上两种情况都是属于在sys_partition_nor.fex 里面设置太大的分区了，需要到合适大小。考虑到SPINOR Flash 不会超过64M, 所以做了这个限制。这个也是merge_full_img 这个应用的缺陷，如果确实需要打包这么大的Nor 固件，可能需要把merge_full_img 这个工具注释掉。

```
#create img for nor programmer
merge_full_img --out full_img.fex \
--boot0 boot0_spinor.fex \
--boot1 ${BOOT1_FILE} \
--mbr ${mbr_file} \
--logic_start ${LOGIC_START} \
--uboot_start ${UBOOT_START} \
--partition sys_partition_nor.bin
if [ $? -ne 0 ]; then
pack_error "merge_full_img failed"
exit 1
fi
```

将scripts/pack_img.sh 里面两处有上述命令的地方，使用“#” 号注释掉即可。
**说明**
**merge_full_img 只是用来做烧录器固件，不影响全志固件的生成。**