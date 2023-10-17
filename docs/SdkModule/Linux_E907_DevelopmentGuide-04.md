## 7 开发使用

### 7.1 rpmsg 内核开发

linux 端请参考driver/rpmsg/rpmsg_client_e907.c 。

melis 端请参考ekernel/subsys/thirdparty/openamp/rpmsg_demo/ 目录下的文件。

### 7.2 rpmsg 用户层接口

控制台调试命令参考测试章节，这里列举代码使用示例。

Linux 端：

```
#include <linux/rpmsg.h>
# 创建端点
int fd;
struct rpmsg_ept_info info;
char ept_dev_name[32];
strcpy(info.name, "test");
info.id = 0xfffff; # id由itctl进行更新
fd = open(ctrl_dev, O_RDWR);
ret = ioctl(fd, RPMSG_CREATE_EPT_IOCTL, &info);
# 当ioctl返回值==0时，端点已经创建成功，设备节点会出现在/dev/rpmsg%d(=info.id)下
close(fd);
#读写设备节点
snprintf(ept_dev_name, 32, "/dev/rpmsg%d", info.id);
fd = open(ept_dev_name, O_RDWR);
write,read,poll...
close(fd);
# 关闭节点
fd = open(ctrl_dev, O_RDWR);
ret = ioctl(fd, RPMSG_DESTROY_EPT_IOCTL, &info);
close(fd);
```

melis 端：

方法1：基于rpmsg_ctrl 驱动，等待主机建立连接

```
// 头文件
#include <openamp/sunxi_helper/openamp.h>
static int ept_cb(struct rpmsg_endpoint *ept, void *data,
size_t len, uint32_t src, void *priv)
{
// 收到数据
}
int bind_cb(struct rpmsg_ept_client *client)
{
// client绑定,每个client代表一个连接
// client->priv和client->ept->priv 可供用户使用
}
int unbind_cb(struct rpmsg_ept_client *client)
{
// 连接关闭
}
int main()
{
// cnt: 监听的数量，即最多对test创建cnt个连接
// 最后一个参数priv,其实里面设置的是client->priv
rpmsg_client_bind("test", ept_cb, bind_cb, unbind_cb,
cnt, NULL);
// do some things
// 取消绑定会unbind所有与其相关的client
rpmsg_client_unbind("test");
}
```

具体代码参考ekernel/subsys/thirdparty/openamp/rpmsg_demo/rpmsg_ctrl/test.c；

方法2：基于rpmsg 原生框架，melis 端主动创建连接，触发主机端的rpmsg driver 的probe。

melis 端代码参考：

1.ekernel/subsys/thirdparty/openamp/rpmsg_demo/demo.c

2.ekernel/subsys/thirdparty/openamp/rpmsg_demo/hearbeat.c

Linux 端参考代码：

1.drivers/rpmsg/rpmsg_client_e907.c

2.drivers/rpmsg/rpmsg_client_heart.c

### 7.3 amp 控制台

SDK 在Linux 端提供了进入E907 控制台的功能，配置步骤如下：

内核配置

```
ckernel
m kernel_menuconfig # 选择下图配置
```

