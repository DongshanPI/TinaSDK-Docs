## 3 XRADIO 系列模组

### 3.1 RF 测试环境搭建

#### 3.1.1 驱动配置

为了支持RF test 工具的使用，必须先配置Xradio 系列的驱动（XR819/XR829），并选择以下配置。

```
make kernel_menuconfig
```

```
Device Drivers > Network device support > Wireless LAN >
    XR829 WLAN support >
        XRadio Driver features >
            Driver debug features >
            	[*] XRADIO ETF Support for RF Test(DEVELOPMENT)
    XR819 WLAN support >
        XRadio Driver features >
            Driver debug features >
            	[*] XRADIO ETF Support for RF Test(DEVELOPMENT)
```

注：

1. 确认在系统的wlan 固件目录（/lib/firmware）中存在boot_xr-xxx.bin，sdd_xr-xxx.bin，etf_xrxxx.bin等文件。
2. 在系统启动后，在测试之前请确认xradio_wlan(三个ko 的形式) 模块已被加载。

#### 3.1.2 Tina 配置

按如下方法配置ETF 工具。

```
make menuconfig
```

```
Utilities > rf test tool
<*> xr819-rftest.........xr819 rf test tools
<*> xr829-rftest.........xr829 rf test tools
```

注意：
由于wlan 与RF 测试共用一个驱动，并且下载固件不一样，因此两者互斥。即测试模式和常规模式不能共存。所以启动etf 工具前，请务必保证进入测试模式。即

若是xradio 模组以三个ko（xradio_wlan,xradio_mac,xradio_core）方式加载的，ETF 测试前需要rmmod xradio_wlan. 若是以xr829/xr819 单个ko 加载的，ETF 

测试时通过带参数的形式加载进入测试模式insmod /lib/modules/xxx/xr829.ko etf_enable=1

### 3.2 ETF CLI 使用说明

ETF 命令行工具可以进行手动测试，也可以被其他程序调用进行自动化测试。

#### 3.2.1 测试命令介绍

ETF 工具命令基本格式，可以通过etf help获取ETF 工具详细的帮助信息。

```
etf cmd [param0] [param1] [param2] [param3]
```

RF 测试模式启动，设备处于运行状态，其他测试命令只能在该命令完成以后才能进行。

```
etf connect
```

RF 测试模式关闭，关闭后设备处于掉电状态。

```
etf disconnect
```

PHY 使能，在进行PHY 和RF 相关操作之前必须先使能PHY。

```
etf enable_phy
```

MAC 地址获取和配置，其中-d为目的地址（A1），-s为源地址（A2），-t为BSSID（A3）。

```
etf get_mac
etf set_mac -d XX:XX:XX:XX:XX:XX -s XX:XX:XX:XX:XX:XX -t XX:XX:XX:XX:XX:XX
```

频段模式和信道配置。其中mode可为DSSS_2GHZ，OFDM_2GHZ，2GHZ。num为信道参数，范围1~14。

```
etf channel [mode] [num]
```

速率配置。

```
etf rate –m [x] –r [y]
```

其中x 和y 意义分别为如下表：

| 模式X | 定义               | 对应速率y                                      |
| ----- | ------------------ | ---------------------------------------------- |
| 0     | 11b short preamble | 1，2，5.5，11                                  |
| 1     | 11b long preamble  | 1，2，5.5，11                                  |
| 2     | 11g                | 6，9，12，18，24，36，48，54                   |
| 4     | 11n                | Greenfield 6.5，13，19.5，26，39，52，58.5，65 |
| 5     | 11n                | Mixed                                          |

功率配置。其中num的范围为2~120，每个速率有对应的默认功率和最大功率，速率配置后自动使用默认功率进行发送；当功率调整超过最大功率时，会配置为最

大功率。

```
etf power_level [num]
```



#### 3.2.2 传导TX 测试

Tx 测试基本格式如下。其中continous 为1 表示连续发送，为0 表示帧数发送，默认为1；当continous 为0 时，num表示要发送的帧数；length表示发送帧的长度。

```
etf tx -c [continous] -n [num] -l [length]
etf tx_stop
```

单载波发送基本格式如下。其中amplitude表示单载波幅度，默认为0dbm；freq为频偏，默认为5MHz；mode表示载波模式，默认为Single Tone Quad。

```
etf tone -a [amplitude] -f [freq] -m [mode]
etf tone_stop
```

示例1：在1 信道，使用11n Mixed 模式MCS7 LongGI 速率，帧长为4095 进行连续发送。

```
etf connect
etf enable_phy
etf channel 2GHZ 1
etf rate -m 5 -r 65
etf tx -c 1 -l 4095
etf tx_stop
etf disconnect
```

示例2：在11 信道，使用11g 模式54Mbps 速率，功率等级为50 进行发送1000 帧。提示：固定帧数发送不需要tx_stop。

```
etf connect
etf enable_phy
etf channel 2GHZ 11
etf rate -m 2 -r 54
etf power_level 50
etf tx -c 0 -n 1000
etf disconnect
```

