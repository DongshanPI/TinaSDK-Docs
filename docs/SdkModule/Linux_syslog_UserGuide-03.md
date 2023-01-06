## 3 不同syslog 方案的对比

以下针对将本地syslog 日志写入到本地文件中的这一需求，对不同的syslog 方案进行对比。

### 3.1 ubox logd + logread

优点：
• OpenWrt 原生自带，稍作配置即可使用。
• 自带获取内核日志以及简单的rotate 功能。
• 不同于busybox 的logread ，ubox 的logread 可同时支持将日志写入文件和从ring buffer 中读取日志的功能。

缺点：
• 依赖于procd 与ubus 。（未测试过在缺少这两者的情况下是否可用）
• rotate 功能只支持将日志文件转存到相同目录下，且只保存一份历史文件，无压缩功能。（未发现有配置项可进行相关的设置）

### 3.2 busybox syslogd + klogd

优点：
• syslogd 自带rotate 功能。在每次往文件写入日志之前，都会先检查文件大小是否已经超过设定的上限值，若是，则执行rotate 操作。因为文件大小的检查是在写入日志的时候进行，而非按一定的时间间隔进行，可保证进行rotate 时日志文件不会超出上限值很多。且因为写入日志与rotate 是在同一进程中实现，对日志文件进行转存时直接重命名即可，不需要再复制一份，在对剩余可用空间的限制上没有logrotate 的copytruncate 那么大。

• syslogd 会保证每秒都重新打开日志文件，不需要担心文件的inode 改变，清空日志时可随意删除日志文件，新的日志文件在下一秒就能继续正常地写入日志。

缺点：

• syslogd 本身不含获取内核日志的功能，需要额外运行klogd 来支持。
• syslogd 不支持自定义前缀、rotate 时压缩的功能，且只能将日志文件转存到同一个目录下，无法自定义目标路径。
• 将日志写入到文件的同时无法使用logread 。

### 3.3 syslog-ng + logrotate

优点：
• syslog-ng 自身功能比较强大，可更为灵活地对日志进行修改、过滤，且自身带有获取内核日志的功能。
• logrotate 能实现更为灵活的rotate 功能，如自定义目标路径、压缩日志文件等。

缺点：
• syslog-ng 本身无法监视文件大小，无法通知logrotate 进行rotate，只能依赖crond 等守护进程定期地执行logrotate ，需要权衡好日志的增长速度和定期检查的时间间隔，否则存储空间有可能会被日志占满。
• 日志文件的inode 不能随意地被改变，否则syslog-ng 可能无法正确地写入日志。因此：
• logrotate 需要配置为copytruncate ，在rotate 时存在“复制文件” 这一过程，对剩余的存储空间有一定的要求，否则rotate 过程会失败。
• 手动清空日志文件内容时不能直接删除日志文件，需要使用类似下面的命令：

```
echo > /var/log/messages
```

