## 4 接口设计

## 4.1 接口函数

### 4.1.1 s32 sunxi_get_platform(s8 *buf, s32 size)

• 作用：获取SoC 平台的名称，实际上是一个BSP 研发代号，如sun8iw11。
• 参数：
• buf: 用于保存平台名称的缓冲区
• size:buf 的大小
• 返回：
• 返回buf 中平台名称的实际拷贝长度（如果size 小于名称长度，返回size）。

### 4.1.2 int sunxi_get_soc_chipid(u8 *chipid)

• 作用：获取SoC 的ChipID（从Efuse 中读到的原始内容，包括数据内容和顺序）。
• 参数：
• chipid：用于保存ChipID 的缓冲区
• 返回：
• 会返回0，无实际意义

### 4.1.3 int sunxi_get_serial(u8 *serial)

• 作用：获取SoC 的序列号（由ChipID 加工而来，格式定义见《chipid 接口的实现方案》。
• 参数：
• serial：用于保存序列号的缓冲区
• 返回：
• 会返回0，无实际意义

### 4.1.4 sunxi_get_soc_chipid_str(char *serial)

• 作用：获取SoC 的ChipID 的第一个字节，要求转换为字符串格式。
• 参数：
• serial：用于打印ChipID 第一个字节的缓冲区
• 返回：
• 只会返回8（4 个字节的十六进制打印长度），无实际意义

### 4.1.5 int sunxi_get_soc_ft_zone_str(char *serial)

• 作用：获取FZ ZONE 的最后一个字节，要求转换为字符串格式。
• 参数：
• serial：用于打印ChipID 第一个字节的缓冲区
• 返回：
• 只会返回8（4 个字节的十六进制打印长度），无实际意义

### 4.1.6 int sunxi_get_soc_rotpk_status_str(char *status)

• 作用：获取rotpk 的状态，是否烧码
• 参数：
• status：用于记录是否烧码的缓冲区；0，未烧；1，已烧
• 返回：
• %d 的长度，无实际意义

### 4.1.7 int sunxi_soc_is_secure(void)

• 作用：获取整个系统的Secure 状态，即安全系统是否启用。
• 参数：
• 无
• 返回：
• 0，未启用安全系统；1，启用

### 4.1.8 unsigned int sunxi_get_soc_bin(void)

• 作用：用于芯片分bin，部分SoC 平台才支持。
• 参数：
• 无
• 返回：
• 0: fail
• 1: normal
• 2: faster
• 3: fastest

### 4.1.9 unsigned int sunxi_get_soc_ver(void)

• 作用：获取SoC 的版本信息。
• 参数：
• 无
• 返回：
• 返回一个十六进制的编号，需要调用者去判断版本号然后做出相应的处理。详情参看dts，
sid 节点。

### 4.1.10 s32 sunxi_efuse_readn(void key_name, void buf, u32

n)
• 作用：读取Efuse 中的一个key 信息。
• 参数：
• key_name - Key 的名称，定义详见sunxi-sid.h
• buf - 用于保存Key 值的缓冲区
• size - buf 的大小
• 返回：
• 0: success
• other: fail

## 4.2 内部函数

### 4.2.1 static s32 sid_get_base(struct device_node **pnode,

void __iomem **base, s8 *compatible, u32 sec)
• 作用：从DTS 中获取指定模块的寄存器基地址。
• 参数：
• pnode - 用于保存获取到的模块node 信息
• base - 用于保存获取到的寄存器基地址
• compatible - 模块名称，用于匹配DTS 中的模块
• 返回：
• 0: success
• other: fail

### 4.2.2 static void sid_put_base(struct device_node *pnode,

void __iomem *base, u32 sec)
• 作用：释放一个模块的基地址。
• 参数：
• pnode - 保存模块node 信息
• base - 该模块的寄存器基地址
• 返回：
• 无

### 4.2.3 static u32 sid_rd_bits(s8 *name, u32 offset, u32 shift,

u32 mask, u32 sec)
• 作用：从一个模块的寄存器中，读取指定位置的bit 信息。
• 参数：
• name - 模块名称，用于匹配DTS 中的模块
• offset - 寄存器相当于基地址的偏移
• shift - 该bit 在寄存器中的位移
• mask - 该bit 的掩码值
• 返回：

• 0,fail
• other，获取到的实际bit 信息