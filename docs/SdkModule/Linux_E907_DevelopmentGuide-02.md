## 5 E907 启动环境

### 5.1 预先工作

选择方案

```
cd tina
source build/envsetup.sh
lunch
选择对应的V85x方案
```

### 5.2 配置boot0 启动e907

e907 在boot0 阶段启动，需要对boot0 进行一些配置

```
cboot0
vim board/sun8iw21p1/common.mk
# 如下图取消注释
保存退出
mboot0 #编译
```

![image-20221121115033101](http://photos.100ask.net/tina-docs/Linux_E907_DevGuide_image-20221121115033101.png)

<center>图5-1: 配置1</center>

#### 5.2.1 关闭RISCV 的IOMMU

本步骤只有需要在boot0 阶段启动E907 的需要配置。打开设备树，注释掉下面2 条属性，因为
e907 在boot0 阶段就启动了，不能打开其IOMMU。

```
cconfigs
vim ../board.dts
```

![image-20221121115124735](http://photos.100ask.net/tina-docs/Linux_E907_DevGuide_image-20221121115124735.png)

<center>图5-2: 关闭IOMMU</center>

### 5.3 配置打包e907 固件

```
cconfigs
cd ../../default/
vim boot_package_nor.cfg # 取消melis-elf选项的注释,如下图
vim boot_package.cfg # 取消melis-elf选项的注释，如下图
保存退出
```

![image-20221121115200889](http://photos.100ask.net/tina-docs/Linux_E907_DevGuide_image-20221121115200889.png)

<center>图5-3: 打包配置</center>

### 5.4 Linux 配置

```
ckernel
m kernel_menuconfig
# 如下图选中2个驱动
mkernel -j
```

![image-20221121115234331](http://photos.100ask.net/tina-docs/Linux_E907_DevGuide_image-20221121115234331.png)

<center>图5-4: 补丁下载</center>

```
mmelis menuconfig # 如下图选中standby支持
```

![image-20221121115354868](http://photos.100ask.net/tina-docs/Linux_E907_DevGuide_image-20221121115354868.png)

<center>图5-5: e907-standby 配置</center>

### 5.5 编译打包

至此关于E907 启动的配置完成，进行编译烧录即可

```
make -j16 # 编译tina
mmelis # 编译melis
p
烧录
```

