## 7 SELinux

SELinux (Security-Enhanced Linux) 是美国国家安全局（NSA）对强制访问控制（MAC，Mandatory Access Control）的实现。
强制访问控制是相对于Linux 传统的自主访问控制（DAC,Discretionary Access Control）一种访问控制机制。
DAC 控制的主体是用户，其最大的缺点是它无法分离用户与进程，进程能够继承用户的访问控制。由于程序是存在漏洞的，一旦被入侵，则入侵者具有该用户在系统上的所有权限。
MAC 中所有的访问权限是由访问控制策略来定义的，用户无法超越策略的限制。SELinux 以最小权限原则（principle of least privilege）为基础，在策略之外的访问都是无权的。

### 7.1 基本概念

#### 7.1.1 主体Subject

可完全等同于进程。

#### 7.1.2 客体Object

被主体访问的系统资源。可以是文件、目录、共享内存、套接字、端口、设备等。

#### 7.1.3 安全上下文Secure Context

SELinux 对主体与客体的标记称做安全上下文，也称为安全标签，或标签。安全上下文的构成是一个四元组，包含User：Role：Type：MLS/MCS。每个字段都可以用来决定访问控制，其中
最重要的是Type，大多数Policy 都是针对Type 来制定的。进程安全上下文被记录在task_struct 中，客体安全上下文来源是文件的扩展属性（xattr）。

#### 7.1.4 策略Policy

安全策略是使用策略语言编写的具体的访问控制规则。

主体访问客体时，SELinux 会根据安全策略来判断访问是否允许。
如策略语句：allow init sshd_exec_t:file {open read execute}; 表明允许init 类型的主体对sshd_exec_t 类型的客体执行file 的open/read/execute 操作。
策略使用策略语言编写的。为了让策略语言起作用，需要用相关的用户态工具将策略语言编译成二进制文件，通过selinuxfs 接口，将二进制文件所表示的策略输入到Security Server 中。

#### 7.1.5 SELinux 的运行模式

SELinux 共有三种运行模式：
• enforcing，默认值，表示会强制禁止违反策略的访问。
• permissive，表示不强制禁止，违反策略会生成一条拒绝访问的信息。
• disable，禁用SELinux。

### 7.2 LSM 框架

LSM（Linux Security Module）是内核为支持不同安全机制的实现所设计的一个通用访问控制框架。目前LSM 框架下访问决策模块包括SELinux、SAMCK、tomoyo、yama、apparmor。

