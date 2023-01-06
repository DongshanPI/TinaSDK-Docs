
## 2 模块介绍

Linux中的音频子系统采用ALSA架构实现。ALSA目前已经成为了Linux的主流音频体系结构。在内核设备驱动层，ALSA提供了alsa-driver，同时在应用层，ALSA为我们提供了alsa-lib，应用程序只要调用alsa-lib提供的API，即可以完成对底层音频硬件的控制。

### 2.1 驱动框架

Tina SDK对各个平台的音频设备驱动均采用ASoC架构实现。ASoC是建立在标准alsa驱动层上，为了更好地支持嵌入式处理器和移动设备中的音频codec的一套软件体系，ASoC将音频系统分为 3 部分：Codec，Platform和Machine。

1. Codec驱动

   ASoC中的一个重要设计原则就是要求Codec驱动是平台无关的,它包含了一些音频的控件(Controls),音频接口,DAMP(动态音频电源管理)的定义和某些Codec IO功能。为了保证硬件无关性,任何特定于平台和机器的代码都要移到Platform和Machine驱动中。
   所有的Codec驱动都要提供以下特性：

       - Codec DAI (Digital Audio Interface)和PCM的配置信息；
       - Codec的IO控制方式(I2C,SPI等);
       - Mixer和其他的音频控件;
       - Codec和ALSA音频操作接口;

2. Platform驱动

   它包含了该SoC平台的音频DMA和音频接口的配置和控制（I2S，PCM，AC97等等）；
   一般不包含与板子或codec相关的代码。

3. Machine驱动单独的Platform和Codec驱动是不能工作的，它必须由Machine驱动把它们结合在一起才能完成整个设备的音频处理工作。


![图2-1: ASoC框图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-1.jpg)

### 2.2 音频接口介绍.

我们提供的音频接口有:

- AudioCodec
- Daudio(I2S)
- Dmic
- Spdif
- MAD

不同芯片平台的音频接口资源会有差异；不同版本的内核，对应的ALSA驱动也有所不同；下面会对各个芯片作详细介绍。


### 2.3 R6音频接口

#### 2.3.1 硬件资源

R6包含 2 个音频模块，分别是内置audiocodec以及daudio0。

![图2-2: R6音频硬件框图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-2.jpg)

#### 2.3.2 时钟源

R6中， 2 个音频模块的时钟源均来自pll_audio。

pll_audio可以输出24.576M或者22.5792M的时钟，分别支持48k系列，44.1k系列的播放录音。


![图2-3: R6时钟源](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-3.jpg)

#### 2.3.3 代码结构

```
linux-3.10/sound/soc/sunxi/
├── sun3iw1_ac101.c // daudio+ac101的machine驱动
├── sun3iw1_codec.c // codec 驱动
├── sun3iw1_codec.h
├── sun3iw1_daudio.c // daudio的platform驱动
├── sun3iw1_daudio.h
├── sun3iw1_sndcodec.c // codec machine驱动
├── sunxi_cpudai.c // codec platform驱动
├── sunxi_cpudai.h
├── sunxi_dma.c //通用文件，提供注册platform驱动的接口及相关函数集
├── sunxi_dma.h
├── sunxi_rw_func.c //通用文件，读写模拟/数字寄存器的接口
└── sunxi_rw_func.h

linux-3.10/sound/soc/codecs/
├── ac101.c // daudio+ac101的codec驱动
└── ac101.h
```

#### 2.3.4 Audiocodec.

硬件特性

- 两路DAC

  - 支持16bit,24bit采样精度
  - 支持8KHz~192KHz采样率

- 一路ADC

  - 支持16bit,24bit采样精度
  - 支持8KHz~48KHz采样率

- 一路模拟输出：一路立体声headphone输出(HPL, HPR)
- 四路模拟输入：MIC,FMINL,FMINR,LINEIN
- 支持同时playback和record(全双工模式)

##### 2.3.4.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
    <*> Advanced Linux Sound Architecture --->
        <*> ALSA for SoC audio support --->
            <*> ASoC support for SUNXI --->
                <*> ASoC support for sun3iw1 audiocodec
                <*> ASoC support for internal-codec cpudai
                <*> ASoC support for sun3iw1 audiocodec machine
```

##### 2.3.4.2 sys_config配置.

```
[sndcodec]
sndcodec_used = 0x1
;------------------------------------------------------------------------------
[cpudai]
cpudai_used = 0x1
;------------------------------------------------------------------------------
[codec]
codec_used = 0x1
headphonevol = 0x3b
maingain = 0
pa_sleep_time = 30
gpio-spk = port:PD03<1><1><default><default>
gpio_shdn = 1
```

sndcodec配置，即machine驱动的相关配置。

| sndcodec配置  | sndcodec配置说明                             |
| :------------ | :------------------------------------------- |
| sndcodec_used | 是否使用sndcodec驱动。 0 ：不使用； 1 ：使用 |

cpudai配置，即platform驱动的相关配置。


| cpudai配置                       | cpudai配置说明        |
| :------------------------------- | :-------------------- |
| cpudai_used 是否使用cpudai驱动。 | 0 ：不使用； 1 ：使用 |


codec配置，即内置audiocodec驱动的相关配置。


| codec配置     | codec配置说明                                                |
| :------------ | :----------------------------------------------------------- |
| codec_used    | 是否使用codec驱动。 0 ：不使用； 1 ：使用                    |
| headphonevol  | headphone volume，可设定范围0~0x3f, 0表示mute, 1~63表示-62dB~0dB, 1dB/step |
| micgain       | mic增益，可设定范围0~0x7, 0:0dB, 1~7:15~33dB, 3dB/step,一般设置0x4,即24dB.如果作为aec回路，则需要设置为0dB |
| pa_sleep_time | 操作PA之后的延时时间(用来避免pop音),单位ms                   |
| gpio-spk      | PA使能引脚                                                   |
| gpio_shdn     | PA引脚使能方式。0:低电平有效； 1 ：高电平有效                |



> 说明

- 如果想要正常加载audiocodec 声卡， 需要把codec,platform,machine 驱动都选上， 即codec_used,cpudai_used,sndcodec_used 都置为1；
- headphonevol 等值会在驱动初始化的时候设置，进入系统后还可以通过amixer 工具对应控件进行再次修改；
- 注意gpio-spk 是否配置正确，是否有其他模块复用了该gpio;
- 除了gpio-spk 指定pa 使能引脚外，驱动中也会检测gpio_num 字段，所以可以直接将gpio 号赋值gpio_num;
- 注意gpio_shdn，实际功放的PA 引脚是高电平有效，还是低电平有效


##### 2.3.4.3 codec数据通路

![图2-4: R6音频通路](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-4.jpg)

R6平台的audiocodec驱动会在播歌的时候自动设置相关通路，默认audio map:

```
播歌
DACL --> HP_L Mux --> HPOUTL
DACR --> HP_R Mux --> HPOUTR
```

录音功能则根据需要操作对应空间使能通路:

```
录制单MIC数据
MICIN --> ADC Mixer -> ADC

录制内部AEC数据(不需要外围回采电路)
Left Output Mixer --> ADC Mixer -> ADC
Right Output Mixer --> ADC Mixer -> ADC
```

R6相关控件如下表：

| 控件名称                        | 功能                                     | 数值                                      |
| :------------------------------ | :--------------------------------------- | :---------------------------------------- |
| ADC INPUT GAIN control          | ADC增益                                  | 0–7,表示-4.5–6dB                          |
| ADC MIC Boost AMP               | enMIC Boost AMP使能                      | 0:关闭; 1:开启                            |
| ADC MIC Boost AMP               | gain control    MIC增益                  | 0–7, 0:0dB, 1~7:15–33dB                   |
| ADC PA speed select             | PA速度选择 0:normal; 1:fast              |                                           |
| ADC mixer mute for FML          | ADC Mixer设置，使能FML通路               | 0:关闭; 1:开启                            |
| ADC mixer mute for FMR          | ADC Mixer设置，使能FMR通路               | 0:关闭; 1:开启                            |
| ADC mixer mute for left output  | ADC Mixer设置，使能left output Mixer通路 | 0:关闭; 1:开启                            |
| ADC mixer mute for linein       | ADC Mixer设置，使能linein通路            | 0:关闭; 1:开启                            |
| ADC mixer mute for mic          | ADC Mixer设置，使能mic通路               | 0:关闭; 1:开启                            |
| ADC mixer mute for right output |                                          | ADC Mixer设置，使能right output Mixer通路 |
| LINEIN GAIN control             | linein到output mixer的增益               | 0–7, 0~7:0–14dB,2dB/step                  |
| MICIN GAIN control              | MIC到outpu mixer的增益                   | 0–7,表示-4.5–6dB                          |
| dac digital volume              | DAC数字音量                              | 0~63,表示0~-73.08dB,-1.16dB/step          |
| head phone volume               | headphone音量                            | 0 表示mute, 1~63表示-62dB~0dB, -1dB/step  |

录音通路设置举例：

1. 录音单声道数据

```
通过MICIN录音：
amixer -D hw:audiocodec cset name='ADC MIC Boost AMP en' 1
amixer -D hw:audiocodec cset name='ADC mixer mute for mic' 1
amixer -D hw:audiocodec cset name='ADC MIC Boost AMP gain control' 4
```

2. 内部AEC(可省去外部AEC电路)

```
amixer -D hw:audiocodec cset name='ADC mixer mute for left ouput' 1
```

#### 2.3.5 Daudio.

硬件特性

- 一路I2S/PCM；
- 支持主从模式
- 支持Left-justified,Right-justified,Standar mode I2S,PCM mode
- 支持i2s,pcm协议格式配置
- 支持mono和stereo模式，最高支持 2 通道
- 支持同时playback和record(全双工模式)
- 支持8~192KHz采样率
- 支持16,24,32bit采样精度

##### 2.3.5.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
        <*> Advanced Linux Sound Architecture --->
            <*> ALSA for SoC audio support --->
                <*> ASoC support for SUNXI --->
                    <*> ASoC support for daudio platform
                    <*> ASoC support for sun3iw1 & ac101 daudio machine
```

##### 2.3.5.2 sys_config配置.

```
[snddaudio0]
snddaudio0_used = 1
over_sample_rate = 128
[daudio0]
daudio0_used = 1
word_select_size = 32
pcm_sync_period = 32
pcm_lsb_first = 0
over_sample_rate = 128
slot_width_select = 16
pcm_sync_type = 0
pcm_start_slot = 0
tdm_config = 1
```

snddaudio0配置，即daudio0 machine驱动的相关配置


| snddaudio配置    | snddaudio配置说明                             |
| :--------------- | :-------------------------------------------- |
| snddaudio0_used  | 是否使用snddaudio驱动。 0 ：不使用； 1 ：使用 |
| over_sample_rate | 支持128fs/192fs/256fs/384fs/512fs/768fs       |


daudio0配置，即daudio0 platform驱动的相关配置

| snddaudio配置    | snddaudio配置说明                          |
| :--------------- | :----------------------------------------- |
| daudio0_used     | 是否使用daudio驱动。 0 ：不使用； 1 ：使用 |
| word_select_size | 支持16bits/20bits/24bits/32bits            |
| pcm_sync_period  | 16/32/64/128/256                           |

| snddaudio配置     | snddaudio配置说明                                 |
| :---------------- | :------------------------------------------------ |
| pcm_lsb_first     | 0: msb first; 1: lsb first                        |
| over_sample_rate  | 支持128fs/192fs/256fs/384fs/512fs/768fs           |
| slot_width_select | 16bits/20bits/24bits/32bits                       |
| pcm_sync_type     | 0: long frame sync; 1: short frame sync           |
| pcm_start_slot    | 0: 1st slot; 1: 2nd slot; 2: 3th slot; 3:4th slot |
| tdm_config        | 0:pcm 1:i2s                                       |

#### 2.3.6 外挂codec:AC101

R6标案使用的AC101作双声道录音，audiocodec则录制回路作AEC下面对R6如何配置使用AC101作简单介绍

##### 2.3.6.1 内核配置

```
Device Drivers --->
    <*> Sound card support --->
        <*> Advanced Linux Sound Architecture --->
            <*> ALSA for SoC audio support --->
                <*> AC101 Codec
                <*> ASoC support for SUNXI --->
                    <*> ASoC support for daudio platform
                    <*> ASoC support for sun3iw1 & ac101 daudio machine
```

##### 2.3.6.2 sys_config&dts配置.

R6通过TWI1控制AC101,而I2S0用于音频数据的传输TWI部分配置,可通过dts进行配置:

```
linux-3.10/arch/arm/boot/dts/sun3iw1p1-sitar-mic2.dts
twi1: twi@0x01c27400{
    ac101@1a {
        compatible = "x-powers,ac101";
        reg = <0x1a>;
        audio_int_ctrl = <&pio PL 12 6 1 1 0>;
        audio_pa_ctrl = <&pio PG 13 1 1 1 0>;
        speaker_val = <0x1b>;
        headset_val = <0x3b>;
        single_speaker_val = <0x19>;
        double_speaker_val = <0x1b>;
        speaker_double_used = <1>;
        earpiece_val = <0x1e>;
        mainmic_val = <0x4>;
        headsetmic_val = <0x4>;
        dmic_used = <0>;
        adc_digital_val = <0xb0b0>;
        agc_used = <0>;
        drc_used = <1>;
        linein_to_spk_used = <0>;
        linein_to_hp_used = <0>;
        linein_to_aif2_used = <0>;
        };
}
```

I2S部分配置可以通过dts配置，也可以通过sys_config覆盖dts的配置

```
[snddaudio0]
snddaudio0_used = 1
over_sample_rate = 128
sunxi,snddaudio-codec = "ac101.1-001a"
sunxi,snddaudio-codec-dai = "ac101"

[daudio0]
daudio0_used = 1
word_select_size = 32
pcm_sync_period = 32
pcm_lsb_first = 0
over_sample_rate = 128
slot_width_select = 16
pcm_sync_type = 0
pcm_start_slot = 0
tdm_config = 1
```

i2s相关格式需要根据AC101 spec进行配置

而snddaudio0中，注意codec的名称，需要与实际AC101的dev name相匹配,而codec-dai名称则与AC101驱动中设置的dai name相匹配

#### 2.3.7 标案音频测试方法

该章节主要介绍在标案上进行播歌，录音的测试命令。

##### 2.3.7.1 播放

如《R6 AudioCodec数据通路》章节所说，驱动代码中已固定配置了播放通路进入系统后直接通过aplay工具进行播放即可，如：

```
aplay -Dhw:audiocodec /mnt/UDISK/1KHz_0dB_16000.wav
```

可通过下面命令调节硬件上的模拟音量:

```
amixer -Dhw:audiocodec cset name='headphone volume' 50
```

##### 2.3.7.2 录音

标案使用AC101进行双声道录音录音前需要配置AC101的音频通路，SDK默认在启动时会进行设置，相关配置脚本在:

```
/etc/init.d/rc.final
```

可以直接通过arecord命令进行录音:

```
arecord -Dhw:sndac1011001a -f S16_LE -r 16000 -c 2 /tmp/test.wav
```

### 2.4 R7s音频接口

#### 2.4.1 硬件资源

R7s包含 2 个音频模块，分别是内置AudioCodec以及Daudio0。


![图2-5: R7s音频硬件框图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-5.jpg)


#### 2.4.2 时钟源

R7s中， 2 个音频模块的时钟源均来自pll_audio。

pll_audio可以输出24.576M或者22.5792M的时钟，分别支持48k系列，44.1k系列的播放录音。


![图2-6: R7s时钟源](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-6.jpg)

#### 2.4.3 代码结构

```
linux-4.9/sound/soc/sunxi/
├── sunxi-pcm.c //提供注册platform驱动的接口及相关函数集
├── sunxi-pcm.h
├── sun8iw8
│   ├── sunxi_codec.c // cpudai驱动
│   ├── sunxi_codecdma.c // codec platform驱动
│   ├── sun8iw8_sndcodec_new.c // codec驱动
│   └── sunxi_sndcodec.c // codec machine驱动
├── sunxi-daudio.c // daudio platform驱动
└── sunxi-snddaudio.c // daudio machine驱动

linux-4.9/sound/soc/soc-utils.c // daudio codec驱动
```

#### 2.4.4 Audiocodec.

硬件特性

- 两路DAC

  - 支持16bit,24bit采样精度
  - 支持8KHz~192KHz采样率

- 两路ADC

  - 支持16bit,24bit采样精度
  - 支持8KHz~48KHz采样率

- 两路模拟输出:

  - 一路立体声LINEOUT输出(LINEOUTP, LINEOUTN)
  - 一路立体声headphone输出(HPOUTL, HPOUTR)

- 两路模拟输入：MIC1,MIC2

  - 支持同时playback和record(全双工模式)
  - 支持ADC的AGC,DRC功能
  - 支持DAC的DRC功能

##### 2.4.4.1 内核配置

```
Device Drivers --->
    <*> Sound card support --->
        <*> Advanced Linux Sound Architecture --->
            <*> ALSA for SoC audio support --->
                <*> Audiocodec for the SUNXI chips
```

##### 2.4.4.2 sys_config配置.

```
[codec]
headphone_vol = 0x3b
lineout_vol = 0x1a
audio_pa_ctrl = port:PB05<1><default><default><0>
adcagc_used = 0
adcdrc_used = 0
dacdrc_used = 0
adchpf_used = 0
dachpf_used = 0
```


| codec配置     | codec配置说明                                                |
| :------------ | :----------------------------------------------------------- |
| headphone_vol | headphone volume，可设定范围0~0x3f, 0表示mute, 1~63表示-62dB~0dB, 1dB/step |
| audio_pa_ctrl | PA使能引脚                                                   |
| adcagc_used   | 1:use adcagc 0:no use                                        |
| adcdrc_used   | 1:use adcdrc 0:no use                                        |
| dacdrc_used   | 1:use dacdrc 0:no use                                        |
| adchpf_used   | 1:use adchpf 0:no use                                        |
| dachpf_used   | 1:use dachpf 0:no use                                        |


##### 2.4.4.3 codec数据通路


![图2-7: R7s音频通路](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-4.jpg)

```
播歌
DACL --> Left Output Mixer --> LINEOUTL
DACR --> Right Output Mixer --> LINEOUTR

录音
MIC1P --> LADC input Mixer --> ADCL
MIC2P --> RADC input Mixer --> ADCR
```

| 控件名称                                 | 功能                                   | 数值                                |
| :--------------------------------------- | :------------------------------------- | :---------------------------------- |
| Lineout volume                           | lineout音量设置                        | 0–31,表示-43.5–0dB                  |
| ADC input gain control                   | ADC增益 0–7,表示-4.5–6dB               |                                     |
| HP_L Mux HP_L                            | Mux设置                                | 0:DACL HPL Switch; 1:MIXER_L Switch |
| HP_R Mux HP_R                            | Mux设置                                | 0:DACR HPR Switch; 1:MIXER_R Switch |
| LADC input Mixer MIC1 boost Switch       | LADC input Mixer设置，使能MIC1通路     | 0:关闭; 1:开启                      |
| LADC input Mixer MIC2 boost Switch       | LADC input Mixer设置，使能MIC2通路     | 0:关闭; 1:开启                      |
| LADC input Mixer l_output mixer Switch   | LADC input Mixer设置，使能l_output通路 | 0:关闭; 1:开启                      |
| LADC input Mixer r_output mixer Switch   | LADC input Mixer设置，使能r_output通路 | 0:关闭; 1:开启                      |
| Left Output Mixer DACL Switch            | Left Output Mixer设置，使能DACL通路    | 0:关闭; 1:开启                      |
| Left Output Mixer DACR Switch            | Left Output Mixer设置，使能DACR通路    | 0:关闭; 1:开启                      |
| Left Output Mixer MIC1Booststage Switch  | Left Output Mixer设置，使能MIC1通路    | 0:关闭; 1:开启                      |
| Left Output Mixer MIC2Booststage Switch  | Left Output Mixer设置，使能MIC2通路    | 0:关闭; 1:开启                      |
| MIC1 boost AMP gain control              | MIC1增益                               | 0–7, 0:0dB,1~7:24–42dB,3dB/step     |
| MIC1_G boost stageoutput mixer control   | MIC1 to L or R output Mixer增益        | 0–7,表示-4.5–6dB                    |
| MIC2 SRC                                 | MIC2 SRC设置                           | 0:MIC3; 1:MIC2                      |
| MIC2 boost AMP gain control              | MIC2增益                               | 0–7, 0:0dB,1~7:24–42dB,3dB/step     |
| MIC2_G boost stage output mixer control  | MIC2 to L or R output Mixer增益        | 0–7,表示-4.5–6dB                    |
| RADC input Mixer MIC1 boost Switch       | RADC input Mixer设置，使能MIC1通路     | 0:关闭; 1:开启                      |
| RADC input Mixer MIC2 boost Switch       | RADC input Mixer设置，使能MIC2通路     | 0:关闭; 1:开启                      |
| RADC input Mixer l_output mixer Switch   | RADC input Mixer设置，使能l_output通路 | 0:关闭; 1:开启                      |
| RADC input Mixer r_output mixer Switch   | RADC input Mixer设置，使能r_output通路 | 0:关闭; 1:开启                      |
| Right Output Mixer DACL Switch           | Right Output Mixer设置，使能DACL通路   | 0:关闭; 1:开启                      |
| Right Output Mixer DACR Switch           | Right Output Mixer设置，使能DACR通路   | 0:关闭; 1:开启                      |
| Right Output Mixer MIC1Booststage Switch | Right Output Mixer设置， 使能MIC1通路  | 0:关闭; 1:开启                      |
| Right Output Mixer MIC2Booststage Switch | Right Output Mixer设置，使能MIC2通路   | 0:关闭; 1:开启                      |
| SPK_L Mux                                | SPK_L Mux设置                          | 0:MIXER_L Switch; 1:MIXR+MIXL       |
| SPK_R Mux                                | SPK_R Mux设置                          | 0:MIXER_L Switch; 1:MIXR+MIXL       |
| digital volume                           | 数字音量设置                           | 0–63,表示-73.08–0dB                 |
| headphone volume                         | headphone音量设置                      | 0–63,0表示mute; 1~63表示-62dB–0dB   |

通路设置举例：

1. 播放通路

```
通过lineout播放：
amixer -D hw:audiocodec cset name='SPK_L Mux' 1
amixer -D hw:audiocodec cset name='SPK_R Mux' 1
amixer -D hw:audiocodec cset name='Right Output Mixer DACR Switch' 1
amixer -D hw:audiocodec cset name='Left Output Mixer DACL Switch' 1
amixer -D hw:audiocodec cset name='digital volume' 6
```

2. 录音通路

```
通过MIC1,MIC2录音:
amixer -D hw:audiocodec cset name='LADC input Mixer MIC1 boost Switch' 1
amixer -D hw:audiocodec cset name='RADC input Mixer MIC2 boost Switch' 1
amixer -D hw:audiocodec cset name='MIC2 SRC' 0
amixer -D hw:audiocodec cset name='MIC1 boost AMP gain control' 4
amixer -D hw:audiocodec cset name='MIC2 boost AMP gain control' 4
```

#### 2.4.5 Daudio.

硬件特性

- 一路I2S/PCM；
- 支持主从模式
- 支持Left-justified,Right-justified,Standar mode I2S,PCM mode
- 支持i2s,pcm协议格式配置
- 支持mono和stereo模式
- 支持同时playback和record(全双工模式)
- 支持8~192KHz采样率
- 支持16,24,32bit采样精度

##### 2.4.5.1 内核配置

```
Device Drivers --->
    <*> Sound card support --->
        <*> Advanced Linux Sound Architecture --->
            <*> ALSA for SoC audio support --->
                <*> Allwinner Digital Audio Support
```

##### 2.4.5.2 sys_config配置.

```
[tdm0]
daudio_used = 0
daudio_master = 4
daudio_select = 1
audio_format = 1
signal_inversion = 1
sample_resolution = 16
slot_width_select = 16
pcm_lrck_period = 32
pcm_lrckr_period = 1
msb_lsb_first = 0
sign_extend = 0
tx_data_mode = 0
rx_data_mode = 0
;i2s_mclk = port:PB08<2><1><default><default>
i2s_bclk = port:PG11<2><1><default><default>
i2s_lrclk = port:PG10<2><1><default><default>
i2s_dout0 = port:PG12<2><1><default><default>
i2s_dout1 =
i2s_dout2 =
i2s_dout3 =
i2s_din = port:PG13<2><1><default><default>
```


| tdm0配置          | tdm0配置说明                                                 |
| :---------------- | :----------------------------------------------------------- |
| daudio_master     | 1: SND_SOC_DAIFMT_CBM_CFM(codec clk & FRM master),即daudio接口作为slave, codec作为master2: SND_SOC_DAIFMT_CBS_CFM(codec clk slave & FRMmaster),一般不用3: SND_SOC_DAIFMT_CBM_CFS(codec clk master & frameslave),一般不用4: SND_SOC_DAIFMT_CBS_CFS(codec clk & FRM slave),即daudio接口作为master, codec作为slave |
| daudio_select     | 0: pcm mode; 1: i2s mode                                     |
| audio_format      | 1: SND_SOC_DAIFMT_I2S(standard i2s format)2: SND_SOC_DAIFMT_RIGHT_J(right justfied format)3: SND_SOC_DAIFMT_LEFT_J(left justfied format)4: SND_SOC_DAIFMT_DSP_A(pcm. MSB is available on 2ndBCLK rising edge after LRC rising edge)5: SND_SOC_DAIFMT_DSP_B(pcm. MSB is available on 1ndBCLK rising edge after LRC rising edge) |
| signal_inversion  | 1: SND_SOC_DAIFMT_NB_NF(normal bit clock + frame)2: SND_SOC_DAIFMT_NB_IF(normal BCLK + inv FRM)3: SND_SOC_DAIFMT_IB_NF(invert BCLK + nor FRM)4: SND_SOC_DAIFMT_IB_IF(invert BCLK + FRM) |
| sample_resolution | 采样精度,16bit, 24bit,32bit                                  |
| slot_width_select | 支持8bit, 16bit, 32bit宽度                                   |
| pcm_lrck_period   | 可配置16/32/64/128/256个bclk                                 |
| pcm_lrckr_period  | 可配置16/32/64/128/256个bclk                                 |
| msb_lsb_first     | 0: msb first; 1: lsb first                                   |
| sign_extend       | 0: zero pending; 1: sign extend                              |
| tx_data_mode      | 0: 16bit linear PCM;1: reserved;2: 8bit u-law;3: 8bit a-law  |
| rx_data_mode      | 0: 16bit linear PCM;1: reserved;2: 8bit u-law;3: 8bit a-law  |
| i2s_bclk          | i2s_bclk引脚                                                 |
| i2s_lrclk         | i2s_lrclk引脚                                                |
| i2s_dout0         | i2s_dout引脚                                                 |
| i2s_din           | i2s_din引脚                                                  |



