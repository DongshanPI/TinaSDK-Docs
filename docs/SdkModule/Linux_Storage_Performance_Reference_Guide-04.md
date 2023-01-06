# 5 读写性能的影响因素



## 5.1 O_SYNC

注意Tina 使用iozone 时，默认参数是使能了O_SYNC 的，降低了cache 的影响。应用正常运行时，一般不使用O_SYNC，可获得比所测数据更佳的性能。
如需测不带O_SYNC 的性能，需修改iozone 参数，测试用例的menuconfig 中提供了ASYNC选项，选上即可。
测试用例运行过程会打印出iozone 的参数，具体参数含义请查看iozone 的帮助。

## 5.2 调频策略

不同调频策略会对读写性能造成影响，建议在测试的时候切换到performance 策略。

```
find . -name scaling_governor #找到调频节点
echo "performance" > /sys/devices/system/cpu/cpufreq/policy0/scaling_governor #修改策略
cat /sys/devices/system/cpu/cpufreq/policy0/scaling_governor #确认策略切换成功
```

## 5.3 其他

对比性能时，需保持其他条件尽可能一致，包括但不限于CPU 频率，DDR 频率，DDR 类型，系统负载等。多次测试会有波动，可以烧录固件后第一次测试的数据为准，或多次取平均。