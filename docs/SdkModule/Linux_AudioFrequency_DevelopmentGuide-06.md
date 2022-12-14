## 6 各平台音频模块注意事项

### 6.1 R328.

1. 只有一个DAC，所以在播放两通道数据的时候，硬件上会将第二个通道的数据丢掉。如果想要将两通道数据都完成播放出来，需要在软件上将两通道合成，可利用alsa插件实现,例如下面配置：

```
pcm.playback {
type plug
slave {
pcm "hw:audiocodec,0"
rate 48000
channels 1
}
}
```

```
alsa插件会将两通道数据的幅度衰减为由原来的一半(如果直接相加，幅度为原来的两倍，有可能造成削顶)，再组合为一通道写入声卡中。
```

2. 使用模拟mic作AEC时，需要将该MIC gain设置为0dB.如果有增益，则会导致录音开始及结束时产生pop音(开关PA产生的pop音，经过差分MIC消除后仍然存在极小的杂音，如果再经过MIC增益放大，则会变为明显的pop音)