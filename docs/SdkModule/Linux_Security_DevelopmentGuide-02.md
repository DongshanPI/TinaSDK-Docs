## 2 安全系统基础

### 2.1 安全系统介绍

安全系统是基于硬件配合软件的安全解决方案。其主要目的是保障系统资源的完整性、保密性、可用性，从而为系统提供一个可信的运行环境。

### 2.2 密码学基础介绍

#### 2.2.1 数据加密模型

（1）明文P。准备加密的文本，称为明文。
（2）密文Y。加密后的文本，称为密文。
（3）加解密算法E(D)。用于实现从明文到密文或从密文到明文的一种转换关系。
（4）密钥K。密钥是加密和解密算法中的关键参数。

![image-20230103095847960](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103095847960.png)

#### 2.2.2 加密算法

对称加密算法：加密、解密用的是同一个密钥。比如AES 算法。

![image-20230103095914998](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103095914998.png)

非对称加密算法：加密、解密用的是不同的密钥，一个密钥公开，即公钥，另一个密钥持有，即私钥。其中一把用于加密，另一把用于解密。比如RSA 算法。
散列（hash）算法：一种摘要算法，把一笔任意长度的数据通过计算得到固定长度的输出，但不能通过这个输出得到原始计算的数据。

![image-20230103095938483](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103095938483.png)

#### 2.2.3 签名与证书

数字签名：数字签名是非对称密钥加密技术与数字摘要技术的应用。数字签名保证信息是由签名者自己签名发送的，签名者不能否认或难以否认；可保证信息自签发后到收到为止未曾作过任何修改，签发的文件是真实文件。

![image-20230103100010011](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103100010011.png)

数字证书：是一个经证书授权中心数字签名的包含公开密钥拥有者信息以及公开密钥的文件，是一种权威性的电子文档。

![image-20230103100026829](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103100026829.png)

### 2.3 TrustZone

TrustZone 是ARM 提出的安全解决方案，旨在提供独立的安全操作系统及硬件虚拟化技术，提供可信的执行环境（Trust Execution Environment）。TrustZone 系统模型如下图所示。TrustZone 技术将软硬件资源隔离成两个环境，分别为安全世界（Secure World）和非安全世界（Normal World），所有需要保密的操作在安全世界执行，其余操作在非安全世界执行，安全世界与非安全世界通过monitor mode 来进行切换。具体可参考《trustzone securitywhitepaper.pdf》。

![image-20230103100105265](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/Linux_Security_DevGuide_image-20230103100105265.png)

#### 2.3.1 OP-TEE

运行在安全世界的系统称为安全操作系统。

很多机构基于TrustZone 推出了自己的安全操作系统，基本都会遵循GP（GlobalPlatform）标准。GlobalPlatform 是一个跨行业的国际标准组织，致力于开发、制定并发布安全芯片的技术标准，以促进多应用产业环境的管理及其安全、可互操作的业务部署。

当前Tina 中采用的是OP-TEE 安全系统。OP-TEE 是Linaro 联合其他公司合作开发的基于ARM TrustZone 技术实现的TEE 方案，遵循GP 标准，主要由三部分组成：
• OP-TEE client (optee_client)：运行在非安全世界用户空间的客户端API。
• OP-TEE Linux Kernel device driver (optee_linuxdriver)：用以控制非安全世界用户空间和安全世界通信的设备驱动。此部分代码在Linux-4.9 mainline 上已经包含。
• OP-TEE Trusted OS (optee_os)：运行在安全世界的可信操作系统。

#### 2.3.2 ARM Trusted Firmware

ARM Trusted Firmware(ATF) 是ARM 官方提供的安全世界软件的参考实现。它统一了ARM底层接口标准，包括电源状态控制接口(Power Status Control 

Interface, PSCI)，安全启动需求(Trusted Board Boot Requirements, TTBR)，安全监控模式调用(Secure Monitor Call,SMC) 等。它还提供了ARMv8 架构下

Exception Level 3(EL3) Secure Monitor 的参考实现。

说明：AW ARM 64 位平台使用ATF 中的bl31 作为Secure Monitor 实现，AW ARM 32 位平台使用OP-TEE 中的SecureMonitor 实现。

### 2.4 硬件安全模块

ARM TrustZone 技术要求安全非安全使用独立的外设资源。在Tina SOC 系列方案中，我们设计了相关硬件模块来控制资源的安全属性。

#### 2.4.1 SPC

Secure Peripherals Control，配置外设的安全属性，只有在安全环境才可以使用该模块。某外设被设定为安全后，该外设只有在安全世界下才能正常访问，非安

全世界写无效，读为0 。

#### 2.4.2 SMC

这里指的是Secure Memory Control（注意与ARM 指令Secure Monitor Call 区分开），配置内存地址的安全属性，只有在安全环境才可以使用该模块。某地址空

间的内存被设定为安全后，该空间的内存只有安全世界可访问，非安全世界写无效，读为0。

#### 2.4.3 SID

Secure ID，控制efuse 的访问。efuse 的访问只能通过sid 模块进行。sid 本身非安全，安全非安全均可访问。但通过sid 访问efuse 时，安全的efuse 只有安全世

界才可以访问，非安全世界访问的结果为0。

#### 2.4.4 efuse

efuse：一次性可编程熔丝技术，是一种OTP（One-Time Programmable，一次性可编程）存储器。efuse 内部数据只能从0 变成1，不能从1 变成0，只能写入一次。efuse 中区域的划分详见各SOC 的SID spec。

#### 2.4.5 CE

Crypto Engine，硬件加解密加速引擎。支持多种对称加密、非对称加密、摘要以及随机数生成算法等。具体见各SOC 的CE spec。

#### 2.4.6 TZMA

TrustZone Memory Access，TZMA 是配置存储器存在安全区域的控制模块。目前仅R528 包含TZMA 模块，用于配置SRAM 区域的安全属性。

### 2.5 相关术语

SMC：Secure Monitor Call，ARM 给出的一条指令，可以让CPU 跳转到Monitor（安全）模式执行。

• RPC：Remote Procedure Control Protocol。optee 中，用于操作Linux 下资源的一种机制。比如，optee 中不能读写文件，就通过RPC 调用Linux 下的文件系统来完成。

• REE：Rich Execution Environment。顾名思义，是资源丰富的执行环境，比如常见的Linux，Android 系统等。TEE：Trusted Execution Environment。可信执行环境，即安全执行环境，在这个区域内，所有的代码，资源都是用户可以信任的。

• TA：Trusted Apps，在TEE 下执行的应用程序，完成用户需要保护的任务，比如对密码的保护。

• PTA：Pesudo Trusted Apps，伪TA，OPTEE 中的一个概念，表明该TA 被集成到了OPTEE OS 中。

• NA：Normal Apps，或称为CA，Client Apps，在REE 下执行的应用程序，完成普通的，不需要保护的任务，比如看普通视频。

• UUID：Universally Unique Identifier，通用唯一识别码。由当前日期和时间，时钟序列，机器识别码（如MAC）组成。

• PRNG：Pesudo Random Number Generator，伪随机数生成器。

• TRNG：True Random Number Generator，真随机数生成器。

• RPMB：Replay Protected Memory Block，是eMMC 中的一个具有安全特性的分区。