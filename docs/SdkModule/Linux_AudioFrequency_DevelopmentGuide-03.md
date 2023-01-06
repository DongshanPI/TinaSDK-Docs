## 3 常用工具及调试方法

### 3.1 3.1 alsa-utils

标准ALSA工具,它使用到alsa-lib标准库，一般常用到的有amixer,aplay,arecord等。

#### 3.1.1 3.1.1 amixer

amixer是命令行的ALSA声卡驱动调节器工具，用于设置mixer control。

使用方法：

• 常用选项

| 选项        | 功能                        |
| ----------- | --------------------------- |
| -D,--device | 指定声卡设备,默认使用defaul |

• 常用命令

| 命令     | 功能                             |
| -------- | -------------------------------- |
| controls | 列出指定声卡的所有控件           |
| contents | 列出指定声卡的所有控件的具体信息 |
| cget     | 获取指定控件的信息               |
| cset     | 设定指定控件的值                 |

举例：

```
获取audiocodec声卡的所有控件名
amixer -Dhw:audiocodec controls
获取当前硬件音量
amixer -Dhw:audiocodec cget name='LINEOUT volume'
设置当前硬件音量
amixer -Dhw:audiocodec cget name='LINEOUT volume' 25
```

#### 3.1.2 3.1.2 aplay

aplay是命令行的ALSA声卡驱动的播放工具，用于播放功能。

使用方法：

| 选项              | 功能                                                         |
| ----------------- | ------------------------------------------------------------ |
| -D,--device       | 指定声卡设备,默认使用default                                 |
| -l,--list-devices | 列出当前所有声卡                                             |
| -t,--file-type    | 指定播放文件的格式,如voc,wav,raw,不指定的情况下会去读取文件<br/>头部作识别 |
| -c,--channels     | 指定通道数                                                   |
| -f,--format       | 指定采样格式                                                 |
| -r,--rate         | 采样率                                                       |
| -d,--duration     | 指定播放的时间                                               |
| --period-size     | 指定period size                                              |
| --buffer-size     | 指定buffer size                                              |

如果播放的是wav文件，可以解析头部，识别通道数，采样率等参数。

举例：

```
aplay -Dhw:audiocodec /mnt/UDISK/test.wav
```

#### 3.1.3 3.1.3 arecord.

arecord是命令行的ALSA声卡驱动的录音工具，用于录音功能。

使用方法：

| 选项              | 功能                                                         |
| ----------------- | ------------------------------------------------------------ |
| -D,--device       | 指定声卡设备,默认使用default                                 |
| -l,--list-devices | 列出当前所有声卡                                             |
| -t,--file-type    | 指定播放文件的格式,如voc,wav,raw,不指定的情况下会去读取文件<br/>头部作识别 |
| -c,--channels     | 指定通道数                                                   |
| -f,--format       | 指定采样格式                                                 |
| -r,--rate         | 采样率                                                       |
| -d,--duration     | 指定播放的时间                                               |
| --period-size     | 指定period size                                              |
| --buffer-size     | 指定buffer size                                              |

举例：

```
录制5s,通道数为2,采样率为16000,采样精度为16bit,保存为wav文件
arecord -Dhw:audiocodec -f S16_LE -r 16000 -c 2 -d 5 /mnt/UDISK/test.wav
```

#### 3.1.4 3.1.4 alsaconf

alsaconf指的是ALSA configuration file，使用alsa-lib打开声卡，操作pcm, mixer时，会加载相关位置上的配置文件，用于指导操作pcm,mixer设备。

首先会读取配置文件/usr/share/alsa/alsa.conf，其中有下面一段hooks。

```
@hooks [
{
func load
files [
{
@func concat
strings [
{ @func datadir }
"/alsa.conf.d/"
]
}
"/etc/asound.conf"
"~/.asoundrc"
]
errors false
}
]
```

这里设定了一个钩子，去读取相关目录配置文件：

```
/usr/share/alsa/alsa.conf.d/
/etc/asound.conf
~/.asoundrc
```

这些配置文件可以设定defaut声卡，自定义pcm设备,alsa插件等功能，具体可以参考:

