## 1 基本介绍

syslog 可以说是一套统一管理系统日志的机制，尤其常用于记录守护进程的输出信息上。因为守护进程不存在控制终端，它的打印不能简单地直接输出到stdin 或

stderr。

使用syslog 时，一般需要关注两部分：syslog 守护进程与syslog 函数。

### 1.1 syslog 守护进程

syslog 守护进程用于统一管理日志。它一般会创建一个数据报（SOCK_DGRAM ）类型的Unix 域套接字（Unix domain socket），将其捆绑到/dev/log （不同的

系统可能会有所不同）。如果支持网络功能，它可能还会创建一个UDP 套接字，并捆绑到端口514。syslog 守护进程从这些套接字中读取日志信息，然后再输出到

设定的目标位置（文件、串口等）。

后面提到的ubox 的logd 、busybox 的syslogd 、syslog-ng 都是syslog 守护进程的不同实现。

### 1.2 syslog 函数

应用程序若想将打印信息发送到syslog 守护进程，就需要通过Unix 域套接字将信息输出到syslog守护进程绑定的路径，标准的做法是通过调用syslog 函数：

```
#include <syslog.h>
void openlog(const char *ident, int option, int facility);
void syslog(int priority, const char *format, ...);
void closelog(void);
```

syslog 函数被应用程序首次调用时，会创建一个Unix 域套接字，并连接到syslog 守护进程的Unix 域套接字绑定的路径名上。这个套接字会一直保持打开，直到进

程终止为止。应用程序也可以显式地调用openlog 和closelog （这两个函数都不是必须要调用的），如果不显式调用，在第一次调用syslog 函数时会自动隐式地调

用openlog ，进程结束后也会自动关闭与syslog 守护进程通信的文件描述符，相当于隐式调用closelog 。

以下是一些参数说明，更详细的请参考syslog 的man 手册。

#### 1.2.1 openlog()

• ident 参数会被添加到每一条日志信息中，一般为程序的名字。

• option 参数支持以下的值，可通过或操作（OR）让其支持多个option ：

| option               | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| LOG_CONS             | 若日志无法通过Unix 域套接字送到syslog 守护进程，则将其输出到<br/>console |
| LOG_NDELAY           | 立即打开至syslog 守护进程Unix 域套接字的连接，不要等到第一次调用<br/>syslog 函数时才建立连接（通常情况下会在第一次调用syslog 时才建立连<br/>接） |
| LOG_NOWAIT           | 不要等待在将消息计入日志过程中可能已经创建的子进程（GNU C 库中不<br/>会创建子进程，因此该选项在Linux 中不会起作用） |
| LOG_ODELAYLOG_NDELAY | 的相反，在第一次调用syslog 函数前不建立连接（这是默<br/>认的行为，可以不显式指定该选项） |
| LOG_PERROR           | 将日志信息也输出到stderr                                     |
| LOG_PID              | 在每条日志信息中添加上进程ID                                 |

• facility 参数用于指定当前应用程序的设施类型，为后续的syslog 调用指定一个设施的默认值。

该参数的存在意义是让syslog 守护进程可以通过配置文件对不同设施类型的日志信息做区分处理。如果应用程序没有调用openlog ，或是调用时facility 参数为0，

可在调用syslog 时将facility 作为priority 参数的一部分传进去。

| facility     | 说明                                   |
| ------------ | -------------------------------------- |
| LOG_AUTH     | 安全/授权信息                          |
| LOG_AUTHPRIV | 安全/授权信息（私用）                  |
| LOG_CRON     | 定时相关的守护进程（cron 和at ）       |
| LOG_DAEMON   | 系统守护进程                           |
| LOG_FTP FTP  | 守护进程                               |
| LOG_KERN     | 内核信息（无法通过用户空间的进程产生） |
| LOG_LPT      | 行式打印机系统                         |
| LOG_MAIL     | 邮件系统                               |
| LOG_NEWS     | USENET 网络新闻系统                    |
| LOG_SYSLOG   | 有syslog 守护进程内部产生的消息        |
| LOG_USER     | 任意的用户级消息（默认）               |
| LOG_UUCP     | UUCP 系统                              |

#### 1.2.2 syslog()

priority 参数可以是上面提到的facility 与下面的level 的组合。level 的优先级从高到低依次排序如下：

| level       | 值   | 说明                             |
| ----------- | ---- | -------------------------------- |
| LOG_EMERG   | 0    | 紧急（系统不可用）（最高优先级） |
| LOG_ALERT   | 1    | 必须立即采取行动                 |
| LOG_CRIT    | 2    | 严重情况（如硬件设备出错）       |
| LOG_ERR     | 3    | 出错情况                         |
| LOG_WARNING | 4    | 警告情况                         |
| LOG_NOTICE  | 5    | 正常但重要的情况（默认值）       |
| LOG_INFO    | 6    | 通告信息                         |
| LOG_DEBUG   | 7    | 调试信息（最低优先级）           |

### 1.3 rotate（转存/轮转）

很多时候都会让syslog 守护进程读取到的日志信息都写入到某个文件中，随着日志的增多，文件大小会不断增大。为了避免日志文件将存储空间占满，需要限制日

志文件的大小并删除过去的日志，该操作就称为rotate（转存/轮转）。

rotate 的实现一般如下：假设syslog 守护进程将日志写入到文件/var/run/messages，当messages 文件大小超过设定值时，会将messages 中的日志信息保存到

别的文件中（假设名字为messages.0），然后清空messages 的内容；当下一次messages 文件的大小又超过设定值时，会再一次将messages 中的内容保存为

messages.0，messages.0 中原有的内容则保存为messages.1。如此类推，若干次之后就会存在messages、messages.0、messages.1、… 、messages.n 几个

文件。一般会设置n 的最大值，超过该值的历史文件就会被删除，从而限制日志文件整体的大小。

我们可以自行编写脚本实现rotate，可以使用专门的工具logrotate，另外有一些syslog 守护进程的实现自带有rotate 的功能，如ubox 的logread 、busybox 的

syslogd 。