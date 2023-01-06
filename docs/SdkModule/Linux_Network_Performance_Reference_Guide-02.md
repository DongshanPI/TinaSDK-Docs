# 2 Wi-Fi 性能测试

Wi-Fi 性能测试可通过rf 测试，iperf 吞吐测试，长时间连接测试，Wi-Fi 多次连接与断开测试。

## 2.1 RF 测试

Wi-Fi rf 测试项目主要包括TX，RX。由于每款无线模组的测试方式都不一样，tina sdk 中仅集成了部分模组的测试工具, 关于测试方法，需要咨询模组厂提供的文档。具体可以参考《Tina_Linux_WiFi_RF 测试_ 使用指南》以下是各个模组测试工具的选择。

```
（1）XR819
make menuconfig
Utilities --->
rf test tool --->
<*> xr819-rftest......................................... xr819 rf test tools


```

```
（2）Realtek
make menuconfig
Utilities --->
rf test tool --->
<*> realtek-rftest..................................... realtek rf test tools
```

```
（3）BCM 系列（AP6212，AP6225…）
make menuconfig
Utilities --->
rf test tool --->
<*> broadcom-rftest................................... broadcom rf test tools
```

## 2.2 iperf 测试

iperf 开源的项目，可用于测试网络性能的工具，可以测试最大的TCP 和UDP 带宽性能。测试iperf 需要准备一台PC 机，路由器，以及需要测试的板子。PC 机需要使用网线跟路由器进行连接，iperf 测试需要在屏蔽房进行测试，避免外部环境干扰。
当然，也可以在办公环境中，对比多个平台测试，比较不同模组的抗干扰能力。

iperf 具有以下参数可供选择。

```
Client/Server:
-f, --format [kmKM] format to report: Kbits, Mbits, KBytes, MBytes
-i, --interval # seconds between periodic bandwidth reports
-l, --len #[KM] length of buffer to read or write (default 8 KB)
-m, --print_mss print TCP maximum segment size (MTU - TCP/IP header)
-o, --output <filename> output the report or error message to this specified file
-p, --port # server port to listen on/connect to
-u, --udp use UDP rather than TCP
-w, --window #[KM] TCP window size (socket buffer size)
-B, --bind <host> bind to <host>, an interface or multicast address
-C, --compatibility for use with older versions does not sent extra msgs
-M, --mss # set TCP maximum segment size (MTU - 40 bytes)
-N, --nodelay set TCP no delay, disabling Nagle's Algorithm
-V, --IPv6Version Set the domain to IPv6
Server specific:
-s, --server run in server mode
-U, --single_udp run in single threaded UDP mode
-D, --daemon run the server as a daemon
Client specific:
-b, --bandwidth #[KM] for UDP, bandwidth to send at in bits/sec
(default 1 Mbit/sec, implies -u)
-c, --client <host> run in client mode, connecting to <host>
-d, --dualtest Do a bidirectional test simultaneously
-n, --num #[KM] number of bytes to transmit (instead of -t)
-r, --tradeoff Do a bidirectional test individually
-t, --time # time in seconds to transmit for (default 10 secs)
-F, --fileinput <name> input the data to be transmitted from a file
-I, --stdin input the data to be transmitted from stdin
-L, --listenport # port to receive bidirectional tests back on
-P, --parallel # number of parallel client threads to run
-T, --ttl # time-to-live, for multicast (default 1)
-Z, --linux-congestion <algo> set TCP congestion control algorithm (Linux only)
Miscellaneous:
-x, --reportexclude [CDMSV] exclude C(connection) D(data) M(multicast) S(settings) V(
server) reports
-y, --reportstyle C report as a Comma-Separated Values
-h, --help print this message and quit
-v, --version print version information and quit
```

在tina 平台中，已经移植好了iperf 工具，只需要在menuconfig 选上以下选项，进行编译打包即可。

```
make munconfig
Network --->
<*> iperf......................... Internet Protocol bandwidth measuring tool
```

### 2.2.1 测试TCP TX

example

```
pc端： iperf -s -i 1 -p 5006
device端： iperf -c <pc_ip> -i 1 -t 20 -p 5006
```

### 2.2.2 测试TCP RX

example

```
pc端： iperf -c <pc_ip> -i 1 -t 20 -p 5006
device端： iperf -s -i 1 -p 5006
```

### 2.2.3 测试UDP TX

example

```
pc端： iperf -c <pc_ip> -i 1 -t 20 -p 5006
device端： iperf -s -i 1 -p 5006
```

### 2.2.4 测试UDP RX

example

```
pc端： iperf -c <pc_ip> -i 1 -u -t 20 -p 5006
device端： iperf -s -i 1 -u -p 5006
```

### 2.2.5 关于吞吐量低的常见问题

• 硬件rf 的指标不正常。
• 检查天线是否正常。
• 周围干扰过大，可以到较为干净环境或屏蔽房进行测试。
• 路由器问题设置问题，如果模组支持HT40，路由器端需要检查是否设置HT40。
• 26M 时钟频偏过大，可以进行ETF 测试进行验证。
• 3.3V 电源没有正常供电。
• 驱动or 固件版本比较低。

## 2.3 长时间连接测试

长时间连接可通过iperf 工具一直与pc 机进行通信，观察并分析是否网络在中途有断开的情况。

## 2.4 Wi-Fi 多次连接与断开

可以对模组进行多次连接与断开测试，测试其连接性是否稳定可靠。tina 平台提供其测试应用，仅供用户进行参考测试。

```
@ssid ： 需要连接的路由器名称
@passwd： 密码
@test_times: 测试次数
@level ： 打印等级（d0 ~ d5）
@说明：最终的测试结果将保存到wifi_long_time_test.log文件中
wifi_longtime_test <ssid> <passwd> <test_times> <level>
```