![image-20230103110205745](https://photos.100ask.net/Tina-Sdk/Linux_Security_DevGuide_image-20230103110205745.png)

SELinux 的决策流程如上图所示。策略管理工具将策略文件载入到内核中的Security server中。当主体访问客体时，首先进行DAC 检测，通过后再进行MAC 检测。

在Security server 与客体管理器之间有一个缓存AVC（Access Vector Cache），用来提高检测效率。主体发出访问客体的请求时，内核中的客体管理器首先查看AVC，如果AVC 中有缓存策略决策结果，根据缓存情况执行放行或拒绝；如果AVC 中没有缓存，安全服务器在策略中查找规则，按规则执行放行或拒绝的决策，策略决策的结果缓存到AVC 中。SELinux 的策略是允许策略，在策略中没有定义的访问都是被禁止的。

#### 7.3 Tina SELinux 开启

Linux 主线很早就包含了SELinux 的实现，Tina 上主要是集成了SELinux 相关库、调试工具、参考策略以及策略加载等组件。
• 库：libsepol、libselinux、libaudit、libcap-ng、libsemanage 等
• 调试工具：policycoreutils、checkpolicy、audit、selinux-python 等。
• 参考策略：refpolicy 与sepolicy。
• 策略加载：busybox 与procd 的init 进程中执行策略加载与安全上下文设置。

**警告**
**我们没有提供一个专门为tina 系统定制的selinux policy， 只是简单的使用refpolicy 和sepolicy（基于android），用户需要根据产品需求开发合适的策略。**

#### 7.3.1 menuconfig 配置

进入Tina 根目录，执行make menuconfig 进入配置主界面，开启如下配置（以refpolicy 为例，最小配置）。

```
Tina Configuration
	Global build settings --->
		[*] NSA SELinux Support
			choose the SELinux Policy (the reference policy) --->
		[*] Compile the kernel with device tmpfs enabled
		[*] Automatically mount devtmpfs after root filesystem is mounted
Base system --->
		<*> busybox --->
			[*] Customize busybox options
			Busybox Settings --->
				[*] Support NSA Security Enhanced Linux
				What kind of applet links to install (as script wrappers) --->
				/bin/sh applet link (as script wrapper) --->
Security --->
	SELINUX --->
		<*> refpolicy
```

可以根据需要加入相关调试工具，如checkpolicy、policycoreutils 等。

#### 7.3.2 kernel_menuconfig 配置

在命令行中进入Tina 根目录，执行make kernel_menuconfig 进入配置主界面，新增如下配置。

```
Linux Kernel Configuration
	General setup --->
		[*] Auditing support
File systems --->
	<*> The Extended 4 (ext4) filesystem
		[*] Ext4 extended attributes
		[*] Ext4 Security Labels
	[*] Miscellaneous filesystems --->
		<*> SquashFS 4.0 - Squashed file system support
			[*] Squashfs XATTR support
Security options --->
	[*] Enable different security models
	[*] Socket and Networking Security Hooks
	[*] NSA SELinux Support
	[*] NSA SELinux boot parameter
	(1) NSA SELinux boot parameter default value (NEW)
	[*] NSA SELinux runtime disable
	[*] NSA SELinux Development Support
	[*] NSA SELinux AVC Statistics
	(1) NSA SELinux checkreqprot default value
Default security module (SELinux) --->
```

注意，不同内核版本可能有细微差异，另上面只列举了ext4/squashfs 文件系统的配置，如果想支持更多的文件系统，请打开对应文件系统的xattr 的支持。
对于Linux-5.4 版本内核，Selinux 配置还需要新增如下配置，在“Ordered list of enabled LSMs” 选项中加入关键字selinux。

```
Linux Kernel Configuration
	Security options --->
	First legacy 'major LSM' to be initialized (SELinux) --->
        (selinux,lockdown,yama,loadpin,safesetid,integrity) Ordered list of enabled LSMs
```

#### 7.3.3 SELinux 初始化

系统启动时，在init 进程里，会加载策略文件、文件上下文到系统中，同时根据加载的策略文件初始化系统的安全上下文。具体的初始化过程需要根据实际情况进行适配，当前Tina 启动后相关log 如下所示。

```
[ 3.734518] device_chose finished 122!
[ 5.124774] SELinux: 32768 avtab hash slots, 117923 rules.
[ 5.199800] SELinux: 32768 avtab hash slots, 117923 rules.
[ 5.234633] SELinux: 6 users, 176 roles, 4773 types, 317 bools
[ 5.241333] SELinux: 129 classes, 117923 rules
[ 5.262431] SELinux: Completing initialization.
[ 5.267662] SELinux: Setting up existing superblocks.
[ 5.330382] audit: type=1403 audit(5.280:3): policy loaded auid=4294967295 ses
=4294967295
SELinux policy load success!
set init context success!
/etc/selinux/targeted/contexts/files/file_contexts load success!
```

启动后，可以执行sestatus 查看当前selinux 状态。

```
root@TinaLinux:/# sestatus
SELinux status: enabled
SELinuxfs mount: /sys/fs/selinux
SELinux root directory: /etc/selinux
Loaded policy name: targeted
Current mode: enforcing
Mode from config file: enforcing
Policy MLS status: disabled
Policy deny_unknown status: denied
Memory protection checking: requested (insecure)
Max kernel policy version: 30
```

查看文件安全上下文。

```
root@TinaLinux:/# ls -Z
system_u:object_r:bin_t bin
system_u:object_r:device_t dev
system_u:object_r:etc_t etc
system_u:object_r:lib_t lib
system_u:object_r:mnt_t mnt
system_u:object_r:unlabeled_t overlay
system_u:object_r:proc_t proc
system_u:object_r:default_t rdinit
system_u:object_r:root_t rom
root:object_r:user_home_dir_t root
system_u:object_r:bin_t sbin
system_u:object_r:sysfs_t sys
system_u:object_r:tmpfs_t tmp
system_u:object_r:usr_t usr
system_u:object_r:default_t var
system_u:object_r:default_t www
```

查看进程安全上下文。

```
root@TinaLinux:/# ps -Z
PID CONTEXT STAT COMMAND
1537 system_u:system_r:kernel_t SW [RTWHALXT]
1549 system_u:system_r:initrc_t S /sbin/swupdate-progress -w
1568 system_u:system_r:init_t S< {ntpd} /bin/busybox /usr/sbin/ntpd
1618 system_u:system_r:sysadm_t R {ps} /bin/busybox /bin/ps -Z
```

### 7.4 策略开发

SELinux 的策略是允许策略，在策略中没有定义的操作都是被禁止的。所以对于一个新程序，需要新增对应的策略。

本节以Tina 上sepolicy-demo 策略为基础，举例说明如何新增策略来实现需要的强制访问控制。

#### 7.4.1 限制主体的权限

要求：限制进程fork_test 不能fork 子进程。

##### 7.4.1.1 fork_test 源代码

写一个简单包含fork 的程序，命名为fork_test.c，编译完成后，生成fork_test 的二进制文件，存放在/usr/bin/下。

```
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
int main()
{
pid_t pid = fork();
if(pid < 0)
printf("fork error\n");
else if(pid == 0)
printf("this is child\n");
else
printf("this is parent, pid = %d\n", pid);
return 0;
}
```

##### 7.4.1.2 添加策略

首先，定义该文件的安全上下文。在tina/package/security/sepolicy-demo/src/file_contexts文件中新增如下行，表明/usr/bin/fork_test 的安全上下文是u:object_r:fork_test_exec:s0

```
/usr/bin/fork_test u:object_r:fork_test_exec:s0
```

其次，定义该进程的安全上下文，以及该进程访问资源的权限。新增tina/package/security/sepolicydemo/src/fork_test.te 文件，其内容如下：

```
type fork_test, domain;
type fork_test_exec, exec_type, file_type;
# permissive fork_test;
init_daemon_domain(fork_test)
domain_auto_trans(shell,fork_test_exec,fork_test)
allow fork_test serial_device:chr_file rw_file_perms;
allow fork_test shell:fd use;
neverallow fork_test self:process fork;
```

• 第一句，定义一个fork_test 的类型，其属性集是domain。Sepolicy 中domain 表示进程属性集。
• 第二句，定义一个fork_test_exec 的类型，其属性集是exec_type 和file_type。表明该类型即是一个可执行的类型，又是一个文件类型。属性集的好处是可以对属性集下的组进行统一处理。
• 第三句，"#permissive fork_test"，前面的# 表示注释。如果不注释，表示fork_test 在执行的时候，如果没有相关的权限也能继续执行，同时会打印一条提示信息，这在开发阶段有好处，因为很难一开始就知道该主体需要多少权限。但是在最终产品上，不允许任何的permissive，否则就起不到强制访问控制的作用了。
• 第四句，init_daemon_domain(fork_test)，这里的init_daemon_domain 是一个宏，该宏的定义位于te_macros 文件中。这句主要就表明允许init 类型的进程执行fork_test_exec 类型的文件，新进程的安全上下文是fork_test。
• 第五句，domain_auto_trans(shell,fork_test_exec,fork_test)，表示shell 类型的进程执行fork_test_exec 类型的文件，新进程的安全上下文自动变为是fork_test。注：如果不进行设置，默认情况下，子进程的安全上下文继承父进程的安全上下文。
• 第六句，allow fork_test serial_device:chr_file rw_file_perms; 允许fork_test 类型的进程对serial_device 的客体进行chr_file 的rw 操作。因为程序中有打印，所以需要添加对串口设备的读写权限。
• 第七句，allow fork_test shell:fd use; fd，文件描述符，表示当进程执行完后，domain 改变时继承fd 的权限。
• 第八句，neverallow fork_test self:process fork; 是不允许fork_test 对自己进行process的fork 操作。

新增完策略之后，编译，提示有一个冲突。如下图所示，原本在domain.te 文件中允许domain对自己有process 的fork 操作，但是在fork_test.te 中又不允许，所以有冲突，将这里的domain减去fork_test。

![image-20230103115326454](https://photos.100ask.net/Tina-Sdk/Linux_Security_DevGuide_image-20230103115326454.png)

##### 7.4.1.3 测试

如下图所示，/usr/bin/fork_test 与/usr/bin/fork_test_bak 内容完全一样，安全上下文有差异。

![image-20230103115347951](https://photos.100ask.net/Tina-Sdk/Linux_Security_DevGuide_image-20230103115347951.png)

fork_test 执行时提示错误，错误信息中的scontext 表示主体的上下文，tcontext 表示客体的安全上下文，tclass 表示操作的类型，fork 表示具体的操作，permissive=0 表示上述的操作不被允许。（如果fork_test.te 中包含了permissive fork_test; 那么这里也会出现提示语句，但是permissive=1，表示允许fork）。
fork_test_bak 文件的类型是system_file，shell 执行该文件时，新进程的安全上下文就是shell，shell 可以对自己进行fork 操作，所以执行成功。

#### 7.4.2 限制客体的访问权限

要求：限制只有特定进程sunxi_info 才能访问/dev/sunxi_soc_info。

##### 7.4.2.1 sunxi_info 源代码

```
#include<stdio.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#define CHECK_SOC_SECURE_ATTR 0x00
#define CHECK_SOC_VERSION 0x01
#define CHECK_SOC_BONDING 0x03
int main(void)
{
int fd = 0;
int cmd;
int arg = 0;
char Buf[4096];
fd = open("/dev/sunxi_soc_info",O_RDWR);
if(fd < 0)
{
printf("Open Dev Mem Erro\n");
return -1;
}
cmd = CHECK_SOC_SECURE_ATTR;
if(ioctl(fd,cmd,&arg) < 0)
{
printf("call cmd CHECK_SOC_SECURE_ATTR fail\n");
return -1;
}
printf("CHECK_SOC_SECURE_ATTR get data is 0x%x\n\n",arg);
cmd = CHECK_SOC_VERSION;
if(ioctl(fd,cmd,&arg) < 0)
{
printf("call cmd CHECK_SOC_VERSION fail\n");
return -1;
}
printf("CHECK_SOC_VERSION get data is 0x%x\n\n",arg);
cmd = CHECK_SOC_BONDING;
if(ioctl(fd,cmd,&Buf) < 0)
{
printf("call cmd CHECK_SOC_BONDING fail\n");
return -1;
}
printf("CHECK_SOC_BONDING get data is %s\n\n",Buf);
close(fd);
return 0;
}
```

##### 7.4.2.2 添加策略

在tina/package/security/sepolicy-demo/src/file_contexts 文件中新增如下两行：

```
/usr/bin/sunxi_info u:object_r:sunxi_info_exec:s0
/dev/sunxi_soc_info u:object_r:sunxi_info_device:s0
```

在tina/package/security/sepolicy-demo/src/device.te 文件中新增如下行：

```
type sunxi_info_device, dev_type;
```

新增tina/package/security/sepolicy-demo/src/sunxi_info.te 文件，内容如下：

```
type sunxi_info, domain;
type sunxi_info_exec, exec_type, file_type;
init_daemon_domain(sunxi_info)
domain_auto_trans(shell,sunxi_info_exec,sunxi_info)
allow sunxi_info { serial_device sunxi_info_device }:chr_file rw_file_perms;
allow sunxi_info shell:fd use;
```

##### 7.4.2.3 测试

![image-20230103115539968](https://photos.100ask.net/Tina-Sdk/Linux_Security_DevGuide_image-20230103115539968.png)

测试结果，/usr/bin/sunxi_info 与/usr/bin/sunxi_info_bak 文件内容一样，安全上下文不同，只有/usr/bin/sunxi_info 才能正确执行。/usr/bin/sunxi_info_bak 执行的时候，提示shell 对sunxi_info_device 设备没有chr_file 的read 与write 权限。
最后一条指令"ls /dev/sunxi_soc_info"执行也失败，提示shell 对sunxi_info_device 设备没有chr_file 的getattr 权限（获取属性权限）。