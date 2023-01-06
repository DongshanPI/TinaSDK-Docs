## 2 syslog 相关软件工具

### 2.1 ubox 的logd 与logread

ubox 是OpenWrt 的工具箱，它的syslog 系统由logd 与logread 两个工具实现（此处的logread是ubox 自己的实现，与下文busybox 提供的logread 不是同一个

工具）。

因为这两个工具是OpenWrt 原生自带，它们在使用上有可能会依赖于procd 和ubus ，目前尚未测试过在非procd init 的环境下是否可用。

使用procd init 时，它们通过开机脚本/etc/init.d/log 自启动，具体如下：

```
#!/bin/sh /etc/rc.common
# Copyright (C) 2013 OpenWrt.org
# start after and stop before networking
START=12
STOP=89
PIDCOUNT=0
USE_PROCD=1
PROG=/sbin/logread
OOM_ADJ=-17
validate_log_section()
{
    uci_validate_section system system "${1}" \
    'log_file:string' \
    'log_size:uinteger' \
    'log_ip:ipaddr' \
    'log_remote:bool:1' \
    'log_port:port:514' \
    'log_proto:or("tcp", "udp"):udp' \
    'log_trailer_null:bool:0' \
    'log_prefix:string'
}
validate_log_daemon()
{
    uci_validate_section system system "${1}" \
    'log_size:uinteger:0' \
    'log_buffer_size:uinteger:0'
}
start_service_daemon()
{
    local log_buffer_size log_size
    validate_log_daemon "${1}"
    [ $log_buffer_size -eq 0 -a $log_size -gt 0 ] && log_buffer_size=$log_size
    [ $log_buffer_size -eq 0 ] && log_buffer_size=16
    procd_open_instance
    procd_set_param oom_adj $OOM_ADJ
    procd_set_param command "/sbin/logd"
    procd_append_param command -S "${log_buffer_size}"
    procd_set_param respawn
    procd_close_instance
}
start_service_file()
{
    PIDCOUNT="$(( ${PIDCOUNT} + 1))"
    local pid_file="/var/run/logread.${PIDCOUNT}.pid"
    local log_file log_size
    validate_log_section "${1}" || {
        echo "validation failed"
        return 1
    }
    [ -z "${log_file}" ] && return
    procd_open_instance
    procd_set_param command "$PROG" -f -F "$log_file" -p "$pid_file"
    [ -n "${log_size}" ] && procd_append_param command -S "$log_size"
    procd_close_instance
}
start_service_remote()
{
    PIDCOUNT="$(( ${PIDCOUNT} + 1))"
    local pid_file="/var/run/logread.${PIDCOUNT}.pid"
    local log_ip log_port log_proto log_prefix log_remote log_trailer_null
    validate_log_section "${1}" || {
        echo "validation failed"
        return 1
    }
    [ "${log_remote}" -ne 0 ] || return
    [ -z "${log_ip}" ] && return
    procd_open_instance
    procd_set_param command "$PROG" -f -r "$log_ip" "${log_port}" -p "$pid_file"
    case "${log_proto}" in
    "udp") procd_append_param command -u;;
    "tcp") [ "${log_trailer_null}" -eq 1 ] && procd_append_param command -0;;
    esac
    [ -z "${log_prefix}" ] || procd_append_param command -P "${log_prefix}"
    procd_close_instance
}
service_triggers()
{
    procd_add_reload_trigger "system"
    procd_add_validation validate_log_section
}
start_service()
{
    config_load system
    config_foreach start_service_daemon system
    config_foreach start_service_file system
    config_foreach start_service_remote system
}
```

可见该脚本会通过config_load system 读取配置，然后将配置作为logd 和logread 的选项参数。具体的配置位于文件/etc/config/system 中。

