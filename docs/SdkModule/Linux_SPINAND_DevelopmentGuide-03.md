## 4 模块配置

### 4.1 uboot 模块配置

```
Device Drivers-->Sunxi flash support-->
[*]Support sunxi nand devices
[*]Support sunxi nand ubifs devices
[*]Support COMM NAND V1 interface
```



如下图：

![image-20221227120338239](https://photos.100ask.net/Tina-Sdk/LinuxSPINANDDevelopmentGuide_003.png)

​																	图 4-1: u-boot-spinand-menuconfig

​	

### 4.2 kernel 模块配置

```
Device Drivers->Memory Technology Device(MTD) support-->sunxi-nand
```

![image-20221227140347684](https://photos.100ask.net/Tina-Sdk/LinuxSPINANDDevelopmentGuide_004.png)

​																	图 4-2: UBI

![image-20221227140501741](https://photos.100ask.net/Tina-Sdk/LinuxSPINANDDevelopmentGuide_005.png)

​																	图 4-3: ker_nand-cfg

![image-20221227140541762](https://photos.100ask.net/Tina-Sdk/LinuxSPINANDDevelopmentGuide_006.png)

​																	图 4-4: ker_spinand

```
Device Drivers->SPI support
```

![image-20221227140943318](https://photos.100ask.net/Tina-Sdk/LinuxSPINANDDevelopmentGuide_007.png)

​																图 4-5: spi-1

![image-20221227141017709](https://photos.100ask.net/Tina-Sdk/LinuxSPINANDDevelopmentGuide_008.png)

​																图 4-6: spi-2



```
Device Drivers->DMA Engine support
```



![image-20221227141059654](https://photos.100ask.net/Tina-Sdk/LinuxSPINANDDevelopmentGuide_009.png)

​																图 4-7: DMA-1

![image-20221227141126258](https://photos.100ask.net/Tina-Sdk/LinuxSPINANDDevelopmentGuide_0010.png)

​																图 4-8: DMA-2



```
Device Drivers->SOC（System On Chip）
```

![image-20221227141153846](https://photos.100ask.net/Tina-Sdk/LinuxSPINANDDevelopmentGuide_0011.png)

​																图 4-9: SID



```
File systems-->Miscellaneous filesystems-->
```

![image-20221227141435508](https://photos.100ask.net/Tina-Sdk/LinuxSPINANDDevelopmentGuide_0012.png)

​																图 4-10: menuconfig_spinand_ubifs



#### 4.3 env.cfg

在 env.cfg 中添加修改下值，setargs_nand_ubi 先 copy 一份 setargs_nand 再添加对应变量

![image-20221227141519171](https://photos.100ask.net/Tina-Sdk/LinuxSPINANDDevelopmentGuide_0013.png)

​																图 4-11: build-mkcmd