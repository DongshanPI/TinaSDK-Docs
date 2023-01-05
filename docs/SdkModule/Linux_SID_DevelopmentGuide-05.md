## 5 可测试性

1. /sys/class/sunxi_info/sys_info
   此节点文件可以打印出一些SoC 信息，包括版本信息、ChipID 等：

```
# cat /sys/class/sunxi_info/sys_info

sunxi_platform : sun50iw10p1
sunxi_secure : secure
sunxi_chipid : 00000000000000000000000000000000
sunxi_chiptype : 00000400
sunxi_batchno : 0x1
```

2. /sys/class/sunxi_info/key_info
   此节点用于获取指定名称的Key 信息。方法是先写入一个Key 名称，然后就可以读取到Key 的内容。执行效果如下：

```
# echo chipid > /sys/class/sunxi_info/key_info ; cat /sys/class/sunxi_info/key_info
0xf1c1b200: 0x00000400
0xf1c1b204: 0x00000000
0xf1c1b208: 0x00000000
0xf1c1b20c: 0x00000000
```



## 6 其他说明

当启用安全系统后，Non-Secure 空间将无法访问大部分的Efuse 信息，这个时候需要通过SMC 指令来读取这些Key 信息。此时不能再使用普通的寄存器读接口readl()，而是调用的SMC 接口：

目前，sunxi_smc_readl() 的实现在源代码sunxi-smc.c，该文件保存在drivers/char/sunxisysinfo。

```
int sunxi_smc_readl(phys_addr_t addr)
```

目前，sunxi_smc_readl() 的实现在源代码sunxi-smc.c，该文件保存在drivers/char/sunxisysinfo。