# 2 经验性能值

Flash 性能与实际使用物料有关，受不同存储介质、不同厂家、不同型号甚至不同老化程度的影响，所以经验值仅供参考。

## 2.1 顺序读写性能经验值

| IC    | 物料类型 | Flash 型号       | 顺序读性能 | 顺序写性能 | 其他说明 |
| ----- | -------- | ---------------- | ---------- | ---------- | -------- |
| R16   | raw nand | K9F4G08U0F       | 40M/s      | 5.6M/s     |          |
| R6    | spi nand | MX35LF1GE4AB     | 4M/s       | 2M/s       | 见注1    |
| R6    | spi nor  | FM25Q128         | 5.6M/s     | 3.1M/s     | 见注2    |
| R30   | mmc      | KLM8G1WEPD-B031  | 77M/s      | 7.7M/s     | 见注3    |
| R333  | spi nand | F50L1G41LB       | 5.7M/s     | 2.0M/s     | 见注1    |
| R328  | spi nand | DS35X1GAXXX      | 12.1M/s    | 4M/s       | 见注4    |
| R328  | spi nand | W25N01GVZE1G     | 6.9M/s     | 2.7M/s     | 见注5    |
| R329  | spi nand | GD5F1GQ4UBYIG    | 7.5M/s     | 2.9M/s     | 见注6    |
| R528  | spi nand | GD5F1GQ4UBYIG    | 5.1M/s     | 2.8M/s     | 见注6    |
| MR813 | mmc      | THGBMBG7D2KBAIL  | 165.56M/s  | 32.18M/s   | 见注7    |
| R528  | mmc      | THGBMJG6C1LBAB7  | 62.5M/s    | 17.4M/s    | 见注8    |
| R528  | mmc      | KLM8G1GESD       | 63M/s      | 20.4M/s    | 见注8    |
| R528  | mmc      | KLM8G1GESD       | 61.8M/s    | 39.5M/s    | 见注9    |
| D1    | spi nand | MX35LF2GE4AD     | 4.8M/s     | 2.9M/s     | 见注10   |
| V853  | emmc     | THGBMJG6C1LBAU7  | 156M/s     | 25M/s      | 见注7    |
| V853  | emmc     | THGBMJG6C1LBAU7  | 69M/s      | 25M/s      | 见注8    |
| V853  | spinand  | MX35LF1GE4AB-Z4I | 7.7M/s     | 2.9M/s     | 见注11   |
| V853  | spi nor  | GD25F256         | 7.4M/s     | 270KB/s    | 见注4    |

**说明**

1. **单线写，双线读，100MHz。**
2. **单线写，单线读，50MHz。**
3. **HS400，50MHz，8 线。**
4. **四线读写，100MHz。**
5. **ubifs，非压缩，四线读写，100MHz。**
6. **ubifs，lzo 压缩，50% 随机数据，四线读写，100MHz，performance 调频策略。**
7. **hs400，100MHz，8 线。**
8. **hs200，150MHz，4 线, 1.8V。**
9. **hs200，150MHz，4 线, 1.8V, 不带O_SYNC**
10. **ubifs，lzo 压缩，50% 随机数据，四线读写，100MHz。performance 调频策略, cpu 频率1440000Hz, dram频率792MHz;**
11. **ubifs，lzo 压缩，50% 随机数据，四线读写，100MHz。performance 调频策略, cpu 频率1104000Hz, dram频率936MHz;**



## 2.2 随机读写性能经验值

| IC    | 物料类型 | Flash 型号       | 顺序读性能 | 顺序写性能 | 其他说明 |
| ----- | -------- | ---------------- | ---------- | ---------- | -------- |
| R6    | raw nand | K9F4G08U0F       | 2486       | 146        | 见注1    |
| R333  | spi nand | F50L1G41LB       | 959        | 266        | 见注1    |
| R329  | spi nand | GD5F1GQ4UBYIG    | 1890       | 592        | 见注2    |
| R528  | spi nand | GD5F1GQ4UBYIG    | 907        | 385        | 见注2    |
| MR813 | mmc      | THGBMJG6C1LBAU   | 6015       | 1596       | 见注3    |
| R528  | mmc      | THGBMJG6C1LBAB7  | 2657       | 830        | 见注4    |
| R528  | mmc      | THGBMJG6C1LBAB7  | 2657       | 830        | 见注4    |
| R528  | mmc      | KLM8G1GESD       | 2582       | 872        | 见注4    |
| R528  | mmc      | KLM8G1GESD       | 2038       | 2220       | 见注5    |
| D1    | spi nand | MX35LF2GE4AD     | 919        | 425        | 见注6    |
| V853  | emmc     | THGBMJG6C1LBAU7  | 4407       | 1363       | 见注3    |
| V853  | emmc     | THGBMJG6C1LBAU7  | 3833       | 1287       | 见注4    |
| V853  | spinand  | MX35LF1GE4AB-Z4I | 1773       | 590        | 见注7    |

**说明**

1. **单线写，双线读，100MHz**
2. **ubifs，lzo 压缩，50% 随机数据，四线读写，100MHz，performance 调频策略。**
3. **hs400，100MHz，8 线**
4. **hs200，150MHz，4 线, 1.8V**
5. **hs200，150MHz，4 线, 1.8V, 不带O_SYNC**
6. **ubifs，lzo 压缩，50% 随机数据，四线读写，100MHz，performance 调频策略, cpu 频率1440000Hz, dram频率792MHz;**
7. **ubifs，lzo 压缩，50% 随机数据，四线读写，100MHz。performance 调频策略, cpu 频率1104000Hz, dram频率936MHz;**