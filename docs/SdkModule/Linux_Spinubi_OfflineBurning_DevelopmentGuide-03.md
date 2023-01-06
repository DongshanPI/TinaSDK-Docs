## 7 计算逻辑区域LEB 总数

用户可见LEB 数= 总物理块数- 8 (boot0) - 24 (boot1) - 8 (secure storage) - 20* 总物理块
数/1024 - 4，规则如下：

1. 减去物理区域块数
2. 减去坏块处理预留数（每1024 物理块最多20 个物理块，即10 个逻辑块）
3. 减去4（2 个用于ubi layout volume，1 个用于LEB 原子写，1 个用于磨损均衡处理）推算方式可以参考u-boot-2018/cmd/ubi_simu.c 的ubi_sim_part 和ubi_simu_create_vol函数。
   正常情况下，ubi 方案sys_partition.fex 中各个分区的大小会按照LEB 大小对齐。假如一款flash 有1024 个block, 每个block 有64 个page, 每个page 有2KB，则逻辑块大小为256K(642K2), 那么PEB 大小是256K，LEB 大小为252K, PEB 中的首逻辑页固定用于
   存放ubi_ec_hdr 和ubi_vid_hdr。
   由于预先不知道物料的容量信息及预留块信息，因此sys_partition.fex（sunxi_mbr.fex）中最后一个分区的size 信息默认先填0，待NAND 驱动初始化完成后才知道用户可见LEB 数有多少个，此时需要根据信息改写sunxi_mbr.fex 中最后一个分区的size。

## 8 动态调整sunxi_mbr 卷

sunxi_mbr.fex 共64k, 共4 个备份，每个备份16K

1. 计算mbr 卷最后分区size, 单位：扇区（512 字节），计算规则如下：
   根据第5 章节计算出的用户可见leb 数转化出总的扇区数total_sector，依次减去分区表中各个
   分区占用的扇区数
2. 回填sunxi_mbr.fex 最后一个分区size
3. 重新计算并回填sunxi_mbr 的crc32
4. 改写其余3 个备份

```
sunxi_mbr_t 结构体：u-boot-2018/include/sunxi_mbr.h，结构体各个成员均使用小端存储。
typedef struct sunxi_mbr
{
unsigned int crc32;
unsigned int version;
unsigned char magic[8];
unsigned int copy;
unsigned int index;
unsigned int PartCount;
unsigned int stamp[1];
sunxi_partition array[SUNXI_MBR_MAX_PART_COUNT];
unsigned int lockflag;
unsigned char res[SUNXI_MBR_RESERVED];
}attribute ((packed)) sunxi_mbr_t;
```

重新计算并回填sunxi_mbr crc32 的代码请参考u-boot-2018/drivers/mtd/aw-spinand/sunxiubi.c 的adjust_sunxi_mbr 函数。

## 9 根据sunxi_mbr 动态生成ubi layout volume

ubi layout volume 可以理解为UBI 模块内部用的分区信息文件，sunxi_mbr 分区是用于全志烧写framework 的分区信息文件。二者记录的分区信息本质上是一样

的，因此烧写时, 可以由sunxi_mbr 卷转化成ubi layout volume。

ubi layout volume 由128 个struct ubi_vtbl_record（u-boot-2018/drivers/mtd/ubi/ubimedia.h）组成, 结构体各个成员使用大端表示。

```
struct ubi_vtbl_record {
__be32 reserved_pebs;
__be32 alignment;
__be32 data_pad;
__u8 vol_type;
__u8 upd_marker;
__be16 name_len;
char name[UBI_VOL_NAME_MAX+1];
__u8 flags;
__u8 padding[23];
__be32 crc;
} __packed;
```