更为详细的说明可参考OpenWrt 的官方文档[Runtime Logging in OpenWrt](https://openwrt.org/docs/guide-user/base-system/log.essentials) 以及[Systemconfiguration /etc/config/system](https://openwrt.org/docs/guide-user/base-system/system_configuration) 。

#### 2.1.1 logd

logd 维护着一个固定大小的ring buffer（环形缓冲区），用于保存收集到的日志（包括内核的日志）。ring buffer 的大小通过-S 参数指定，可通过配

置/etc/config/system 中的log_buffer_size进行修改，单位为KB。

#### 2.1.2 logread

logread 用于读取logd 的ring buffer 的内容，并输出到文件或网络上的远程机器（通过TCP/UDP 套接字）。它支持的选项有如下：

```
Usage: logread [options]
Options:
    -s <path> Path to ubus socket
    -l <count> Got only the last 'count' messages
    -e <pattern> Filter messages with a regexp
    -r <server> <port> Stream message to a server
    -F <file> Log file
    -S <bytes> Log size
    -p <file> PID file
    -h <hostname> Add hostname to the message
    -P <prefix> Prefix custom text to streamed messages
    -f Follow log messages
    -u Use UDP as the protocol
    -0 Use \0 instead of \n as trailer when using TCP
```

• 直接执行logread 会将当前ring buffer 中的日志全打印出来（类似于dmesg ）。
• 加上-f 则会持续地运行着，并输出ring buffer 中新的日志。
• 使用-F "$log_file" 可指定将日志输出到哪一个文件中，-S "$log_size" 可指定文件的大小，其中log_file 和log_size 都可在/etc/config/system 中进行设置。（实测发现其自带有rotate 的功能，当log_file 的大小超过log_size 时，会加上“.0” 后缀转存到同一个目录下，默认只保存一份历史文件。暂未发现是否可配置保存超过一份的历史转存文件。）

### 2.2 busybox 的syslogd、klogd 与logread

busybox 自带有一些syslog 工具，一般用到的主要为syslogd 、klogd 和logread （此处的logread与上文的ubox 的logread 不是同一个工具），均位于它

menuconfig 的“System LoggingUtilities” 下。

后文busybox 相关的内容均基于1.27.2 版本进行阐述。

#### 2.2.1 syslogd

busybox 的syslogd 用于读取/dev/log 中的日志，并决定将其发送到文件、共享内存中的circular buffer 或网络等位置，且其自带有简单的rotate 功能。

它支持的特性可在menuconfig 中进行配置，将所有特性都选上后它支持的选项如下：

```
Usage: syslogd [OPTIONS]
System logging utility
    -n Run in foreground
    -R HOST[:PORT] Log to HOST:PORT (default PORT:514)
    -L Log locally and via network (default is network only if -R)
    -C[size_kb] Log to shared mem buffer (use logread to read it)
    -K Log to kernel printk buffer (use dmesg to read it)
    -O FILE Log to FILE (default: /var/log/messages, stdout if -)
    -s SIZE Max size (KB) before rotation (default 200KB, 0=off)
    -b N N rotated logs to keep (default 1, max 99, 0=purge)
    -l N Log only messages more urgent than prio N (1-8)
    -S Smaller output
    -D Drop duplicates
    -f FILE Use FILE as config (default:/etc/syslog.conf)
```

• 特性“Rotate message files”（FEATURE_ROTATE_LOGFILE）即为rotate 功能，对应
-s 指定日志文件的限制大小以及-b 指定保存多少份历史的转存文件。
• 特性“Remote Log support”（FEATURE_REMOTE_LOG）即为网络功能的支持，对应-R和-L 选项。
• 特性“Support -D (drop dups) option”（FEATURE_SYSLOGD_DUP）对应-D 选项，会丢弃掉内容相同的重复日志。判断日志是否相同不光看其主体信息，时间戳等附加的信息也会考虑在内，如“Jan 1 08:00:00 root: foobar” 和“Jan 1 08:00:01 root: foobar” 会被认为是两条不同的日志，只有完全相同的日志才会被丢弃掉。
• 特性“Support syslog.conf”（FEATURE_SYSLOGD_CFG）支持使用配置文件，默认为/etc/syslog.conf ，也可通过-f 指定其他的文件。可在配置文件中根据facility 与level 将日志输出到不同的目标位置，例子如下：

```
# 将所有日志输出到文件/var/log/messages

*.* /var/log/messages

# 将所有日志输出到console
*.* /dev/console
# 将facility 为LOG_KERN 的日志输出到/var/log/kernel
kern.* /var/log/kernel
# 将facility 为LOG_USER 且level 高于LOG_NOTICE 的日志输出到/var/log/user
user.notice /var/log/user
```

• 特性“Read buffer size in bytes”（FEATURE_SYSLOGD_READ_BUFFER_SIZE）用于设置syslogd 从/dev/log 中读取内容时的buffer 大小，它规定了单条日志消息的最大长度，超出的部分会被截断丢弃掉。
• 特性“Circular Buffer support”（FEATURE_IPC_SYSLOG）对应-C[size_kb] 选项，用于将日志送至共享内存的circular buffer 中， 可以通过logread 读取出来。circular buffer 的大小可通过“Circular buffer size in Kbytes (minimum 4KB)”（FEATURE_
IPC_SYSLOG_BUFFER_SIZE）进行设置。
• 特性“Linux kernel printk buffer support”（FEATURE_KMSG_SYSLOG）对应-K 选项，用于将日志输出到Linux 内核的printk buffer 中，可通过dmesg 读取出来。
• 剩余的一些选项：-O 用于指定直接将日志输出到哪个文件；-S 用于精简日志消息，去除hostname、facility、level 等内容，只保留时间戳、进程名字以及消息的内容部分。

**注意：**当前版本的syslogd 中-f 、-C 、-O 几个选项对应的功能是冲突的，无法同时使用。相关部分的代码如下（位于busybox/sysklogd/syslogd.c 的timestamp_and_log 函数中）：

```
/* Log message locally (to file or shared mem) */
#if ENABLE_FEATURE_SYSLOGD_CFG
    {
        bool match = 0;
        logRule_t *rule;
        uint8_t facility = LOG_FAC(pri);
        uint8_t prio_bit = 1 << LOG_PRI(pri);
        for (rule = G.log_rules; rule; rule = rule->next) {
          if (rule->enabled_facility_priomap[facility] & prio_bit) {
            log_locally(now, G.printbuf, rule->file);
            match = 1;
        }
    }
    if (match)
    	return;
    }
#endif
	if (LOG_PRI(pri) < G.logLevel) {
#if ENABLE_FEATURE_IPC_SYSLOG
        if ((option_mask32 & OPT_circularlog) && G.shbuf) {
            log_to_shmem(G.printbuf);
            return;
        }
#endif
    	log_locally(now, G.printbuf, &G.logFile);
    }
```

可见使能了配置文件的特性后（对应宏ENABLE_FEATURE_SYSLOGD_CFG ），在读取配置文件并将日志写到目标位置后就直接return 了，不会再执行将日志输出

到共享内存区域（对应宏ENABLE_FEATURE_IPC_SYSLOG ）或直接输出到某个文件（最后的那句log_locally(now, G.printbuf, &G.logFile) ）的代码。

#### 2.2.2 syslog.conf

##### 2.2.2.1 syslog.conf 的格式

syslog.conf 文件指明syslogd 程序纪录日志的行为，该程序在启动时查询配置文件。

**如果没有改配置文件的话，默认的会写到/var/log/messages 中。**

该文件由不同程序或消息分类的单个条目组成，每个占一行。对每类消息提供一个选择域和一个动作域。这些域由tab 隔开：选择域指明消息的类型和优先级；动

作域指明syslogd 接收到一个与选择标准相匹配的消息时所执行的动作。每个选项是由设备和优先级组成。当指明一个优先级时，syslogd 将纪录一个拥有相同或

更高优先级的消息。所以如果指明“crit”，那所有标为crit、alert 和emerg 的消息将被纪录。每行的行动域指明当选择域选择了一个给定消息后应该把他发送到哪

儿。

如下所示：

类型. 级别; 类型. 级别[TAB] 动作

##### 2.2.2.2 类型facility：

保留字段中的“类型” 代表信息产生的源头，可以是：

```
auth 认证系统，即询问用户名和口令
cron 系统定时系统执行定时任务时发出的信息
daemon 某些系统的守护程序的syslog,如由in.ftpd产生的log
kern 内核的syslog信息
lpr 打印机的syslog信息
mail 邮件系统的syslog信息
mark 定时发送消息的时标程序
news 新闻系统的syslog信息
user 本地用户应用程序的syslog信息
uucp uucp子系统的syslog信息
local0..7 种本地类型的syslog信息,这些信息可以又用户来定义
\* 代表以上各种设备
```

##### 2.2.2.3 级别priority：

保留字段中的“级别” 代表信息的重要性，可以是：

```
emerg 紧急，处于Panic状态。通常应广播到所有用户；
alert 告警，当前状态必须立即进行纠正。例如，系统数据库崩溃；
crit 关键状态的警告。例如，硬件故障；
err 其它错误；
warning 警告；
notice 注意；非错误状态的报告，但应特别处理；
info 通报信息；
debug 调试程序时的信息；
none 通常调试程序时用，指示带有none级别的类型产生的信息无需送出。如*.debug;mail.none表示调试时除
邮件信息外其它信息都送出。
```

##### 2.2.2.4 动作action：

“动作” 域指示信息发送的目的地。可以是：

```
/filename 日志文件。由绝对路径指出的文件名，此文件必须事先建立；
@host 远程主机； @符号后面可以是ip,也可以是域名，默认在/etc/hosts文件下loghost这个别名已经指
定给了本机。
user1,user2 指定用户。如果指定用户已登录，那么他们将收到信息；

* 所有用户。所有已登录的用户都将收到信息。
```



##### 2.2.2.5 具体实例：

我们来看看/etc/syslog.conf 文件中的实例：

```
……
*.err;kern.debug;daemon.notice;mail.crit [TAB] /var/adm/messages
……
```

这行中的“action” 就是我们常关心的那个/var/adm/messages 文件，输出到它的信息源头“selector”是：
• *.err - 所有的一般错误信息；
• kern.debug - 核心产生的调试信息；
• daemon.notice - 守护进程的注意信息；
• mail.crit - 邮件系统的关键警告信息

例如，如果想把所有邮件消息纪录到一个文件中，如下：

```
#Log all the mail messages in one place
mail.* /var/log/maillog
```

其他设备也有自己的日志。UUCP 和news 设备能产生许多外部消息。它把这些消息存到自己的日志（/var/log/spooler）中并把级别限为“err” 或更高。例如：

```
# Save mail and news errors of level err and higher in aspecial file.

uucp,news.crit /var/log/spooler
```

当一个紧急消息到来时，可能想让所有的用户都得到。也可能想让自己的日志接收并保存。

```
#Everybody gets emergency messages， plus log them on anther machine
*.emerg *
*.emerg @linuxaid.com.cn
```

alert 消息应该写到root 和tiger 的个人账号中：

```
#Root and Tiger get alert and higher messages
*.alert root,tiger
```

有时syslogd 将产生大量的消息。例如内核（“kern” 设备）可能很冗长。用户可能想把内核消息纪录到/dev/console 中。下面的例子表明内核日志纪录被注释掉了：

```
#Log all kernel messages to the console
#Logging much else clutters up the screen
#kern.* /dev/console
```

用户可以在一行中指明所有的设备。下面的例子把info 或更高级别的消息送到/var/log/messages，除了mail 以外。级别“none” 禁止一个设备：

```
#Log anything（except mail）of level info or higher
#Don't log private authentication messages!
*.info:mail.none;authpriv.none /var/log/messages
```

在有些情况下，可以把日志送到打印机，这样网络入侵者怎么修改日志都没有用了。通常要广泛纪录日志。Syslog 设备是一个攻击者的显著目标。一个为其他主机

维护日志的系统对于防范服务器攻击特别脆弱，因此要特别注意。有个小命令logger 为syslog（3）系统日志文件提供一个shell 命令接口，使用户能创建日志文件

中的条目。
用法：logger
例如：logger This is a test！
它将产生一个如下的syslog 纪录：Jan 1 00:08:49 TinaLinux user.notice root: this is a test!

#### 2.2.3 klogd

busybox 的syslogd 无法直接获取到内核的日志信息，该功能需要通过klogd 实现。在运行syslogd之后再运行klogd 即可。

klogd 获取内核日志的方法有两种：1) 通过klogctl() 接口；2) 通过/proc 或设备节点。选用哪种方法可通过menuconfig 中的“Use the klogctl() 

