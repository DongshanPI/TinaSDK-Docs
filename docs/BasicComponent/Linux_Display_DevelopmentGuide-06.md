# Linux_Display sysfs 接口描述
## 8 sysfs 接口描述

以下两个函数在下面接口的demo 中会使用到。

```
    const int MAX_LENGTH = 128;
    const int MAX_DATA = 128;
    static ssize_t read_data(const char *sysfs_path, char *data)
    {
        ssize_t err = 0;
        FILE *fp = NULL;
        fp = fopen(sysfs_path, "r");
        if (fp) {
            err = fread(data, sizeof(char), MAX_DATA ,fp);
            fclose(fp);
        }
        return err;
    }
    static ssize_t write_data(const char *sysfs_path, const char *data, size_t len)
    {
        ssize_t err = 0;
        int fd = -1;
        fd = open(sysfs_path, O_WRONLY);
        if (fp) {
            errno = 0;
            err = write(fd, data, len);
            if (err < 0) {
                err = -errno;
    }
    close(fd);
    } else {
        ALOGE("%s: Failed to open file: %s error: %s", __FUNCTION__, sysfs_path,
        strerror(errno));
        err = -errno;
    }
    return err;
    }
```

### 8.1 enhance

#### 8.1.1 enhance_mode

• 系统节点

```
/sys/class/disp/disp/attr/disp
/sys/class/disp/disp/attr/enhance_mode
```

• 参数

| 参数                 | 说明                                                |
| -------------------- | --------------------------------------------------- |
| disp display channel | 比如0: disp0, 1：disp1                              |
| enhance_mode         | 0：standard, 1: enhance, 2: soft, 3: enahnce + demo |

• 返回值

no。

• 描述

该接口用于设置色彩增强的模式。

• 示例

```
//设置disp0 的色彩增强的模式为增强模式
echo 0 > /sys/class/disp/disp/attr/disp;
echo 1 > /sys/class/disp/disp/attr/enhance_mode;
//设置disp1 的色彩增强的模式为柔和模式
echo 1 > /sys/class/disp/disp/attr/disp;
echo 2 > /sys/class/disp/disp/attr/enhance_mode;
//设置disp0 的色彩增强的模式为增加模式，并且开启演示模式
echo 0 > /sys/class/disp/disp/attr/disp;
echo 3 > /sys/class/disp/disp/attr/enhance_mode;
```

c/c++ 代码：

```
char sysfs_path[MAX_LENGTH];
char sysfs_data[MAX_DATA];
unisgned int disp = 0
unsigned int enhance_mode = 1;
snprintf(sysfs_path,sizeof(sysfs_full_path),"sys/class/disp/disp/attr/disp");
snprintf(sysfs_data, sizeof(sysfs_data),"%d",disp);
write_data(sysfs_path, sys_data, strlen(sysfs_data));
snprintf(sysfs_path,sizeof(sysfs_full_path),
"/sys/class/disp/disp/attr/enhance_mode");
snprintf(sysfs_data, sizeof(sysfs_data), "%d",enhance_mode);
write_data(sysfs_path, sys_data, strlen(sysfs_data));
```

#### 8.1.2 enhance_bright/contrast/saturation/edge/detail/denoise

• 系统节点

```
/sys/class/disp/disp/attr/disp
/sys/class/disp/disp/attr/enhance_bright /* 亮度*/
/sys/class/disp/disp/attr/enhance_contrast /* 对比度*/
/sys/class/disp/disp/attr/enhance_saturation /* 饱和*/
/sys/class/disp/disp/attr/enhance_edge /* 边缘锐度*/
/sys/class/disp/disp/attr/enhance_detail /* 细节增强*/
/sys/class/disp/disp/attr/enhance_denoise /* 降噪*/
```

• 参数

```
disp display channel，比如0: disp0, 1：disp1。
enhance_xxx： 范围：0~100，数据越大，调节幅度越大。
```

