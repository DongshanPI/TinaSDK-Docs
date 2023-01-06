## 3 模块设计

### 3.1 结构框图

SID 驱动内部的功能划分如下图所示：

![image-20221219105727865](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_SID_DevGuide_image-20221219105727865.png)

总体上，SID 驱动内部可以分为两大部分：
1.SID Register RW，封装了对寄存器按位读取的接口，以及获取指定compatible 的模块基地址等。
2.SID Api，以API 的方式提供一些功能接口：获取Key、获取SoC Version、获取SecureEnable、获取ChipID 等。

### 3.2 关键数据定义

#### 3.2.1 常量及宏定义

##### 3.2.1.1 key 的名称定义

在获取Key 的时候，调用者需要知道Key 的名称，以此作为索引的依据。Key 名称详见sunxisid.h：

```
1 #define EFUSE_CHIPID_NAME "chipid"
2 #define EFUSE_BROM_CONF_NAME "brom_conf"
3 #define EFUSE_BROM_TRY_NAME "brom_try"
4 #define EFUSE_THM_SENSOR_NAME "thermal_sensor"
5 #define EFUSE_FT_ZONE_NAME "ft_zone"
6 #define EFUSE_TV_OUT_NAME "tvout"
7 #define EFUSE_OEM_NAME "oem"
9 #define EFUSE_WR_PROTECT_NAME "write_protect"
10 #define EFUSE_RD_PROTECT_NAME "read_protect"
11 #define EFUSE_IN_NAME "in"
12 #define EFUSE_ID_NAME "id"
13 #define EFUSE_ROTPK_NAME "rotpk"
14 #define EFUSE_SSK_NAME "ssk"
15 #define EFUSE_RSSK_NAME "rssk"
16 #define EFUSE_HDCP_HASH_NAME "hdcp_hash"
17 #define EFUSE_HDCP_PKF_NAME "hdcp_pkf"
18 #define EFUSE_HDCP_DUK_NAME "hdcp_duk"
19 #define EFUSE_EK_HASH_NAME "ek_hash"
20 #define EFUSE_SN_NAME "sn"
21 #define EFUSE_NV1_NAME "nv1"
22 #define EFUSE_NV2_NAME "nv2"
23 #define EFUSE_BACKUP_KEY_NAME "backup_key"
24 #define EFUSE_RSAKEY_HASH_NAME "rsakey_hash"
25 #define EFUSE_RENEW_NAME "renewability"
26 #define EFUSE_OPT_ID_NAME "operator_id"
27 #define EFUSE_LIFE_CYCLE_NAME "life_cycle"
28 #define EFUSE_JTAG_SECU_NAME "jtag_security"
29 #define EFUSE_JTAG_ATTR_NAME "jtag_attr"
30 #define EFUSE_CHIP_CONF_NAME "chip_config"
31 #define EFUSE_RESERVED_NAME "reserved"
32 #define EFUSE_RESERVED2_NAME "reserved2"
33 /* For KeyLadder */
34 #define EFUSE_KL_SCK0_NAME "keyladder_sck0"
35 #define EFUSE_KL_KEY0_NAME "keyladder_master_key0"
36 #define EFUSE_KL_SCK1_NAME "keyladder_sck1"
37 #define EFUSE_KL_KEY1_NAME "keyladder_master_key1"
```

sunxi-sid.h 不是所有key 都能访问，一般可以访问的已经在dts 定义。

#### 3.2.2 关键数据结构

##### 3.2.2.1 soc_ver_map

用于管理多个SoC 的Version 信息，方便用查表的方式实现SoC Version API。其中有两个分量：id，即BondingID；rev[]，用于保存BondingID 和Version 的各种组合值。定义在sunxi-sid.c 中：

```
#define SUNXI_VER_MAX_NUM 8
struct soc_ver_map {
u32 id;
u32 rev[SUNXI_VER_MAX_NUM];
};
```

对于一个SoC 定义一个soc_ver_map 结构数组，使用id 和不同Version 在rev[] 中查找对应的组合值。

##### 3.2.2.2 soc_ver_reg

SoC Version、BondingID、SecureEnable 的存储位置因SoC 而异，所以定义了一个结构来记录这类信息的位置，包括属于那个模块（基地址）、偏移、掩码、位移等。定义见sunxisid.c：

```
#define SUNXI_SOC_ID_INDEX 1
#define SUNXI_SECURITY_ENABLE_INDEX 2
struct soc_ver_reg {
s8 compatile[48];
u32 offset;
u32 mask;
u32 shift;
};
```

每个SoC 会定义一个soc_ver_reg 数组，目前各元素的定义如下：
0 - SoC Version 信息在寄存器中的位置。
1 - BondingID 信息在寄存器中的位置。
2 - SecureEnable 信息在寄存器中的位置。

#### 3.2.3 全局变量

定义几个static 全局变量，用于保存解析后的ChipID、SoC_Ver 等信息：

```
static unsigned int sunxi_soc_chipid[4];
static unsigned int sunxi_serial[4];
static int sunxi_soc_secure;
static unsigned int sunxi_soc_bin;
static unsigned int sunxi_soc_ver;
```

### 3.3 模块流程设计

#### 3.3.1 SoC 信息读取流程

本节中，这里把SoC Ver、ChipID、SecureEnable 信息统称为“SoC 信息”，因为他们的读取过程非常相似。都是遵循以下流程：

![image-20221219110111514](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_SID_DevGuide_image-20221219110111514.png)

#### 3.3.2 Efuse Key 读取流程

在读取Efuse 中Key 的时候，需要判断是否存在、以及访问权限，过程有点复杂，用以下流程图进行简单说明。

![image-20221219110133623](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_SID_DevGuide_image-20221219110133623.png)