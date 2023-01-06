## 3 模块接口说明

### 3.1 i2c-core 接口

#### 3.1.1 i2c_transfer()

- 函数原型：int i2c_transfer(struct i2c_adapter *adap, struct i2c_msg *msgs, int num)

- 作用：完成 I2C 总线和 I2C 设备之间的一定数目的 I2C message 交互。

- 参数：

  - adap：指向所属的 I2C 总线控制器；

  - msgs：i2c_msg 类型的指针；

  - num：表示一次需要处理几个 I2C msg

- 返回：

  - \>0：已经处理的 msg 个数；

  - <0：失败；



#### 3.1.2 i2c_master_recv()

- 函数原型：int i2c_master_recv(const struct i2c_client *client, char *buf, int count)

- 作用：通过封装 i2c_transfer() 完成一次 I2c 接收操作。

- 参数：

  - client：指向当前 I2C 设备的实例；
  - buf：用于保存接收到的数据缓存；

  - count：数据缓存 buf 的长度

- 返回：
  -  \>0：成功接收的字节数；
  -  <0：失败；



#### 3.1.3 i2c_master_send()

- 函数原型：int i2c_master_send(const struct i2c_client *client, const char *buf, int count)

- 作用：通过封装 i2c_transfer() 完成一次 I2c 发送操作。

- 参数：

  - client：指向当前 I2C 从设备的实例；

  - buf：要发送的数据；
  - count：要发送的数据长度

- 返回：

  - \>0：成功发送的字节数；

  - <0：失败；



#### 3.1.4 i2c_smbus_read_byte()

- 函数原型：s32 i2c_smbus_read_byte(const struct i2c_client *client)

- 作用：从 I2C 总线读取一个字节。（内部是通过 i2c_transfer() 实现，以下几个接口同。）

- 参数：
  - client：指向当前的 I2C 从设备

- 返回：
  - \>0：读取到的数据；
  - <0：失败；



#### 3.1.5 i2c_smbus_write_byte()

- 函数原型：s32 i2c_smbus_write_byte(const struct i2c_client *client, u8 value)

- 作用：从 I2C 总线写入一个字节。

- 参数：
  - client：指向当前的 I2C 从设备；
  - value：要写入的数值

- 返回：
  - 0：成功；
  - <0：失败；



#### 3.1.6 i2c_smbus_read_byte_data()

- 函数原型：s32 i2c_smbus_read_byte_data(const struct i2c_client *client, u8 command)

- 作用：从 I2C 设备指定偏移处读取一个字节。

- 参数：
  - client：指向当前的 I2C 从设备；
  - command：I2C 协议数据的第 0 字节命令码（即偏移值）；

- 返回：

  - \>0：读取到的数据；

  - <0：失败；



#### 3.1.7 i2c_smbus_write_byte_data()

- 函数原型：s32 i2c_smbus_write_byte_data(const struct i2c_client *client, u8 command,u8 value)

- 作用：从 I2C 设备指定偏移处写入一个字节。

- 参数：

  - client：指向当前的 I2C 从设备；

  - command：I2C 协议数据的第 0 字节命令码（即偏移值）；

  - value：要写入的数值；

- 返回：

  - 0：成功；

  - <0：失败；



#### 3.1.8 i2c_smbus_read_word_data()

- 函数原型：s32 i2c_smbus_read_word_data(const struct i2c_client *client, u8 command)

- 作用：从 I2C 设备指定偏移处读取一个 word 数据（两个字节，适用于 I2C 设备寄存器是 16 位的情况）。

- 参数：

  - client：指向当前的 I2C 从设备；

  - command：I2C 协议数据的第 0 字节命令码（即偏移值）；

- 返回：

  - \>0：读取到的数据；

  - <0：失败；



#### 3.1.9 i2c_smbus_write_word_data()

- 函数原型：s32 i2c_smbus_write_word_data(const struct i2c_client *client, u8 command,u16 value)

- 作用：从 I2C 设备指定偏移处写入一个 word 数据（两个字节）。

- 参数：

  - client：指向当前的 I2C 从设备；

  - command：I2C 协议数据的第 0 字节命令码（即偏移值）；

  - value：要写入的数值

- 返回：

  - 0：成功；

  - <0：失败；



#### 3.1.10 i2c_smbus_read_block_data()

- 函数原型：s32 i2c_smbus_read_block_data(const struct i2c_client *client, u8 command,u8 *values)

- 作用：从 I2C 设备指定偏移处读取一块数据。 

- 参数：

  - client：指向当前的 I2C 从设备；

  - command：I2C 协议数据的第 0 字节命令码（即偏移值）；

  - values：用于保存读取到的数据；

- 返回：
  - \>0：读取到的数据长度；
  - <0：失败；



#### 3.1.11 i2c_smbus_write_block_data()

- 函数原型：s32 i2c_smbus_write_block_data(const struct i2c_client *client, u8 command,u8 length, const u8 *values)

- 作用：从 I2C 设备指定偏移处写入一块数据（长度最大 32 字节）。

- 参数：

  - client：指向当前的 I2C 从设备；

  - command：I2C 协议数据的第 0 字节命令码（即偏移值）；

  - length：要写入的数据长度；

  - values：要写入的数据；

- 返回：
  - 0：成功；
  - <0：失败；



### 3.2 i2c 用户态调用接口

i2c 的操作在内核中是当做字符设备来操作的，可以通过利用文件读写接口（open，write，read，ioctrl）等操作内核目录中的/dev/i2c-* 文件来条用相关的接口，i2c 相关的操作定义在i2c-dev.c 里面，本节将介绍比较重要的几个接口：

#### 3.2.1 i2cdev_open()

- 函数原型：static int i2cdev_open(struct inode *inode, struct file *file)

- 作用：程序（C 语言等）使用 open(file) 时调用的函数。打开一个 i2c 设备，可以像文件读写的方式往 i2c 设备中读写数据

- 参数：

  - inode：inode 节点；

  - file：file 结构体；

  - 返回：文件描述符



#### 3.2.2 i2cdev_read()

- 函数原型：static ssize_t i2cdev_read(struct file *file, char __user *buf, size_t count,loff_t *offset)

- 作用：程序（C 语言等）调用 read() 时调用的函数。像往文件里面读数据一样从 i2c 设备中读数据。底层调用 i2c_xfer 传输数据

- 参数：

  - file：file 结构体；

  - buf，写数据 buf； 

  - offset, 文件偏移。

- 返回：

  - 非空：返回读取的字节数；

  - <0：失败；



#### 3.2.3 i2cdev_write()

- 函数原型：static ssize_t i2cdev_write(struct file *file, const char __user *buf,size_t count, loff_t *offset)

- 作用：程序（C 语言等）调用 write() 时调用的函数。像往文件里面写数据一样往 i2c 设备中写数据。底层调用 i2c_xfer 传输数据

- 参数：

  - file：file 结构体；

  - buf：读数据 buf； 

  - offset, 文件偏移。

- 返回：

  - 0：成功；

  - <0：失败；



#### 3.2.4 i2cdev_ioctl()

- 函数原型：static long i2cdev_ioctl(struct file *file, unsigned int cmd, unsigned long arg)

- 作用：程序（C 语言等）调用 ioctl() 时调用的函数。像对文件管理 i/o 一样对 i2c 设备管理。该功能比较强大，可以修改 i2c 设备的地址，往 i2 设备里面读写数据，使用 smbus 等等，详细的可以查阅该函数。

- 参数：

  - file：file 结构体；

  - cmd：指令；

  - arg：其他参数。

- 返回：

  - 0：成功；

  - <0：失败；