interface”（FEATURE_KLOGD_KLOGCTL）进行设置。

klogd 在获取到内核日志后，再通过syslog 函数将日志发送给syslog 守护进程。

**注意：**

虽然klogd 是使用openlog("kernel", 0, LOG_KERN) ，但从源码中的注释来看，在glibc 中LOG_KERN 可能会被替换为LOG_USER ，因此在使用klogd 过程中需要

注意，**内核日志的facility 有可能为user而非kern。**

#### 2.2.4 logread

busybox 的logread 用于从syslogd 共享内存的circular buffer 中读取日志信息， 它需要syslogd 运行时带上-C[size_kb] 选项， 并且需要关闭支持配置文件的特性（FEATURE_SYSLOGD_CFG）。

### 2.3 syslog-ng

syslog-ng 是syslog 守护进程的又一种实现，它本身并不依赖于ubox 或busybox，是一个独立的应用软件。它支持更为丰富的配置项，可以对日志进行更为灵活

的处理。

syslog-ng 的配置文件为/etc/syslog-ng.conf ， 详细的语法可参考官方文档https://www.syslog-ng.com/technical-documents/list/syslog-ng-open-source-edition ，下面是一份配置的例子：

```
@version:3.9
options {
    chain_hostnames(no);
    create_dirs(yes);
    flush_lines(0);
    keep_hostname(yes);
    log_fifo_size(256);
    log_msg_size(1024);
    stats_freq(0);
    flush_lines(0);
    use_fqdn(no);
    time_reopen(1); # 连接断开后等待多少秒后重新建立连接（默认为60 秒）
    keep_timestamp(no); # 不保存日志信息自带的时间戳，用syslog-ng 收到该日志的时间作为时间戳
};
# 定义一个template，可使用template 对日志的各部分内容进行处理
# 使用了此处的template 的日志，会只显示时间戳、日志头部（程序名字等）以及主体信息，
# 相比于默认的日志信息会少了主机名
template t_without_hostname {
    template("${DATE} ${MSGHDR}${MESSAGE}\n");
};
# 定义日志的source，即从哪里获取日志
# 此处表示从syslog-ng 内部以及通过Unix 数据报套接字从/dev/log 获取日志
source src {
    internal();
    unix-dgram("/dev/log");
};
# 从/proc/kmsg 中获取内核的日志
source kernel {
	file("/proc/kmsg" program_override("kernel"));
};
# 定义日志的destination，即将日志送往哪里
# 此处表示将日志输出到文件/var/log/messages，并使用刚刚定义的template 去掉主机名
destination messages {
file("/var/log/messages" template(t_without_hostname));
};
# 将日志输出到console，并使用刚刚定义的template 去掉主机名
destination console {
	file("/dev/console" template(t_without_hostname));
};
# 定义log，用于决定将哪些source 的日志送往哪些destination
log {
    source(src);
    source(kernel);
    destination(messages);
    destination(console);
};
```

