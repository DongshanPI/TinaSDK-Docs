## 4 Secure OS

ARM 利用CPU 分时复用的思路，设计了SMC 指令切换到另外一个特殊状态再结合SOC 级别的硬件IP 构建了被称为ARM TrustZone 的安全技术。
Tina 从SOC 层面支持ARM Trustzone, 但要设计满足Linux 系统安全标准和需求的安全方案，除了实现ARM TrustZone，还必须有一套软件可信执行环境TEE。Tina 采用的OP-TEE便是一种特定安全系统实现，它严格遵循ARM TrustZone 和TEE/GP 等产业标准。

### 4.1 optee 总体框架

optee 系统，是由运行在TEE 环境下的optee os、TA、以及运行在REE 环境下的client、driver、NA 组成，一共五个部分。optee 总体架构如下图所示：

![image-20230103102922052](https://photos.100ask.net/Tina-Sdk/Linux_Security_DevGuide_image-20230103102922052.png)

### 4.2 开启Secure OS

#### 4.2.1 Secure OS 镜像

Tina 固件在打包会自动把Secure OS 镜像打包到安全固件中。Secure OS 镜像位于device/config/chips/{IC}/bin/optee_{CHIP}.bin。
TEE 环境使用的内存有3 个部分，各部分大小与起始地址在Secure OS 编译时指定。各部分作用如下：
• 共享内存。REE 与TEE 通过smc 指令进行交互，smc 只能通过寄存器交换有限的数据，更多的数据通过共享内存进行交换。REE 和TEE 都有访问权限。
• optee os 内存。optee_os 专用的内存。optee_os 被加载到此处开始运行。REE 无权访问。
• TA 内存堆。加载TA、放置TA 堆、栈的内存空间。由optee_os 进行分配。分配给某一个TA的内存只能由该TA 或optee_os 访问，其他TA 无法访问。REE 无权访问。

**警告**：
在内核中需要为TEE 环境预留内存，预留内存的大小与地址需要按照optee_{CHIP}.bin 编译时指定的大小与地址来设置。

假设R328 Secure 环境需要使用的内存如下：

```
1. SHARE MEM: 0x41900000-0x41A00000
2. OPTEE OS: 0x41A00000-0x41B00000
3. OPTEE TA: 0x41B00000-0x41C00000
```

在文件tina/lichee/linux-4.9/arch/arm/boot/dts/sun8iw18p1.dtsi 也必须预留3M 的内存。

![image-20230103103045891](https://photos.100ask.net/Tina-Sdk/Linux_Security_DevGuide_image-20230103103045891.png)

#### 4.2.2 内核支持optee 驱动

在内核中使能optee 驱动，执行make kernel_menuconfig，选中如下几项：

```
Device Drivers --->
	<*> Trusted Execution Environment support
		TEE drivers --->
			<*> OP-TEE
```

