## 4 接口描述

RTC 驱动会注册生成串口设备/dev/rtcN，应用层的使用只需遵循Linux 系统中的标准RTC 编程方法即可。

### 4.1 打开/关闭RTC 设备

使用标准的文件打开函数：

```
1 int open(const char *pathname, int flags);
2 int close(int fd);
```

需要引用头文件：

```
1 #include <sys/types.h>
2 #include <sys/stat.h>
3 #include <fcntl.h>
4 #include <unistd.h>
```

### 4.2 设置和获取RTC 时间

同样使用标准的ioctl 函数：

```
1 int ioctl(int d, int request, ...);
```

需要引用头文件：

```
1 #include <sys/ioctl.h>
2 #include <linux/rtc.h>
```