#### 2.4.6 标案音频测试方法

该章节主要介绍在标案上进行播歌，录音的测试命令。

#### 2.4.6.1 播放

```
amixer -D hw:audiocodec cset name='SPK_L Mux' 1
amixer -D hw:audiocodec cset name='SPK_R Mux' 1
amixer -D hw:audiocodec cset name='Lineout volume' 24
amixer -D hw:audiocodec cset name='Right Output Mixer DACR Switch' 1
amixer -D hw:audiocodec cset name='Left Output Mixer DACL Switch' 1
amixer -D hw:audiocodec cset name='digital volume' 0
aplay -Dhw:audiocodec /mnt/UDISK/1KHz_0dB_16000.wav
```

可通过下面命令调节硬件上的模拟音量:


```
amixer -Dhw:audiocodec cset name='Lineout volume' 50
```

#### 2.4.6.2 录音

表示下使用audiocodec进行MIC1,MIC2录音。

```
amixer -D hw:audiocodec cset name='LADC input Mixer MIC1 boost Switch' 1
amixer -D hw:audiocodec cset name='RADC input Mixer MIC2 boost Switch' 1
amixer -D hw:audiocodec cset name='MIC1 boost AMP gain control' 4
amixer -D hw:audiocodec cset name='MIC2 SRC' 0
amixer -D hw:audiocodec cset name='MIC2 boost AMP gain control' 4

arecord -Dhw:audiocodec -f S16_LE -r 16000 -c 2 /tmp/test.wav
```

## 2.5 R11音频接口

### 2.5.1 硬件资源

R11包含 2 个音频模块，分别是内置AudioCodec以及Daudio0。


![图2-8: R11音频硬件框图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-8.jpg)

### 2.5.2 时钟源

R11中， 2 个音频模块的时钟源均来自pll_audio。

pll_audio可以输出24.576M或者22.5792M的时钟，分别支持48k系列，44.1k系列的播放录音。


![图2-9: R11时钟源](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-9.jpg)

### 2.5.3 代码结构

```
linux-3.4/sound/soc/sunxi/
├── audiocodec
│ ├── sun8iw8_sndcodec_new.c // codec 驱动
│ ├── sun8iw8_sndcodec.h
│ ├── sunxi_codec.c // cpu dai驱动
│ ├── sunxi_sndcodec.c // codec machine驱动
│ ├── sunxi_codecdma.c // codec platform驱动
│ └── sunxi_codecdma.h
└── daudio0
    ├── snddaudio0.c // daudio codec驱动
    ├── sunxi-daudio0.c // daudio cpu dai驱动
    ├── sunxi-daudio0.h
    ├── sunxi-daudiodma0.c // daudio platform 驱动
    ├── sunxi-daudiodma0.h
    └── sunxi-snddaudio0.c // daudio machine驱动
```

### 2.5.4 AudioCodec

硬件特性

- 两路DAC

  - 支持16bit,24bit采样精度
  - 支持8KHz~192KHz采样率

- 两路ADC

  - 支持16bit,24bit采样精度
  - 支持8KHz~48KHz采样率

- 一路模拟输出:一路立体声LINEOUT输出(LINEOUTP, LINEOUTN)
- 一路路模拟输入：MIC1
- 支持同时playback和record(全双工模式)
- 支持ADC的AGC,DRC功能
- 支持DAC的DRC功能

#### 2.5.4.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
    <*> Advanced Linux Sound Architecture --->
        <*> ALSA for SoC audio support --->
            <*> Audiocodec for the SUNXI chips
            <*> Audiocodec Machine for codec chips
            <*> Audiocodec for the SUN8IW8 chips
```

#### 2.5.4.2 sys_config配置.

```
[audio0]
headphone_vol = 0x3b
lineout_vol = 0x1a
audio_pa_ctrl = port:PB05<1><default><default><0>
audio_pa_active_level = 1
adcagc_used = 0
adcdrc_used = 0
dacdrc_used = 0
adchpf_used = 0
dachpf_used = 0
```


| audio0配置            | audio0配置说明                                               |
| :-------------------- | :----------------------------------------------------------- |
| headphone_vol         | headphone volume，可设定范围0~0x3f, 0表示mute,1~63表示-62dB~0dB, 1dB/step |
| lineout_vol           | lineout volume，可设定范围0~0x1f, 0或者 1 表示mute,2~31表示-43.5dB~0dB, 1.5dB/step |
| audio_pa_ctrl         | PA使能引脚                                                   |
| audio_pa_active_level | 1:high level active; 0:low level active                      |
| adcagc_used           | 1:use adcagc 0:no use                                        |
| adcdrc_used           | 1:use adcdrc 0:no use                                        |
| dacdrc_used           | 1:use dacdrc 0:no use                                        |
| adchpf_used           | 1:use adchpf 0:no use                                        |
| dachpf_used           | 1:use dachpf 0:no use                                        |


#### 2.5.4.3 codec数据通路

![图2-10: R11音频通路](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-10.jpg)

```
播歌
DACL --> Left Output Mixer --> LINEOUTL
DACR --> Right Output Mixer --> LINEOUTR

录音
MIC1P --> LADC input Mixer --> ADCL
```

| 控件名称                                 | 功能                                                     | 数值             |
| :--------------------------------------- | :------------------------------------------------------- | :--------------- |
| Lineout volume                           | lineout 音量设置0–31, 表示-43.5–0dB                      |                  |
| ADC input gain control                   | ADC 增益0–7, 表示-4.5–6dB                                |                  |
| LADC input Mixer MIC1 boost Switch       | LADC input Mixer 设置，使能MIC 通路 0: 关闭; 1: 开启     |                  |
| Left Output Mixer DACL Switch            | Left Output Mixer 设置，使能DACL 通路 0: 关闭; 1: 开启   |                  |
| Left Output Mixer DACR Switch            | Left Output Mixer 设置，使能DACR 通路 0: 关闭; 1: 开启   |                  |
| Left Output Mixer MIC1Booststage Switch  | Left Output Mixer 设置，使能MIC1 通路 0: 关闭; 1: 开启   |                  |
| MIC1 boost AMP gain control              | MIC1 增益0–7, 0:0dB, 1~7:24–42dB,3dB/step                |                  |
| MIC1_G boost stage output mixer control  | MIC1 to L or R output Mixer 增益0–7, 表示-4.5–6dB        |                  |
| Right Output Mixer DACL Switch           | Right Output Mixer 设置，使能DACL 通路                   | 0: 关闭; 1: 开启 |
| Right Output Mixer DACR Switch           | Right Output Mixer 设置， 使能DACR 通路 0: 关闭; 1: 开启 |                  |
| Right Output Mixer MIC1Booststage Switch | Right Output Mixer 设置， 使能MIC1 通路 0: 关闭; 1: 开启 |                  |
| SPK_L Mux SPK_L Mux                      | 设置0:MIXER_L Switch; 1:MIXR+MIXL                        |                  |
| SPK_R Mux SPK_R Mux                      | 设置0:MIXER_L Switch; 1:MIXR+MIXL                        |                  |
| digital volume                           | 数字音量设置0–63, 表示-73.08–0dB                         |                  |


通路设置举例：

1. 播放通路

```
通过lineout播放：
amixer -D hw:audiocodec cset name='SPK_L Mux' 1
amixer -D hw:audiocodec cset name='SPK_R Mux' 1
amixer -D hw:audiocodec cset name='Right Output Mixer DACR Switch' 1
amixer -D hw:audiocodec cset name='Left Output Mixer DACL Switch' 1
amixer -D hw:audiocodec cset name='digital volume' 0
```

2. 录音通路

```
通过MIC1录音:
amixer -D hw:audiocodec cset name='LADC input Mixer MIC1 boost Switch' 1
amixer -D hw:audiocodec cset name='MIC1 boost AMP gain control' 4
```

### 2.5.5 Daudio.

硬件特性

- 一路I2S/PCM；
- 支持主从模式
- 支持Left-justified,Right-justified,Standar mode I2S,PCM mode
- 支持i2s,pcm协议格式配置
- 支持mono和stereo模式，最高支持 2 通道
- 支持同时playback和record(全双工模式)
- 支持8~192KHz采样率
- 支持16,24,32bit采样精度

#### 2.5.5.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
    <*> Advanced Linux Sound Architecture --->
        <*> ALSA for SoC audio support --->
            <*> SoC daudio0 tdm interface for SUNXI chips
            <*> Daudio0 Public Machine for SUNXI chips
```

#### 2.5.5.2 sys_config配置.

```
[tdm0]
daudio_used = 1
daudio_master = 4
daudio_select = 1
audio_format = 1
signal_inversion = 1
sample_resolution = 16
slot_width_select = 16
pcm_lrck_period = 32
pcm_lrckr_period = 1
msb_lsb_first = 0
sign_extend = 0
tx_data_mode = 0
rx_data_mode = 0
;i2s_mclk = port:PB08<2><1><default><default>
i2s_bclk = port:PG11<2><1><default><default>
i2s_lrclk = port:PG10<2><1><default><default>
i2s_dout0 = port:PG12<2><1><default><default>
i2s_dout1 =
i2s_dout2 =
i2s_dout3 =
i2s_din = port:PG13<2><1><default><default>
```

| tdm0配置      | tdm0配置说明                                                 |
| :------------ | :----------------------------------------------------------- |
| daudio_master | 1: SND_SOC_DAIFMT_CBM_CFM(codec clk & FRM master),即daudio接口作为slave, codec作为master2: SND_SOC_DAIFMT_CBS_CFM(codec clk slave & FRMmaster),一般不用3: SND_SOC_DAIFMT_CBM_CFS(codec clk master & frameslave),一般不用4: SND_SOC_DAIFMT_CBS_CFS(codec clk & FRM slave),即daudio接口作为master, codec作为slave |
| daudio_select | 0: pcm mode; 1: i2s mode                                     |

deaudio_format |1: SND_SOC_DAIFMT_I2S(standard i2s format)2: SND_SOC_DAIFMT_RIGHT_J(right justfied format)3: SND_SOC_DAIFMT_LEFT_J(left justfied format)4: SND_SOC_DAIFMT_DSP_A(pcm. MSB is available on 2ndBCLK rising edge after LRC rising edge)5: SND_SOC_DAIFMT_DSP_B(pcm. MSB is available on 1nd BCLK rising edge after LRC rising edge)|
|signal_inversion |1: SND_SOC_DAIFMT_NB_NF(normal bit clock + frame)2: SND_SOC_DAIFMT_NB_IF(normal BCLK + inv FRM)3: SND_SOC_DAIFMT_IB_NF(invert BCLK + nor FRM)4: SND_SOC_DAIFMT_IB_IF(invert BCLK + FRM)|
| sample_resolution |采样精度,16bit, 24bit,32bit|
| slot_width_select| 支持8bit, 16bit, 32bit宽度|
| pcm_lrck_period |可配置16/32/64/128/256个bclk|
| pcm_lrckr_period |可配置16/32/64/128/256个bclk|
| msb_lsb_first |0: msb first; 1: lsb first|
| sign_extend |0: zero pending; 1: sign extend|
| tx_data_mode |0: 16bit linear PCM;1: reserved;2: 8bit u-law;3: 8bit a-law|
| rx_data_mode |0: 16bit linear PCM;1: reserved;2: 8bit u-law;3: 8bit a-law|
| i2s_bclk |i2s_bclk引脚|
| i2s_lrclk |i2s_lrclk引脚|
| i2s_dout0 |i2s_dout引脚|
| i2s_din| i2s_din引脚|

### 2.5.6 标案音频测试方法

该章节主要介绍在标案上进行播歌，录音的测试命令。


#### 2.5.6.1 播放

```
amixer -D hw:audiocodec cset name='SPK_L Mux' 1
amixer -D hw:audiocodec cset name='SPK_R Mux' 1
amixer -D hw:audiocodec cset name='Lineout volume' 24
amixer -D hw:audiocodec cset name='Right Output Mixer DACR Switch' 1
amixer -D hw:audiocodec cset name='Left Output Mixer DACL Switch' 1
amixer -D hw:audiocodec cset name='digital volume' 0
aplay -Dhw:audiocodec /mnt/UDISK/1KHz_0dB_16000.wav
```

可通过下面命令调节硬件上的模拟音量:

```
amixer -Dhw:audiocodec cset name='Lineout volume' 50
```

#### 2.5.6.2 录音

表示下使用AudioCodec进行单声道录音

```
amixer -D hw:audiocodec cset name='LADC input Mixer MIC1 boost Switch' 1
amixer -D hw:audiocodec cset name='MIC1 boost AMP gain control' 4

arecord -Dhw:audiocodec -f S16_LE -r 16000 -c 1 /tmp/test.wav
```

## 2.6 R16音频接口

### 2.6.1 硬件资源

R16包含 3 个音频模块，分别是内置AudioCodec,I2S0以及I2S1。


![图2-11: R16音频硬件框图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-11.jpg)

### 2.6.2 时钟源

R16中， 3 个音频模块的时钟源均来自pll_audio。

pll_audio可以输出24.576M或者22.5792M的时钟，分别支持48k系列，44.1k系列的播放录音。


![图2-12: R16时钟源](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-12.jpg)

### 2.6.3 代码结构

```
linux-3.4/sound/soc/sunxi/
├── audiocodec
│   ├── sun8iw5_machine.c // codec machine驱动
│   ├── sun8iw5_sndcodec.c // codec驱动
│   ├── sun8iw5_sndcodec.h
│   ├── sunxi_codecdma.c // codec platform驱动
│   ├── sunxi_codecdma.h
│   └── sunxi_codec.c // cpu dai驱动
├── i2s0
│ ├── sndi2s0.c // i2s codec驱动
│ ├── sunxi-i2s0dma.c // i2s platform驱动
│ ├── sunxi-i2s0dma.h
│ ├── sunxi-i2s0.c // i2s cpu dai驱动
│ ├── sunxi-i2s0.h
│ └── sunxi-sndi2s0.c // i2s machine驱动
└── i2s1
    ├── sndi2s1.c // i2s codec驱动
    ├── sunxi-i2s1dma.c // i2s platform驱动
    ├── sunxi-i2s1dma.h
    ├── sunxi-i2s1.c // i2s cpu dai驱动
    ├── sunxi-i2s1.h
    └── sunxi-sndi2s1.c // i2s machine驱动
```

### 2.6.4 AudioCodec

硬件特性

- 两路DAC

  - 支持16bit,24bit采样精度
  - 支持8KHz~192KHz采样率

- 两路ADC

  - 支持16bit,24bit采样精度
  - 支持8KHz~48KHz采样率

- 两路模拟输出:

  - 一路立体声headphone输出(HPOUTL,HPOUTR)
  - 一路立体声phoneout输出(PHONEOUTP,PHONEOUTN)

- 四路路模拟输入：MIC1,MIC2,linein,phonein
- 支持headphone驱动
- 支持同时playback和record(全双工模式)

#### 2.6.4.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
    <*> Advanced Linux Sound Architecture --->
        <*> ALSA for SoC audio support --->
            <*> Audiocodec for the SUNXI chips
            <*> Audiocodec Machine for sun8iw5 chips
            <*> Audiocodec for the SUN8IW5 chips
```

#### 2.6.4.2 sys_config配置.

```
[audio0]
audio_used = 1
headphone_vol = 0x3b
pa_double_used = 1
headphone_direct_used = 1
headset_mic_vol = 3
main_mic_vol = 1
;audio_linein_detect = port:PB07<0><default><default><0>
audio_pa_ctrl = port:PD11<1><default><default><0>
pa_gpio_reverse = 0
aif2_used = 0
aif3_used = 0
headphone_mute_used = 0
aif1_lrlk_div = 0x40
```


| audio0配置            | audio0配置说明                                               |
| :-------------------- | :----------------------------------------------------------- |
| audio0                | 是否使用audiocodec驱动。 0 ：不使用； 1 ：使用               |
| headphone_vol         | headphone volume，可设定范围0~0x3f, 0表示mute,1~63表示-62dB~0dB, 1dB/step |
| pa_double_used        | 是否同时使用两个DAC， 0 ：不使用； 1 ：使用                  |
| headphone_direct_used | 是否使用headphone输出， 0 ：不使用； 1 ：使用                |
| main_mic_vol          | MIC1默认增益, 0–7, 0:0dB, 1~7:24–42dB,3dB/step               |
| headset_mic_vol       | MIC2默认增益, 0–7, 0:0dB, 1~7:24–42dB,3dB/step               |
| audio_pa_ctrl         | PA使能引脚                                                   |
| pa_gpio_reverse       | PA使能引脚是否颠倒, 0:正常,即high level active; 1:颠倒,即low level active |
| aif1_lrlk_div         | aif1的lrck分频系数                                           |


#### 2.6.4.3 codec数据通路

![图2-13: R16音频通路](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-13.jpg)

```
通过HPOUTL/R播歌
AIF1DACL --> AIF1IN0L Mux --> DACL Mixer --> Left Output Mixer --> HP_L Mux --> HPOUTL
AIF1DACR --> AIF1IN0R Mux --> DACR Mixer --> Right Output Mixer --> HP_R Mux --> HPOUTR

通过MIC1录音
AIF1ADCL <-- AIF1OUT0L Mux <-- AIF1 AD0L Mixer <-- ADCL Mux <-- LEFT ADC input Mixer <--
MIC1 PGA <-- MIC1P/N
```

R16相关控件如下表：



| 控件名称                                             | 功能                                                   | 数值                                                         |
| ---------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| Headphone<br/>Switch                                 | Headphone通路使能                                      | 0:关闭; 1:开启                                               |
| ADC input gain                                       | ADC增益                                                | 0–7,表示-4.5–6dB                                             |
| ADC volume                                           | ADCL/ADCR音量设置                                      | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| ADCL Mux                                             | ADCL Mux设置,只支持0:ADC                               | 0:ADC                                                        |
| ADCR Mux                                             | ADCR Mux设置,只支持0:ADC                               | 0:ADC                                                        |
| AIF1 AD0L Mixer<br/>ADCL Switch                      | AIF1 AD0L Mixer设<br/>置，使能ADCL通路                 | 0:关闭; 1:开启                                               |
| AIF1 AD0L Mixer<br/>AIF1 DA0L Switch                 | AIF1 AD0L Mixer设置，使能AIF1 DA0L通路                 | 0:关闭; 1:开启                                               |
| AIF1 AD0L Mixer<br/>AIF2 DACL Switch                 | AIF1 AD0L Mixer设置，使能AIF2 DACL通路                 | 0:关闭; 1:开启                                               |
| AIF1 AD0L Mixer<br/>AIF2 DACR<br/>Switch             | AIF1 AD0L Mixer设置，使能AIF2 DACR通路                 | 0:关闭; 1:开启                                               |
| AIF1 AD0R Mixer<br/>ADCR Switch                      | AIF1 AD0R Mixer设置，使能ADCR通路                      | 0:关闭; 1:开启                                               |
| AIF1 AD0R Mixer<br/>AIF1 DA0R Switch                 | AIF1 AD0R Mixer设置，使能AIF1 DA0R通路                 | 0:关闭; 1:开启                                               |
| AIF1 AD0R Mixer<br/>AIF2 DACL Switch                 | AIF1 AD0R Mixer设置，使能AIF2 DACL通路                 | 0:关闭; 1:开启                                               |
| AIF1 AD0R Mixer<br/>AIF2 DACR<br/>Switch             | AIF1 AD0R Mixer设置，使能AIF2 DACR通路                 | 0:关闭; 1:开启                                               |
| AIF1 AD1L Mixer<br/>ADCL Switch                      | AIF1 AD1L Mixer设置，使能ADCL通路                      | 0:关闭; 1:开启                                               |
| AIF1 AD1L Mixer<br/>AIF2 DACL Switch                 | AIF1 AD1L Mixer设置，使能AIF2 DACL通路                 | 0:关闭; 1:开启                                               |
| AIF1 AD1R Mixer<br/>ADCR Switch                      | AIF1 AD1R Mixer设置，使能ADCR通路                      | 0:关闭; 1:开启                                               |
| AIF1 AD1R Mixer<br/>AIF2 DACR Switch                 | AIF1 AD1R Mixer设置，使能AIF2 DACR通路                 | 0:关闭; 1:开启                                               |
| AIF1 ADC timeslot<br/>0 mixer gain                   | AIF1 ADC0L/ADC0R Mixer,数字增益                        | 0:0dB; 1:-6dB;                                               |
|                                                      |                                                        | 对于ADC0L Mixer,<br/>bit0:AIF2 DACR;<br/>bit1:ADCL;<br/>bit2:AIF2 DACL;<br/>bit3:AIF2 DA0L;<br/>对于ADC0R Mixer,<br/>bit0:AIF2 DACL;<br/>bit1:ADCR;<br/>bit2:AIF2 DACR;<br/>bit3:AIF2 DA0R;<br/> |
| AIF1 ADC timeslot<br/>0 volume                       | AIF1 ADC0L/ADC0R音量设置                               | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| AIF1 ADC timeslot<br/>1 mixer gain                   | AIF1 ADC1L/ADC1R Mixer,数字增益                        | 0:0dB; 1:-6dB;                                               |
|                                                      |                                                        | 对于ADC1L Mixer,<br/>bit0:ADCL;<br/>bit1:AIF2 DACL;<br/>对于ADC1R Mixer,<br/>bit0:ADCR;<br/>bit1:AIF2 DACR; |
| AIF1 ADC timeslot<br/>1 volume                       | AIF1 ADC1L/ADC1R 音量设置                              | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| AIF1 DAC timeslot<br/>0 volume                       | AIF1 DAC0L/DAC0R 音量设置                              | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| AIF1 DAC timeslot<br/>1 volume                       | AIF1 DAC1L/DAC1R 音量设置                              | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| AIF1IN0L Mux                                         | AIF1IN0L Mux设置                                       | 0:AIF1_DA0L; 1:AIF1_DA0R;<br/>2:SUM_AIF1DA0L_AIF1DA0R;<br/>3:AVE_AIF1DA0L_AIF1DA0R |
| AIF1IN0R Mux                                         | AIF1IN0R Mux设置                                       | 0:AIF1_DA0R; 1:AIF1_DA0L;<br/>2:SUM_AIF1DA0L_AIF1DA0R;<br/>3:AVE_AIF1DA0L_AIF1DA0R |
| AIF1IN1L Mux                                         | AIF1IN1L Mux设置                                       | 0:AIF1_DA1L; 1:AIF1_DA1R;<br/>2:SUM_AIF1DA1L_AIF1DA1R;<br/>3:AVE_AIF1DA1L_AIF1DA1R |
| AIF1IN1R Mux                                         | AIF1IN1R Mux设置                                       | 0:AIF1_DA1R; 1:AIF1_DA1L;<br/>2:SUM_AIF1DA1L_AIF1DA1R;<br/>3:AVE_AIF1DA1L_AIF1DA1R |
| AIF1OUT0L Mux                                        | AIF1OUT0L Mux设置                                      | 0:AIF1_AD0L; 1:AIF1_AD0R;<br/>2:SUM_AIF1AD0L_AIF1AD0R;<br/>3:AVE_AIF1AD0L_AIF1AD0R |
| AIF1OUT0R Mux                                        | AIF1OUT0R Mux设置                                      | 0:AIF1_AD0R; 1:AIF1_AD0L;<br/>2:SUM_AIF1AD0L_AIF1AD0R;<br/>3:AVE_AIF1AD0L_AIF1AD0R |
| AIF1OUT1L Mux                                        | AIF1OUT1L Mux设置                                      | 0:AIF1_AD1L; 1:AIF1_AD1R;<br/>2:SUM_AIF1AD1L_AIF1AD1R;<br/>3:AVE_AIF1AD1L_AIF1AD1R |
| AIF1OUT1R Mux                                        | AIF1OUT1R Mux设置                                      | 0:AIF1_AD1R; 1:AIF1_AD1L;<br/>2:SUM_AIF1AD1L_AIF1AD1R;<br/>3:AVE_AIF1AD1L_AIF1AD1R |
| DAC mixer gain                                       | DAC mixer增益                                          | 0:0dB; 1:-6dB;                                               |
|                                                      |                                                        | 对于DACL Mixer,<br/>bit0:ADCL;<br/>bit1:AIF2 DACL;<br/>bit2:AIF1 DAC1L;<br/>bit3:AIF1 DAC0L;<br/>对于DACR Mixer,<br/>bit0:ADCR;<br/>bit1:AIF2 DACR;<br/>bit2:AIF1 DAC1R;<br/>bit3:AIF1 DAC0R; |
| DAC volume                                           | DACL/DACR音量设置                                      | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| DACL Mixer<br/>ADCL Switch                           | DACL Mixer设置，使能<br/>ADCL通路                      | 0:关闭; 1:开启                                               |
| DACL Mixer<br/>AIF1DA0L Switch                       | DACL Mixer设置，使能<br/>AIF1DA0L通路                  | 0:关闭; 1:开启                                               |
| DACL Mixer<br/>AIF1DA1L Switch                       | DACL Mixer设置，使能<br/>AIF1DA1L通路                  | 0:关闭; 1:开启                                               |
| DACL Mixer<br/>AIF2DACL Switch                       | DACL Mixer设置，使能<br/>AIF2DACL通路                  | 0:关闭; 1:开启                                               |
| DACR Mixer<br/>ADCR Switch                           | DACR Mixer设置，使<br/>能ADCR通路                      | 0:关闭; 1:开启                                               |
| DACR Mixer<br/>AIF1DA0R Switch                       | DACR Mixer设置，使<br/>能AIF1DA0R通路                  | 0:关闭; 1:开启                                               |
| DACR Mixer<br/>AIF1DA1R Switch                       | DACR Mixer设置，使<br/>能AIF1DA1R通路                  | 0:关闭; 1:开启                                               |
| DACR Mixer<br/>AIF2DACR Switch                       | DACR Mixer设置，使<br/>能AIF2DACR通路                  | 0:关闭; 1:开启                                               |
| External Speaker<br/>Switch                          | 使能Headphone以及<br/>PA                               | 0:关闭; 1:开启                                               |
| HP_L Mux                                             | HP_L Mux设置                                           | 0:DACL ; 1:Left Output Mixer                                 |
| HP_R Mux                                             | HP_R Mux设置                                           | 0:DACR ; 1:Right Output Mixer                                |
| LEFT ADC input<br/>Mixer<br/>Lout_Mixer_Switch       | LEFT ADC input Mixer<br/>设置，使能Lout Mixer通路      | 0:关闭; 1:开启                                               |
| LEFT ADC input<br/>Mixer MIC1 boost<br/>Switch       | LEFT ADC input Mixer<br/>设置，使能MIC1通路            | 0:关闭; 1:开启                                               |
| LEFT ADC input<br/>Mixer MIC2 boost<br/>Switch       | LEFT ADC input Mixer<br/>设置，使能MIC2通路            | 0:关闭; 1:开启                                               |
| LEFT ADC input<br/>Mixer<br/>Rout_Mixer_Switch       | LEFT ADC input Mixer<br/>设置，使能Rout Mixer<br/>通路 | 0:关闭; 1:开启                                               |
| Left Output Mixer<br/>DACL Switch                    | Left Output Mixer设<br/>置，使能DACL通路               | 0:关闭; 1:开启                                               |
| Left Output Mixer<br/>DACR Switch                    | Left Output Mixer设<br/>置，使能DACR通路               | 0:关闭; 1:开启                                               |
| Left Output Mixer<br/>MIC1Booststage<br/>Switch      | Left Output Mixer设<br/>置，使能MIC1通路               | 0:关闭; 1:开启                                               |
| Left Output Mixer<br/>MIC2Booststage<br/>Switch      | Left Output Mixer设<br/>置，使能MIC2通路               | 0:关闭; 1:开启                                               |
| MIC1 boost<br/>amplifier gain                        | MIC1增益                                               | 0–7, 0:0dB, 1~7:24–42dB,3dB/step                             |
| MIC2 SRC                                             | MIC2 SRC设置                                           | 0:MIC3; 1:MIC2                                               |
| MIC2 boost<br/>amplifier gain                        | MIC2增益                                               | 0–7, 0:0dB, 1~7:24–42dB,3dB/step                             |
| RIGHT ADC input<br/>Mixer<br/>Lout_Mixer_Switch      | RIGHT ADC input<br/>Mixer设置，使能Lout<br/>Mixer通路  | 0:关闭; 1:开启                                               |
| RIGHT ADC input<br/>Mixer MIC1 boost<br/>Switch      | RIGHT ADC input<br/>Mixer设置，使能MIC1<br/>通路       | 0:关闭; 1:开启                                               |
| RIGHT ADC input<br/>Mixer MIC2 boost<br/>Switch      | RIGHT ADC input<br/>Mixer设置，使能MIC2<br/>通路       | 0:关闭; 1:开启                                               |
| RIGHT ADC input<br/>Mixer<br/>Rout_Mixer_Switch      | RIGHT ADC input<br/>Mixer设置，使能Rout<br/>Mixer通路  | 0:关闭; 1:开启                                               |
| Right Output<br/>Mixer DACL<br/>Switch               | Right Output Mixer设<br/>置，使能DACL通路              | 0:关闭; 1:开启                                               |
| Right Output<br/>Mixer DACR<br/>Switch               | Right Output Mixer设<br/>置，使能DACR通路              | 0:关闭; 1:开启                                               |
| Right Output<br/>Mixer<br/>MIC1Booststage<br/>Switch | Right Output Mixer设<br/>置，使能MIC1通路              | 0:关闭; 1:开启                                               |
| Right Output<br/>Mixer<br/>MIC2Booststage<br/>Switch | Right Output Mixer设<br/>置，使能MIC2通路              | 0:关闭; 1:开启                                               |
| digital volume                                       | 数字音量设置                                           | 0–63,表示-73.08–0dB                                          |
| headphone<br/>volume                                 | headphone音量设置                                      | 0–63,0表示mute; 1~63表<br/>示-62dB–0dB                       |

### 2.6.5 Daudio.

硬件特性


• 两路I2S/PCM；

• 支持主从模式

- 支持Left-justified,Right-justified,Standar mode I2S,PCM mode
- 支持i2s,pcm协议格式配置
- 支持mono和stereo模式，支持 8 通道输出和 2 通道输入
- 支持同时playback和record(全双工模式)
- 支持8~192KHz采样率
- 支持16,24,32bit采样精度

#### 2.6.5.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
<*> SoC i2s0 interface for SUNXI chips
<*> SoC i2s1 interface for SUNXI chips
```