直接执行命令syslog-ng 即可运行syslog-ng，下面是一个procd 式自启动脚本的例子：

```
#!/bin/sh /etc/rc.common
# Copyright (C) 2006-2016 OpenWrt.org
START=50
STOP=99
USE_PROCD=1
start_service() {
    [ -f /etc/syslog-ng.conf ] || return 1
    procd_open_instance
    procd_set_param command /usr/sbin/syslog-ng
    procd_close_instance
}
stop_service() {
	/usr/sbin/syslog-ng-ctl stop
}
reload_service() {
    stop
    start
}
```

syslog-ng 自身并不具备rotate 的功能，无法限制日志文件的大小，一般会通过logrotate 或自行编写脚本实现。

### 2.4 logrotate

logrotate 是专门用于对日志文件进行rotate 的工具，支持将日志文件进行压缩、转存到不同目录等特性。

使用logrotate 时需要加上配置文件的路径，如logrotate /etc/logrotate.conf ，配置文件的语法可

参考https://jlk.fjfi.cvut.cz/arch/manpages/man/logrotate.8 。

通常可在配置文件中使用include 命令将某个路径下所有的配置文件都包含进来，如在/etc/logrotate.conf 中加上一句include /etc/logrotate.d 可包

含/etc/logrotate.d 目录下的配置文件，然后该目录下可以按照不同应用、不同日志文件对配置文件进行区分，方便解耦。