示例3：在1 信道，进行单载波连续发送的示例。单载波发送必须先进行连续发送。

```
etf connect
etf enable_phy
etf channel 2GHZ 1
etf tx -c 1
etf tone
etf tone_stop
etf tx_stop
etf disconnect
```

#### 3.2.3 传导RX 测试

Rx 测试基本格式如下。Rx 测试无参数，停止后会返回统计数据。

```
etf rx
etf rx_stop
```

Rx 停止后返回数据如下：

```
Rx mode is: OFDM_PREAMBLE
Smoothing: YES!
Sounding PPDU: NO!
A-MPDU: NO!
Short GI: 800ns
CFO: -6.256104
SNR: 11.671869
RSSI: -49.000000
EVM: 2.713441
RCPI: -52.500000
Total: 1107
AbortError: 405
CRCError: 232
Sending CMD OK!
```

具体返回值意义说明：

| 名称       | 描述                       | 备注       |
| ---------- | -------------------------- | ---------- |
| Total      | 所有检测到帧的总数         |            |
| AbortError | 无法解调帧的总数错误帧总数 |            |
| CRCError   | CRC 发生错误的帧           | 错误帧总数 |
| Rx mode    | 最后一帧的调制模式         |            |
| A-MPDU     | 是否为聚合帧               |            |
| RSSI       | 接收信号强度，单位dbm      |            |

示例1：在1 信道，进行连续接收的示例。

```
etf connect
etf enable_phy
etf channel 2GHZ 1
etf rx
etf rx_stop
etf disconnect
```

示例2：在11 信道，11b only 模式，进行连续接收的示例。

```
etf connect
etf enable_phy
etf channel DSSS_2GHZ 11
etf rx
etf rx_stop
etf disconnect
```

### 3.3 WiFi 指令合集

#### 3.3.1 传导TX 测试

在11b 模式带宽11M 信道1 场景下测试

```
rmmod xradio_wlan //上电后卸载一次即可
etf connect
etf enable_phy
etf channel 2GHZ 1
etf rate -m 1 -r 11
etf tx //可以不设置侦长等信息，直接tx
etf tx_stop //每次切换成其他模式需要先stop再输指令
```

在11g 模式带宽54M 信道1 场景下测试

```
rmmod xradio_wlan //上电后卸载一次即可
etf connect
etf enable_phy
etf channel 2GHZ 1
etf rate -m 2 -r 54
etf tx //可以不设置侦长等信息，直接tx
etf tx_stop //每次切换成其他模式需要先stop再输指令
```

在11n 模式带宽HT20 速率MCS7 信道1 场景下测试

```
rmmod xradio_wlan //上电后卸载一次即可
etf connect
etf enable_phy
etf channel 2GHZ 1
etf rate -m 5 -r 65
etf tx //可以不设置侦长等信息，直接tx
etf tx_stop //每次切换成其他模式需要先stop再输指令
```

在11n 模式带宽HT40 速率MCS7 信道1 场景下测试（XR819 没有40M 模式，XR829 才有）

```
rmmod xradio_wlan //上电后卸载一次即可
etf connect
etf enable_phy
etf bandwidth 40M //设置40M带宽
etf subchannel LOWER //设置信道组合方式,向上模式；也可以设置成LOWER向下模式
etf channel 2GHZ 1 //注意LOWER模式IQ仪器需要选择n+2信道，如软件设置信道1，仪器选择信道3
//注意UPPER模式IQ仪器需要选择n-2信道，如软件设置信道3，仪器选择信道1
etf rate -m 5 -r 65
etf tx -w 40M -u LOWER
etf tx_stop //每次切换成其他模式需要先stop再输指令
```

备注：

subchannel可为LOWER或UPPER。此处的LOWER和UPPER含义为设置信道为组成40M 带宽的低/高频信道，如下图所示。故5LOWER 和9UPPER均表示40M 的

中心频率在7 信道（2442MHz）。40M 中心频率的计算方法如下：所设信道的中心频率+10M（对于LOWER的情况）或所设信道的中心频率-10M（对于UPPER的情况）。

![Tina_Linux_WiFi_RFTesting_Development_Guide-image-20230103100325051](http://photos.100ask.net/tina-docs/Tina_Linux_WiFi_RFTesting_Development_Guide-image-20230103100325051.png)

#### 3.3.2 传导RX 测试

在11b 或11g 或11n 模式带宽HT20 场景下测试

```
rmmod xradio_wlan //上电后卸载一次即可
etf connect
etf enable_phy
etf channel 2GHZ 1
//仪器发信号前先进入rx模式
etf rx
//仪器发完之后按输入rx_stop指令，查看结果
etf rx_stop
```

在11n 模式带宽HT40 速率MCS7 场景下测试

```
rmmod xradio_wlan //上电后卸载一次即可
etf connect
etf enable_phy
etf bandwidth 40M //设置40M带宽
etf subchannel LOWER //设置信道组合方式，也可以设置成UPPER模式
etf channel 2GHZ 1
//仪器发信号前先进入rx模式
etf rx
//仪器发完之后按输入rx_stop指令，查看结果
etf rx_stop
```