#### 2.6.5.2 sys_config配置.

I2S0,I2S1的配置方法是一样的，下面仅描述I2S0

```
[i2s0]
i1s0_used = 1
i2s0_channel = 2
i2s0_master = 4
i2s0_select = 1
audio_format = 1
signal_inversion = 1
over_sample_rate = 512
sample_resolution = 16
word_select_size = 32
pcm_sync_period = 256
msb_lsb_first = 0
slot_index = 0
slot_width = 16
frame_width = 1
tx_data_mode = 1
rx_data_mode = 1
i2s0_mclk =
i2s0_bclk = port:PB05<2><1><default><default>
i2s0_lrclk = port:PB04<2><1><default><default>
i2s0_dout0 = port:PB06<2><1><default><default>
i2s0_dout1 =
i2s0_dout2 =
i2s0_dout3 =
i2s0_din = port:PB07<2><1><default><default>
```

| i2s0配置          | i2s0配置说明                                                 |
| ----------------- | ------------------------------------------------------------ |
| i2s0_used         | 是否使用i2s驱动。 0 ：不使用； 1 ：使用                      |
| i2s0_master       | 1: SND_SOC_DAIFMT_CBM_CFM(codec clk & FRMmaster),<br/>即daudio接口作为slave, codec作为master<br/>2: SND_SOC_DAIFMT_CBS_CFM(codec clk slave &<br/>FRM master),一般不用<br/>3: SND_SOC_DAIFMT_CBM_CFS(codec clk master &<br/>frame slave),一般不用<br/>4: SND_SOC_DAIFMT_CBS_CFS(codec clk & FRM<br/>slave),即daudio接口作为master, codec作为slave |
| audio_format      | 1: SND_SOC_DAIFMT_I2S(standard i2s format)<br/>2: SND_SOC_DAIFMT_RIGHT_J(right justfied format)<br/>3: SND_SOC_DAIFMT_LEFT_J(left justfied format)<br/>4: SND_SOC_DAIFMT_DSP_A(pcm. MSB is available<br/>on 2nd BCLK rising edge after LRC rising edge)<br/>5: SND_SOC_DAIFMT_DSP_B(pcm. MSB is available<br/>on 1nd BCLK rising edge after LRC rising edge) |
| signal_inversion  | 1: SND_SOC_DAIFMT_NB_NF(normal bit clock +<br/>frame)<br/>2: SND_SOC_DAIFMT_NB_IF(normal BCLK + inv<br/>FRM)<br/>3: SND_SOC_DAIFMT_IB_NF(invert BCLK + nor FRM)<br/>4: SND_SOC_DAIFMT_IB_IF(invert BCLK + FRM)<br/> |
| over_sample_rate  | 支持128fs/192fs/256fs/384fs/512fs/768fs                      |
| sample_resolution | 采样精度,16bit, 24bit,32bit                                  |
| word_select_size  | 支持16bits/20bits/24bits/32bits                              |
| pcm_sync_period   | 16/32/64/128/256                                             |
| msb_lsb_first     | 0: msb first; 1: lsb first                                   |
| slot_index        | 0: 1st slot; 1: 2nd slot; 2: 3th slot; 3:4th slot            |
| slot_width        | 8: 8 clocks width; 16: 16 clocks width                       |
| frame_width       | 0: long frame sync; 1: short frame sync                      |
| tx_data_mode      | 0: 16bit linear PCM;1: reserved;2: 8bit u-law;3: 8bit a-law  |
| rx_data_mode      | 0: 16bit linear PCM;1: reserved;2: 8bit u-law;3: 8bit a-law  |
| i2s0_mclk         | i2s0_mclk引脚                                                |
| i2s0_bclk         | i2s0_bclk引脚                                                |
| i2s0_lrclk        | i2s0_lrclk引脚                                               |
| i2s0_dout0        | i2s0_dout引脚                                                |
| i2s0_din          | i2s0_din引脚                                                 |


### 2.6.6 标案音频测试方法

该章节主要介绍在标案上进行播歌，录音的测试命令。

#### 2.6.6.1 播放

```
通过speaker播放
amixer cset name='AIF1IN0L Mux' 'AIF1_DA0L';
amixer cset name='AIF1IN0R Mux' 'AIF1_DA0R';
amixer cset name='DACL Mixer AIF1DA0L Switch' 1;
amixer cset name='DACR Mixer AIF1DA0R Switch' 1;
amixer cset name='HP_L Mux' 'DACL HPL Switch' ;
amixer cset name='HP_R Mux' 'DACR HPR Switch';
amixer cset name='External Speaker Switch' 1;
aplay -Dhw:audiocodec /mnt/UDISK/1KHz_0dB_16000.wav
```

```
mixer cset name='AIF1IN0L Mux' 'AIF1_DA0L';
amixer cset name='AIF1IN0R Mux' 'AIF1_DA0R';
amixer cset name='DACL Mixer AIF1DA0L Switch' 1;
amixer cset name='DACR Mixer AIF1DA0R Switch' 1;
amixer cset name='HP_L Mux' 'DACL HPL Switch' ;
amixer cset name='HP_R Mux' 'DACR HPR Switch';
amixer cset name='Headphone Switch' 1;
aplay -Dhw:sndcodec /mnt/UDISK/1KHz_0dB_16000.wav
```

可通过下面命令调节硬件上的模拟音量:

```
amixer -Dhw:sndcodec cset name='headphone volume' 58
```

#### 2.6.6.2 录音

表示下使用audiocodec进行单声道录音

```
amixer cset name='LEFT ADC input Mixer MIC1 boost Switch' 1
amixer cset name='AIF1 AD0L Mixer ADCL Switch' 1
amixer cset name='AIF1OUT0L Mux' 'AIF1_AD0L'
amixer cset name='MIC1 boost amplifier gain' 4
arecord -Dhw:sndcodec -f S16_LE -r 16000 -c 1 /tmp/test.wav
```

## 2.7 R18音频接口

### 2.7.1 硬件资源

R18包含 4 个音频模块，分别是内置AudioCodec以及Daudio0,Daudio1,Daudio2。

![图2-14: R18音频硬件框图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-14.jpg)

### 2.7.2 时钟源

R18中， 4 个音频模块的时钟源均来自pll_audio

pll_audio可以输出24.576M或者22.5792M的时钟，分别支持48k系列，44.1k系列的播放录音。


![图2-15: R18时钟源](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-15.jpg)

### 2.7.3 代码结构

```
linux-4.4/sound/soc/sunxi/
├── sun50iw1-codec.c // codec驱动
├── sun50iw1-codec.h
├── sun50iw1-sndcodec.c // codec machine驱动
├── sunxi-inter-i2s.c // codec platform驱动
├── sunxi-daudio.c // daudio platform驱动
├── sunxi-daudio.h
├── sunxi-snddaudio.c // daudio machine驱动
├── sunxi-snddaudio.h
├── sunxi-pcm.c //通用文件，提供注册platform驱动的接口及相关函数集
├── sunxi-pcm.h
├── sunxi_rw_func.c //通用文件，读写模拟/数字寄存器的接口
├── sunxi_rw_func.h
├── spdif-utils.c // spdif codec驱动
├── sunxi-sndspdif.c // spdif machine驱动
├── sunxi-spdif.c // spdif platform驱动
├── sunxi-spdif.h
├── sunxi-hdmi.c // hdmi codec驱动
└── sunxi-sndhdmi.c // hdmi machine驱动
```

```
hdmi platform模型使用的是sunxi-daudio.c
linux-4.4/sound/soc/soc-utils.c // snd-soc-dummmy驱动,可用于daudio codec模型
linux-4.4/sound/soc/codecs/ac108.c // ac108 codec驱动
linux-4.4/sound/soc/codecs/tas5731.c // tas5731数字功放codec驱动
```

### 2.7.4 AudioCodec

硬件特性


• 两路DAC

- 支持16bit,24bit采样精度
- 支持8KHz~192KHz采样率
- 两路ADC
- 支持16bit,24bit采样精度
- 支持8KHz~48KHz采样率
- 四路模拟输出:
- 一路立体声earpiece输出(EAROUTP,EAROUTN)
- 一路立体声phoneout输出(PHONEOUTP,PHONEOUTN)
- 一路立体声headphone输出(HPOUTL,HPOUTR)
- 一路立体声lineout输出(LINEOUTL,LINEOUTR)
- 四路路模拟输入：MIC1,MIC2,linein,phonein
- 支持headphone驱动
- 支持earpiece驱动
- 支持同时playback和record(全双工模式)
- 支持适用于DAC的DRC功能
- 支持适用于ADC的AGC,DRC功能

#### 2.7.4.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner Sun50iw1 Codec Support
```

#### 2.7.4.2 sys_config配置.

```
[sndcodec]
sndcodec_used = 0x1
aif2fmt = 0x3
aif3fmt = 0x3
aif2master = 0x1
linein_detect = port:PH05<6><default><default><default>
hp_detect_case = 0x1
;------------------------------------------------------------------------------
[i2s]
i2s_used = 0x1
;-------------------------------------------------------------------------------
[codec]
codec_used = 0x1
headphonevol = 0x38
```

```
spkervol = 0x1d
earpiecevol = 0x1e
maingain = 0x4
headsetmicgain = 0x4
adcagc_cfg = 0x0
adcdrc_cfg = 0x0
adchpf_cfg = 0x1
dacdrc_cfg = 0x0
dachpf_cfg = 0x0
aif2config = 0x0
aif3config = 0x0
aif1_lrlk_div = 0x40
aif2_lrlk_div = 0x40
pa_sleep_time = 0x0a
dac_digital_vol = 0x9898
gpio-spk =
```

sndcodec配置，即machine驱动的相关配置

| sndcodec配置                         | sndcodec配置说明              |
| ------------------------------------ | ----------------------------- |
| sndcodec_used 是否使用sndcodec驱动。 | 0 ：不使用； 1 ：使用         |
| linein_detect                        | linein检测引脚                |
| hp_detect_case                       | jack irq level, 0:low; 1:high |

codec配置，即内置audiocodec驱动的相关配置

| codec配置       | codec配置说明                                                |
| --------------- | ------------------------------------------------------------ |
| codec_used      | 是否使用codec驱动。0 ：不使用； 1 ：使用                     |
| headphonevol    | headphone volume，可设定范围0~0x3f, 0表示mute, 1~63表<br/>示-62dB~0dB, 1dB/step |
| spkervol        | spk(lineout) volume,可设定范围0~0x1f, 0或者 1 表示mute, 2~31<br/>表示-43.5dB~0dB, 1.5dB/step |
| earpiecevol     | earpiece volume,可设定范围0~0x1f, 0或者 1 表示mute, 2~31表<br/>示-43.5dB~0dB, 1.5dB/step |
| maingain        | MIC1默认增益, 0–7, 0:0dB, 1~7:24–42dB,3dB/step               |
| headsetmicgain  | MIC2默认增益, 0–7, 0:0dB, 1~7:24–42dB,3dB/step               |
| adcagc_cfg      | 是否使用adcagc. 0:不适用； 1 ：使用                          |
| adcdrc_cfg      | 是否使用adcdrc. 0:不适用； 1 ：使用                          |
| adchpf_cfg      | 是否使用adchpf. 0:不适用； 1 ：使用                          |
| dacdrc_cfg      | 是否使用dacdrc. 0:不适用； 1 ：使用                          |
| dachpf_cfg      | 是否使用dachpf. 0:不适用； 1 ：使用                          |
| aif1_lrlk_div   | aif1的lrck分频系数                                           |
| pa_sleep_time   | 使能pa之前等待的时间，单位ms                                 |
| dac_digital_vol | DACL/DACR数字音量,0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB, 0.75dB/step,如0xA0表示0dB, 0x98表示-6dB |

| codec配置 | codec配置说明 |
| --------- | ------------- |
| gpio-spk  | PA使能引脚    |

#### 2.7.4.3 codec数据通路

![图2-16: R18音频通路](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-16.jpg)

```
通过HPOUTL/R播歌
AIF1DACL --> AIF1IN0L Mux --> DACL Mixer --> HP_L Mux --> HPOUTL
AIF1DACR --> AIF1IN0R Mux --> DACR Mixer --> HP_R Mux --> HPOUTR
通过MIC1,2录音
AIF1ADCL <-- AIF1OUT0L Mux <-- AIF1 AD0L Mixer <-- ADCL Mux <-- LADC input Mixer <-- MIC1
PGA <-- MIC1P/N
AIF1ADCR <-- AIF1OUT0R Mux <-- AIF1 AD0R Mixer <-- ADCR Mux <-- RADC input Mixer <-- MIC2
PGA <-- MIC2P/N
```

R18相关控件如下表：

| 控件名称                                                     | 功能                                                  | 数值                                                         |
| ------------------------------------------------------------ | ----------------------------------------------------- | ------------------------------------------------------------ |
| Headphone Switch                                             | Headphone通路使能                                     | 0:关闭; 1:开启                                               |
| Linein_detect<br/>Switch                                     | Linein检测使能                                        | 0:关闭; 1:开启                                               |
| ADC input gain<br/>control                                   | ADC增益                                               | 0–7,表示-4.5–6dB                                             |
| ADC volume                                                   | ADCL/ADCR音量设置                                     | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| ADCL Mux                                                     | ADCL Mux设置,只支持<br/>0:ADC                         | 0:ADC                                                        |
| ADCR Mux                                                     | ADCR Mux设置,只支持<br/>0:ADC                         | 0:ADC                                                        |
| AIF1 AD0L Mixer<br/>ADCL Switch                              | AIF1 AD0L Mixer设<br/>置，使能ADCL通路                | 0:关闭; 1:开启                                               |
| AIF1 AD0L Mixer<br/>AIF1 DA0L Switch                         | AIF1 AD0L Mixer设<br/>置，使能AIF1 DA0L通路           | 0:关闭; 1:开启                                               |
| AIF1 AD0L Mixer<br/>AIF2 DACL Switch                         | AIF1 AD0L Mixer设<br/>置，使能AIF2 DACL通路           | 0:关闭; 1:开启                                               |
| AIF1 AD0L Mixer<br/>AIF2 DACR Switch                         | AIF1 AD0L Mixer设<br/>置，使能AIF2 DACR通路           | 0:关闭; 1:开启                                               |
| AIF1 AD0R Mixer<br/>ADCR Switch                              | AIF1 AD0R Mixer设<br/>置，使能ADCR通路                | 0:关闭; 1:开启                                               |
| AIF1 AD0R Mixer<br/>AIF1 DA0R Switch                         | AIF1 AD0R Mixer设<br/>置，使能AIF1 DA0R通路           | 0:关闭; 1:开启                                               |
| AIF1 AD0R Mixer<br/>AIF2 DACL Switch                         | AIF1 AD0R Mixer设<br/>置，使能AIF2 DACL通路           | 0:关闭; 1:开启                                               |
| AIF1 AD0R Mixer<br/>AIF2 DACR Switch                         | AIF1 AD0R Mixer设<br/>置，使能AIF2 DACR通路           | 0:关闭; 1:开启                                               |
| AIF1 AD1L Mixer<br/>ADCL Switch                              | AIF1 AD1L Mixer设<br/>置，使能ADCL通路                | 0:关闭; 1:开启                                               |
| AIF1 AD1L Mixer<br/>AIF2 DACL Switch                         | AIF1 AD1L Mixer设<br/>置，使能AIF2 DACL通路           | 0:关闭; 1:开启                                               |
| AIF1 AD1R Mixer<br/>ADCR Switch                              | AIF1 AD1R Mixer设<br/>置，使能ADCR通路                | 0:关闭; 1:开启                                               |
| AIF1 AD1R Mixer<br/>AIF2 DACR Switch                         | AIF1 AD1R Mixer设<br/>置，使能AIF2 DACR通路           | 0:关闭; 1:开启                                               |
| AIF1 ADC timeslot<br/>0 mixer gain                           | AIF1 ADC0L/ADC0R <br/>Mixer,数字增益                  | 0:0dB; 1:-6dB;                                               |
|                                                              |                                                       | 对于ADC0L Mixer,<br/>bit0:AIF2 DACR;<br/>bit1:ADCL;<br/>bit2:AIF2 DACL;<br/>bit3:AIF2 DA0L;<br/>对于ADC0R Mixer,<br/>bit0:AIF2 DACL;<br/>bit1:ADCR;<br/>bit2:AIF2 DACR;<br/>bit3:AIF2 DA0R; |
| AIF1 ADC timeslot<br/>0 volume                               | AIF1 ADC0L/ADC0R<br />音量设置                        | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| AIF1 ADC timeslot<br/>1 mixer gain                           | AIF1 ADC1L/ADC1R<br />Mixer,数字增益                  | 0:0dB; 1:-6dB;                                               |
|                                                              |                                                       | 对于ADC1L Mixer,<br/>bit0:ADCL;<br/>bit1:AIF2 DACL;<br/>对于ADC1R Mixer,<br/>bit0:ADCR;<br/>bit1:AIF2 DACR; |
| AIF1 ADC timeslot<br/>1 volume                               | AIF1 ADC1L/ADC1R<br/>音量设置                         | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| AIF1 DAC timeslot<br/>0 volume                               | AIF1 DAC0L/DAC0R<br/>音量设置                         | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| AIF1 DAC timeslot<br/>1 volume                               | AIF1 DAC1L/DAC1R<br/>音量设置                         | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| AIF1IN0L Mux                                                 | AIF1IN0L Mux设置                                      | 0:AIF1_DA0L; 1:AIF1_DA0R;<br/>2:SUM_AIF1DA0L_AIF1DA0R;<br/>3:AVE_AIF1DA0L_AIF1DA0R |
| AIF1IN0R Mux                                                 | AIF1IN0R Mux设置                                      | 0:AIF1_DA0R; 1:AIF1_DA0L;<br/>2:SUM_AIF1DA0L_AIF1DA0R;<br/>3:AVE_AIF1DA0L_AIF1DA0R |
| AIF1IN1L Mux                                                 | AIF1IN1L Mux设置                                      | 0:AIF1_DA1L; 1:AIF1_DA1R;<br/>2:SUM_AIF1DA1L_AIF1DA1R;<br/>3:AVE_AIF1DA1L_AIF1DA1R |
| AIF1IN1R Mux                                                 | AIF1IN1R Mux设置                                      | 0:AIF1_DA1R; 1:AIF1_DA1L;<br/>2:SUM_AIF1DA1L_AIF1DA1R;<br/>3:AVE_AIF1DA1L_AIF1DA1R |
| AIF1OUT0L Mux                                                | AIF1OUT0L Mux设置                                     | 0:AIF1_AD0L; 1:AIF1_AD0R;<br/>2:SUM_AIF1AD0L_AIF1AD0R;<br/>3:AVE_AIF1AD0L_AIF1AD0R |
| AIF1OUT0R Mux                                                | AIF1OUT0R Mux设置                                     | 0:AIF1_AD0R; 1:AIF1_AD0L;<br/>2:SUM_AIF1AD0L_AIF1AD0R;<br/>3:AVE_AIF1AD0L_AIF1AD0R |
| AIF1OUT1L Mux                                                | AIF1OUT1L Mux设置                                     | 0:AIF1_AD1L; 1:AIF1_AD1R;<br/>2:SUM_AIF1AD1L_AIF1AD1R;<br/>3:AVE_AIF1AD1L_AIF1AD1R |
| AIF1OUT1R Mux                                                | AIF1OUT1R Mux设置                                     | 0:AIF1_AD1R; 1:AIF1_AD1L;<br/>2:SUM_AIF1AD1L_AIF1AD1R;<br/>3:AVE_AIF1AD1L_AIF1AD1R |
| DAC mixer gain                                               | DAC mixer增益                                         | 0:0dB; 1:-6dB;                                               |
|                                                              |                                                       | 对于DACL Mixer,<br/>bit0:ADCL;<br/>bit1:AIF2 DACL;<br/>bit2:AIF1 DAC1L;<br/>bit3:AIF1 DAC0L;<br/>对于DACR Mixer,<br/>bit0:ADCR;<br/>bit1:AIF2 DACR;<br/>bit2:AIF1 DAC1R;<br/>bit3:AIF1 DAC0R; |
| DAC volume                                                   | DACL/DACR音量设置                                     | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| DACL Mixer ADCL<br/>Switch                                   | DACL Mixer设置，使能<br/>ADCL通路                     | 0:关闭; 1:开启                                               |
| DACL Mixer<br/>AIF1DA0L Switch                               | DACL Mixer设置，使能<br/>AIF1DA0L通路                 | 0:关闭; 1:开启                                               |
| DACL Mixer<br/>AIF1DA1L Switch                               | DACL Mixer设置，使能<br/>AIF1DA1L通路                 | 0:关闭; 1:开启                                               |
| DACL Mixer<br/>AIF2DACL Switch                               | DACL Mixer设置，使能<br/>AIF2DACL通路                 | 0:关闭; 1:开启                                               |
| DACR Mixer ADCR<br/>Switch                                   | DACR Mixer设置，使<br/>能ADCR通路                     | 0:关闭; 1:开启                                               |
| DACR Mixer<br/>AIF1DA0R Switch                               | DACR Mixer设置，使<br/>能AIF1DA0R通路                 | 0:关闭; 1:开启                                               |
| DACR Mixer<br/>AIF1DA1R Switch                               | DACR Mixer设置，使<br/>能AIF1DA1R通路                 | 0:关闭; 1:开启                                               |
| DACR Mixer<br/>AIF2DACR Switch                               | DACR Mixer设置，使<br/>能AIF2DACR通路                 | 0:关闭; 1:开启                                               |
| EAR Mux                                                      | EAR Mux设置                                           | 0:DACR; 1:DACL; 2:Right Analog<br/>Mixer; 3:Left Analog Mixer |
| Earpiece Switch                                              | Earpiece通路使能                                      | 0:关闭; 1:开启                                               |
| External Speaker<br/>Switch                                  | 使能Headphone以及<br/>PA                              | 0:关闭; 1:开启                                               |
| HP_L Mux                                                     | HP_L Mux设置                                          | 0:DACL ; 1:Left Output Mixer                                 |
| HP_R Mux                                                     | HP_R Mux设置                                          | 0:DACR ; 1:Right Output Mixer                                |
| LADC input Mixer<br/>LINEINL                                 | LADC input Mixer设<br/>置，使能LINEINL通路            | 0:关闭; 1:开启                                               |
| LADC input Mixer<br/>MIC1 boost Switch                       | LADC input Mixer设<br/>置，使能MIC1通路               | 0:关闭; 1:开启                                               |
| LADC input Mixer<br/>MIC2 boost Switch                       | LADC input Mixer设<br/>置，使能MIC2通路               | 0:关闭; 1:开启                                               |
| LADC input Mixer<br/>l_output mixer<br/>Switch               | LADC input Mixer设<br/>置，使能l_output<br/>mixer通路 | 0:关闭; 1:开启                                               |
| LADC input Mixer<br/>r_output mixer<br/>Switch               | LADC input Mixer设<br/>置，使能r_output<br/>mixer通路 | 0:关闭; 1:开启                                               |
| LINEINL/R to L_R<br/>output mixer gain<br/>Left Output Mixer<br/>DACL Switch | Left Output Mixer设<br/>置，使能DACL通路              | 0:关闭; 1:开启                                               |
| Left Output Mixer<br/>DACR Switch                            | Left Output Mixer设<br/>置，使能DACR通路              | 0:关闭; 1:开启                                               |
| Left Output Mixer<br/>LINEINL Switch                         | Left Output Mixer设<br/>置，使能LINEINL通路           | 0:关闭; 1:开启                                               |
| Left Output Mixer<br/>MIC1Booststage<br/>Switch              | Left Output Mixer设<br/>置，使能MIC1通路              | 0:关闭; 1:开启                                               |
| Left Output Mixer<br/>MIC2Booststage<br/>Switch              | Left Output Mixer设<br/>置，使能MIC2通路              | 0:关闭; 1:开启                                               |
| MIC1 boost<br/>amplifier gain                                | MIC1增益                                              | 0–7, 0:0dB, 1~7:24–42dB,3dB/step                             |
| MIC1_G boost<br/>stage output mixer<br/>control              | MIC1 to L or R output<br/>Mixer增益                   | 0–7,表示-4.5–6dB                                             |
| MIC2 BST stage to<br/>L_R outp mixer<br/>gain                | MIC2 to L or R output<br/>Mixer增益                   | 0–7,表示-4.5–6dB                                             |
| MIC2 SRC                                                     | MIC2 SRC设置                                          | 0:MIC3; 1:MIC2                                               |
| MIC2 boost AMP<br/>gain control                              | MIC2增益                                              | 0–7, 0:0dB, 1~7:24–42dB,3dB/step                             |
| RADC input Mixer<br/>LINEINR Switch                          | RADC input Mixer设<br/>置，使能LINEINR通路            | 0:关闭; 1:开启                                               |
| RADC input Mixer<br/>MIC1 boost Switch                       | RADC input Mixer设<br/>置，使能MIC1通路               | 0:关闭; 1:开启                                               |
| RADC input Mixer<br/>MIC2 boost Switch                       | RADC input Mixer设<br/>置，使能MIC2通路               | 0:关闭; 1:开启                                               |
| RADC input Mixer<br/>l_output mixer<br/>Switch               | RADC input Mixer设<br/>置，使能l_output<br/>mixer通路 | 0:关闭; 1:开启                                               |
| RADC input Mixer<br/>l_output Switch                         | RADC input Mixer设<br/>置，使能l_output<br/>mixer通路 | 0:关闭; 1:开启                                               |
| Right Output Mixer<br/>DACL Switch                           | Right Output Mixer设<br/>置，使能DACL通路             | 0:关闭; 1:开启                                               |
| Right Output Mixer<br/>DACR Switch                           | Right Output Mixer设<br/>置，使能DACR通路             | 0:关闭; 1:开启                                               |
| Right Output Mixer<br/>LINEINR Switch                        | Right Output Mixer设<br/>置，使能LINEINR通路          | 0:关闭; 1:开启                                               |
| Right Output Mixer<br/>MIC1Booststage<br/>Switch             | Right Output Mixer设<br/>置，使能MIC1通路             | 0:关闭; 1:开启                                               |
| Right Output Mixer<br/>MIC2Booststage<br/>Switch             | Right Output Mixer设<br/>置，使能MIC2通路             | 0:关闭; 1:开启                                               |
| SPK_L Mux                                                    | SPK_L Mux设置                                         | 0:MIXEL Switch; 1:MIXL MIXR<br/>Switch                       |
| SPK_R Mux                                                    | SPK_R Mux设置                                         | 0:MIXER Switch; 1:MIXR MIXL<br/>Switch                       |
| digital volume                                               | 数字音量设置                                          | 0–63,表示-73.08–0dB                                          |
| earpiece volume                                              | earpiece音量设置                                      | 0–31,表示-43.5–0dB                                           |
| headphone volume                                             | headphone音量设置                                     | 0–63,0表示mute; 1~63表<br/>示-62dB–0dB                       |
| speaker volume                                               | speaker(lineout)音量设置                              | 0–31,表示-43.5–0dB                                           |

### 2.75 Daudio.

硬件特性

• 三路I2S/PCM；

• 支持主从模式

- 支持Left-justified,Right-justified,Standar mode I2S,PCM mode
- 支持i2s,pcm协议格式配置
- 支持同时playback和record(全双工模式)
- 支持8~192KHz采样率
- 支持16,24,32bit采样精度

#### 2.7.5.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner Digital Audio Support
```

