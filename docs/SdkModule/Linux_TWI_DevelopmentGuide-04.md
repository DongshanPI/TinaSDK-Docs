## 4 模块使用范例

### 4.1 利用 i2c-core 接口读写 TWI 设备

在内核源码中有现成的 i2c 设备驱动实例：tina/lichee/kernel/linux-5.4/drivers/misc/eeprom/at24.c, 这是一个 EEPROM 的 I2C 设备驱动，为了验证 I2C 总线驱动，所以其中通过 sysfs 节点实现读写访问。下面对这个文件的一些关键点进行展示介绍：

```c
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/module.h>
#include <linux/of_device.h>
#include <linux/slab.h>
#include <linux/delay.h>
#include <linux/mutex.h>
#include <linux/mod_devicetable.h>
#include <linux/bitops.h>
#include <linux/jiffies.h>
#include <linux/property.h>
#include <linux/acpi.h>
#include <linux/i2c.h>
#include <linux/nvmem-provider.h>
#include <linux/regmap.h>
#include <linux/pm_runtime.h>
#include <linux/gpio/consumer.h>

#define EEPROM_ATTR(_name) \ 
{ 							\
	.attr = { .name = #_name,.mode = 0444 }, \
	.show = _name_00_show, \ 
}

struct i2c_client *this_client;

static const struct i2c_device_id at24_ids[] = {
	{ "24c16", 0 },
	{ /* END OF LIST */ }
};
MODULE_DEVICE_TABLE(i2c, at24_ids);

static int eeprom_i2c_rxdata(char *rxdata, int length)
{
    int ret;
    struct i2c_msg msgs[] = {
        {
            .addr = this_client->addr,
            .flags = 0,
            .len = 1,
            .buf = &rxdata[0],
        },
        {
            .addr = this_client->addr,
            .flags = I2C_M_RD,
            .len = length,
            .buf = &rxdata[1],
        },
	};
    ret = i2c_transfer(this_client->adapter, msgs, 2);
    if (ret < 0)
    	pr_info("%s i2c read eeprom error: %d\n", __func__, ret);
    	
    return ret;
}    

static int eeprom_i2c_txdata(char *txdata, int length)
{
    int ret;
    struct i2c_msg msg[] = {
        {
            .addr = this_client->addr,
            .flags = 0,
            .len = length,
            .buf = txdata,
        },
    };
    ret = i2c_transfer(this_client->adapter, msg, 1);
    if (ret < 0)
	    pr_err("%s i2c write eeprom error: %d\n", __func__, ret);
	    
    return 0;
}

static ssize_t read_show(struct kobject *kobj, struct kobj_attribute *attr,
char *buf)
{
    int i;
    u8 rxdata[4];
    rxdata[0] = 0x1;
    eeprom_i2c_rxdata(rxdata, 3);
    
    for(i=0;i<4;i++)
	    printk("rxdata[%d]: 0x%x\n", i, rxdata[i]);
    
    return sprintf(buf, "%s\n", "read end!");
}

static ssize_t write_show(struct kobject *kobj, struct kobj_attribute *attr,
char *buf)
{
    int i;
    static u8 txdata[4] = {0x1, 0xAA, 0xBB, 0xCC};
    for(i=0;i<4;i++)
    	printk("txdata[%d]: 0x%x\n", i, txdata[i]);
    	
    eeprom_i2c_txdata(txdata,4);
    
    txdata[1]++;
    txdata[2]++;
    txdata[3]++;
    
    return sprintf(buf, "%s\n", "write end!");
}

static struct kobj_attribute read = EEPROM_ATTR(read);
static struct kobj_attribute write = EEPROM_ATTR(write);

static const struct attribute *test_attrs[] = {
    &read.attr,
    &write.attr,
    NULL,
};

static int at24_probe(struct i2c_client *client, const struct i2c_device_id *id)
{
    int err;
    this_client = client;
    printk("1..at24_probe \n");
    err = sysfs_create_files(&client->dev.kobj,test_attrs);
    printk("2..at24_probe \n");
    
    if(err){
    	printk("sysfs_create_files failed\n");
	}
	printk("3..at24_probe \n");

	return 0;
}
static int at24_remove(struct i2c_client *client)
{
	return 0;
}

static struct i2c_driver at24_driver = {
    .driver = {
        .name = "at24",
        .owner = THIS_MODULE,
    },
    .probe = at24_probe,
    .remove = at24_remove,
    .id_table = at24_ids,
};

static int __init at24_init(void)
{
    printk("%s %d\n", __func__, __LINE__);
    return i2c_add_driver(&at24_driver);
}
module_init(at24_init);
	
static void __exit at24_exit(void)
{
	printk("%s()%d - \n", __func__, __LINE__);
	i2c_del_driver(&at24_driver);
}
module_exit(at24_exit);
```





### 4.2 利用用户态接口读写 TWI 设备

如果配置了 i2c devices interface，可以直接利用文件读写函数来操作 I2C 设备。下面这个程序直接读取 /dev/i2c-* 来读写 i2c 设备：

```
#include <sys/ioctl.h>
#include <fcntl.h>
#include <linux/i2c-dev.h>
#include <linux/i2c.h>
#define CHIP "/dev/i2c-1"
#define CHIP_ADDR 0x50
int main()
{
    unsigned char rddata;
    unsigned char rdaddr[2] = {0, 0}; /* 将要读取的数据在芯片中的偏移量 */
    unsigned char wrbuf[3] = {0, 0, 0x3c}; /* 要写的数据，头两字节为偏移量 */
    printf("hello, this is i2c tester\n");
    int fd = open(CHIP, O_RDWR);
    if (fd < 0)
    {
        printf("open "CHIP"failed\n");
        goto exit;
    }
    if (ioctl(fd, I2C_SLAVE_FORCE, CHIP_ADDR) < 0)
    { /* 设置芯片地址 */
        printf("oictl:set slave address failed\n");
        goto close;
    }
    printf("input a char you want to write to E2PROM\n");
    wrbuf[2] = getchar();
    printf("write return:%d, write data:%x\n", write(fd, wrbuf, 3), wrbuf[2]);
    sleep(1);
    printf("write address return: %d\n",write(fd, rdaddr, 2)); /* 读取之前首先设置读取的偏移量 */
    printf("read data return:%d\n", read(fd, &rddata, 1));
    printf("rddata: %c\n", rddata);
    close(fd);
    
exit:
    return 0;
}
```

