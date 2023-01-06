## 4 其他一些的注意事项

### 4.1 Unix 域套接字（Unix domain socket）是可靠的

syslog 是靠Unix 域套接字实现IPC（Inter-Process Communication，进程间通信），协议族为AF_LOCAL （或AF_UNIX ），不管套接字的类型为字节流

（SOCK_STREAM ）还是数据报（SOCK_DGRAM），它都是可靠的，在使用Unix 域套接字通信的过程中，如果读操作一端阻塞且缓冲区满了，写操作的一端也同

样会阻塞，在此过程中不会有数据被丢弃。

因此，当syslog 守护进程因为某些原因阻塞或运行耗时变长时，若此时缓冲区已经满了，有可能会影响到调用syslog 函数的应用程序的性能。应用程序在设计时就

需要考虑syslog 函数可能的影响，不能无节制地使用syslog 函数进行打印，也不能认为它总会很快地就执行完。

关于缓冲区，应该跟内核的套接字设置有关。对于Unix 域数据报套接字，从测试结果来看/proc/sys/net/unix/max_dgram_qlen 会影响其缓冲区大小，但具体的

机制还不清楚。它的默认值为10，可使用sysctl 进行修改：

```
sysctl -w net.unix.max_dgram_qlen=XX
```