https://www.alsa-project.org/alsa-doc/alsa-lib/conf.html

https://www.alsa-project.org/alsa-doc/alsa-lib/pcm_plugins.html

tina sdk下有相关软件包会设置/etc/asound.conf，可以用作参考。

使用方法：

tina根目录下执行make menuconfig,选择alsa-conf-aw软件包。

![3-1-4-1](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-3-1-4-1.jpg)

![3-1-4-2](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-3-1-4-2.jpg)

它会生成/etc/asound.conf文件，下面作简单介绍：

```
设定amixer操作的defautl声卡(执行snd_hctl_open会获取该配置)
ctl.!default {
type hw
card audiocodec
}
设定default声卡(执行snd_pcm_open会获取该配置)
pcm.!default {
type asym
playback.pcm "PlaybackDmix"
capture.pcm "CaptureDsnoop"
}

使用dmix插件，可以混合播歌，即支持多次打开声卡进行播歌
pcm.PlaybackDmix {
type plug
slave.pcm {
type dmix
ipc_key 1111
ipc_perm 0666
slave {
pcm "hw:audiocodec"
rate 48000
channels 2
}
}
}

使用dsnoop插件，可以混合录音，即支持多次打开声卡进行录音
pcm.CaptureDsnoop {
type plug
slave.pcm {
type dsnoop
ipc_key 1111
ipc_perm 0666
slave {
pcm "hw:audiocodec,0"
rate 48000
channels 2
}
}
}

使用dmix插件以及softvol插件，softvol插件可以增加一个control，用于控制音量(软件上作调节)
pcm.PlaybackDmix {
type plug
slave.pcm {
type softvol
slave.pcm {
type dmix
ipc_key 1111
ipc_perm 0666
slave {
pcm "hw:audiocodec,0"
rate 48000
channels 1
}
}
control {
name "Soft Volume Master"
card audiocodec
}
min_dB -51.0
max_dB 0.0
resolution 256
}
}
```




### 3.2 3.2 tinyalsa-utils.

tinyalsa是alsa-lib的一个简化版。它提供了pcm和control的基本接口；没有太多太复杂的操作、功能。可以按需使用接口。tinyalsa-utils是基于tinyalsa的一些工具，下面对几个常用的工具作介绍。

#### 3.2.1 3.2.1 tinymix

与amixer作用类似，用于操作mixer control。

• 常用选项

| 选项     | 功能                       |
| -------- | -------------------------- |
| -D,–card | 指定声卡设备,默认使用card0 |

• 常用命令

| 命令     | 功能                             |
| -------- | -------------------------------- |
| controls | 列出指定声卡的所有控件           |
| contents | 列出指定声卡的所有控件的具体信息 |
| get      | 获取指定控件的信息               |
| set      | 设定指定控件的值                 |

举例：

```
获取card0的所有控件名
tinymix -D 0 controls
获取card0当前硬件音量
tinymix -D 0 get 'LINEOUT volume'
设置card0当前硬件音量
tinymix -D 0 set 'LINEOUT volume' 25
```

#### 3.2.2 3.2.2 tinyplay

与aplay作用类似,用于操作声卡设备进行播放

• 常用选项

| 选项            | 功能                       |
| --------------- | -------------------------- |
| -D,–card        | 指定声卡设备,默认使用card0 |
| -p,–period-size | 指定period大小,单位为帧    |
| -c,–channels    | 指定通道数                 |
| -r,–rate        | 指定采样率                 |
| -b,–bits        | 指定采样精度               |

如果播放的是wav文件,可以解析头部,识别通道数,采样率等参数

举例：

```
tinyplay -D 0 /tmp/16000-stere-10s.wav
```

#### 3.2.3 3.2.3 tinycap

与arecord作用类似，用于操作声卡进行录音功能

• 常用选项

| 选项            | 功能                       |
| --------------- | -------------------------- |
| -D,–device      | 指定声卡设备,默认使用card0 |
| -p,–period-size | 指定period大小,单位为帧    |
| -c,–channels    | 指定通道数                 |
| -r,–rate        | 指定采样率                 |
| -b,–bits        | 指定采样精度               |

举例：

