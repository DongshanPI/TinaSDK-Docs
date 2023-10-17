## 5 在Tina 中使用syslog

### 5.1 ubox 的logd 与logread

一般使用procd init 的方案都会默认选上这两个工具。

logd 由PACKAGE_logd 提供，menuconfig 中的位置为：

```
make menuconfig --->
    Base system --->
    	<*> logd
```

![Tina_Linux_syslog_Development_Guide-image-20230102174959204](https://photos.100ask.net/Tina-Sdk/Tina_Linux_syslog_Development_Guide-image-20230102174959204.png)

<center>图5-1: logd 配置图</center>

logread 由PACKAGE_ubox 提供，menuconfig 中的位置为：

```
make menuconfig --->
    Base system --->
    	<*> ubox
```

![Tina_Linux_syslog_Development_Guide-image-20230102175031723](https://photos.100ask.net/Tina-Sdk/Tina_Linux_syslog_Development_Guide-image-20230102175031723.png)

<center>图5-2: ubox 配置图</center>

它们的开机脚本/etc/init.d/log 由PACKAGE_logd 提供；配置项位于文件/etc/config/system 中，默认由PACKAGE_base-files 提供，若想修改默认的配置，可以在

target/allwinner/<方案名字>/base-files/etc/config/ 目录下放置一份自定义的system 以覆盖默认的文件。

### 5.2 busybox 的syslogd、klogd 与logread

busybox 的syslog 工具在menuconfig 中的位置为：

```
make menuconfig --->
    Base system --->
        busybox --->
            System Logging Utilities --->
                [*] klogd
                *** klogd should not be used together with syslog to kernel printk buffer ***
                [*] Use the klogctl() interface
                [*] logger
                [*] logread
                [*] Double buffering
                [*] syslogd
                [*] Rotate message files
                [ ] Remote Log support
                [*] Support -D (drop dups) option
                [*] Support syslog.conf
                (256) Read buffer size in bytes
                [*] Circular Buffer support
                (4) Circular buffer size in Kbytes (minimum 4KB)
                [*] Linux kernel printk buffer support
```

对应配置项的内容请参考前文的章节。

busybox 的syslog 工具没有自带开机脚本，若想开机自启需要自行编写, 在rc.final 增加开机自启动，如下：

```
#挂载sd卡
mount /dev/mmcblk0p1 /mnt/sdcard/
mkdir /mnt/sdcard/log
#开启rotate功能，每个log大小问4M，最多记录10个
syslogd -s 4096 -b 10 &
sleep 1
#同时记录kernel的log
klogd &
```

创建/etc/syslog.conf 文件，把log 文件记录到sd 卡中，内容如下：

```
*.* /mnt/sdcard/log/message
```

### 5.3 syslog-ng

syslog-ng 在menuconfig 中的位置为：

```
make menuconfig --->
    Administration --->
    	<*> syslog-ng
```

![Tina_Linux_syslog_Development_Guide-image-20230102175320862](https://photos.100ask.net/Tina-Sdk/Tina_Linux_syslog_Development_Guide-image-20230102175320862.png)

<center>图5-3: syslog-ng 配置图</center>

它自带有一份procd 式的开机脚本（会自动拷贝到小机端）以及一份配置文件的范例（不会自动拷贝到小机端），均位于package/admin/syslog-ng/files 目录

下。可以参考配置文件范例syslog-ng.conf_example 自定义一份syslog-ng.conf 放到小机端的/etc 目录下。

### 5.4 logrotate

logrotate 在menuconfig 中的位置为：

```
make menuconfig --->
    Utilities --->
    	<*> logrotate
```

![Tina_Linux_syslog_Development_Guide-image-20230102175409032](https://photos.100ask.net/Tina-Sdk/Tina_Linux_syslog_Development_Guide-image-20230102175409032.png)

<center>图5-4: logrotate 配置图</center>

它自带有一份配置文件logrotate.conf ，位于package/utils/logrotate/files 目录下，会自动拷贝到小机端的/etc 目录下。

配置文件带有一些全局的配置项，并且会include /etc/logrotate.d ，因此自定义的配置可放置在小机端的/etc/logrotate.d 目录下，执行logrotate 

/etc/logrotate.conf 时会被自动调用到（注意文件的权限需要为0644 或0444 ）。