#### 2.7.5.2 sys_config配置.

```
[snddaudio0]
snddaudio0_used = 1
;-----------------------------------------------------------------------------
[daudio0]
daudio0_used = 1
pcm_lrck_period = 0x60
pcm_lrckr_period = 0x01
slot_width_select = 0x18
pcm_lsb_first = 0x0
tx_data_mode = 0x0
rx_data_mode = 0x0
daudio_master = 0x04
audio_format = 0x01
signal_inversion = 0x01
```

```
frametype = 0x0
tdm_config = 0x01
clk_active = 0x0
```

snddaudio0配置，即daudio0 machine驱动的相关配置

| snddaudio配置   | snddaudio配置说明                             |
| --------------- | --------------------------------------------- |
| snddaudio0_used | 是否使用snddaudio驱动。 0 ：不使用； 1 ：使用 |

daudio0配置，即daudio0 platform驱动的相关配置

| daudio配置        | daudio配置说明                                               |
| ----------------- | ------------------------------------------------------------ |
| daudio0_used      | 是否使用daudio驱动。 0 ：不使用； 1 ：使用                   |
| daudio_master     | 1: SND_SOC_DAIFMT_CBM_CFM(codec clk & FRM<br/>master),即daudio接口作为slave, codec作为master<br/>2: SND_SOC_DAIFMT_CBS_CFM(codec clk slave &<br/>FRM master),一般不用<br/>3: SND_SOC_DAIFMT_CBM_CFS(codec clk master &<br/>frame slave),一般不用<br/>4: SND_SOC_DAIFMT_CBS_CFS(codec clk & FRM<br/>slave),即daudio接口作为master, codec作为slave |
| audio_format      | 1: SND_SOC_DAIFMT_I2S(standard i2s format)<br/>2: SND_SOC_DAIFMT_RIGHT_J(right justfied format)<br/>3: SND_SOC_DAIFMT_LEFT_J(left justfied format)<br/>4: SND_SOC_DAIFMT_DSP_A(pcm. MSB is available<br/>on 2nd BCLK rising edge after LRC rising edge)<br/>5: SND_SOC_DAIFMT_DSP_B(pcm. MSB is available<br/>on 1nd BCLK rising edge after LRC rising edge) |
| signal_inversion  | 1: SND_SOC_DAIFMT_NB_NF(normal bit clock +<br/>frame)<br/>2: SND_SOC_DAIFMT_NB_IF(normal BCLK + inv<br/>FRM)<br/>3: SND_SOC_DAIFMT_IB_NF(invert BCLK + nor FRM)<br/>4: SND_SOC_DAIFMT_IB_IF(invert BCLK + FRM) |
| slot_width_select | 支持8bit, 16bit, 32bit宽度                                   |
| pcm_lrck_period   | 一般可配置16/32/64/128/256个bclk                             |
| msb_lsb_first     | 0: msb first; 1: lsb first                                   |
| frametype         | 0: short frame = 1 clock width; 1: long frame = 2 clock width |
| tdm_config        | 0: pcm mode; 1: i2s mode                                     |
| tx_data_mode      | 0: 16bit linear PCM;1: reserved;2: 8bit u-law;3: 8bit<br/>a-law |

| daudio配置   | daudio配置说明                                              |
| ------------ | ----------------------------------------------------------- |
| rx_data_mode | 0: 16bit linear PCM;1: reserved;2: 8bit u-law;3: 8bit a-law |

具体Daudio外接codec,数字功放的配置，可参考《R18外挂codec:ac108》《R18外挂数字功放TAS5731》

### 2.7.6 SPDIF

硬件特性

• 支持S/PDIF_OUT

- 支持mono和stereo模式
- 输出支持22.05kHz, 24kHz, 32kHz, 44.1kHz, 48kHz, 88.2kHz, 96kHz, 176.4kHz,
  192kHz采样率
- 支持16bit,24bit采样精度

#### 2.7.6.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner SPDIF Support
```

#### 2.7.6.2 sys_config配置.

```
[spdif]
spdif_used = 0
[sndspdif]
sndspdif_used = 0
```

spdif配置，即platform驱动的相关配置

| spdif配置  | spdif配置说明                             |
| ---------- | ----------------------------------------- |
| spdif_used | 是否使用spdif驱动。 0 ：不使用； 1 ：使用 |

sndspdif配置，即machine驱动的相关配置

| sndspdif配置  | sndspdif配置说明                             |
| ------------- | -------------------------------------------- |
| sndspdif_used | 是否使用sndspdif驱动。 0 ：不使用； 1 ：使用 |

- sys_config中不需要配置codec驱动相关信息

因为machine驱动代码中默认配置了”spdif-utils”作为codec驱动,代码路径：

```
linux-4.4/sound/soc/sunxi/spdif-utils.c
```

### 2.7.7 外挂codec:AC108

R18标案tulip-noma搭配了MIC子板，含有两片AC108,每片最高可录 4 通道

下面对R18如何配置使用AC108作简单介绍

#### 2.7.7.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner Digital Audio Support
CODEC drivers --->
<*> Sunxi AC108 Codec
```

#### 2.7.7.2 sys_config&dts配置.

R18通过twi1控制AC108,而i2s0用于音频数据的传输

twi部分配置,可通过dts进行配置:

```
twi1: twi@0x01c2b000 {
status = "okay";
ac108@35{
compatible = "Allwinnertech,MicArray_1";
debug_mode = <0>;
pga_gain = <0x32>;
ref_pga_gain = <0x08>;
ref_chip_addr = <0x3b>;
ref_channel_num = <0x2>;
pa_double_used = <0x1>;
```

```
codec_mic_used = <0x0>;
gpio-power = <&r_pio PL 12 1 1 1 1>;
twi_bus = <1>;
voltage_enable = "nocare";
power_vol = <0x0>;
slot_width = <0x18>;
reg = <0x35>;
};
ac108@3b{
compatible = "Allwinnertech,MicArray_0";
reg = <0x3b>;
debug_mode = <0>;
pga_gain = <0x32>;
ref_pga_gain = <0x08>;
ref_chip_addr = <0x3b>;
ref_channel_num = <0x2>;
pa_double_used = <0x1>;
codec_mic_used = <0x0>;
twi_bus = <1>;
voltage_enable = "nocare";
gpio-power = <&r_pio PL 12 1 1 1 1>;
power_vol = <0x0>;
slot_width = <0x18>;
};
};
```

I2S部分需要配置sys_config以及dts

- sys_config部分主要涉及i2s相关格式，需要根据AC108spec进行配置，sdk默认daudio0
  配置可正常运行AC108
- dts部分主要需要指定ASOC codec以及codec-dai驱动的名称，如

```
snddaudio0:sound@1 {
sunxi,snddaudio-codec = "ac108.1-0035";
sunxi,snddaudio-codec-dai = "ac108-pcm1";
};
```

#### 2.7.7.3 使用

进入系统后,通过命令cat /proc/asound/cards列出当前声卡信息，如果发现ac108相关声
卡，说明已经正常加载驱动

无需额外设置音频通路，可直接用下面命令进行录音:

```
arecord -Dhw:sndac10810035 -f S16_LE -r 16000 -c 8 /tmp/test.wav
```

### 2.7.8 外挂数字功放TAS5731.

R18标案tulip-noma搭配了一片数字功放TAS5731


下面对R18如何配置使用tas5731作简单介绍

#### 2.7.8.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner Digital Audio Support
CODEC drivers --->
<*> TAS5731 PA
```

#### 2.7.8.2 sys_config&dts配置.

R18通过TWI0控制数字功放,而I2S1用于音频数据的传输

twi部分配置,可通过dts进行配置:

```
twi0: twi@0x01c2ac00 {
status = "okay";
tas5731-codec@1b{
compatible = "Allwinnertech,tas5731_PA";
tas5731_power = <&pio PH 8 1 1 1 1>;
tas5731_reset = <&pio PB 2 1 1 1 1>;
amp_poweren = <&r_pio PL 7 1 1 1 1>;
regulator_name = "vcc-amp";
reg = <0x1b>;
};
};
```

I2S部分需要配置sys_config以及dts

- sys_config部分主要涉及i2s相关格式，需要根据具体数字功放进行配置，sdk默认daudio1
  配置可正常运行tas5731
- dts部分主要需要指定ASOC codec以及codec-dai驱动的名称，如

```
snddaudio1:sound@2 {
sunxi,snddaudio-codec = "tas5731-codec.0-001b";
sunxi,snddaudio-codec-dai = "tas5731_audio";
};
```

#### 2.7.8.3 使用

进入系统后,通过命令cat /proc/asound/cards列出当前声卡信息，如果发现tas5731相关声
卡，说明已经正常加载驱动


无需额外设置音频通路，可直接用下面命令进行播歌:

```
aplay -Dhw:sndtas5731codec /mnt/UDISK/16000-stere-10s.wav
```

### 2.7.9 HDMI音频接口

R18使用I2S2将音频数据传输到HDMI模块，并且I2S2也只能用于HDMI。

#### 2.7.9.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner HDMI Audio Support
<*> Allwinner Digital Audio Support
```

#### 2.7.9.2 sys_config配置.

```
[daudio2]
daudio2_used = 1
[sndhdmi]
sndhdmi_used = 1
```

daudio2配置，即daudio2 platform驱动的相关配置

| daudio2配置  | daudio2配置说明                             |
| ------------ | ------------------------------------------- |
| daudio2_used | 是否使用daudio2驱动。 0 ：不使用； 1 ：使用 |

sndhdmi配置，即sndhdmi machine驱动的相关配置

| sndhdmi配置  | sndhdmi配置说明                             |
| ------------ | ------------------------------------------- |
| sndhdmi_used | 是否使用sndhdmi驱动。 0 ：不使用； 1 ：使用 |

### 2.7.10 标案音频测试方法.

该章节主要介绍在标案上进行播歌，录音的测试命令。


#### 2.7.10.1播放

```
amixer -Dhw:audiocodec cset name='AIF1IN0L Mux' 'AIF1_DA0L'
amixer -Dhw:audiocodec cset name='AIF1IN0R Mux' 'AIF1_DA0R'
amixer -Dhw:audiocodec cset name='DACL Mixer AIF1DA0L Switch' 1
amixer -Dhw:audiocodec cset name='DACR Mixer AIF1DA0R Switch' 1
amixer -Dhw:audiocodec cset name='HP_R Mux' 'DACR HPR Switch'
amixer -Dhw:audiocodec cset name='HP_L Mux' 'DACL HPL Switch'
amixer -Dhw:audiocodec cset name='Headphone Switch' 1
aplay -Dhw:audiocodec /mnt/UDISK/1KHz_0dB_16000.wav
```

可通过下面命令调节硬件上的模拟音量:

```
amixer -Dhw:audiocodec cset name='headphone volume' 60
```

#### 2.7.10.2录音

```
amixer -Dhw:audiocodec cset name='LADC input Mixer MIC1 boost Switch' 1
amixer -Dhw:audiocodec cset name='RADC input Mixer MIC2 boost Switch' 1
amixer -Dhw:audiocodec cset name='AIF1 AD0L Mixer ADCL Switch' 1
amixer -Dhw:audiocodec cset name='AIF1 AD0R Mixer ADCR Switch' 1
amixer -Dhw:audiocodec cset name='AIF1OUT0L Mux' 'AIF1_AD0L'
amixer -Dhw:audiocodec cset name='AIF1OUT0R Mux' 'AIF1_AD0R'
arecord -Dhw:audiocodec -f S16_LE -r 16000 -c 2 /tmp/test.wav
```

## 2.8 R30音频接口

### 2.8.1 硬件资源

R30包含 5 个音频模块，分别是内置AudioCodec,Daudio0,Daudio1,Daudio2以及Dmic


![图2-17: R30音频硬件框图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-17.jpg)

### 2.8.2 时钟源

R30中， 5 个音频模块的时钟源均来自pll_audio。

pll_audio可以输出24.576M或者22.5792M的时钟，分别支持48k系列，44.1k系列的播
放录音。

![图2-18: R30时钟源](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-18.jpg)

### 2.8.3 代码结构

```
linux-4.9/sound/soc/sunxi/
├── sun50iw3-codec.c // codec驱动
├── sun50iw3-codec.h
├── sun50iw3-sndcodec.c // codec machine驱动
├── sunxi-inter-i2s.c // codec platform驱动
```

```
├── sunxi-inter-i2s.h
├── sunxi-daudio.c // daudio platform驱动
├── sunxi-daudio.h
├── sunxi-dmic.c // dmic platform驱动
├── sunxi-dmic.h
├── sunxi-pcm.c //通用文件，提供注册platform驱动的接口及相关函数集
├── sunxi-pcm.h
├── sunxi_rw_func.c //通用文件，读写模拟/数字寄存器的接口
├── sunxi_rw_func.h
├── sunxi-snddaudio.c // daudio machine驱动
├── sunxi-snddaudio.h
├── sunxi-snddmic.c // dmic machine驱动
└── sunxi-snddmic.h
linux-4.9/sound/soc/codecs/dmic.c // dmic codec驱动
linux-4.9/sound/soc/soc-utils.c // daudio codec驱动
```

### 2.8.4 AudioCodec

硬件特性

• 两路DAC

- 支持16bit,24bit采样精度
- 支持8KHz~192KHz采样率
- 两路ADC
- 支持16bit,24bit采样精度
- 支持8KHz~48KHz采样率
- 四路模拟输出:
- 一路立体声earpiece输出(EAROUTP,EAROUTN)
- 一路立体声phoneout输出(PHONEOUTP,PHONEOUTN)
- 一路立体声headphone输出(HPOUTL,HPOUTR)
- 一路立体声lineout输出(LINEOUTL,LINEOUTR)
- 四路路模拟输入：MIC1,MIC2,linein,phonein
- 支持headphone驱动
- 支持earpiece驱动
- 支持同时playback和record(全双工模式)
- 支持适用于DAC的DRC功能
- 支持适用于ADC的AGC,DRC功能