```
录制通道数为2,采样率为16000,采样精度为16bit,保存为wav文件
tinycap -D 0 -b 16 -r 16000 -c 2 /mnt/UDISK/test.wav
```

### 3.3 3.3 dump寄存器

我们sunxi平台均提供了sunxi_dump驱动，用于查看读写寄存器。


节点位于/sys/class/sunxi_dump目录。

但是audiocodec模拟寄存器的操作会有些特殊，我们一般在audio驱动中都会增加相关调试节点，去操作自己模块的寄存器,以便调试。

#### 3.3.1 3.3.1 dump audiocodec寄存器

audiocodec驱动的寄存器调试节点位于:

```
/sys/devices/platform/soc/codec/audio_reg_debug/audio_reg
```

使用方法:

通过echo写入下列参数

参数1: 0-read; 1-write

参数2: 1-digitar reg; 2-analog reg

参数3: reg value

参数4: write value

举例：

查看所有寄存器状态:

```
cat /sys/devices/platform/soc/codec/audio_reg_debug/audio_reg
打印如下(其中地址为0x300以上的寄存器为模拟寄存器，其他均为数字寄存器)：
dump audio reg:
SUNXI_DAC_DPC [0x000]: 0x0 Save:0x0
SUNXI_DAC_FIFO_CTL [0x010]: 0x3004000 Save:0x0
SUNXI_DAC_FIFO_STA [0x014]: 0x80800c Save:0x0
SUNXI_DAC_CNT [0x024]: 0xb4014 Save:0x0
SUNXI_DAC_DG [0x028]: 0x0 Save:0x0
SUNXI_ADC_FIFO_CTL [0x030]: 0xe000400 Save:0x0
SUNXI_ADC_FIFO_STA [0x038]: 0x0 Save:0x0
SUNXI_ADC_CNT [0x044]: 0x0 Save:0x0
SUNXI_ADC_DG [0x04c]: 0x0 Save:0x0
SUNXI_DAC_DAP_CTL [0x0f0]: 0x0 Save:0x0
SUNXI_ADC_DAP_CTL [0x0f8]: 0x0 Save:0x0
SUNXI_HP_CTL [0x300]: 0x0 Save:0x0
SUNXI_MIX_DAC_CTL [0x303]: 0x0 Save:0x0
SUNXI_LINEOUT_CTL0 [0x305]: 0x10 Save:0x0
SUNXI_LINEOUT_CTL1 [0x306]: 0x19 Save:0x0
SUNXI_MIC1_CTL [0x307]: 0x34 Save:0x0
SUNXI_MIC2_MIC3_CTL [0x308]: 0x4 Save:0x0
SUNXI_LADCMIX_SRC [0x309]: 0x4 Save:0x0
SUNXI_RADCMIX_SRC [0x30a]: 0x8 Save:0x0
SUNXI_XADCMIX_SRC [0x30b]: 0x10 Save:0x0
```

```
SUNXI_ADC_CTL [0x30d]: 0x3 Save:0x0
SUNXI_MBIAS_CTL [0x30e]: 0x21 Save:0x0
SUNXI_APT_REG [0x30f]: 0xd6 Save:0x0
SUNXI_OP_BIAS_CTL0 [0x310]: 0x55 Save:0x0
SUNXI_OP_BIAS_CTL1 [0x311]: 0x55 Save:0x0
SUNXI_ZC_VOL_CTL [0x312]: 0x2 Save:0x0
SUNXI_BIAS_CAL_CTRL [0x315]: 0x0 Save:0x0
```

查看某个数字寄存器状态：

```
echo 0,1,0x10 > /sys/devices/platform/soc/codec/audio_reg_debug/audio_reg
打印如下：
[ 127.036609] sunxi-internal-codec codec: ret:3, reg_group:1, reg_offset:16, reg_val:0x0
[ 127.045557] sunxi-internal-codec codec:
[ 127.045557]
[ 127.045557] Reg[0x10] : 0x03004000
[ 127.045557]
表示0x10数字寄存器的值为0x03004000
```

查看某个模拟寄存器状态：

