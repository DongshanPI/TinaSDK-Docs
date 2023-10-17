## 3 接口描述

### 3.1 设备注册接口

接口定义在 include/linux/spi/spi.h，主要包含 spi_register_driver 与 spi_unregister_driver 接口，其中给出了快速注册的 SPI 设备驱动的宏 module_spi_driver()，定义如下：

```
#define module_spi_driver(__spi_driveSPI \
module_driver(__spi_driver, spi_register_driver, \
	spi_unregister_driver)
```



#### 3.1.1 spi_register_driver()

*•* 函数原型:int spi_register_driver(struct spi_driver *sdrv)

*•* 功能描述: 注册一个 SPI 设备驱动。

*•* 参数说明：

​	*•* sdrv，spi_driver 类型的指针，其中包含了 SPI 设备的名称、probe 等接口信息。

*•* 返回值：返回 0 表示成功，返回其他值表示失败。



#### 3.1.2 spi_unregister_driver()

*•* 函数原型：void spi_unregister_driver(struct spi_driver *sdrv)

*•* 功能描述：注销一个 SPI 设备驱动。

*•* 参数说明：

​	*•* sdrv，spi_driver 类型的指针，其中包含了 SPI 设备的名称、probe 等接口信息。

*•* 返回值：无





### 3.2 数据传输接口

SPI 设备驱动使用 “struct spi_message” 向 SPI 总线请求读写 I/O。一个 spi_message 中包含了一个操作序列，每一个操作称作 spi_transfer，这样方便 SPI 总线驱动中串行的执行一个个原子的序列。内核线程使用队列实现了异步传输的功能，对于同一个数据传输的发起者，既然异步方式无需等待数据传输完成即可返回，返回后，该发起者可以立刻又发起一个 message，而这时上一个 message 还没有处理完。对于另外一个不同的发起者来说，也有可能同时发起一次 message 传输请求。

![](https://photos.100ask.net/Tina-Sdk/LinuxSPIDevelopmentGuide_005.png)

​															图 3-1: Linux SPI 数据传输流程



```
struct spi_transfer {
    const void *tx_buf;
    void *rx_buf;
    unsigned len;
    dma_addr_t tx_dma;
    dma_addr_t rx_dma;
    unsigned cs_change:1;
    u8 bits_per_word;
    u16 delay_usecs;
    u32 speed_hz;
    struct list_head transfer_list;
};

struct spi_message {
    struct list_head transfers;
    struct spi_device *spi;
    unsigned is_dma_mapped:1;
    void (*complete)(void *context);
    void *context;
    unsigned actual_length;
    int status;
    struct list_head queue;
    void *state;
};
```



#### 3.2.1 spi_message_init()

*•* 函数原型：void spi_message_init(struct spi_message *m)

*•* 功能描述：初始化一个 SPI message 结构，主要是清零和初始化 transfer 队列。

*•* 参数说明：

​	*•* m:spi_message 类型的指针。

*•* 返回值：无



#### 3.2.2 spi_message_add_tail()

*•* 函数原型：void spi_message_add_tail(struct spi_transfer *t, struct spi_message *m)

*•* 功能描述：向 SPI message 中添加一个 transfer。 

*•* 参数说明：

​	*•* t: 指向待添加到 SPI transfer 结构; 

​	*•* m:spi_message 类型的指针。

*•* 返回值：无



#### 3.2.3 spi_sync()

*•* 函数原型：int spi_sync(struct spi_device *spi, struct spi_message *message)

*•* 功能描述：启动、并等待 SPI 总线处理完指定的 SPI message。 

*•* 参数说明：

​	*•* spi，指向当前的 SPI 设备; 

​	*•* m，spi_message 类型的指针，其中有待处理的 SPI transfer 队列。

*•* 返回值：0，成功；小于 0，失败。