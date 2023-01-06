## 8 其他

### 8.1 rpmsg 需知

1. 端点是rpmsg 通信的基础；每个端点都有自己的src 和dst 地址，范围（1 - 1023，除了0x35）

2. rpmsg 每次发送数据最大为512 -16 字节；（数据块大小为512，头部占用16 字节）

3. rpmsg 使用name server 机制，当E907 创建的端点名，和linux 注册的rpmsg 驱动名一样的时候，rpmsg bus 总线会调用其probe 接口。所以如果需要

  Linux 端主动发起创建端点并通知e907，则需要借助上面提到的rpmsg_ctrl 驱动。

4. rpmsg 是串行调用回调的，故建议rpmsg_driver 的回调中不要调用耗时长的函数，避免影响其他rpmsg 驱动的运行

### 8.2 rpbuf 简介

rpbuf 全志基于rpmsg 开发的一套通信机制，它主要解决rpmsg 不适合传输大数据量的问题。
其实现原理是使用rpmsg 传输数据的地址，而不是数据的本身，避免了数据的多次拷贝以及每次
传输不能大于496 字节的限制。
rpbuf 中使用名字和长度来唯一标识一个buffer，故不能创建相同名字的buffer。
rpbuf 中的buffer 有3 个状态：

1. remote_dummy_buffers：该buffer 远端已创建，本地未创建
2. local_dummy_buffers：该buffer 本地已创建，远端未创建
3. buffers：远端、本地已创建，此时buffer 才可用

### 8.3 修改e907 地址

目前在perf1 板子上，给e907 预留的内存为：0x48000000 开始的4M 空间

如果需要修改E907 固件的运行地址和大小，可按如下步骤进行修改：

#### 8.3.1 修改设备树(Linux)

```
cconfigs
vim ../board.dts
# 找到e907_dram项，修改成想要的地址，例如这里向修改成0x49000000
e907_dram: riscv_memserve {
reg = <0x0 0x49000000 0x0 0x00400000>;
no-map;
};
# 重新编译内核
mkernel
```

#### 8.3.2 修改配置项(melis)

```
mmelis menuconfig
# 如下图进行修改;e907没有mmu，故第一项和第二项相等
# 将第一项和第二项改成0x49000000
# 第三项为大小,可按需修改
```

![image-20221121183344375](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121183344375.png)

<center>图8-1: e907 dram config</center>

#### 8.3.3 修改链接脚本(melis)

```
cmelis
vim source/projects/v853-e907-ver1-board/kernel.lds
# 如下图所示,按照所需修改DRAM_SEG_KRN 项目
mmelis -j16
```

![image-20221121183431822](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221121183431822.png)

<center>图8-2: e907 lds config</center>

### 8.4 添加新板级注意事项

当用户需要添加新的板子时，需要注意修改build/expand_melis.sh 来支持mmelis, cmelis命令。例如，用户在添加了新的板级v853_user，在melis 添加了新的板

级e907_user，则需要对build/expand_melis.sh文件进行如下修改：

![image-20221122091718934](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221122091718934.png)

<center>图8-3: 新板级配置</center>

### 8.5 melis 系统

#### 8.5.1 常用命令

1. help：列出当前系统支持的所有命令
2. p addr [len]：打印内存数据
3. m addr value：修改内存数据
4. top：显示当前系统各个线程CPU 占用率
5. ps：显示当前系统各个线程状态
6. free：查看当前系统内存信息

#### 8.5.2 自定义命令

当用户想要在e907 控制台上执行自定义的命令时候，可以用FINSH_FUNCTION_EXPORT_ALIAS导出自定义的函数。例如：

![image-20221122091828089](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221122091828089.png)

<center>图8-4: 添加自定义命令</center>

![image-20221122091847075](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_E907_DevGuide_image-20221122091847075.png)

<center>图8-5: 执行自定义命令</center>