#### 2.8.4.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner Sun50iw3 Codec Support
```

#### 2.8.4.2 sys_config配置.

```
[sndcodec]
sndcodec_used = 0x1
aif2fmt = 0x3
aif3fmt = 0x3
aif2master = 0x1
hp_detect_case = 0x0
;------------------------------------------------------------------------------
[i2s]
i2s_used = 0x1
;-------------------------------------------------------------------------------
[codec]
codec_used = 0x1
headphonevol = 0x3b
spkervol = 0x1b
maingain = 0x4
headsetmicgain = 0x4
adcagc_cfg = 0x0
adcdrc_cfg = 0x0
adchpf_cfg = 0x0
dacdrc_cfg = 0x0
dachpf_cfg = 0x0
aif2config = 0x0
aif3config = 0x0
gpio-spk = port:PB3<1><default><default><0>
```

sndcodec配置，即machine驱动的相关配置

| sndcodec配置   | sndcodec配置说明                                             |
| -------------- | ------------------------------------------------------------ |
| sndcodec_used  | 是否使用sndcodec驱动。 0 ：不使用； 1 ：使用                 |
| aif2fmt        | 1: SND_SOC_DAIFMT_I2S(standard i2s format)<br/>2: SND_SOC_DAIFMT_RIGHT_J(right justfied format)<br/>3: SND_SOC_DAIFMT_LEFT_J(left justfied format)<br/>4: SND_SOC_DAIFMT_DSP_A(pcm. MSB is available on 2nd<br/>BCLK rising edge after LRC rising edge)<br/>5: SND_SOC_DAIFMT_DSP_B(pcm. MSB is available on 1nd<br/>BCLK rising edge after LRC rising edge) |
| aif3fmt        | 1: SND_SOC_DAIFMT_I2S(standard i2s format)<br/>2: SND_SOC_DAIFMT_RIGHT_J(right justfied format)<br />3: SND_SOC_DAIFMT_LEFT_J(left justfied format)<br/>4: SND_SOC_DAIFMT_DSP_A(pcm. MSB is available on 2nd<br/>BCLK rising edge after LRC rising edge)<br/>5: SND_SOC_DAIFMT_DSP_B(pcm. MSB is available on 1nd<br/>BCLK rising edge after LRC rising edge) |
| aif2master     | 1: SND_SOC_DAIFMT_CBM_CFM(codec clk & FRM master),<br/>即aif接口选择master模式<br/>2: SND_SOC_DAIFMT_CBS_CFS(codec clk & FRM slave),即aif<br/>接口选择slave模式 |
| hp_detect_case | jack irq level, 0:low; 1:high                                |

I2S配置，即audiocodec platform驱动的相关配置,内部aif接口用的I2S(与I2S0,I2S1接口无关)

| i2s配置  | i2s配置说明                             |
| -------- | --------------------------------------- |
| i2s_used | 是否使用i2s驱动。 0 ：不使用； 1 ：使用 |

codec配置，即内置audiocodec驱动的相关配置

| codec配置      | codec配置说明                                                |
| -------------- | ------------------------------------------------------------ |
| codec_used     | 是否使用codec驱动。 0 ：不使用； 1 ：使用                    |
| headphonevol   | 初始化headphone volume，可设定范围0~0x3f,表示0~-62dB,<br/>-1dB/step |
| spkervol       | 初始化speaker volume，可设定范围0~0x1f, 0或者 1 表示mute,<br/>2~31表示-43.5dB~0dB, 1.5dB/step |
| headsetmicgain | 指的是MIC2增益,可设定范围0~0x7, 0:0dB, 1~7:15~33dB,<br/>3dB/step,一般设置0x4,即24dB |
| adcinputgain   | adc增益，可设定范围0~0x7,表示-4.5~6dB, 1.5dB/step,一般设置0x3,即0dB |
| adcagc_cfg     | 是否使用adcagc. 0:不使用； 1 ：使用                          |
| adcdrc_cfg     | 是否使用adcdrc. 0:不使用； 1 ：使用                          |
| adchpf_cfg     | 是否使用adchpf. 0:不使用； 1 ：使用                          |
| dacdrc_cfg     | 是否使用dacdrc. 0:不使用； 1 ：使用                          |
| aif2config     | 是否使用aif2. 0:不使用； 1 ：使用                            |
| aif3config     | 是否使用aif3. 0:不使用； 1 ：使用                            |
| gpio-spk       | PA使能引脚                                                   |

#### 2.8.4.3 codec数据通路

![图2-19: R30音频通路](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-19.jpg)

```
通过SPKL/R播歌
AIF1DACL --> AIF1IN0L Mux --> DACL Mixer --> Left Output Mixer --> SPK_L Mux --> SPKL
AIF1DACR --> AIF1IN0R Mux --> DACR Mixer --> Right Output Mixer --> SPK_R Mux --> SPKR
通过LINEOUTL/R播歌
AIF1DACL --> AIF1IN0L Mux --> DACL Mixer --> Left Output Mixer --> LINEOUTL Mux -->
LINEOUTL
AIF1DACR --> AIF1IN0R Mux --> DACR Mixer --> Right Output Mixer --> LINEOUTR Mux -->
LINEOUTR
通过HPOUTL/R播歌
AIF1DACL --> AIF1IN0L Mux --> DACL Mixer --> HP_L Mux --> HPOUTL
AIF1DACR --> AIF1IN0R Mux --> DACR Mixer --> HP_R Mux --> HPOUTR
通过MIC1,2录音
AIF1ADCL <-- AIF1OUT0L Mux <-- AIF1 AD0L Mixer <-- LADC input Mixer <-- MIC1 PGA <-- MIC1P/
N
AIF1ADCR <-- AIF1OUT0R Mux <-- AIF1 AD0R Mixer <-- RADC input Mixer <-- MIC2 PGA <-- MIC2P/
N
通过LINEINL/R录音
AIF1ADCL <-- AIF1OUT0L Mux <-- AIF1 AD0L Mixer <-- LADC input Mixer <-- LINEINN
AIF1ADCR <-- AIF1OUT0R Mux <-- AIF1 AD0R Mixer <-- RADC input Mixer <-- LINEINP
```

R30相关控件如下表：

| 控件名称                                         | 功能                                                  | 数值                                                         |
| ------------------------------------------------ | ----------------------------------------------------- | ------------------------------------------------------------ |
| Headphone Switch                                 | Headphone通路使能                                     | 0:关闭; 1:开启                                               |
| Lineout Switch                                   | Lineout通路使能                                       | 0:关闭; 1:开启                                               |
| ADC input gain<br/>control                       | ADC增益                                               | 0–7,表示-4.5–6dB                                             |
| ADC volume                                       | ADCL/ADCR音量设置                                     | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| AIF1 AD0L Mixer<br/>ADCL Switch                  | AIF1 AD0L Mixer设<br/>置，使能ADCL通路                | 0:关闭; 1:开启                                               |
| AIF1 AD0L Mixer<br/>AIF1 DA0L Switch             | AIF1 AD0L Mixer设<br/>置，使能AIF1 DA0L通<br/>路      | 0:关闭; 1:开启                                               |
| AIF1 AD0L Mixer<br/>AIF2 DACL Switch             | AIF1 AD0L Mixer设<br/>置，使能AIF2 DACL通<br/>路      | 0:关闭; 1:开启                                               |
| AIF1 AD0L Mixer<br/>AIF2 DACR Switch             | AIF1 AD0L Mixer设<br/>置，使能AIF2 DACR通<br/>路      | 0:关闭; 1:开启                                               |
| AIF1 AD0R Mixer<br/>ADCR Switch                  | AIF1 AD0R Mixer设<br/>置，使能ADCR通路                | 0:关闭; 1:开启                                               |
| AIF1 AD0R Mixer<br/>AIF1 DA0R Switch             | AIF1 AD0R Mixer设<br/>置，使能AIF1 DA0R通<br/>路      | 0:关闭; 1:开启                                               |
| AIF1 AD0R Mixer<br/>AIF2 DACL Switch             | AIF1 AD0R Mixer设<br/>置，使能AIF2 DACL通<br/>路      | 0:关闭; 1:开启                                               |
| AIF1 AD0R Mixer<br/>AIF2 DACR Switch             | AIF1 AD0R Mixer设<br/>置，使能AIF2 DACR通<br/>路      | 0:关闭; 1:开启                                               |
| AIF1 AD1L Mixer<br/>ADCL Switch                  | AIF1 AD1L Mixer设<br/>置，使能ADCL通路                | 0:关闭; 1:开启                                               |
| AIF1 AD1L Mixer<br/>AIF2 DACL Switch             | AIF1 AD1L Mixer设<br/>置，使能AIF2 DACL通<br/>路      | 0:关闭; 1:开启                                               |
| AIF1 AD1R Mixer<br/>ADCR Switch                  | AIF1 AD1R Mixer设<br/>置，使能ADCR通路                | 0:关闭; 1:开启                                               |
| AIF1 AD1R Mixer<br/>AIF2 DACR Switch             | AIF1 AD1R Mixer设<br/>置，使能AIF2 DACR通<br/>路      | 0:关闭; 1:开启                                               |
| AIF1 ADC timeslot<br/>0 mixer gain               | AIF1 ADC0L/ADC0R<br/>Mixer,数字增益                   | 0:0dB; 1:-6dB;                                               |
|                                                  |                                                       | 对于ADC0L Mixer,<br/>bit0:AIF2 DACR;<br/>bit1:ADCL;<br/>bit2:AIF2 DACL;<br/>bit3:AIF2 DA0L;<br/>对于ADC0R Mixer,<br/>bit0:AIF2 DACL;<br/>bit1:ADCR;<br/>bit2:AIF2 DACR;<br/>bit3:AIF2 DA0R; |
| AIF1 ADC timeslot<br/>0 volume                   | AIF1 ADC0L/ADC0R<br/>音量设置                         | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| AIF1 ADC timeslot<br/>1 mixer gain               | AIF1 ADC1L/ADC1R<br/>Mixer,数字增益                   | 0:0dB; 1:-6dB;                                               |
|                                                  |                                                       | 对于ADC1L Mixer,<br/>bit0:ADCL;<br/>bit1:AIF2 DACL;<br/>对于ADC1R Mixer,<br/>bit0:ADCR;<br/>bit1:AIF2 DACR; |
| AIF1 ADC timeslot<br/>1 volume                   | AIF1 ADC1L/ADC1R<br/>音量设置                         | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| AIF1 DAC timeslot<br/>0 volume                   | AIF1 DAC0L/DAC0R<br/>音量设置                         | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| AIF1 DAC timeslot<br/>1 volume                   | AIF1 DAC1L/DAC1R<br />音量设置                        | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| AIF1IN0L Mux                                     | AIF1IN0L Mux设置                                      | 0:AIF1_DA0L; 1:AIF1_DA0R;<br/>2:SUM_AIF1DA0L_AIF1DA0R;<br/>3:AVE_AIF1DA0L_AIF1DA0R |
| AIF1IN0R Mux                                     | AIF1IN0R Mux设置                                      | 0:AIF1_DA0R; 1:AIF1_DA0L;<br/>2:SUM_AIF1DA0L_AIF1DA0R;<br/>3:AVE_AIF1DA0L_AIF1DA0R |
| AIF1IN1L Mux                                     | AIF1IN1L Mux设置                                      | 0:AIF1_DA1L; 1:AIF1_DA1R;<br/>2:SUM_AIF1DA1L_AIF1DA1R;<br/>3:AVE_AIF1DA1L_AIF1DA1R |
| AIF1IN1R Mux                                     | AIF1IN1R Mux设置                                      | 0:AIF1_DA1R; 1:AIF1_DA1L;<br/>2:SUM_AIF1DA1L_AIF1DA1R;<br/>3:AVE_AIF1DA1L_AIF1DA1R |
| AIF1OUT0L Mux                                    | AIF1OUT0L Mux设置                                     | 0:AIF1_AD0L; 1:AIF1_AD0R;<br/>2:SUM_AIF1AD0L_AIF1AD0R;<br/>3:AVE_AIF1AD0L_AIF1AD0R |
| AIF1OUT0R Mux                                    | AIF1OUT0R Mux设置                                     | 0:AIF1_AD0R; 1:AIF1_AD0L;<br/>2:SUM_AIF1AD0L_AIF1AD0R;<br/>3:AVE_AIF1AD0L_AIF1AD0R |
| AIF1OUT1L Mux                                    | AIF1OUT1L Mux设置                                     | 0:AIF1_AD1L; 1:AIF1_AD1R;<br/>2:SUM_AIF1AD1L_AIF1AD1R;<br/>3:AVE_AIF1AD1L_AIF1AD1R |
| AIF1OUT1R Mux                                    | AIF1OUT1R Mux设置                                     | 0:AIF1_AD1R; 1:AIF1_AD1L;<br/>2:SUM_AIF1AD1L_AIF1AD1R;<br/>3:AVE_AIF1AD1L_AIF1AD1R |
| DAC mixer gain                                   | DAC mixer增益                                         | 0:0dB; 1:-6dB;                                               |
|                                                  |                                                       | 对于DACL Mixer,<br/>bit0:ADCL;<br/>bit1:AIF2 DACL;<br/>bit2:AIF1 DAC1L;<br/>bit3:AIF1 DAC0L;<br/>对于DACR Mixer,<br/>bit0:ADCR;<br/>bit1:AIF2 DACR;<br/>bit2:AIF1 DAC1R;<br/>bit3:AIF1 DAC0R; |
| DAC volume                                       | DACL/DACR音量设置                                     | 0–0xff, 0表示mute, 0x1~0xff表<br/>示-119.25dB~71.25dB,<br/>0.75dB/step,如0xA0表示0dB |
| DACL Mixer ADCL<br/>Switch                       | DACL Mixer设置，使能<br/>ADCL通路                     | 0:关闭; 1:开启                                               |
| DACL Mixer<br/>AIF1DA0L Switch                   | DACL Mixer设置，使能<br/>AIF1DA0L通路                 | 0:关闭; 1:开启                                               |
| DACL Mixer<br/>AIF1DA1L Switch                   | DACL Mixer设置，使能<br/>AIF1DA1L通路                 | 0:关闭; 1:开启                                               |
| DACL Mixer<br/>AIF2DACL Switch                   | DACL Mixer设置，使能<br/>AIF2DACL通路                 | 0:关闭; 1:开启                                               |
| DACR Mixer ADCR<br/>Switch                       | DACR Mixer设置，使<br/>能ADCR通路                     | 0:关闭; 1:开启                                               |
| DACR Mixer<br/>AIF1DA0R Switch                   | DACR Mixer设置，使<br/>能AIF1DA0R通路                 | 0:关闭; 1:开启                                               |
| DACR Mixer<br/>AIF1DA1R Switch                   | DACR Mixer设置，使<br/>能AIF1DA1R通路                 | 0:关闭; 1:开启                                               |
| DACR Mixer<br/>AIF2DACR Switch                   | DACR Mixer设置，使<br/>能AIF2DACR通路                 | 0:关闭; 1:开启                                               |
| External Speaker<br/>Switch                      | 使能Headphone以及<br/>PA                              | 0:关闭; 1:开启                                               |
| HP_L Mux                                         | HP_L Mux设置                                          | 0:DACL ; 1:Left Output Mixer                                 |
| HP_R Mux                                         | HP_R Mux设置                                          | 0:DACR ; 1:Right Output Mixer                                |
| LADC input Mixer<br/>LINEINL                     | LADC input Mixer设<br/>置，使能LINEINL通路            | 0:关闭; 1:开启                                               |
| LADC input Mixer<br/>MIC1 boost Switch           | LADC input Mixer设<br/>置，使能MIC1通路               | 0:关闭; 1:开启                                               |
| LADC input Mixer<br/>MIC2 boost Switch           | LADC input Mixer设<br/>置，使能MIC2通路               | 0:关闭; 1:开启                                               |
| LADC input Mixer<br/>l_output mixer<br/>Switch   | LADC input Mixer设<br/>置，使能l_output<br/>mixer通路 | 0:关闭; 1:开启                                               |
| LADC input Mixer<br/>r_output mixer<br/>Switch   | LADC input Mixer设<br/>置，使能r_output<br/>mixer通路 | 0:关闭; 1:开启                                               |
| LINEINL/R to L_R<br/>output mixer gain           | LINEINL/R to L or R<br/>output Mixer增益              | 0–7,表示-4.5–6dB                                             |
| LINEOUTL Mux                                     | LINEOUTL Mux设置                                      | 0:left output mixer; 1:left+right<br/>output mixer           |
| LINEOUTR Mux                                     | LINEOUTR Mux设置                                      | 0:right output mixer; 1:left+right<br/>output mixer          |
| Left Output Mixer<br/>DACL Switch                | Left Output Mixer设<br/>置，使能DACL通路              | 0:关闭; 1:开启                                               |
| Left Output Mixer<br/>DACR Switch                | Left Output Mixer设<br/>置，使能DACR通路              | 0:关闭; 1:开启                                               |
| Left Output Mixer<br/>LINEINL Switch             | Left Output Mixer设<br/>置，使能LINEINL通路           | 0:关闭; 1:开启                                               |
| Left Output Mixer<br/>MIC1Booststage<br/>Switch  | Left Output Mixer设<br/>置，使能MIC1通路              | 0:关闭; 1:开启                                               |
| Left Output Mixer<br/>MIC2Booststage<br/>Switch  | Left Output Mixer设<br/>置，使能MIC2通路              | 0:关闭; 1:开启                                               |
| MIC1 boost AMP<br/>gain control                  | MIC1增益                                              | 0–7, 0:0dB, 1~7:24–42dB,3dB/step                             |
| MIC1_G boost<br/>stage output mixer<br/>control  | MIC1 to L or R output<br/>Mixer增益                   | 0–7,表示-4.5–6dB                                             |
| MIC2 BST stage to<br/>L_R outp mixer<br/>gain    | MIC2 to L or R output<br/>Mixer增益                   | 0–7,表示-4.5–6dB                                             |
| MIC2 SRC                                         | MIC2 SRC设置                                          | 0:MIC3; 1:MIC2                                               |
| MIC2 boost AMP<br/>gain control                  | MIC2增益                                              | 0–7, 0:0dB, 1~7:24–42dB,3dB/step                             |
| RADC input Mixer<br/>LINEINR Switch              | RADC input Mixer设<br/>置，使能LINEINR通路            | 0:关闭; 1:开启                                               |
| RADC input Mixer<br/>MIC1 boost Switch           | RADC input Mixer设<br/>置，使能MIC1通路               | 0:关闭; 1:开启                                               |
| RADC input Mixer<br/>MIC2 boost Switch           | RADC input Mixer设<br/>置，使能MIC2通路               | 0:关闭; 1:开启                                               |
| RADC input Mixer<br/>l_output mixer<br/>Switch   | RADC input Mixer设<br/>置，使能l_output<br/>mixer通路 | 0:关闭; 1:开启                                               |
| RADC input Mixer<br/>l_output Switch             | RADC input Mixer设<br/>置，使能l_output<br/>mixer通路 | 0:关闭; 1:开启                                               |
| Right Output Mixer<br/>DACL Switch               | Right Output Mixer设<br/>置，使能DACL通路             | 0:关闭; 1:开启                                               |
| Right Output Mixer<br/>DACR Switch               | Right Output Mixer设<br/>置，使能DACR通路             | 0:关闭; 1:开启                                               |
| Right Output Mixer<br/>LINEINR Switch            | Right Output Mixer设<br/>置，使能LINEINR通路          | 0:关闭; 1:开启                                               |
| Right Output Mixer<br/>MIC1Booststage<br/>Switch | Right Output Mixer设<br/>置，使能MIC1通路             | 0:关闭; 1:开启                                               |
| Right Output Mixer<br/>MIC2Booststage<br/>Switch | Right Output Mixer设<br/>置，使能MIC2通路             | 0:关闭; 1:开启                                               |
| SPK_L Mux                                        | SPK_L Mux设置                                         | 0:MIXEL Switch; 1:MIXL MIXR<br/>Switch                       |
| SPK_R Mux                                        | SPK_R Mux设置                                         | 0:MIXER Switch; 1:MIXR MIXL<br/>Switch                       |
| digital volume                                   | 数字音量设置                                          | 0–63,表示-73.08–0dB                                          |
| headphone volume                                 | headphone音量设置                                     | 0–63,0表示mute; 1~63表<br/>示-62dB–0dB                       |
| lineout volume                                   | lineout音量设置                                       | 0–31,表示-43.5–0dB                                           |
| speaker volume                                   | speaker(lineout)音量<br/>设置                         | 0–31,表示-43.5–0dB                                           |

### 2.8.5 Daudio.

硬件特性

• 三路I2S/PCM；

• 支持主从模式

- 支持Left-justified,Right-justified,Standar mode I2S,PCM mode
- 支持i2s,pcm协议格式配置
- 支持同时playback和record(全双工模式)
- 支持8~192KHz采样率
- 支持16,24,32bit采样精度

#### 2.8.5.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner Digital Audio Support
```

#### 2.8.5.2 sys_config配置.

```
[snddaudio0]
snddaudio0_used = 1
[daudio0]
pcm_lrck_period = 0x80
slot_width_select = 0x20
pcm_lsb_first = 0x0
tx_data_mode = 0x0
rx_data_mode = 0x0
```

```
daudio_master = 0x04
audio_format = 0x01
signal_inversion = 0x01
frametype = 0x0
tdm_config = 0x01
mclk_div = 0x1
daudio0_used = 1
```

snddaudio0配置，即daudio0 machine驱动的相关配置

| snddaudio配置   | snddaudio配置说明                             |
| --------------- | --------------------------------------------- |
| snddaudio0_used | 是否使用snddaudio驱动。 0 ：不使用； 1 ：使用 |

daudio0配置，即daudio0 platform驱动的相关配置

| daudio配置        | daudio配置说明                                               |
| ----------------- | ------------------------------------------------------------ |
| daudio0_used      | 是否使用daudio驱动。 0 ：不使用； 1 ：使用                   |
| daudio_master     | 1: SND_SOC_DAIFMT_CBM_CFM(codec clk & FRM<br/>master),即daudio接口作为slave, codec作为master<br/>2: SND_SOC_DAIFMT_CBS_CFM(codec clk slave &<br/>FRM master),一般不用<br/>3: SND_SOC_DAIFMT_CBM_CFS(codec clk master &<br/>frame slave),一般不用<br/>4: SND_SOC_DAIFMT_CBS_CFS(codec clk & FRM<br/>slave),即daudio接口作为master, codec作为slave |
| audio_format      | 1: SND_SOC_DAIFMT_I2S(standard i2s format)<br/>2: SND_SOC_DAIFMT_RIGHT_J(right justfied format)<br/>3: SND_SOC_DAIFMT_LEFT_J(left justfied format)<br/>4: SND_SOC_DAIFMT_DSP_A(pcm. MSB is available<br/>on 2nd BCLK rising edge after LRC rising edge)<br/>5: SND_SOC_DAIFMT_DSP_B(pcm. MSB is available<br/>on 1nd BCLK rising edge after LRC rising edge) |
| signal_inversion  | 1: SND_SOC_DAIFMT_NB_NF(normal bit clock +<br/>frame)<br/>2: SND_SOC_DAIFMT_NB_IF(normal BCLK + inv<br/>FRM)<br/>3: SND_SOC_DAIFMT_IB_NF(invert BCLK + nor FRM)<br/>4: SND_SOC_DAIFMT_IB_IF(invert BCLK + FRM) |
| slot_width_select | 支持8bit, 16bit, 32bit宽度                                   |
| pcm_lrck_period   | 一般可配置16/32/64/128/256个bclk                             |
| msb_lsb_first     | 0: msb first; 1: lsb first                                   |
| frametype         | 0: short frame = 1 clock width; 1: long frame = 2<br/>clock width |
| tdm_config        | 0: pcm mode; 1: i2s mode                                     |

| daudio配置   | daudio配置说明                                               |
| ------------ | ------------------------------------------------------------ |
| tx_data_mode | 0: 16bit linear PCM;1: reserved;2: 8bit u-law;3: 8bit<br/>a-law |
| rx_data_mode | 0: 16bit linear PCM;1: reserved;2: 8bit u-law;3: 8bit<br/>a-law |

### 2.8.6 DMIC

硬件特性

• 支持 8 路输入

- 支持8~48KHz采样率
- 支持16/24bit采样精度

#### 2.8.6.1 内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner DMIC Support
```

#### 2.8.6.2 sys_config配置.

```
[dmic]
dmic_used = 0
[snddmic]
snddmic_used = 0
```

dmic配置，即platform驱动的相关配置

| dmic配置  | dmic配置说明                             |
| --------- | ---------------------------------------- |
| dmic_used | 是否使用dmic驱动。 0 ：不使用； 1 ：使用 |

snddmic配置，即machine驱动的相关配置

| snddmic配置  | snddmic配置说明                             |
| ------------ | ------------------------------------------- |
| snddmic_used | 是否使用snddmic驱动。 0 ：不使用； 1 ：使用 |

### 2.8.7 标案音频测试方法

该章节主要介绍在标案上进行播歌，录音的测试命令。

#### 2.8.7.1 播放

```
通过SPKL/R播歌,例如喇叭播歌
amixer -Dhw:audiocodec cset name='AIF1IN0L Mux' 'AIF1_DA0L'
amixer -Dhw:audiocodec cset name='AIF1IN0R Mux' 'AIF1_DA0R'
amixer -Dhw:audiocodec cset name='DACL Mixer AIF1DA0L Switch' 1
amixer -Dhw:audiocodec cset name='DACR Mixer AIF1DA0R Switch' 1
amixer -Dhw:audiocodec cset name='Left Output Mixer DACL Switch' 1
amixer -Dhw:audiocodec cset name='Right Output Mixer DACR Switch' 1
amixer -Dhw:audiocodec cset name='SPK_R Mux' 'MIXER_Switch'
amixer -Dhw:audiocodec cset name='SPK_L Mux' 'MIXEL_Switch'
amixer -Dhw:audiocodec cset name='External Speaker Switch' 1
aplay -Dhw:audiocodec /mnt/UDISK/1KHz_0dB_16000.wav
可通过下面命令调节硬件上的模拟音量:
amixer -Dhw:audiocodec cset name='speaker volume' 28
```

```
通过LINEOUTL/R播歌
amixer -Dhw:audiocodec cset name='AIF1IN0L Mux' 'AIF1_DA0L'
amixer -Dhw:audiocodec cset name='AIF1IN0R Mux' 'AIF1_DA0R'
amixer -Dhw:audiocodec cset name='DACL Mixer AIF1DA0L Switch' 1
amixer -Dhw:audiocodec cset name='DACR Mixer AIF1DA0R Switch' 1
amixer -Dhw:audiocodec cset name='Left Output Mixer DACL Switch' 1
amixer -Dhw:audiocodec cset name='Right Output Mixer DACR Switch' 1
amixer -Dhw:audiocodec cset name='LINEOUTL Mux' 'LOMIX'
amixer -Dhw:audiocodec cset name='LINEOUTR Mux' 'ROMIX'
amixer -Dhw:audiocodec cset name='Lineout Switch' 1
aplay -Dhw:audiocodec /mnt/UDISK/1KHz_0dB_16000.wav
可通过下面命令调节硬件上的模拟音量:
amixer -Dhw:audiocodec cset name='lineout volume' 28
```

```
通过HPOUTL/R播歌,例如耳机
amixer -Dhw:audiocodec cset name='AIF1IN0L Mux' 'AIF1_DA0L'
amixer -Dhw:audiocodec cset name='AIF1IN0R Mux' 'AIF1_DA0R'
amixer -Dhw:audiocodec cset name='DACL Mixer AIF1DA0L Switch' 1
amixer -Dhw:audiocodec cset name='DACR Mixer AIF1DA0R Switch' 1
amixer -Dhw:audiocodec cset name='HP_R Mux' 'DACR HPR Switch'
amixer -Dhw:audiocodec cset name='HP_L Mux' 'DACL HPL Switch'
amixer -Dhw:audiocodec cset name='Headphone Switch' 1
aplay -Dhw:audiocodec /mnt/UDISK/1KHz_0dB_16000.wav
```

```
可通过下面命令调节硬件上的模拟音量:
```

```
amixer -Dhw:audiocodec cset name='headphone volume' 60
```

#### 2.8.7.2 录音

```
通过MIC1,MIC2录音
amixer -Dhw:audiocodec cset name='LADC input Mixer MIC1 boost Switch' 1
amixer -Dhw:audiocodec cset name='RADC input Mixer MIC2 boost Switch' 1
amixer -Dhw:audiocodec cset name='AIF1 AD0L Mixer ADCL Switch' 1
amixer -Dhw:audiocodec cset name='AIF1 AD0R Mixer ADCR Switch' 1
amixer -Dhw:audiocodec cset name='AIF1OUT0L Mux' 'AIF1_AD0L'
amixer -Dhw:audiocodec cset name='AIF1OUT0R Mux' 'AIF1_AD0R'
amixer -Dhw:audiocodec cset name='MIC1 boost AMP gain control' 4
amixer -Dhw:audiocodec cset name='MIC2 boost AMP gain control' 4
arecord -Dhw:audiocodec -f S16_LE -r 16000 -c 2 /tmp/test.wav
```


## 2.9 R328音频接口.

### 2.9.1 硬件资源

R328音频接口丰富，包含 6 个音频模块，分别是内置AudioCodec,Daudio0,Daudio1,Daudio2,
Dmic,Spdif。

另外还支持MAD作语音唤醒检测(详细请看R328 MAD章节)。

![图2-34: R328音频硬件框图](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-34.jpg)


### 2.9.2 时钟源.

R328中， 6 个音频模块的时钟源均来自pll_audio。

pll_audio可以输出24.576M或者22.5792M的时钟，分别支持48k系列，44.1k系列的播
放录音。

![图2-35: R328时钟源](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-35.jpg)

### 2.9.3 代码结构

```
linux-4.9/sound/soc/sunxi/
├── spdif-utils.c // spdif codec驱动
├── sun8iw18-codec.c // codec驱动
├── sun8iw18-codec.h
├── sun8iw18-sndcodec.c // codec machine驱动
├── sunxi-cpudai.c // codec platform驱动
├── sunxi-daudio.c // daudio platform驱动
├── sunxi-daudio.h
├── sunxi-dmic.c // dmic platform驱动
├── sunxi-dmic.h
├── sunxi-mad.c //提供MAD相关功能接口
├── sunxi-mad.h
├── sunxi-pcm.c //通用文件，提供注册platform驱动的接口及相关函数集
├── sunxi-pcm.h
├── sunxi_rw_func.c //通用文件，读写模拟/数字寄存器的接口
├── sunxi_rw_func.h
├── sunxi-snddaudio.c // daudio machine驱动
├── sunxi-snddaudio.h
├── sunxi-snddmic.c // dmic machine驱动
├── sunxi-snddmic.h
├── sunxi-sndspdif.c // spdif machine驱动
├── sunxi-spdif.c // spdif platform驱动
└── sunxi-spdif.h
linux-4.9/sound/soc/codecs/dmic.c // dmic codec驱动
linux-4.9/sound/soc/soc-utils.c // daudio codec驱动
```

### 2.9.4 AudioCodec

硬件特性

• 一路DAC

- 支持16bit,24bit采样精度
- 支持8KHz~192KHz采样率
- 三路ADC
- 支持16bit,24bit采样精度
- 支持8KHz~48KHz采样率
- 一路模拟输出：一路差分输出lineoutP/N，支持单端lineout输出
- 三路模拟输入：MIC1,MIC2,MIC3
- 支持同时playback和record(全双工模式)
- DAC及ADC均支持 5 段DRC
- DAC FIFO长度128*24bits, ADC FIFO长度128*24bits

#### 2.9.4.1内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner Sun8iw18 Codec Support
```