| attribute name | type   | value | comment                                              |
| -------------- | ------ | ----- | ---------------------------------------------------- |
| reserved_pebs  | __be32 |       | 卷大小/LEB size, 对于ubi layout volume，固定为2      |
| alignment      | __be32 | 1     |                                                      |
| data_pad       | __be32 | 0     |                                                      |
| vol_type       | __u8   | 1     | 动态卷：1，静态卷：2，当前方案均是动态卷             |
| upd_marker     | __u8   | 0     |                                                      |
| name_len       | __be16 |       | 卷名长度                                             |
| name[128]      | char   |       |                                                      |
| flags          | __u8   |       | 分区内最后一个卷udisk，flags UBI_VTBL_AUTORESIZE_FLG |
| padding[23]    | __u8   | 0     |                                                      |
| crc            | __be32 |       | crc32_le                                             |

ubi layout volume 的内容填充及烧写方法请参考u-boot-2018/cmd/ubi_simu.c 的ubi_simu_create_vol 和wr_vol_table 函数

注意ubi 中crc32_le 算法与sunxi_mbr 的crc32 算法不一样。

ubi 中crc32_le 参考crc32_le.c 用法sunxi_mbr 中crc32 参考crc32.c 用法

## 10 烧写逻辑卷

PEB = ubi_ec_hdr + ubi_vid_hdr + LEB
其中ubi_ec_hdr 和ubi_vid_hdr 存放于PEB 的首逻辑页（logical page0）。

ubi_ec_hdr 存放于0 字节偏移处，大小与物理页size 对齐

ubi_vid_hdr 存放于1 个物理页size 偏移处，大小也与物理页size 对齐

### 10.1 ubi_ec_hdr

ubi_ec_hdr：主要用于存储PEB 的擦除次数信息，需动态生成crc32_le 校验值。

struct ubi_ec_hdr 位于u-boot-2018/drivers/mtd/ubi/ubi-media.h，结构体各个成员使用大端表示。

```
struct ubi_ec_hdr {
__be32 magic;
__u8 version;
__u8 padding1[3];
__be64 ec; /* Warning: the current limit is 31-bit anyway! */
__be32 vid_hdr_offset;
__be32 data_offset;
__be32 image_seq;
__u8 padding2[32];
__be32 hdr_crc;
} __packed;
```

| attribute name | type   | value              | comment  |
| -------------- | ------ | ------------------ | -------- |
| magic          | __be32 | 0x55424923         | UBI#     |
| version        | __u8   | 1                  |          |
| padding1[3]    | __u8   | 0                  |          |
| ec             | __be64 | 1                  |          |
| vid_hdr_offset | __be32 | physical page size | 2048     |
| data_offset    | __be32 | logical page size  | 4096     |
| image_seq      | __be32 | 0                  |          |
| padding2[32]   | __u8   | 0                  |          |
| hdr_crc        | __be32 |                    | crc32_le |

ubi_ec_hdr 的填充方法请参考u-boot-2018/cmd/ubi_simu.c 的fill_ec_hdr 函数。

### 10.2 ubi_vid_hdr

ubi_vid_hdr：存放PEB 和LEB&Volume 映射信息，需动态生成crc32_le 校验值

struct ubi_vid_hdr 位于u-boot-2018/drivers/mtd/ubi/ubi-media.h，结构体各个成员使用大端表示。

```
struct ubi_vid_hdr {
__be32 magic;
__u8 version;
__u8 vol_type;
__u8 copy_flag;
__u8 compat;
__be32 vol_id;
__be32 lnum;
__u8 padding1[4];
__be32 data_size;
__be32 used_ebs;
__be32 data_pad;
__be32 data_crc;
__u8 padding2[4];
__be64 sqnum;
__u8 padding3[12];
__be32 hdr_crc;
} __packed;
```

![image-20221219104521069](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_SPINAND-UBI_Offline_ProgDevGuide_image-20221227.png)

ubi_vid_hdr 的填充方法请参考u-boot-2018/cmd/ubi_simu.c 的fill_vid_hdr 函数。

## 11 数据对齐

有数据对齐需求时，不能填充0xff 数据，可选择填充全0版