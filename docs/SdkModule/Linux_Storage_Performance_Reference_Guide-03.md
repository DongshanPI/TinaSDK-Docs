# 3 顺序读写性能

## 3.1 顺序写性能理论值计算

物料的数据手册一般会提供擦除和写的耗时，关注数据手册中Block Erase time、PageProgram time 此类关键字数值。以GD25Q256E（spi nor）为例，Block Erase time:0.12s/0.15s typical(Block size 64KByte) ,Page Program time: 0.25ms typical(Page
size 256Byte)。

```c
| High Speed Clock Frequency | Fast Program/Erase Speed |
| 133MHz for fast read with 30PF load | Page Program time: 0.25ms typical |
| Dual I/O Data transfer up to 266Mbits/s | Sector Erase time: 45ms typical |
| Quad I/O Data transfer up to 532Mbits/s | Block Erase time: 0.12s/0.15s typical |
```

上面的Quad 的传输速率，是通过133MHZ * 4 line 计算到的，是一个理论数据，而实际的使用场景，我们要读数据前要用1 line 发送5(6)Bytes 数据，即cmd + addr[3(4)] + dummy(大于16M 的FLASH，需要发4byte 地址), 其次我们SPI 控制器最大输出频率100Mhz。假设
发一次命令读N bytes 数据，则命令和数据所占时间的比例为5 : (N/4), 那么实际4 line 的极限速度等于(N/4) / [5+(N/4)] * CLK * 4Mbits/s。以100Mhz 4line 为例，理论极限速度为47.68MB/s。

```c
speed = 32MByte/(erase_time + program_time + spi_time)
= 32MByte/(0.15s*(32MByte/64KByte) + 0.25ms*(32MByte/256Byte) + 32MByte/47.68MB/s)
= 32MByte/(76.8s + 32.8s +0.67s)
= 290KByte/s
```

以GD5F1GQ4UAYIG（spinand）为例，Block Erase time: 3ms typical(Block size128KByte) ,Page Program time: 0.4ms typical(Page size 2048Byte)。

```c
speed = 128MByte/(erase_time + program_time)
= 128MByte/(3ms*(128MByte/128KByte) + 0.4ms*(128MByte/2048Byte) + 128MByte/47.68MB/s)
= 128MByte/(3072ms + 26214.4ms + 2684ms)
= 4.0MByte/s
```

## 3.2 顺序性能测试方法

Tina 测试平台有2 个顺序读写性能的测试用例，分别如下。

```c
/spec/storage/seq #适用于>64M 内存的方案
/spec/storage/tiny-seq #适用于<=64M 内存的方案和使用ubifs的存储方案
```

特别注意的是，在测试文件数据量非常小时，内存对测试影响太大，测试出来的读数据会非常不准确。例如，对spinor 的测试分区只有5M 大小，而内存有64M，测试出的读可能达到100+M/s，此时的读数据不具有参考价值。

## 3.3 顺序性能解读

顺序读写性能以读写速度(KB/s;MB/s) 作为衡量标准，主要体现大文件连续读写的性能。此时，速度值越大，顺序读写性能越好。
不同存储介质的读写性能是有差异的，甚至同一种存储介质，不同厂家不同型号可能都有差别。
以mmc 为例，有的mmc 写性能只能达到10M/s，而有的mmc 写性能达到150M/s。一般来说，MMC 的规格书中有体现性能估值。
常见的，不同介质顺序读写性能排序如下。

```c
读： mmc > raw nand > spinor > spinand
写： mmc > raw nand > spinand > spinor
```



# 4 随机读写性能

## 4.1 随机性能测试方法

Tina 测试平台有1 个随机读写性能的测试用例，且只适用于>64M内存方案。

```c
/spec/storage/rand
```

## 4.2 随机性能解读

随机读写性能以IOPS(IO per second) 为衡量标准，理解为每秒处理多少个IO 请求。此指标反应的是小文件的读写性能。此数值越高，表示其随机读写性能越好。
与顺序读写相似的是，其数值也与实际物料，当前工作模式有关。