#### 2.9.4.2 sys_config配置

```
[sndcodec]
sndcodec_used = 0x1
;------------------------------------------------------------------------------
[cpudai]
cpudai_used = 0x1
;-------------------------------------------------------------------------------
[codec]
codec_used = 0x1
digital_vol = 0x0
lineout_vol =0x1a
mic1gain = 0x4
mic2gain = 0x4
mic3gain = 0x0
adcgain = 0x3
adcagc_cfg = 0x0
adcdrc_cfg = 0x0
adchpf_cfg = 0x0
```

```
dacdrc_cfg = 0x0
dachpf_cfg = 0x0
pa_ctl_level = 0x1
pa_msleep_time = 160
gpio-spk = port:PH9<1><1><1><1>
```

sndcodec配置，即machine驱动的相关配置

| sndcodec配置  | sndcodec配置说明                             |
| ------------- | -------------------------------------------- |
| sndcodec_used | 是否使用sndcodec驱动。 0 ：不使用； 1 ：使用 |

cpudai配置，即platform驱动的相关配置

| cpudai配置  | cpudai配置说明                             |
| ----------- | ------------------------------------------ |
| cpudai_used | 是否使用cpudai驱动。 0 ：不使用； 1 ：使用 |

codec配置，即内置audiocodec驱动的相关配置

| codec配置      | codec配置说明                                                |
| -------------- | ------------------------------------------------------------ |
| codec_used     | 是否使用codec驱动。 0 ：不使用； 1 ：使用                    |
| digital_vol    | 初始化digital volume，可设定范围0~0x3f,表示0~-73.08dB,<br/>-1.16dB/step |
| lineout_vol    | lineout volume，可设定范围0~0x1f,表示-43.5dB~0dB, 1.5dB/step |
| mic1gain       | mic1增益，可设定范围0~0x7, 0:0dB, 1~7:15~33dB, 3dB/step,一<br/>般设置0x4,即24dB |
| mic2gain       | mic2增益，可设定范围0~0x7, 0:0dB, 1~7:15~33dB, 3dB/step,一<br/>般设置0x4,即24dB |
| mic3gain       | mic3增益，可设定范围0~0x7, 0:0dB, 1~7:15~33dB, 3dB/step,一<br/>般设置0x4,即24dB.如果作为aec回路，则需要设置为0dB |
| adcgain        | adc增益，可设定范围0~0x7,表示-4.5~6dB, 1.5dB/step,一般设置<br/>0x3,即0dB |
| adcdrc_cfg     | 是否使用adcdrc. 0:不适用； 1 ：使用                          |
| adchpf_cfg     | 是否使用adchpf. 0:不适用； 1 ：使用                          |
| dacdrc_cfg     | 是否使用dacdrc. 0:不适用； 1 ：使用                          |
| dachpf_cfg     | 是否使用dachpf. 0:不适用； 1 ：使用                          |
| pa_ctl_level   | PA引脚使能方式。0:低电平有效； 1 ：高电平有效                |
| pa_msleep_time | 操作PA之后的延时时间(用来避免pop音)                          |
| gpio-spk       | PA使能引脚                                                   |

**说明**

- **如果想要正常加载** **_audiocodec_** **声卡，需要把** **_codec,platform,machine_** **驱动都选上，即**
  **_codec_used,cpudai_used,sndcodec_used_** **都置为** **_1_** **；**
- **_digital_vol,lineout_vol_** **等值会在驱动初始化的时候设置，进入系统后还可以通过** **_amixer_** **工具对应控件进行再次修改；**
- **注意** **_gpio-spk_** **是否配置正确，是否有其他模块复用了该** **_gpio;_**
- **注意** **_pa_ctl_level_** **，实际功放的** **_PA_** **引脚是高电平有效，还是低电平有效**

#### 2.9.4.3 codec数据通路

![图2-36: R328音频通路](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-36.jpg)

```
播歌
Playback --> DACL --> Left LINEOUT Mux --> LINEOUTL --> External Speaker
Playback --> DACR --> Right LINEOUT Mux --> LINEOUTR --> External Speaker
录音
MIC1 --> MIC1 PGA ---> Left Input Mixer --> ADCL --> Capture
MIC2 --> MIC2 PGA ---> Right Input Mixer --> ADCL --> Capture
MIC3 --> MIC3 PGA ---> Xadc Input Mixer --> ADCL --> Capture
```

R328所有控件如下表：

| 控件名称                                | 功能                                     | 数值                                            |
| --------------------------------------- | ---------------------------------------- | ----------------------------------------------- |
| Lineout Switch                          | 使能lineout                              | 0:关闭; 1:开启                                  |
| ADC gain volume                         | ADC增益                                  | 0–7,表示-4.5–6dB,具体计<br/>算请看注释 1        |
| External Speaker Switch                 | 使能lineout以及PA                        | 0:关闭; 1:开启                                  |
| LINEOUT volume                          | lineout音量设置                          | 0–31,表示-43.5–0dB,具体<br/>计算请看注释 2      |
| Left Input Mixer DACL<br/>Switch        | Left Input Mixer设置，使<br/>能DACL通路  | 0:关闭; 1:开启                                  |
| Left Input Mixer MIC1<br/>Boost Switch  | Left Input Mixer设置，使<br/>能MIC1通路  | 0:关闭; 1:开启                                  |
| Left Input Mixer MIC2<br/>Boost Switch  | Left Input Mixer设置，使<br/>能MIC2通路  | 0:关闭; 1:开启                                  |
| Left Input Mixer MIC3<br/>Boost Switch  | Left Input Mixer设置，使<br/>能MIC3通路  | 0:关闭; 1:开启                                  |
| Left LINEOUT Mux                        | Left Lineout Mux设置                     | 0:DACL; 1:NULL(空)                              |
| Right LINEOUT Mux                       | Right Lineout Mux设置                    | 0:NULL(空); 1:DACL                              |
| MIC1 gain volume                        | MIC1 Boost AMP gain                      | 0–7, 0:0dB, 1~7:15–33dB,<br/>具体计算请看注释 3 |
| MIC2 gain volume                        | MIC2 Boost AMP gain                      | 与MIC1 gain volume设置<br/>一样                 |
| MIC3 gain volume                        | MIC3 Boost AMP gain                      | 与MIC1 gain volume设置<br/>一样                 |
| Right Input Mixer DACL<br/>Switch       | Right Input Mixer设置，<br/>使能DACL通路 | 0:关闭; 1:开启                                  |
| Right Input Mixer MIC1<br/>Boost Switch | Right Input Mixer设置，<br/>使能MIC1通路 | 0:关闭; 1:开启                                  |
| Right Input Mixer MIC2<br/>Boost Switch | Right Input Mixer设置，<br/>使能MIC2通路 | 0:关闭; 1:开启                                  |
| Right Input Mixer MIC3<br/>Boost Switch | Right Input Mixer设置，<br/>使能MIC3通路 | 0:关闭; 1:开启                                  |
| Xadc Input Mixer DACL<br/>Switch        | Xadc Input Mixer设置，使<br/>能DACL通路  | 0:关闭; 1:开启                                  |
| Xadc Input Mixer MIC1<br/>Boost Switch  | Xadc Input Mixer设置，使<br/>能MIC1通路  | 0:关闭; 1:开启                                  |
| Xadc Input Mixer MIC2<br/>Boost Switch  | Xadc Input Mixer设置，使<br/>能MIC2通路  | 0:关闭; 1:开启                                  |
| Xadc Input Mixer MIC3<br/>Boost Switch  | Xadc Input Mixer设置，使<br/>能MIC3通路  | 0:关闭; 1:开启                                  |
| codec hub mode                          | 使能audiocodec hub功能                   | 0:关闭; 1:开启                                  |
| digital volume                          | 数字端音量设置                           | 0–63,表示-73.08–0dB,具<br/>体计算请看注释 4     |



• 注释 1


```
ADC gain volume计算方法：
应用层可设置范围:0～ 7
对应实际硬件设置的范围:-4.5~6dB, step: 1.5dB
换算方法:-4.5+(n\*1.5)
举例，设置0dB:
-4.5+(n\*1.5) = 0
n = 3
所以应用层上输入下面命令设置为0dB:
amixer -D hw:audiocodec cset name='ADC gain volume' 3
```

• 注释 2

```
LINEOUT volume计算方法：
应用层可设置范围： 0 ～31 (设置为 0 或者 1 时，就是mute)
对应实际硬件设置的范围：-43.5~0dB, step：1.5dB
换算方法:-43.5+((n-2)\*1.5)
举例 1 ，设置0dB:
-43.5+((n-2)\*1.5) = 0
n = 31
所以应用层上输入下面命令设置为0dB:
amixer -D hw:audiocodec cset name='LINEOUT volume' 31
举例 2 ，设置-6dB:
-43.5+((n-2)\*1.5) = -6
n = 27
所以应用层上输入下面命令设置为-6dB:
amixer -D hw:audiocodec cset name='LINEOUT volume' 27
```

• 注释 3

```
MIC1 gain volume计算方法：
应用层可设置范围： 0 ～7 (设置为 0 时，就是0dB)
对应实际硬件设置的范围：0dB或者15~33dB, step：3dB
换算方法:15+((n-1)*3)
举例，设置24dB:
15+((n-1)*3) = 24
n = 4
所以应用层上输入下面命令设置为24dB:
amixer -D hw:audiocodec cset name='MIC1 gain volume' 4
```

• 注释 4

```
digital volume计算方法：
应用层可设置范围： 0 ～ 63
对应实际硬件设置的范围：-73.08~0dB, step：1.16dB
换算方法:-73.08+(n*1.16)
举例 1 ，设置0dB:
-73.08+(n*1.16) = 0
n = 63
所以应用层上输入下面命令设置为0dB:
amixer -D hw:audiocodec cset name='digital volume' 63
举例 2 ，设置-5.8dB:
-73.08+(n*1.16) = -5.8
```

```
n = 58
所以应用层上输入下面命令设置为-5.8dB:
amixer -D hw:audiocodec cset name='digital volume' 58
```

通路设置举例：

1. 播放通路

```
通过Speaker播放，差分输出：
amixer -D hw:audiocodec cset name='External Speaker Switch' 1
amixer -D hw:audiocodec cset name='digital volume' 63
amixer -D hw:audiocodec cset name='LINEOUT volume' 25
amixer -D hw:audiocodec cset name='Right LINEOUT Mux' 1
```

2. 录音通路

```
通过模拟MIC1, MIC2录音：
amixer -D hw:audiocodec cset name='Left Input Mixer MIC1 Boost Switch' 1
amixer -D hw:audiocodec cset name='Right Input Mixer MIC2 Boost Switch' 1
amixer -D hw:audiocodec cset name='MIC1 gain volume' 4
amixer -D hw:audiocodec cset name='MIC2 gain volume' 4
```

3. 常用AEC通路

```
有两种AEC回路方式，具体看硬件如何设计
1)外部AEC
MIC1,MIC2录音；MIC3作为AEC,外部SPKP/N连接到MIC3.
amixer -D hw:audiocodec cset name='Left Input Mixer MIC1 Boost Switch' 1
amixer -D hw:audiocodec cset name='Right Input Mixer MIC2 Boost Switch' 1
amixer -D hw:audiocodec cset name='Xadc Input Mixer MIC3 Boost Switch' 1
amixer -D hw:audiocodec cset name='MIC1 gain volume' 4
amixer -D hw:audiocodec cset name='MIC2 gain volume' 4
amixer -D hw:audiocodec cset name='MIC3 gain volume' 0
2)内部AEC(可省去外部AEC电路)
MIC1,MIC2录音；MIC3作为AEC,使能内部DACL到MIC3的通路.
amixer -D hw:audiocodec cset name='Left Input Mixer MIC1 Boost Switch' 1
amixer -D hw:audiocodec cset name='Right Input Mixer MIC2 Boost Switch' 1
amixer -D hw:audiocodec cset name='Xadc Input Mixer DACL Switch' 1
amixer -D hw:audiocodec cset name='MIC1 gain volume' 4
amixer -D hw:audiocodec cset name='MIC2 gain volume' 4
```

### 2.14.5 Daudio

硬件特性

• 三路I2S/PCM,可用于蓝牙通话，语音采集，数字功放；

• 支持主从模式

- 支持Left-justified,Right-justified,Standar mode I2S,PCM mode


- 支持i2s,pcm协议格式配置
- 支持mono和stereo模式，最高支持 8 通道
- 支持同时playback和record(全双工模式)
- 支持8~192KHz采样率
- 支持16,24,32bit采样精度
- 支持 3 路MCLK输出

#### 2.14.5.1内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner Digital Audio Support
```

#### 2.14.5.2 sys_config配置

```
[snddaudio0]
snddaudio0_used = 0
daudio_master = 4
audio_format = 1
signal_inversion = 1
[daudio0]
daudio0_used = 0
slot_width_select = 32
pcm_lrck_period = 128
msb_lsb_first = 0
sign_extend = 0
frametype = 0
mclk_div = 1
tdm_config = 1
tx_data_mode = 0
rx_data_mode = 0
```

snddaudio0配置，即daudio0 machine驱动的相关配置

| snddaudio配置    | snddaudio配置说明                                            |
| ---------------- | ------------------------------------------------------------ |
| snddaudio0_used  | 是否使用snddaudio驱动。 0 ：不使用； 1 ：使用                |
| daudio_master    | 1: SND_SOC_DAIFMT_CBM_CFM(codec clk & FRM<br/>master),即daudio接口作为slave, codec作为master<br/>2: SND_SOC_DAIFMT_CBS_CFM(codec clk slave &<br/>FRM master),一般不用<br/>3: SND_SOC_DAIFMT_CBM_CFS(codec clk master &<br/>frame slave),一般不用<br/>4: SND_SOC_DAIFMT_CBS_CFS(codec clk & FRM<br/>slave),即daudio接口作为master, codec作为slave |
| audio_format     | 1: SND_SOC_DAIFMT_I2S(standard i2s format)<br/>2: SND_SOC_DAIFMT_RIGHT_J(right justfied format)<br/>3: SND_SOC_DAIFMT_LEFT_J(left justfied format)<br/>4: SND_SOC_DAIFMT_DSP_A(pcm. MSB is available<br/>on 2nd BCLK rising edge after LRC rising edge)<br/>5: SND_SOC_DAIFMT_DSP_B(pcm. MSB is available<br/>on 1nd BCLK rising edge after LRC rising edge) |
| signal_inversion | 1: SND_SOC_DAIFMT_NB_NF(normal bit clock +<br/>frame)<br/>2: SND_SOC_DAIFMT_NB_IF(normal BCLK + inv<br/>FRM)<br/>3: SND_SOC_DAIFMT_IB_NF(invert BCLK + nor FRM)<br/>4: SND_SOC_DAIFMT_IB_IF(invert BCLK + FRM) |

daudio0配置，即daudio0 platform驱动的相关配置

| daudio配置        | daudio配置说明                                               |
| ----------------- | ------------------------------------------------------------ |
| daudio0_used      | 是否使用daudio驱动。 0 ：不使用； 1 ：使用                   |
| slot_width_select | 支持8bit, 16bit, 32bit宽度                                   |
| pcm_lrck_period   | 一般可配置16/32/64/128/256个bclk                             |
| msb_lsb_first     | 0: msb first; 1: lsb first                                   |
| sign_extend       | 0: zero pending; 1: sign extend                              |
| frametype         | 0: short frame = 1 clock width; 1: long frame = 2<br/>clock width |
| mclk_div          | 0: not output(normal setting this);<br/>1/2/4/6/8/12/16/24/32/48/64/96/128/176/192:给外部<br/>codec提供时钟，频率是pll_audio/mclk_div |
| tdm_config        | 0: pcm mode; 1: i2s mode                                     |
| tx_data_mode      | 0: 16bit linear PCM;1: reserved;2: 8bit u-law;3: 8bit<br/>a-law |
| rx_data_mode      | 0: 16bit linear PCM;1: reserved;2: 8bit u-law;3: 8bit<br/>a-law |

注意事项：

- daudio machine驱动的配置(snddaudio),一般来说还需要配置codec name以及codec
  dai name


1. 例如daudio0使用了AC108作为外挂codec:

   ```
   [snddaudio0]
   snddaudio0_used = 1
   sunxi,snddaudio-codec = "ac108.1-003b";
   sunxi,snddaudio-codec-dai = "ac108-pcm0";
   daudio_master = 4
   audio_format = 1
   signal_inversion = 2
   [daudio0]
   daudio0_used = 1
   slot_width_select = 32
   pcm_lrck_period = 128
   msb_lsb_first = 0
   sign_extend = 0
   frametype = 0
   mclk_div = 1
   tdm_config = 1
   tx_data_mode = 0
   rx_data_mode = 0
   注意名称需要与codec驱动中配置的名称一致，如ac108驱动，路径：
   linux-4.9/sound/soc/codecs/ac108.c
   代码中snd_soc_register_codec注册codec驱动，其中codec device name为ac108.1-003b,
   codec dai name为ac108-pcm0
   ```

   

2. 例如daudio2与bluetooth模组相连(没有实际的codec驱动)，那么这时候codec

```
name, codec dai name需要配置为dummy codec,可以如下配置：
   [snddaudio2]
   snddaudio2_used = 1
   sunxi,snddaudio-codec = "snd-soc-dummy"
   sunxi,snddaudio-codec-dai = "snd-soc-dummy-dai"
   daudio_master = 1
   audio_format = 5
   signal_inversion = 2
因为驱动中解析snddaudio-codec等字段时，判断出错的时候则使用默认codec"snd-soc-dummy",
所以如果sunxi,snddaudio-codec和sunxi,snddaudio-codec-dai不配置，或者配置为空的时候，
则默认使用dummy codec:
[snddaudio2]
snddaudio2_used = 1
daudio_master = 1
audio_format = 5
signal_inversion = 2
或者
[snddaudio2]
snddaudio2_used = 1
sunxi,snddaudio-codec =
sunxi,snddaudio-codec-dai =
daudio_master = 1
audio_format = 5
signal_inversion = 2
```

### 2.14.6 Dmic

硬件特性

• 支持 8 路输入

- 支持8~48KHz采样率
- 支持16/24bit采样精度

#### 2.14.6.1内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner DMIC Support
```

#### 2.14.6.2 sys_config配置

配置如下:

```
[dmic]
dmic_used = 0
[snddmic]
snddmic_used = 0
```

dmic配置，即platform驱动的相关配置

```
dmic配置 dmic配置说明
dmic_used 是否使用dmic驱动。 0 ：不使用； 1 ：使用
```

snddmic配置，即machine驱动的相关配置

```
snddmic配置 snddmic配置说明
snddmic_used 是否使用snddmic驱动。 0 ：不使用； 1 ：使用
```

- sys_config中不需要配置codec驱动相关信息

因为machine驱动代码中默认配置了”dmic-codec”作为codec驱动,代码路径：

```
linux-4.9/sound/soc/codecs/dmic.c
```

### 2.14.7 SPDIF

硬件特性

• 支持S/PDIF_OUT和S/PDIF_IN

- 支持mono和stereo模式(mono模式下由硬件自动拓展为stereo)
- 输出支持22.05kHz, 24kHz, 32kHz, 44.1kHz, 48kHz, 88.2kHz, 96kHz, 176.4kHz,
  192kHz采样率
- 输入支持44.1KHz,48KHz采样率
- 输出和输入支持16bit,24bit采样精度

#### 2.14.7.1内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner SPDIF Support
```

#### 2.14.7.2 sys_config配置

```
[sndspdif]
sndspdif_used = 0
[spdif]
spdif_used = 0
```

spdif配置，即platform驱动的相关配置

```
spdif配置 spdif配置说明|
spdif_used 是否使用spdif驱动。 0 ：不使用； 1 ：使用|
```

sndspdif配置，即machine驱动的相关配置

```
sndspdif配置sndspdif配置说明
```

sndspdif_used是否使用sndspdif驱动。 0 ：不使用； 1 ：使用

- sys_config中不需要配置codec驱动相关信息


因为machine驱动代码中默认配置了”spdif-utils”作为codec驱动,代码路径：

```
linux-4.9/sound/soc/sunxi/spdif-utils.c
```

### 2.14.8 MAD

硬件特性

- 支持三路I2S，一路DMIC PCM音频传输接口，时分复用，固定16bit
- 支持16KHz,48KHz采样率
- 支持基于能量识别的语音检测模块LPSD
- 支持一块128KB的SRAM,可用于保存音频数据

#### 2.14.8.1内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner Mad Support
```

#### 2.14.8.2 sys_config配置

```
[mad]
mad_used = 1
lpsd_clk_src_cfg = 0
standby_sram_io_type = 1
```

| mad配置  | mad配置说明                             |
| -------- | --------------------------------------- |
| mad_used | 是否使用mad驱动。 0 ：不使用； 1 ：使用 |

#### 2.14.8.3 mixer控件说明

1. mad绑定到I2S

| 控件名称                             | 功能                         | 数值                      |
| ------------------------------------ | ---------------------------- | ------------------------- |
| daudio bind mad Function             | 是否绑定MAD功能              | 0:不绑定; 1:绑定          |
| lpsd channel sel Function            | 选择作为能量唤醒的通道       | 0:通道0; 1:通道1;         |
|                                      |                              | 如此类推，最高可指定通道7 |
| mad_standby channel sel<br/>Function | 设定休眠时mad录音通<br/>道数 | 0:表示使用实际录音通道数; |

2. mad绑定到dmic

| 控件名称                             | 功能                    | 数值                                                         |
| ------------------------------------ | ----------------------- | ------------------------------------------------------------ |
| dmic bind mad Function               | 是否绑定MAD功能         | 0:不绑定; 1:绑定                                             |
| lpsd channel sel Function            | 选择作为能量唤醒的通道  | 0:通道0; 1:通道1;                                            |
|                                      |                         | 如此类推，最高可指定通道 7                                   |
| mad_standby channel sel<br/>Function | 设定休眠时mad录音通道数 | 0:表示使用实际录音通道数; <br/>1:表示只录制两通道<br/>2:表示只录制四通道 |

#### 2.14.8.4使用说明

固件上的配置，只要修改sys_config以及内核配置即可。

应用上需要使能MAD相关的mixer control。

I2S设置举例，例如使用的是AC108:

```
mad使能,绑定mad到daudio中
amixer -Dhw:sndac1081003b cset name='daudio bind mad Function' 1
设置通道 0 作为唤醒通道
amixer -Dhw:sndac1081003b cset name='lpsd channel sel Function' 0
设定mad standby时,录音的通道数
amixer -Dhw:sndac1081003b cset name='mad_standby channel sel Function' 2
```

DMIC设置举例:

```
amixer -Dhw:snddmic cset name='dmic bind mad Function' 1
amixer -Dhw:snddmic cset name='lpsd channel sel Function' 0
amixer -Dhw:snddmic cset name='mad_standby channel sel Function' 0
```

然后应用正常进行录音即可，如果需要进入休眠，有下面几点必须实现的:

1. 暂定录音、播放。snd_pcm_pause将playback,capture均暂停;


2. 设置wakeup_count。更新当前唤醒次数;
3. 进入休眠。写mem到/sys/power/state即可;

Tina SDK中有一个能量唤醒demo可供参考。

make menuconfig选中mad-demo软件包

```
Allwinner --->
	<*> mad-demo
```

执行mad-ac108-demo，默认配置(脚本/usr/bin/mad-ac108-demo上设定了默认配置):

• 使用通道 0 作为唤醒通道;

- 录制 4 通道，16bit, 16K;
- 每次录音5s后进入休眠，可通过语音能量唤醒;

执行mad-dmic-demo，默认配置(脚本/usr/bin/mad-dmic-demo上设定了默认配置):

• 使用通道 0 作为唤醒通道;

- 录制 4 通道，16bit, 16K;
- 每次录音5s后进入休眠，可通过语音能量唤醒;

如果想查看录音数据，可以增加 dump 参数, 例如mad-dmic-demo dump, 录音文件保存
在/mnt/UDISK/目录下。

#### 2.14.8.5能量唤醒阈值参数

能量唤醒模块lpsd，识别能量主要有两个方向，瞬时能量和累计能量（前者比如是关门声，后者
比如是不断说话）能量检测参数配置均在/sys/module/sunxi_mad/parameters/目录下

lpsd_rrun和lpsd_rstop的推荐值：

| lpsd_rrun | lpsd_rstop |
| --------- | ---------- |
| 77        | 88         |
| 77        | 108        |
| 77        | 128        |
| 77        | 148        |

