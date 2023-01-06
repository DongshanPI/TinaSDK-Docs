## 4 RTL 系列模组

### 4.1 RF 测试环境搭建

#### 4.1.1 驱动配置

为了支持RF test 工具的使用，必须配置RTL 系列的驱动。

```
make kernel_menuconfig
```

```
Device Drivers > Network device support > Wireless LAN
<M> Realtek 8189F SDIO WiFi
<M> Realtek 8723D SDIO or SPI WiFi
```



#### 4.1.2 Tina 配置

按如下方法配置ETF 工具。

```
make menuconfig
```

```
Utilities > rf test tool >
<*> realtek-rftest.......realtek rf test tools
rtk_hciattach >
<*> rtk_hciattach.........Realtek BT HCI UART initialization tools
```



### 4.2 rtwpriv 测试命令

#### 4.2.1 传导TX 测试

```
ifconfig wlan0 up

# Enable Device for MP operation

rtwpriv wlan0 mp_start

# enter MP mode

rtwpriv wlan0 mp_channel 7

# set channel to 1 . 2, 3, 4~13 etc.now is channel 7

rtwpriv wlan0 mp_bandwidth 40M=0,shortGI=0

# set 20M mode and long GI；set 40M is 40M=1.

rtwpriv wlan0 mp_ant_tx a

# Select antenna A for operation

if device have 2x2 antennam select antenna "a" or "b" and "ab" for operation.
rtwpriv wlan0 mp_txpower patha=44
# set path A and path B Tx power level , the Range is 0~63.
rtwpriv wlan0 mp_rate 135
# set OFDM data rate to 54Mbps,ex: CCK 1M = 2, CCK 5.5M = 11 ;
OFDM 6M=12、54M = 108 ;
N Rate: MCS0 = 128, MCS1 = 129 MCS 2=130....MCS15 = 143 etc.
rtwpriv wlan0 mp_ctx count=%100,pkt
# start continuous Packet Tx
rtwpriv wlan0 mp_ctx stop
#stop continuous Tx
rtwpriv wlan0 mp_stop
# exit MP mode
ifconfig wlan0 down
# close WLAN interface
```

#### 4.2.2 传导RX 测试

```
ifconfig wlan0 up

# Enable Device for MP operation

rtwpriv wlan0 mp_start

# Enter MP mode

rtwpriv wlan0 mp_channel 13

# Set channel to 1 . 2, 3, 4~13 etc.

rtwpriv wlan0 mp_bandwidth 40M=0,shortGI=0

# Set 20M mode and long GI or set to 40M is 40M=1.

rtwpriv wlan0 mp_ant_rx a

# Select antenna A for operation

if device have 2x2 antennam select antenna "a" or "b" and "ab" for operation.
rtwpriv wlan0 mp_arx start

# start air Rx teseting.

rtwpriv wlan0 mp_arx phy

# get the statistics.

rtwpriv wlan0 mp_reset_stats
#Stop air Rx test and show the Statistics / Reset Counter.
rtwpriv wlan0 mp_arx stop or rtwpriv wlan0 mp_reset_stats
rtwpriv wlan0 mp_stop

# exit MP mode

ifconfig wlan0 down

# close WLAN interface
```