以下是针对某一份日志文件进行单独配置的例子：

```
# 针对文件的/var/log/messages 的配置，花括号中的配置项可覆盖全局配置
/var/log/messages {
    hourly # 每小时均进行转存（实测转存周期小于一小时也可成功运行，
    # 但如果设为daily、weekly 等貌似在转存周期太短时会执行失败）
    size 2M # 文件在大于2M 时才会转存
    rotate 9 # 保存9 份历史转存日志文件
    olddir /data # 被转存的历史日志文件会保存到/data 目录下
    createolddir # 若历史日志文件的目标目录不存在则会自动创建
    compress # 对历史日志文件进行压缩（默认使用gzip）
    copytruncate # 在转存时对原始日志文件复制一份后再进行截断，对复制后的文件进行转存；
    # 而不是直接将原始日志文件移动到目标路径，避免原始日志文件的inode 发生变化。
}
```

注意事项：

1. 配置文件的权限需要为0644 或0444 ，否则logrotate 执行时会有以下报错：

  ```
  error: Ignoring XXX because of bad file mode - must be 0644 or 0444.
  ```

2. logrotate 本身属于单次执行后就退出的应用程序，并非守护进程，需要借助其他守护进程（如crond ）定期来执行。下面是/etc/crontabs/root 的一个示例，让root 用户每隔3 分钟执行一次logrotate ：

  ```
  */3 * * * * /usr/sbin/logrotate /etc/logrotate.conf
  ```

3. 一般都需要配置为copytruncate ，除非当前使用的syslog 守护进程支持重新打开日志文件的特性（如busybox 的syslogd 每秒都会重新打开日志文件），否则默认logrotate 进行rotate 时会直接对原始日志文件进行重命名，再创建一个与原始日志文件同名的空白文件，此时日志文件虽然名字相同但inode 不同，而syslog 守护进程还是继续操作原本的inode，导致后续的日志没有正确地写入。

4. 配置为copytruncate 时需要确保rotate 时刻剩余的可用空间大于原始日志文件的大小。因为copytruncate 需要先将日志文件复制一份后再进行rotate，若剩余空间不足导致复制操作失败，后续的整个rotate 过程也无法完成。

### 2.5 logger

logger 用于在shell 中向syslog 守护进程发送消息，使用方法类似于echo 命令：

```
logger "foobar"
```