• 返回值
no。

• 描述

该接口用于设置图像的亮度/对比度/饱和度/边缘锐度/细节增强/降噪的调节幅度。

• 示例

```
//设置disp0 的图像亮度为80
echo 0 > /sys/class/disp/disp/attr/disp;
echo 80 > /sys/class/disp/disp/attr/enhance_bright;
//设置disp1 的饱和度为50
echo 1 > /sys/class/disp/disp/attr/disp;
echo 50 > /sys/class/disp/disp/attr/enhance_saturation;
```

c/c++ 代码：

```
char sysfs_path[MAX_LENGTH];
char sysfs_data[MAX_DATA];
unisgned int disp = 0
unsigned int enhance_bright = 80;
snprintf(sysfs_path,sizeof(sysfs_full_path),"sys/class/disp/disp/attr/disp");
snprintf(sysfs_data, sizeof(sysfs_data),"%d",disp);
write_data(sysfs_path, sys_data, strlen(sysfs_data));
snprintf(sysfs_path,sizeof(sysfs_full_path),
"/sys/class/disp/disp/attr/enhance_bright");
snprintf(sysfs_data, sizeof(sysfs_data), "%d",enhance_bright);
write_data(sysfs_path, sys_data, strlen(sysfs_data));
```

### 8.2 hdmi edid

#### 8.2.1 edid

• 系统节点

```
/sys/class/hdmi/hdmi/attr/edid
```

• 参数

no。

• 返回值

Edid data(1024 bytes)。

• 描述

该接口用于读取EDID 的裸数据。

• 示例

```
// 读取edid数据
cat /sys/class/hdmi/hdmi/attr/edid
```

c/c++ 代码：

```
#define EDID_MAX_LENGTH 1024
char sysfs_path[MAX_LENGTH];
char sysfs_data[EDID_MAX_LENGTH];
ssize_t edid_length;
snprintf(sysfs_path,sizeof(sysfs_full_path),"/sys/class/hdmi/hdmi/attr/edid");
edid_length = read_data(sysfs_path, sys_data);
```

#### 8.2.2 hpd

• 系统节点

```
/sys/class/switch/hdmi/state
```

• 参数

no。

• 返回值

Hdmi hotplut state, 0: unplug; 1: plug in。

• 描述

```
该接口用于读取HDMI 的热插拔状态。
```

• 示例

```
// 读取HDMI热插拔状态
cat /sys/class/switch/hdmi/state
```

c/c++代码：

```
char sysfs_path[MAX_LENGTH];
char sysfs_data[MAX_DATA];
int hpd;
snprintf(sysfs_path,sizeof(sysfs_full_path),"/sys/class/hdmi/hdmi/attr/edid");
read_data(sysfs_path, sys_data);
hpd = atoi(sys_data);
If (hpd)
printf("hdmi plug in\n");
else
printf("hdmi unplug \n");
```

#### 8.2.3 hdcp_enable

• 系统节点

```
/sys/class/hdmi/hdmi/attr/hdcp_enable
```

• 参数

enable: 0: disable hdmi hdcp function；1：enable hdmi hdcp function。

• 返回值

No returns。

• 描述

该接口用于使能、关闭hdmi hdcp 功能。

• 示例

```
// 开启hdmi hdcp功能
echo 1 > /sys/class/hdmi/hdmi/attr/hdcp_enable
// 关闭hdmi hdcp功能
echo 0 > /sys/class/hdmi/hdmi/attr/hdcp_enable
```

c/c++ 代码：

```
char sysfs_path[MAX_LENGTH];
char sysfs_data[MAX_DATA];
snprintf(sysfs_path,sizeof(sysfs_full_path),"/sys/class/hdmi/hdmi/attr/hdcp_enable");
snprintf(sysfs_data, sizeof(sysfs_data),"%d",1);
write_data(sysfs_path, sys_data, strlen(sysfs_data));
```
