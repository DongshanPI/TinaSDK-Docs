## 4 使用例子

### 4.1 uboot shell 使用

#### 4.1.1 sunxi_flash

mem_addr：内存地址，0x40000000 之后可以随便选取如：0x45000000，0x46000000
part_name：分区文件名，boot-resource、env、boot、rootfs
size：可以省略，默认读取整个分区文件

1. sunxi_flash read [size] 读取flash 中的分区文件到内存中
   例：使用sunxi_flash read 命令将boot 分区读入到0x49000000 中，然后使用md 命令读取
   0x49000000 中的内容。

![image-20221216112650227](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216112650227.png)

验证方法：

1. 0x49000000 读入前与读入后数据有没有发生变化
2. 在out/pack_out 目录下找到对应的分区文件，使用hexdump -Cv boot.fex -n 500 命
   令输出分区文件的数据，对比一致即读入成功。

![image-20221216112713299](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216112713299.png)

2. sunxi_flash write [size] 将内存中的数据，写入到分区中
   例：
   1）使用mm 命令修改内存内容

![image-20221216112743939](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216112743939.png)

2）使用sunxi_flash write 0x44000000 env 将内存中的数据写入env 分区

![image-20221216112759532](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216112759532.png)

3）重新将env 分区读入内存中，对比一致表示写入成功

![image-20221216112809980](https://photos.100ask.net/Tina-Sdk/Linux_Nor_DevGuide_image-20221216112809980.png)