1. 瞬时能量检测参数，主要是lpsd_rrun和lpsd_rstop。


- 一般我们只对stop值进行修改;
- 如果录音数据经常缺少唤醒词的第一个字，则可以尝试降低stop值，可以有效提高唤醒词数据
  的完整性。但同时会提高误唤醒率，环境噪音也会很容易触发能量检测，唤醒系统;
- 如果想要降低误唤醒率(环境噪音造成唤醒)，则可以尝试提高stop值。同样的，这会导致一些
  唤醒词录音数据不完整，例如一些音量较低，音调较低的语料;
- 唤醒词识别率以及误唤醒率无法同时兼得，客户需要根据实际需求、场景，权衡配置参数;

2. 累积能量检测参数，主要是lpsd_th。

• 我们建议使用默认值 1200 。建议修改范围50~1200;

#### 2.14.8.6注意事项

1. MAD绑定动作，需要在应用打开声卡前就设置好;

2. 应用操作上的一些要求，具体请查看《MAD使用说明》章节;

3. 如果读取wakeup_count时一直阻塞，说明当前仍有wake_lock处于激活状态，例如usb
   线连接着PC，usb驱动会保持一个wake_lock，不让系统进入休眠，所以需要拔掉usb或
   者连接到usb适配器上，或者改动代码，去掉usb驱动中wake_lock的使用;

### 2.14.9 VAD.

VAD是基于MAD实现的，可以通过内部AudioCodec的ADC采集音频数据，并作能量唤醒。
由于硬件上MAD功能只能用于I2S或者DMIC，内部codec无法直接关联到MAD，因此通
过I2S作为音频数据的桥梁，实现了VAD功能，使得模拟MIC也可以利用MAD功能作能量唤
醒。

VAD完整的数据通路：

```
ADC RxFiFo ---> I2S TxFiFo ---> I2S RxFiFo ---> MAD SRAM ---> MEM
```

#### 2.14.9.1内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
[*] Allwinner I2S PCM DMA MAP Support
<*> Allwinner Mad Support
<*> Allwinner Sun8iw18 Codec Support
<*> Allwinner Digital Audio Support
```

#### 2.14.9.2 sys_config配置

需要使能MAD配置:

```
[mad]
mad_used = 1
lpsd_clk_src_cfg = 0
standby_sram_io_type = 1
```

| mad配置  | mad配置说明                             |
| -------- | --------------------------------------- |
| mad_used | 是否使用mad驱动。 0 ：不使用； 1 ：使用 |

#### 2.14.9.3 mixer控件说明

| 控件名称                             | 功能                    | 数值                                                         |
| ------------------------------------ | ----------------------- | ------------------------------------------------------------ |
| codec I2S Port                       | 指定VAD使用的I2S        | 0:不适用; 1:使用I2S0;<br/> 2:使用I2S1;<br/>3:使用I2S2        |
| sndcodec bind mad Function           | 是否绑定MAD功能         | 0:不绑定; 1:绑定                                             |
| lpsd channel sel Function            | 选择作为能量唤醒的通道  | 0:通道0; 1:通道1;                                            |
|                                      |                         | 如此类推，最高可指定<br/>通道 7                              |
| mad_standby channel sel<br/>Function | 设定休眠时mad录音通道数 | 0:表示使用实际录音通<br/>道数; 1:表示只录制两<br/>通道2:表示只录制四通道 |

注意：

对于控件“codec I2S Port”，需要指定实际没有使用(sys_config没有使能的)的一路I2S。设置举例：

- 实际没有使用I2S0(sys_config中snddaudio0,audio0均没有配置),那么这里可以设置为1,表示VAD使用I2S0;
- 实际没有使用I2S1(sys_config中snddaudio1,audio1均没有配置),那么这里可以设置为2,表示VAD使用I2S1;

设置举例:


```
amixer -Dhw:audiocodec cset name='codec I2S Port' 2
amixer -Dhw:audiocodec cset name='sndcodec bind mad Function' 1
amixer -Dhw:audiocodec cset name='lpsd channel sel Function' 0
amixer -Dhw:audiocodec cset name='mad_standby channel sel Function' 0
```

#### 2.14.9.4使用说明

VAD的使用与MAD类似。

固件上的配置，只要修改sys_config以及内核配置即可。

应用上除了打开内部audiocodec的录音通路之外，还需要下面一些配置：

```
vad需要使用一路i2s,这里指定使用i2s1
amixer -Dhw:audiocodec cset name='codec I2S Port' 2
使能mad,绑定mad到audiocodec中
amixer -Dhw:audiocodec cset name='sndcodec bind mad Function' 1
设置通道 0 作为唤醒通道
amixer -Dhw:audiocodec cset name='lpsd channel sel Function' 0
```

然后应用正常进行录音即可，如果需要进入休眠，有下面几点必须实现的：

1. 暂定录音、播放。snd_pcm_pause将playback,capture均暂停
2. 设置wakeup_count。更新当前唤醒次数
3. 进入休眠。写mem到/sys/power/state即可

Tina SDK中有一个能量唤醒demo可供参考make menuconfig选中mad-demo软件包

```
Allwinner --->
	<*> mad-demo
```

执行vad-demo，默认配置(脚本/usr/bin/vad-demo上设定了默认配置):

• 使用I2S1

- 录制 2 通道，16bit, 16K
- 每次录音5s后进入休眠，可通过语音能量唤醒

如果想查看录音数据，可以执行vad-demo dump,录音文件保存在/mnt/UDISK/vad-test.wav能量唤醒阈值的调整，可以参考《能量唤醒阈值参数》


#### 2.14.9.5注意事项

VAD同样需要注意MAD注意事项章节中提到的几点。

另外需要注意，VAD隐式使用了一路I2S，所以硬件上需要保留一路I2S，并且sys_config中不能使能该I2S配置

### 2.14.10标案音频测试方法.

该章节主要介绍在标案上进行播歌，录音的测试命令

#### 2.14.10.1播放

```
通过Speaker播放
amixer -D hw:audiocodec cset name='External Speaker Switch' 1
amixer -D hw:audiocodec cset name='Right LINEOUT Mux' 1
aplay -Dhw:audiocodec /mnt/UDISK/1KHz_0dB_16000.wav
```

#### 2.14.10.2录音

```
通过MIC1,MIC2录制两通道
amixer -D hw:audiocodec cset name='Left Input Mixer MIC1 Boost Switch' 1
amixer -D hw:audiocodec cset name='Right Input Mixer MIC2 Boost Switch' 1
amixer -D hw:audiocodec cset name='MIC1 boost volume' 4
amixer -D hw:audiocodec cset name='MIC2 boost volume' 4
arecord -Dhw:audiocodec -f S16_LE -r 16000 -c 2 /tmp/test.wav
```

#### 

## 2.20 V853音频接口

V853包含多个音频模块，分别是内置AudioCodec,I2S0,I2S1,DMIC。


### 2.20.1 时钟源.

V853中，音频模块的时钟源来自pll_audio0。

pll_audio0可输出22.5792M和24.576M频率的时钟，分别支持44.1k系列、48k系列的播放录音，但无法同时输出。

![图2-48: V853时钟源](https://cdn.staticaly.com/gh/DongshanPI/Docs-Photos@master/Tina-Sdk/OpenRemoved_Tina_Linux_Audio_development_Guide-2-48.jpg)

### 2.20.2 代码结构

```
linux-4.9/sound/soc/sunxi_v2/
├── snd_sun8iw21_codec.c // codec驱动
├── snd_sun8iw21_codec.h
├── snd_sunxi_aaudio.c
├── snd_sunxi_common.c
├── snd_sunxi_common.h
├── snd_sunxi_daudio.c //daudio platform驱动
├── snd_sunxi_daudio.h
├── snd_sunxi_dmic.c // dmic platform驱动
├── snd_sunxi_dmic.h
├── snd_sunxi_log.h
├── snd_sunxi_mach.c
├── snd_sunxi_mach.h
├── snd_sunxi_mach_utils.c
├── snd_sunxi_mach_utils.h
├── snd_sunxi_pcm.c
├── snd_sunxi_pcm.h
├── snd_sunxi_rxsync.c
├── snd_sunxi_rxsync.h
├── snd_sunxi_txhub.c
└── snd_sunxi_txhub.h
```

### 2.20.3 AudioCodec

#### 2.20.3.1硬件特性

• 一路DAC

- 支持16bit,20bit有效采样精度
- 支持8KHz~192KHz采样率
- 两路ADC
- 支持16bit,20bit有效采样精度
- 支持8KHz~48KHz采样率
- 一路模拟输出：
- 一路立体声输出LINEOUT，支持single/differ
- 两路模拟输入：MIC1,MIC2
- MIC支持single/differ,
- 支持同时playback和record(全双工模式)
- DAC及ADC均支持DRC
- 支持mono模式，最高支持 2 通道

#### 2.20.3.2内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support V2 --->
<*> Allwinner AAUDIO support
<*> Allwinner DAUDIO Support
<*> Allwinner Function Components
<*> Components Rx Sync
```

#### 2.20.3.3 DTS配置.

**2.20.3.3.1 DeviceTree 配置说明** 设备树为芯片平台的模块配置，面对芯片特性进行配
置，设备树文件的路径为：lichee/linux-4.9/arch/arm/boot/dts/sun8iw21p1.dtsi

```
codec:codec@0x02030000 {
#sound-dai-cells = <0>;
compatible = "allwinner,sunxi-snd-codec";
reg = <0x0 0x02030000 0x0 0x34C>;
clocks = <&clk_pll_audio>,
```

```
<&clk_codec_dac>,
<&clk_codec_adc>;
status = "disabled";
};
codec_plat:codec_plat {
#sound-dai-cells = <0>;
compatible = "allwinner,sunxi-snd-aaudio";
playback_cma = <128>;
capture_cma = <128>;
tx_fifo_size = <128>;
rx_fifo_size = <128>;
dac_txdata = <0x02030020>;
adc_txdata = <0x02030040>;
status = "disabled";
};
codec_mach:codec_mach {
compatible = "allwinner,sunxi-snd-mach";
soundcard-mach,name = "audiocodec";
soundcard-mach,pin-switches = "MIC1", "MIC2", "LINEIN",
"LINEOUT", "SPK";
soundcard-mach,routing = "MIC1_PIN", "MIC1",
"MIC2_PIN", "MIC2",
"LINEINL_PIN", "LINEIN",
"LINEINR_PIN", "LINEIN",
"LINEOUT", "LINEOUTL_PIN",
"SPK", "LINEOUTL_PIN";
status = "disabled";
soundcard-mach,cpu {
/* pll freq = 24.576M or 22.5792M * pll-fs */
soundcard-mach,pll-fs = <1>;
sound-dai = <&codec_plat>;
};
soundcard-mach,codec {
sound-dai = <&codec>;
};
};
```

配置项说明（仅对常用项进行展开）：

AudioCodec模块由 3 个设备树节点构建。

1. ASoC层codec: codec

<center>表2-128: AudioCodec codec节点配置项(linux4.9)</center>

| 配置项名称       | 配置项说明                             |
| ---------------- | -------------------------------------- |
| #sound-dai-cells | machine层检测codec和platform节点的标志 |
| reg              | 设置audiocodec寄存器起始地址和地址长度 |
| clocks           | 设置audiocodec所需的时钟源和模块时钟   |

2. ASoC层platform: codec_plat

<center>表2-129: AudioCodec codec_plat节点配置项(linux4.9)</center>

| 配置项名称       | 配置项说明                                                   |
| ---------------- | ------------------------------------------------------------ |
| #sound-dai-cells | machine层检测codec和platform节点的标志                       |
| playback_cma     | 设置播放流DMA申请的size大小，必须为(2ˆn)Kbyte，默认 128      |
| capture_cma      | 设置录音流DMA申请的size大小，必须为(2ˆn)Kbyte，默认 128      |
| tx_fifo_size     | 设置播放流snd_pcm_runtime的fifo_size大小，用于声卡硬件参数<br/>限定，默认 128 |
| rx_fifo_size     | 设置录音流snd_pcm_runtime的fifo_size大小，用于声卡硬件参数<br/>限定，默认 128 |
| dac_txdata       | 设置播放流DMA搬运地址（audiocodec模块的tx_fifo寄存器地址）   |
| adc_txdata       | 设置录音流DMA搬运地址（audiocodec模块的rx_fifo寄存器地址）   |

3. ASoC层machine: codec_mach

<center>表2-130: AudioCodec codec_mach节点配置项(linux4.9)</center>

| 配置项名称          | 配置项说明                                                   |
| ------------------- | ------------------------------------------------------------ |
| soundcard-<br/>mach | machine层配置前缀                                            |
| name                | 声卡名字                                                     |
| pin-switches        | 用于定义模块接口开关（需参考驱动代码dapm进行设定）           |
| routing             | 用于定义模块接口开关所链接的dapm通路（需参考驱动代码dapm进<br/>行设定） |
| cpu                 | machine层所绑定的cpu节点（即platform层），用sound-dai属<br/>性指定节点 |
| codec               | machine层所绑定的codec节点（即codec层），用sound-dai属<br/>性指定节点 |
| pll-fs              | 指定模块时钟源频率（24.576M or 22.5792M * pll-fs）           |

**2.20.3.3.2 board.dts板级配置说明** 

board.dts用于保存板级平台设备差异化的信息的补充，面对 **板型特性** 进行配置，其配置信息会覆盖device tree默认配置信息，board.dts文件的路径为：/device/config/chips/v853/configs/{BOARD}/board.dts

```
&codec {
/* external-avcc; */
/* avcc-supply = <&reg_aldo1>; */
avcc-vol = <1800000>; /* uv */
lineout_vol = <31>;
mic1gain = <31>;
mic2gain = <31>;
```

```
adcdelaytime = <0>;
/* lineout-single; */
/* mic1-single; */
/* mic2-single; */
pa_pin_max = <1>; /* set pa */
pa_pin_0 = <&pio PH 4 1 1 1 1>;
pa_pin_level_0 = <1>;
pa_pin_msleep_0 = <0>;
tx_hub_en;
rx_sync_en;
status = "okay";
};
&codec_plat {
status = "okay";
};
&codec_mach {
status = "okay";
soundcard-mach,cpu {
sound-dai = <&codec_plat>;
};
soundcard-mach,codec {
sound-dai = <&codec>;
};
};
```

配置项介绍：

<center>表2-131: AudioCodec模块板级配置项</center>

| 配置项名称             | 配置值范围                      | 配置项说明                                                   |
| ---------------------- | ------------------------------- | ------------------------------------------------------------ |
| status                 | “okay”, “disabled”              | 使能或关闭该节点驱动                                         |
| avcc-external          | 注释为false,反之为<br/>ture     | avcc电源是否为外围电路提供                                   |
| avcc-supply            | 注释,引用pmu提供的<br/>电源节点 | avcc若为外部pmu供电，可选择该项指定<br/>对应的pmu电源        |
| avcc-vol u32           | （缺省值:<br/>1800000 ）        | avcc电压值设定，单位uV，需符合实际硬<br/>件需求              |
| dvcc-external          | 注释为false,反之为<br/>ture     | dvcc电源是否为外围电路提供                                   |
| dvcc-supply            | 注释,引用pmu提供的<br/>电源节点 | dvcc若为外部pmu供电，可选择该项指定<br/>对应的pmu电源        |
| dvcc-vol               | u32（缺省值:<br/>1800000 ）     | dvcc电压值设定，单位uV，需符合实际硬<br/>件需求              |
| adc-dig-vol-<br/>(n)   | 0~255                           | adc(n)数字音量，(n)代表adc序号，从 1<br/>开始递增            |
| mic(n)_vol             | 0~31                            | mic(n)默认输入音量（增益），(n)代表<br/>mic序号，从 1 开始递增 |
| dac-dig-vol            | 0~63 dac                        | 数字总音量                                                   |
| dac-dig-vol-<br/>(n)   | 0~255                           | dac(n)数字音量，(n)代表dac序号，从 1<br/>开始递增            |
| lineout-vol            | 0~31                            | lineout默认输出音量（增益）                                  |
| hp-vol                 | 0~7                             | 耳机默认输出音量（增益）                                     |
| pa-pin-max             | u32（正常为 1 或 2 ）           | 标定外部功放芯片使能引脚数量                                 |
| pa-pin-(n)             | pio提供的引脚节点               | 指定第(n)个功放使能引脚                                      |
| pa-pin-level-<br/>(n)  | 0~1                             | 指定功放芯片使能电平                                         |
| pa-pin-<br/>msleep-(n) | u32（正常小于 200 ）            | 设置功放芯片使能所需的sleep时长，用于<br/>规避pop声，单位ms  |
| adcdelaytime           | u32（需符合IC规格）             | 设置adc录音延迟时长，单位ms                                  |
| tx-hub-en              | 注释为false,反之为<br/>ture     | 选择是否注册txhub控件                                        |
| rx-sync-en             | 注释为false,反之为<br/>ture     | 选择是否注册rxsync控件                                       |



#### 2.20.3.4 codec数据通路

```
通过Lineout播歌
Playback --> DACL --> LINEOUTL Output Select --> LINEOUTL --> LINEOUT
Playback --> DACR --> LINEOUTR Output Select --> LINEOUTR --> LINEOUT
录音
MIC1 --> MIC1 Input Select --> ADC1 Input --> ADC1 --> Capture
MIC2 --> MIC2 Input Select --> ADC2 Input --> ADC2 --> Capture
LINE-in录音
LINEINL --> ADC1 Input --> ADC1 --> Capture
LINEINR --> ADC2 Input --> ADC2 --> Capture
```

V853所有控件如下表：

<center>表2-132: amixer控件表</center>

| 控件名称                  | 功能                                              | 数值                                                         |
| ------------------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| ADC1 ADC2 swap            | 将adc1和adc2进行通道<br/>交换                     |                                                              |
| ADC1 volume               | ADC1数字音量设置                                  | 0~0xFF, 0:Mute;<br/>1~0xFF:-119.25dB~ 71.24dB,<br/>0.75dB/step,默认0xA0=0dB |
| ADC2 volume               | ADC2数字音量设置                                  | 0~0xFF, 0:Mute;<br/>1~0xFF:-119.25dB~ 71.24dB,<br/>0.75dB/step,默认0xA0=0dB |
| ADCDRC                    | 开启ADC DRC功能                                   |                                                              |
| ADCHPF                    | 开启adc hpf功能                                   |                                                              |
| DAC volume                | DACL,DACR音量设置                                 | 0~0xFF, 0:Mute;<br/>1~0xFF:-119.25dB~ 71.24dB,<br/>0.75dB/step,默认0xA0=0dB |
| DACDRC                    | 开启dac drc功能                                   |                                                              |
| DACHPF                    | 开启dac hpf功能                                   |                                                              |
| LINEIN Switch             | 是否使能ADC->LINEIN<br/>的通路                    | 0:关闭; 1:使能                                               |
| LINEINL gain<br/>volume   | LINEINL增益                                       | 0:0dB; 1:6dB                                                 |
| LINEINR gain<br/>volume   | LINEINR增益                                       | 0:0dB; 1:6dB                                                 |
| LINEOUT Output<br/>Select | Lineout输出选择                                   | 0:单端; 1:差分                                               |
| LINEOUT Switch            | 是否使能Lineout通路                               | 0:关闭; 1:使能                                               |
| LINEOUT volume            | Lineout音量设置                                   | 0~31,表示-43.5~0dB                                           |
| MIC1 Input Select         | MIC1输入模式                                      | 0:差分输入; 1:单端输入                                       |
| MIC1 Switch               | 是否使能ADC1->MIC1的<br/>通路                     | 0:关闭; 1:使能                                               |
| MIC1 gain volume          | MIC1增益                                          | 0~31,表示0~36dB, 0:0dB,1~3:6dB,<br/>4~31:9~36dB, 1dB/step    |
| MIC2 Input Select         | MIC2输入模式                                      | 0:差分输入; 1:单端输入                                       |
| MIC2 Switch               | 是否使能ADC2->MIC2的<br/>通路                     | 0:关闭; 1:使能                                               |
| MIC2 gain volume          | MIC2增益                                          | 0~31,表示0~36dB, 0:0dB,1~3:6dB,<br/>4~31:9~36dB, 1dB/step    |
| SPK Switch                | 是否使能Speaker通路(使<br/>用功放)                | 0:关闭; 1:使能                                               |
| digital volume            | 数字端音量设置                                    | 0~63,表示-73.08~0dB                                          |
| rx sync mode              | 使能同步录音（和其它开启<br/>rx sync mode的声卡） |                                                              |
| tx hub mode               | 使能同源播放（和其它开启<br/>tx hub mode的声卡）  |                                                              |


### 2.20.4 Daudio

#### 2.20.4.1硬件特性

• 两路I2S/PCM,可用于蓝牙通话，语音采集，数字功放；

• 支持主从模式

- 支持Left-justified,Right-justified,Standar mode I2S,PCM mode
- 支持i2s,pcm协议格式配置
- 支持同时playback和record(全双工模式)
- 支持8~192KHz采样率
- 支持16,24,32bit采样精度
- 支持 2 路MCLK输出

#### 2.20.4.2内核配置

```
Device Drivers --->
    <*> Sound card support --->
        <*> Advanced Linux Sound Architecture --->
            <*> ALSA for SoC audio support --->
                Allwinner SoC Audio support V2 --->
                <*> Allwinner AAUDIO support
                <*> Allwinner DAUDIO Support
                <*> Allwinner Function Components
                <*> Components Rx Sync
```

#### 2.20.4.3 DTS配置.

**2.20.4.3.1 DeviceTree 配置说明** 设备树为芯片平台的模块配置，面对芯片特性进行配置，设备树文件的路径为：lichee/linux-4.9/arch/arm/boot/dts/sun8iw21p1.dtsi

```
daudio0_plat:daudio0_plat@0x02032000 {
#sound-dai-cells = <0>;
compatible = "allwinner,sunxi-snd-plat-daudio";
reg = <0x0 0x02032000 0x0 0x7c>;
clocks = <&clk_pll_audio>, <&clk_i2s0>;
playback_cma = <128>;
capture_cma = <128>;
tx_fifo_size = <128>;
rx_fifo_size = <128>;
status = "disabled";
};
daudio0_mach:daudio0_mach{
compatible = "allwinner,sunxi-snd-mach";
soundcard-mach,format = "i2s";
soundcard-mach,name = "snddaudio0";
```

```
status = "disabled";
soundcard-mach,cpu {
sound-dai = <&daudio0_plat>;
};
soundcard-mach,codec {
};
};
daudio1_plat:daudio1_plat@0x02033000 {
#sound-dai-cells = <0>;
compatible = "allwinner,sunxi-snd-plat-daudio";
reg = <0x0 0x02033000 0x0 0x7c>;
clocks = <&clk_pll_audio>, <&clk_i2s1>;
playback_cma = <128>;
capture_cma = <128>;
tx_fifo_size = <128>;
rx_fifo_size = <128>;
status = "disabled";
};
daudio1_mach:daudio1_mach{
compatible = "allwinner,sunxi-snd-mach";
soundcard-mach,format = "i2s";
soundcard-mach,name = "snddaudio1";
status = "disabled";
soundcard-mach,cpu {
sound-dai = <&daudio1_plat>;
};
soundcard-mach,codec {
};
};
```

配置项说明（仅对常用项进行展开）：

I2S/PCM模块由 2 个或 3 个设备树节点构建。

1. ASoC层codec:非必须节点，若无，则绑定虚拟codec节点。
2. ASoC层platform: daudio(n)_plat

<center>表2-133: I2S/PCM daudio(n)_plat节点配置项(linux4.9)</center>

| 配置项名称       | 配置项说明                                                   |
| ---------------- | ------------------------------------------------------------ |
| #sound-dai-cells | machine层检测codec和platform节点的标志                       |
| reg              | 设置I2S/PCM寄存器起始地址和地址长度                          |
| clocks           | 设置I2S/PCM所需的时钟源和模块时钟                            |
| playback_cma     | 设置播放流DMA申请的size大小，必须为(2ˆn)Kbyte，默认 128      |
| capture_cma      | 设置录音流DMA申请的size大小，必须为(2ˆn)Kbyte，默认 128      |
| tx_fifo_size     | 设置播放流snd_pcm_runtime的fifo_size大小，用于声卡硬件参数<br/>限定，默认 128 |
| rx_fifo_size     | 设置录音流snd_pcm_runtime的fifo_size大小，用于声卡硬件参数<br/>限定，默认 128 |

3. ASoC层machine: daudio(n)_mach

<center>表2-134: I2S/PCM daudio(n)_mach节点配置项(linux4.9)</center>

| 配置项名称     | 配置项说明                                                   |
| -------------- | ------------------------------------------------------------ |
| soundcard-mach | machine层配置前缀                                            |
| name           | 声卡名字                                                     |
| cpu            | machine层所绑定的cpu节点（即platform层），用<br/>sound-dai属性指定节点 |
| codec          | machine层所绑定的codec节点（即codec层），用<br/>sound-dai属性指定节点。若该子节点下无sound-dai属性，<br/>即代表使用虚拟codec，用于辅助生成声卡 |
| pll-fs         | 指定模块时钟源频率（24.576M or 22.5792M * pll-fs）           |



**说明**

- **_daudio(n)_plat_** **代表** **_daudio0_plat, daudio1_plat,_** **···（取决于芯片规格）；**
- **_daudio(n)_mach_** **代表** **_daudio0_mach, daudio1_mach,_** **···（取决于芯片规格）；**

**2.20.4.3.2 board.dts板级配置说明** board.dts用于保存板级平台设备差异化的信息的补
充，面对 **板型特性** 进行配置，其配置信息会覆盖device tree默认配置信息，board.dts文件的
路径为：/device/config/chips/v853/configs/{BOARD}/board.dts