```
echo 0,2,0x5 > /sys/devices/platform/soc/codec/audio_reg_debug/audio_reg
打印如下：
[ 306.126103] sunxi-internal-codec codec: ret:3, reg_group:2, reg_offset:5, reg_val:0x0
[ 306.134971] sunxi-internal-codec codec:
[ 306.134971]
[ 306.134971] Reg[0x05] : 0x10
[ 306.134971]
表示0x05模拟寄存器的值为0x10
```

改写某个数字寄存器:

```
echo 1,1,0x24,0 > /sys/devices/platform/soc/codec/audio_reg_debug/audio_reg
表示将0x24数字寄存器写为0x0
```

改写某个模拟寄存器:

```
echo 1,2,0x3,0x1 > /sys/devices/platform/soc/codec/audio_reg_debug/audio_reg
表示将0x03模拟寄存器写为0x1
```

#### 3.3.2 3.3.2 dump daudio寄存器

查看spec可以知道i2s模块的寄存器基地址


```
i2s0: 0x05090000
i2s1: 0x05091000
i2s2: 0x05092000
```

可以通过sunxi_dump节点查询寄存器状态，例如查看i2s0的寄存器:

```
cd /sys/class/sunxi_dump
echo 0x05090000,0x050900a0 > dump
cat dump
```

#### 3.3.3 3.3.3 dump dmic寄存器

查看spec可以知道dmic模块的寄存器基地址

```
dmic: 0x05095000
```

可以通过sunxi_dump节点查询寄存器状态:

```
cd /sys/class/sunxi_dump
echo 0x05095000,0x05095050 > dump
cat dump
```

#### 3.3.4 3.3.4 dump spdif寄存器

查看spec可以知道spdif模块的寄存器基地址

```
spdif: 0x05093000
```

可以通过sunxi_dump节点查询spdif寄存器状态:

```
cd /sys/class/sunxi_dump
echo 0x05093000,0x05093040 > dump
cat dump
```

### 3.4 3.4 sound procfs.

通过procfs文件系统下面的声卡相关节点，可以得到各个声卡各个音频流的状态。实际调试中会非常有用。


内核需要选中下面选项才能在procfs下生成对应节点：

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
[*] Sound Proc FS Support
[*] Verbose procfs contents
```

以card0为例看下提供的节点信息：

```
ddd
/proc/asound/card0/
├── id /*声卡名称*/
├── pcm0c /* pcm0录音流*/
│ ├── info /* pcm信息*/
│ └── sub0
│ ├── hw_params /*硬件参数信息*/
│ ├── info /* pcm信息*/
│ ├── status /* pcm流运行状态*/
│ └── sw_params /*软件参数信息*/
└── pcm0p /* pcm0播放流*/
├── info
└── sub0
```

其中，hw_params, status都能拿到比较有用的信息：

```
cat /proc/asound/card0/pcm0c/sub0/hw_params
access: RW_INTERLEAVED /*交错模式排列通道*/
format: S16_LE /*当前音频流的采样精度*/
subformat: STD
channels: 2 /*通道数*/
rate: 16000 (16000/1) /*采样率*/
period_size: 320 /*周期(决定dma中断时间,例如这里period_time=320/16000=20ms)
*/
buffer_size: 2560 /*内核ALSA框架中环形缓冲区大小,决定能够缓存多少个period */
cat /proc/asound/card0/pcm0c/sub0/status
state: RUNNING /*音频流运行状态,RUNNING, SETUP等状态*/
owner_pid : 22653
trigger_time: 81828.078175765
tstamp : 82373.796969347 /*开始运行后的时间戳信息*/
delay : 256
avail : 256 /*当前可用音频数据帧数*/
avail_max : 320
-----
hw_ptr : 8731456 /*硬件逻辑指针，单位(帧) */
appl_ptr : 8731200 /*应用逻辑指针，单位(帧) */
```

- 从period_size可以知道当前dma中断频率，太快会影响系统响应速度，太慢可能就存在一定延时。
- buffer_size可以知道缓存区大小，太小容易因调度不及时出现xrun,太大同样存在一定延时。
- 从hw_ptr, appl_ptr可以知道当前录音/播音的帧数，是否发生过xrun等。