![image-20221121120738002](https://photos.100ask.net/Tina-Sdk/Linux_E907_DevGuide_image-20221121120738002.png)

<center>图7-1: rpmsg config</center>

Tina 配置

```
croot
m menuconfig # 选择下图配置
```

![image-20221121175441744](https://photos.100ask.net/Tina-Sdk/Linux_E907_DevGuide_image-20221121175441744.png)

<center>图7-2: amp_shell config</center>

melis 配置

![image-20221121175506540](https://photos.100ask.net/Tina-Sdk/Linux_E907_DevGuide_image-20221121175506540.png)

<center>图7-3: amp_shell config</center>

编译& 打包& 下载

```
mmelis -j32
make -j32
p
```

使用

在echo start > /sys/kernel/debug/remoteproc/remoteproc0/state 后检查有无rpmsg_ctrl 成功创建的log 或者是否存在/dev/rpmsg_ctrl0 节点。如果正常，直

接在Linux 控制台啊输入amp_shell 即可进入e907 控制台，amp_exit退出控制台。支持执行多次amp_shell，开启多个控制台。

![image-20221121175556831](https://photos.100ask.net/Tina-Sdk/Linux_E907_DevGuide_image-20221121175556831.png)

<center>图7-4: amp_shell test</center>

![image-20221121175611748](https://photos.100ask.net/Tina-Sdk/Linux_E907_DevGuide_image-20221121175611748.png)

<center>图7-5: amp_shell test</center>

### 7.4 大数据传输

由于rpmsg 特性，不适合传输大数据量；如需使用大数据传输，请参考本章节。

#### 7.4.1 配置

内核打开rpbuf 驱动：

```
m kernel_menuconfig
Device Drivers --->
	RPBuf drivers --->
		-*- RPBuf device interface
		<*> RPMsg-based RPBuf service driver
		<*> Allwinner RPBuf controller driver
		<*> Allwinner RPBuf sample driver
```

Note：Allwinner RPBuf sample driver 是一个简单的rpbuf 内核层使用demo，可以不使能。

e907 配置：

```
Kernel Setup
	Subsystem support
		Allwinner Components Support
			RPBuf framework
                [*] RPMsg-based RPBuf service component
                [*] RPBuf controller component
                [*] RPMsg-based RPBuf service component demo
            OpenAMP Support
           	  [*]  RPBuf demo
```

Tina 打开rpbuf_demo 软件包：

```
m menuconfig
Allwinner --->
    RPBuf --->
        <*> rpbuf_demo
        <*> rpbuf_test
```

#### 7.4.2 测试

rpmsg_test：会自动生成随机数据并附带MD5 校验值，另一端收到会重新计算MD5 并与收到的进行对比。

```
(e907) rpbuf_test -c -N "rpbuf_demo" -L 0x100000 # 创建size=0x100000的buffer
(Linux) rpbuf_test -d 1000 -s -L 0x100000 -N "rpbuf_demo" # 发送测试数据
(Linux) rpbuf_test -r -t 1000 -L 0x100000 -N "rpbuf_demo" # 接收数据
(e907) rpbuf_test -s -L 0x100000 -N "rpbuf_demo" #发送测试数据
(e907) rpbuf_test -N "rpbuf_demo" -d # 删除buffer
```

出现success 表明校验成功。

过程log 如下：

![image-20221121182820753](https://photos.100ask.net/Tina-Sdk/Linux_E907_DevGuide_image-20221121182820753.png)

<center>图7-6: Linux 端log</center>

![image-20221121182838167](https://photos.100ask.net/Tina-Sdk/Linux_E907_DevGuide_image-20221121182838167.png)

<center>图7-7: e907 端log</center>

rpbuf_demo：用于在控制台简单传输数据

```
(e907) rpbuf_demo -c -N "rpbuf_demo" -L 0x1000 # 创建size=4k的buffer
(Linux) rpbuf_demo -d 1000 -L 0x1000 -N "rpbuf_demo" -s "hello" # 发送数据,并在1000ms后释放
buffer
(Linux) rpbuf_demo -r -t 1000 -L 0x1000 -N "rpbuf_demo" # 接收数据
(e907) rpbuf_demo -s "hello" -N "rpbuf_demo" # 发送数据
(e907) rpbuf_demo -N "rpbuf_demo" -d # 删除buffer
```

![image-20221121182906578](https://photos.100ask.net/Tina-Sdk/Linux_E907_DevGuide_image-20221121182906578.png)

<center>图7-8: Linux 端log</center>

![image-20221121182927251](https://photos.100ask.net/Tina-Sdk/Linux_E907_DevGuide_image-20221121182927251.png)

<center>图7-9: e907 端log</center>

#### 7.4.3 使用

内核层接口，参考drivers/rpbuf/rpbuf_sample_sunxi.c：
Linux 端使用流程：

1. 在需要用到rpbuf 接口的驱动的设备树节点种添加一条属性：rpbuf = <&rpbuf_controller0>;，
   可以创建多个controller，当面默认只有一个rpbuf_controller0；

2. 获取controller：调用controller = rpbuf_get_controller_by_of_node(np, 0);

3. 创建buffer：调用rpbuf_alloc_buffer(controller, name, len, ops, cbs, priv)；

4. 接收数据：收到数据时候会调用cbd->rx_cb 回调

5. 判断状态：创建出的buffer 不一样马上可用，需要用判断状态，调用rpbuf_buffer_is_available(buffer)

6. 发送数据：

7. buf_va = rpbuf_buffer_va(buffer);

8. buf_len = rpbuf_buffer_len(buffer);

9. 直接对buf_va 地址进行写入即可

10. rpbuf_transmit_buffer(buffer, offset, data_len);

11. 释放buffer：rpbuf_free_buffer(buffer);

    应用层端口，具体细节可以参考package/allwinner/rpbuf/

    1. 创建buffer
    2. fd = open(0, O_RDWR)；
    3. ioctl(fd, RPBUF_CTRL_DEV_IOCTL_CREATE_BUF, &buffer->info);
    4. buf_fd = open(buf_dev_path, O_RDWR);
    5. addr = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED, buf_fd, 0);
    6. 接收数据：
    7. rpbuf_receive_buffer_block(buffer, &offset, &data_len) 阻塞接收
    8. rpbuf_receive_buffer_nonblock(buffer, &offset, &data_len) 非阻塞接收
    9. 发送数据：
    10. buf_va = rpbuf_buffer_va(buffer);
    11. 直接对buf_va 地址进行写入即可
    12. rpbuf_transmit_buffer(buffer, offset, data_len);
    13. 释放buffer：
    14. close(buf_fd);
    15. ioctl(fd, RPBUF_CTRL_DEV_IOCTL_DESTROY_BUF, &buffer->info);
    16. close(fd);

e907 端接口，可以参考ekernel/subsys/aw/rpbuf/rpbuf_demo/rpbuf_demo.c
e907 端使用流程：

1. 获取controller：调用controller = rpbuf_get_controller_by_id(0); 代码默认提供一个controller，
   这里直接使用
2. 创建buffer：调用rpbuf_alloc_buffer(controller, name, len, ops, cbs, priv)；
3. 接收数据：收到数据时候会调用cbd->rx_cb 回调
4. 判断状态：创建出的buffer 不一样马上可用，需要用判断状态，调用rpbuf_buffer_is_available
   (buffer)
5. 发送数据：
6. buf_va = rpbuf_buffer_va(buffer);
7. buf_len = rpbuf_buffer_len(buffer);
8. 直接对buf_va 地址进行写入即可
9. rpbuf_transmit_buffer(buffer, offset, data_len);
10. 释放buffer：rpbuf_free_buffer(buffer);



#### 7.4.4 Note

1. 关于controller：代码默认已经提供了一个基于rpmsg 实现的controller0 了，正常情况下
   直接使用改controller 即可
2. 关于rpbuf_alloc_buffer 的ops 参数：controller0 已经基于ion 实现了内存分配函数。如
   无必要，使用controller 的内存分配函数即可，即创建buffer 时，ops 参数置NULL。
3. 关于互斥：由于通信双方都能拿到的buffer 的地址，难以在驱动实现互斥，所以需要在具体
   应用上自行保证互斥。