```
&daudio0_plat {
tdm_num = <0>;
tx_pin = <0>;
rx_pin = <0>;
/* pinctrl_used; */
/* pinctrl-names= "default","sleep"; */
/* pinctrl-0 = <&daudio0_pins_a>; */
/* pinctrl-1 = <&daudio0_pins_b>; */
tx_hub_en;
rx_sync_en;
status = "okay";
};
&daudio0_mach {
soundcard-mach,format = "i2s";
soundcard-mach,frame-master = <&daudio0_cpu>;
soundcard-mach,bitclock-master = <&daudio0_cpu>;
/* soundcard-mach,frame-inversion; */
/* soundcard-mach,bitclock-inversion; */
soundcard-mach,slot-num = <2>;
soundcard-mach,slot-width = <32>;
status = "okay";
daudio0_cpu: soundcard-mach,cpu {
sound-dai = <&daudio0_plat>;
soundcard-mach,pll-fs = <1>; /* pll freq = 24.576M or 22.5792M * pll-fs */
soundcard-mach,mclk-fs = <0>; /* mclk freq = pcm rate * mclk-fs */
};
daudio0_codec: soundcard-mach,codec {
```

```
};
};
&daudio1_plat {
tdm_num = <1>;
tx_pin = <0>;
rx_pin = <0>;
/* pinctrl_used; */
/* pinctrl-names= "default","sleep"; */
/* pinctrl-0 = <&daudio1_pins_a>; */
/* pinctrl-1 = <&daudio1_pins_b>; */
tx_hub_en;
rx_sync_en;
status = "disabled";
};
&daudio1_mach {
soundcard-mach,format = "i2s";
soundcard-mach,frame-master = <&daudio1_cpu>;
soundcard-mach,bitclock-master = <&daudio1_cpu>;
/* soundcard-mach,frame-inversion; */
/* soundcard-mach,bitclock-inversion; */
soundcard-mach,slot-num = <2>;
soundcard-mach,slot-width = <32>;
status = "disabled";
daudio1_cpu: soundcard-mach,cpu {
sound-dai = <&daudio1_plat>;
soundcard-mach,pll-fs = <1>; /* pll freq = 24.576M or 22.5792M * pll-fs */
soundcard-mach,mclk-fs = <0>; /* mclk freq = pcm rate * mclk-fs */
};
daudio1_codec: soundcard-mach,codec {
};
};
```

配置项介绍：

<center>表2-135: I2S/PCM模块板级配置项</center>

| 配置项名称              | 配置值范围                                    | 配置项说明                                                   |
| ----------------------- | --------------------------------------------- | ------------------------------------------------------------ |
| status                  | “okay”, “disabled”                            | 使能或关闭该节点驱动                                         |
| tdm-num                 | 0~3                                           | 指定I2S序号，需和daudio(n)_plat的(n)<br/>对应                |
| tx-pin                  | 0~3                                           | 指定I2S所使用的DOUT引脚序号                                  |
| rx-pin                  | 0~3                                           | 指定I2S所使用的DIN引脚序号                                   |
| tx-hub-en               | 注释为false,反之为<br/>ture                   | 选择是否注册txhub控件                                        |
| rx-sync-en              | 注释为false,反之为<br/>ture                   | 选择是否注册rxsync控件                                       |
| format                  | “i2s”,“right_j”,“left_j”,<br/>“dsp_a”,“dsp_b” | 选择tdm协议格式                                              |
| frame-master            | cpu子节点，codec子<br/>节点                   | 选择LRCK信号主模式                                           |
| bitclock-<br/>master    | cpu子节点，codec子<br/>节点                   | 选择BCLK信号主模式                                           |
| frame-<br/>inversion    | 注释为false,反之为<br/>ture                   | LRCK信号是否翻转                                             |
| bitclock-<br/>inversion | 注释为false,反之为<br/>ture                   | BCLK信号是否翻转                                             |
| slot-num                | 1~16                                          | slot数量（可简单理解为支持最大通道数）                       |
| slot-width              | 8, 16, 24, 32                                 | 单个slot宽度（可简单理解为支持最大数据精<br/>度）            |
| mclk-fp                 | 注释为false,反之为<br/>ture                   | ture: mclk以固定频段输出；false: mclk以<br/>采样率倍数输出   |
| mclk-fs                 | u32                                           | 固定频段：mclk = mclk-fs * 12.288M or 11.2896M<br/>采样率倍数：mclk = mclk-fs * pcm rate |

### 2.20.5 DMIC.

硬件特性

• 支持 4 路输入

- 支持8~48KHz采样率
- 支持16/24bit采样精度

#### 2.20.5.1内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support V2 --->
<*> Allwinner DMIC support
```

#### 2.20.5.2 DTS配置.

**2.20.5.2.1 DeviceTree 配置说明** 设备树为芯片平台的模块配置，面对芯片特性进行配
置，设备树文件的路径为：lichee/linux-4.9/arch/arm/boot/dts/sun8iw21p1.dtsi

```
dmic_plat:dmic_plat@0x02031000 {
#sound-dai-cells = <0>;
compatible = "allwinner,sunxi-snd-plat-dmic";
```

```
reg = <0x0 0x02031000 0x0 0x50>;
clocks = <&clk_pll_audio>, <&clk_dmic>;
capture_cma = <128>;
rx_fifo_size = <128>;
status = "disabled";
};
dmic_mach:dmic_mach{
compatible = "allwinner,sunxi-snd-mach";
soundcard-mach,name = "snddmic";
soundcard-mach,capture_only;
status = "disabled";
soundcard-mach,cpu {
sound-dai = <&dmic_plat>;
};
soundcard-mach,codec {
};
};
```

配置项说明（仅对常用项进行展开）：

DMIC模块由 2 个设备树节点构建。

1. ASoC层codec:无，绑定虚拟codec节点。
2. ASoC层platform: dmic_plat

<center>表2-136: DMIC dmic_plat节点配置项(linux4.9)</center>

| 配置项名称       | 配置项说明                                                   |
| ---------------- | ------------------------------------------------------------ |
| #sound-dai-cells | machine层检测codec和platform节点的标志                       |
| reg              | 设置DMIC寄存器起始地址和地址长度                             |
| clocks           | 设置DMIC所需的时钟源和模块时钟                               |
| capture_cma      | 设置录音流DMA申请的size大小，必须为(2ˆn)Kbyte，默认 128      |
| rx_fifo_size     | 设置录音流snd_pcm_runtime的fifo_size大小，用于声卡硬件参数<br/>限定，默认 128 |



3. ASoC层machine: dmic_mach

<center>表2-137: DMIC dmic_mach节点配置项(linux4.9)</center>

| 配置项名称          | 配置项说明                                                   |
| ------------------- | ------------------------------------------------------------ |
| soundcard-<br/>mach | machine层配置前缀                                            |
| name                | 声卡名字                                                     |
| cpu                 | machine层所绑定的cpu节点（即platform层），用sound-dai属性<br/>指定节点 |
| codec               | machine层所绑定的codec节点（即codec层），用sound-dai属性<br/>指定节点（使用虚拟codec） |
| capture_only        | 设置仅录音，不进行播放流设备创建<br/>pll-fs 指定模块时钟源频率（24.576M or 22.5792M * pll-fs） |



**2.20.5.2.2 board.dts板级配置说明** board.dts用于保存板级平台设备差异化的信息的补
充，面对 **板型特性** 进行配置，其配置信息会覆盖device tree默认配置信息，board.dts文件的
路径为：/device/config/chips/v853/configs/{BOARD}/board.dts

```
&dmic_plat {
    rx_chmap = <0x76543210>;
    data_vol = <0xB0>;
    rxdelaytime = <0>;
    pinctrl_used;
    pinctrl-names = "default","sleep";
    pinctrl-0 = <&dmic_pins_a>;
    pinctrl-1 = <&dmic_pins_b>;
    rx_sync_en;
    status = "disabled";
};
&dmic_mach {
    status = "disabled";
    soundcard-mach,cpu {
        sound-dai = <&dmic_plat>;
        soundcard-mach,pll-fs = <1>; /* pll freq = 24.576M or 22.5792M * pll-fs */
    };
    soundcard-mach,codec {
    };
};
```

配置项介绍：

<center>表2-138: DMIC模块板级配置项</center>

| 配置项名称       | 配置值范围                      | 配置项说明                                            |
| ---------------- | ------------------------------- | ----------------------------------------------------- |
| status           | “okay”, “disabled”              | 使能或关闭该节点驱动                                  |
| rx-sync-en       | 注释为false,反之为<br/>ture     | 选择是否注册rxsync控件                                |
| avcc-<br/>supply | 注释,引用pmu提供的<br/>电源节点 | avcc若为外部pmu供电，可选择该项指定对应的<br/>pmu电源 |



### 2.20.6 RX_SYNC多声卡同步功能

RX_SYNC功能用于同时使用到两个录音声卡(不同音频硬件接口)，可以保证两个声卡同时开始录音，保证延迟稳定不变。


例如内部ADC+外部ADC(使用I2S)的语音方案，它就可以保证内部ADC和I2S RX的同步性。

AudioCodec, I2S, DMIC均可以使用RX_SYNC功能，除了它们对应的驱动配置外，还需要额外配置内核,dts等地方。

#### 2.20.6.1内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support V2 --->
<*> Allwinner AAUDIO support
<*> Allwinner DAUDIO Support
<*> Allwinner Function Components
<*> Components Rx Sync
```

#### 2.20.6.2 dts配置.

在希望使用到rx_sync功能的音频模块上添加rx_sync_en = <1>字段:

```
codec:codec@2030000 {
	rx_sync_en = <0x01>;
};
dmic:dmic@2031000 {
	rx_sync_en = <0x01>;
};
daudio0:daudio@2032000 {
	rx_sync_en = <0x01>;
};
```

注意，配置了rx_sync_en字段的模块，需要都开启录音后，才会真正开始录音。

一般我们会使用multi插件将两个声卡数据合并(默认asound.conf中添加了CaptureMulti1可供参考)，arecord -DCaptureMulti1 -f S16_LE -r 16000 -c 7 /tmp/test.wav &。

### 2.20.7 标案音频测试方法.

该章节主要介绍在标案上进行播歌，录音的测试命令


#### 2.20.7.1播放

```
通过LINEOUT->Speaker播放
amixer -Dhw:audiocodec cset name='LINEOUTL Output Select' 1
amixer -Dhw:audiocodec cset name='LINEOUTR Output Select' 1
amixer -Dhw:audiocodec cset name='LINEOUT Switch' 1
amixer -D hw:audiocodec cset name='LINEOUT volume' 15
aplay -Dhw:audiocodec /mnt/UDISK/1KHz_0dB_16000.wav
或者利用默认/etc/asound.conf配置的pcm设备进行播放：
aplay -Ddefault /mnt/UDISK/1KHz_0dB_16000.wav
通过Headphone播放
amixer -D hw:audiocodec cset name='Headphone Switch' 1
amixer -D hw:audiocodec cset name='Headphone volume' 3
aplay -Dhw:audiocodec /mnt/UDISK/1KHz_0dB_16000.wav
```

#### 2.20.7.2录音

```
通过内部的MIC1,MIC2录制双通道
amixer -D hw:audiocodec cset name='MIC1 Input Select' 0
amixer -D hw:audiocodec cset name='MIC2 Input Select' 0
amixer -D hw:audiocodec cset name='MIC1 Switch' 1
amixer -D hw:audiocodec cset name='MIC2 Switch' 1
amixer -D hw:audiocodec cset name='MIC1 gain volume' 19
amixer -D hw:audiocodec cset name='MIC2 gain volume' 19
arecord -Dhw:audiocodec -f S16_LE -r 16000 -c 2 /tmp/test.wav
```

## 2.21 F133音频接口

F133包含多个音频模块，分别是内置AudioCodec,I2S1,I2S2,DMIC,SPDIF。

### 2.21.1 时钟源.

F133中，音频模块的时钟源来自pll_audio0以及pll_audio1_div5。

pll_audio0可以输出22.5792M的时钟,而pll_audio1_div5输出24.576M的时钟，分别支
持44.1k系列，48k系列的播放录音。


### 2.21.2 AudioCodec

#### 2.21.2.1硬件特性

• 两路DAC

- 支持16bit,20bit有效采样精度
- 支持8KHz~192KHz采样率
- 三路ADC
- 支持16bit,20bit有效采样精度
- 支持8KHz~48KHz采样率
- 模拟输出：
- 一路立体声输出HPOUTL,HPOUTR
- 模拟差分输入：
- 一路差分麦克风输入MIC3P/N
- 一路立体声line-in输入LINEINL,LINEINR
- 一路立体声FM-in输入FMINL,FMINR
- 支持同时playback和record(全双工模式)
- DAC及ADC均支持DRC

#### 2.21.2.2内核配置

kernel_menuconfig配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner Sun20iw1 Codec Support
<*> Allwinner Audio Simple Card
[ ] Allwinner RX SYNC Support
```


| 配置项名称                       | 配置项说明   |
| -------------------------------- | ------------ |
| Allwinner Sun20iw1 Codec Support | audiocodec   |
| Allwinner Audio Simple Card      | 绑定声卡     |
| Allwinner RX SYNC Support        | 同步功能组件 |

#### 2.21.2.3 DTS配置.

board.dts板级配置

```
&codec {
/* MIC and headphone gain setting */
mic1gain = <0x1F>;
mic2gain = <0x1F>;
mic3gain = <0x1F>;
/* ADC/DAC DRC/HPF func enabled */
¦ /* 0x1:DAP_HP_EN; 0x2:DAP_SPK_EN; 0x3:DAP_HPSPK_EN */
adcdrc_cfg = <0x0>;
adchpf_cfg = <0x1>;
dacdrc_cfg = <0x0>;
dachpf_cfg = <0x0>;
/* Volume about */
digital_vol = <0x00>;
lineout_vol = <0x1a>;
headphonegain = <0x03>;
/* Pa enabled about */
pa_level = <0x01>;
pa_pwr_level = <0x01>;
pa_msleep_time = <0x78>;
/* gpio-spk = <&pio PF 2 GPIO_ACTIVE_HIGH>;*/
/* gpio-spk-pwr = <&pio PF 4 GPIO_ACTIVE_HIGH>; */
/* CMA config about */
playback_cma = <128>;
capture_cma = <256>;
/* regulator about */
/* avcc-supply = <&reg_aldo1>; */
/* hpvcc-supply = <&reg_eldo1>; */
status = "okay";
};
```

| 部分配置项名称 | 配置取值           | 配置项说明                                     |
| -------------- | ------------------ | ---------------------------------------------- |
| mic3gain       | 0-31               | mic3增益                                       |
| digital_vol    | 0-63               | DAC数字音量                                    |
| headphonegain  | 0-7                | 耳机播放增益                                   |
| pa_level       | 0-1                | 功放芯片使能电平                               |
| pa_pwr_level   | 0-1                | 功放供电使能电平                               |
| pa_msleep_time | u32,一般小于 200 x | u32,一般小于 200 设置功放芯片使能所需sleep时间 |
| status         | “okay”/“disable”   | 使能或者关闭本节点                             |



### 2.21.3 Daudio

#### 2.21.3.1硬件特性

• 两路I2S/PCM I2S1 I2S2


- 支持8-192k采样率
- 支持16 24 32采样精度
- 支持1-16多通道录音播放
- 支持 5 种TDM模式
  - I2S standard mode
  - Left-justified mode
  - Right-justified mode
  - DSP-A mode(short frame PCM mode)
  - DSP-B mode(long frame PCM mode)
- 支持回环模式
- 支持同时playback和capture
- 支持多声卡同步录音

#### 2.21.3.2内核配置

kernel_menuconfig配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner Audio Simple Card
<*> Allwinner Digital Audio Support
[ ] Allwinner RX SYNC Support
```

| 配置项名称                       | 配置项说明   |
| -------------------------------- | ------------ |
| Allwinner Sun20iw1 Codec Support | I2S/PCM      |
| Allwinner Audio Simple Card      | 绑定声卡     |
| Allwinner RX SYNC Support        | 同步功能组件 |

#### 2.21.3.3 DTS配置.

board.dts板级配置

```
&daudio1 {
mclk_div = <0x01>;
frametype = <0x00>;
tdm_config = <0x01>;
sign_extend = <0x00>;
msb_lsb_first = <0x00>;
pcm_lrck_period = <0x80>;
slot_width_select = <0x20>;
pinctrl-names = "default", "sleep";
```

```
pinctrl-0 = <&daudio1_pins_a>;
pinctrl-1 = <&daudio1_pins_b>;
pinctrl_used = <0x0>;
status = "disabled";
};
&sounddaudio1 {
status = "disabled";
daudio1_master: simple-audio-card,codec {
/* sound-dai = <&ac108>; */
};
};
```

设备树配置sun20iw1p1.dtsi

```
daudio1:daudio@2033000 {
#sound-dai-cells = <0>;
compatible = "allwinner,sunxi-daudio";
reg = <0x0 0x02033000 0x0 0xa0>;
clocks = <&ccu CLK_PLL_AUDIO0>,
¦<&ccu CLK_I2S1>,
¦<&ccu CLK_BUS_I2S1>;
clock-names = "pll_audio", "i2s1", "i2s1_bus";
resets = <&ccu RST_BUS_I2S1>;
dmas = <&dma 4>, <&dma 4>;
dma-names = "tx", "rx";
interrupts-extended = <&plic0 43 IRQ_TYPE_LEVEL_HIGH>;
sign_extend = <0x00>;
tx_data_mode = <0x00>;
rx_data_mode = <0x00>;
msb_lsb_first = <0x00>;
pcm_lrck_period = <0x80>;
slot_width_select = <0x20>;
frametype = <0x00>;
tdm_config = <0x01>;
tdm_num = <0x01>;
mclk_div = <0x00>;
clk_parent = <0x01>;
capture_cma = <128>;
playback_cma = <128>;
tx_num = <4>;
tx_chmap1 = <0x76543210>;
tx_chmap0 = <0xFEDCBA98>;
rx_num = <4>;
rx_chmap3 = <0x03020100>;
rx_chmap2 = <0x07060504>;
rx_chmap1 = <0x0B0A0908>;
rx_chmap0 = <0x0F0E0D0C>;
asrc_function_en = <0x00>;
rx_sync_en = <0x00>;
device_type = "daudio1";
status = "disabled";
};
sounddaudio1: sounddaudio1@20330a0 {
reg = <0x0 0x020330a0 0x0 0x4>;
compatible = "sunxi,simple-audio-card";
simple-audio-card,name = "snddaudio1";
simple-audio-card,format = "i2s";
status = "disabled";
simple-audio-card,cpu {
sound-dai = <&daudio1>;
};
};
```

| 部分配置项名称                            | 配置取值                    | 配置项说明                                                   |
| ----------------------------------------- | --------------------------- | ------------------------------------------------------------ |
| status                                    | “okay”<br/>“disable”        | 使能或者关闭snddaudio驱动                                    |
| mclk_div                                  | 0-192                       | mclk分频系数，取值为<br/>0/1/2/4/8/12/16/24/32/48/64/96/128/176/192 |
| frametype                                 | 0-1                         | 0:short frame(1 clock width) 1:long<br/>frame(2 clock width) |
| tdm_config                                | 0-1                         | 0:PCM mode 1:I2S mode                                        |
| sign_extend                               | 0-1                         | 0:zero pending 1:sign extend                                 |
| msb_lsb_first                             | 0-1                         | 0:msb first 1:lsb first                                      |
| pcm_lrck_period                           | 16/32/64/128/256            | 一般可配置为16/32/64/128/256个<br/>bclk                      |
| slot_width_select                         | 8/16/32                     | slot支持8/16/32bit宽度                                       |
| tx_data_mode                              | 0/1/2/3                     | 0:16bit linear PCM 1:reserved<br/>2:8bit u-law 3:8bit a-law  |
| rx_data_mode                              | 0/1/2/3                     | 0:16bit linear PCM 1:reserved<br/>2:8bit u-law 3:8bit a-law  |
| simple-audio-card,name                    | string                      | 声卡名称                                                     |
| simple-audio-card,format                  | string                      | I2S tdm模式,取值为<br/>“i2s”/“right_j”/“left_j”/“dsp_a”/“dsp_b” |
| simple-audio-card,frame-<br/>master       | 注释为false,反<br/>之为true | 配置frame clk主从关系，不配置则<br/>soc为主，反之soc为从     |
| simple-audio-card,bitclock-<br/>master    | 注释为false,反<br/>之为true | 配置bit clk主从关系，不配置则soc为<br/>主，反之soc为从       |
| simple-audio-card,frame-<br/>inversion    | 注释为false,反<br/>之为true | 配置frame clk极性取反;不配置则是正<br/>常极性                |
| simple-audio-card,bitclock-<br/>inversion | 注释为false,反<br/>之为true | 配置bit clk极性取反;不配置则是正常<br/>极性                  |
| simple-audio-<br/>card,capture_only       | 注释为false,反<br/>之为true | 仅支持录音                                                   |
| simple-audio-<br/>card,playback_only      | 注释为false,反<br/>之为true | 仅支持播放                                                   |


### 2.21.4 DMIC.

#### 2.21.4.1硬件特性

- 支持8-48k采样率
- 支持16 24采样精度
- 支持1-8多通道录音播放
- 支持多声卡同步录音

#### 2.21.4.2内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner Audio Simple Card
<*> Allwinner DMIC Support
[ ] Allwinner RX SYNC Support
```

| 配置项名称                       | 配置项说明   |
| -------------------------------- | ------------ |
| Allwinner Sun20iw1 Codec Support | dmic         |
| Allwinner Audio Simple Card      | 绑定声卡     |
| Allwinner RX SYNC Support        | 同步功能组件 |

#### 2.21.4.3 DTS配置.

```
&dmic {
pinctrl-names = "default","sleep";
pinctrl-0 = <&dmic_pins_a>;
pinctrl-1 = <&dmic_pins_b>;
rx_sync_en = <0x00>;
status = "okay";
};
&dmic_codec {
status = "okay";
};
&sounddmic {
status = "okay";
};
```

| 部分配置项名称 | 配置取值              | 配置项说明          |
| -------------- | --------------------- | ------------------- |
| status         | “okay”/“disable”      | 使能或者关闭本节点  |
| rx_sync_en     | 注释为false反之为true | 是否注册rx_sync组件 |

### 2.21.5 SPDIF

#### 2.21.5.1硬件特性

- 支持22.05-192k采样率
- 支持16 24采样精度
- 支持1-2多通道录音播放
- 支持多声卡同步录音
- 支持回环模式
- 支持IEC-60958协议
- 支持IEC-61937协议

#### 2.21.5.2内核配置

```
Device Drivers --->
<*> Sound card support --->
<*> Advanced Linux Sound Architecture --->
<*> ALSA for SoC audio support --->
Allwinner SoC Audio support --->
<*> Allwinner Audio Simple Card
<*> Allwinner SPDIF Support
[ ] Allwinner RX SYNC Support
```

| 配置项名称                       | 配置项说明   |
| -------------------------------- | ------------ |
| Allwinner Sun20iw1 Codec Support | spdif        |
| Allwinner Audio Simple Card      | 绑定声卡     |
| Allwinner RX SYNC Support        | 同步功能组件 |

#### 2.21.5.3 DTS配置.

```
&spdif {
pinctrl-names = "default","sleep";
pinctrl-0 = <&spdif_pins_a>;
pinctrl-1 = <&spdif_pins_b>;
```

```
rx_sync_en = <0x00>;
status = "okay";
};
&soundspdif {
status = "okay";
};
```

| 部分配置项名称 | 配置取值              | 配置项说明          |
| -------------- | --------------------- | ------------------- |
| status         | “okay” “disable”      | 使能或者关闭本节点  |
| rx_sync_en     | 注释为false反之为true | 是否注册rx_sync组件 |

### 2.21.6 标案音频测试方法.

该章节主要介绍在标案上进行播歌，录音的测试命令

##### 2.21.6.1播放

```
通过Headphone播放
1.开机后推送测试音频48000.wav到小机端,pc命令：adb push 48000.wav /tmp/
2.播放该音频文件, aplay /tmp/48000.wav
```

##### 2.21.6.2录音

```
mic:
1.使能mic3通路：amixer set "ADC3 Input MIC3 Boost" on
2.设置mic3增益：amixer cset name="MIC3 gain volume" 31
3.录音5s：arecord -D hw:audiocodec -f S16_LE -r 44100 -c 1 /tmp/test.wav --period-size 1024
--buffer-size 4096 -d 5
linein:
1.使能linein通路：amixer set "ADC1 Input LINEINL" on;amixer set "ADC2 Input LINEINR" on
2.设置linein增益：amixer cset name="LINEINL gain volume" 1;amixer cset name="LINEINR gain
volume" 1
3.录音5s：arecord -D hw:audiocodec -f S16_LE -r 44100 -c 2 /tmp/test.wav --period-size 1024
--buffer-size 4096 -d 5
fmin:
1.使能fmin通路：amixer set "ADC1 Input FMINL" on;amixer set "ADC2 Input FMINR" on
2.设置fmin增益：amixer cset name="FMINL gain volume" 1;amixer cset name="FMINR gain volume" 1
3.录音5s：arecord -D hw:audiocodec -f S16_LE -r 44100 -c 2 /tmp/test.wav --period-size 1024
--buffer-size 4096 -d 5
I2S:
1.推送音频文件到小机端:adb push 16000.wav /tmp/16000.wav
```

2.配置I2S Loopback功能:amixer -D hw:snddaudio2 cset name='sunxi daudio loopback debug' 1
3.开始录音：arecord -D hw:snddaudio2 -f S16_LE -r 44100 -c 2 /tmp/test_su.wav --period-size1024 --buffer-size 4096 &
4.开始播放：aplay -D hw:snddaudio2 /tmp/16000.wav
5.aplay播放结束后，killall arecord结束录音任务
6.通过adb命令把test_su.wav拉出来，在PC端查看音频数据是否跟播放的内容一致