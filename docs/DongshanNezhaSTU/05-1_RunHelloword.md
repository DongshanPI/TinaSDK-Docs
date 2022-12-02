# 运行输出hello word



## 配置开发环境

首先我们需要获取 东山哪吒STU 开发板 配套的交叉编译工具链。因为最初的工具链是 阿里平头哥提供，他们的工具链 与 GNU社区标准的工具链存在一定的差异，所以我们暂时不能使用 社区版本。

由于目前工具链没有提供windows版本，所以只能在 Linux下进行，操作，请先参考上述章节 配置ubuntu 虚拟机章节，进行配置，并配置好。



## 获取交叉编译工具链

我们的源码都存放在不同的git仓库内，其中以github为主要托管，也是最新的状态，同时也会使用 gitee作为备用站点，根据大家的实际情况，来进行选择。

* 对于可以访问github的同学 请使用如下命令获取源码

```bash
git clone https://github.com/DongshanPI/eLinuxCore_dongshannezhastu
cd  eLinuxCore_dongshannezhastu
git submodule update  --init --recursive
```



* 对于无法访问GitHub的同学 请使用如下命令获取源码。

```bash
git clone https://gitee.com/weidongshan/eLinuxCore_dongshannezhastu.git
cd  eLinuxCore_dongshannezhastu
git submodule update  --init --recursive
```



获取完成源码后，需要进入到交叉编译工具链路径到 内，用于验证是否可用。

```bash
book@virtual-machine:~/eLinuxCore_dongshannezhastu/toolchain/riscv64-glibc-gcc-thead_20200702/bin$ ./riscv64-unknown-linux-gnu-gcc -v
Using built-in specs.
COLLECT_GCC=./riscv64-unknown-linux-gnu-gcc
COLLECT_LTO_WRAPPER=/home/book/NezhaSTU/eLinuxCore_dongshannezhastu/toolchain/riscv64-glibc-gcc-thead_20200702/bin/../libexec/gcc/riscv64-unknown-linux-gnu/8.1.0/lto-wrapper
Target: riscv64-unknown-linux-gnu
Configured with: /ldhome/software/toolsbuild/slave/workspace/riscv64_build_linux_x86_64/build/../source/riscv/riscv-gcc/configure --target=riscv64-unknown-linux-gnu --with-mpc=/ldhome/software/toolsbuild/slave/workspace/riscv64_build_linux_x86_64/lib-for-gcc-x86_64-linux/ --with-mpfr=/ldhome/software/toolsbuild/slave/workspace/riscv64_build_linux_x86_64/lib-for-gcc-x86_64-linux/ --with-gmp=/ldhome/software/toolsbuild/slave/workspace/riscv64_build_linux_x86_64/lib-for-gcc-x86_64-linux/ --prefix=/ldhome/software/toolsbuild/slave/workspace/riscv64_build_linux_x86_64/install --with-sysroot=/ldhome/software/toolsbuild/slave/workspace/riscv64_build_linux_x86_64/install/sysroot --with-system-zlib --enable-shared --enable-tls --enable-languages=c,c++,fortran --disable-libmudflap --disable-libssp --disable-libquadmath --disable-nls --disable-bootstrap --src=../../source/riscv/riscv-gcc --enable-checking=yes --with-pkgversion='C-SKY RISCV Tools V1.8.4 B20200702' --enable-multilib --with-abi=lp64d --with-arch=rv64gcxthead 'CFLAGS_FOR_TARGET=-O2  -mcmodel=medany' 'CXXFLAGS_FOR_TARGET=-O2  -mcmodel=medany' CC=gcc CXX=g++
Thread model: posix
gcc version 8.1.0 (C-SKY RISCV Tools V1.8.4 B20200702)

```

完成以后 我们就可以加入到 系统的 PATH环境变量内。

首先 需要获取 交叉编译工具链 所在的绝对路径，进入到  <code>eLinuxCore_dongshannezhastu/toolchain/riscv64-glibc-gcc-thead_20200702/bin</code>目录下执行 **pwd** 命令，即可得到绝对路径 ` /home/book/eLinuxCore_dongshannezhastu/toolchain/riscv64-glibc-gcc-thead_20200702/bin` 。

```bash
book@virtual-machine:~/eLinuxCore_dongshannezhastu/toolchain/riscv64-glibc-gcc-thead_20200702/bin$ pwd
/home/book/eLinuxCore_dongshannezhastu/toolchain/riscv64-glibc-gcc-thead_20200702/bin
```

接下来，可以在终端下执行如下命令，讲这个加入到系统 环境变量内，这样就可以在任意位置执行  交叉编译工具链了。

```bash
export PATH=$PATH:/home/book/eLinuxCore_dongshannezhastu/toolchain/riscv64-glibc-gcc-thead_20200702/bin
```

注意：此方式只针对当前的终端有效，如果你关闭了这个终端，再次开启终端 需要重新执行才可以。

还有另一种永久生效的方式 就是写入到 系统环境变量里面，需要修改  **/etc/environment** 在末尾加上 你获取到的交叉编译工具链绝对路径,注意修改需要使用 sudo 命令。

```bash
book@virtual-machine:~$ cat /etc/environment
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/home/book/eLinuxCore_dongshannezhastu/toolchain/riscv64-glibc-gcc-thead_20200702/bin"
```



## 编写Hello word程序

* 配置好交叉编译工具链以后，就可以开始编写我们的应用程序了，如下为一个最简单的 hello word打印示例程序。

```c
#include <stdio.h>
int main (void)
{
    printf("hello word!\n");
    return 0;
}    
```

编写完成后，保存到 helloword.c

之后我们执行 如下编译命令进行编译 

```bash
book@virtual-machine:~$ vim helloword.c 
book@virtual-machine:~$ riscv64-unknown-linux-gnu-gcc -o helloword helloword.c
book@virtual-machine:~$ file helloword
helloword: ELF 64-bit LSB executable, UCB RISC-V, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-riscv64xthead-lp64d.so.1, for GNU/Linux 4.15.0, with debug_info, not stripped
```

## 拷贝到开发板

怎么拷贝文件到开发板上？ 有U盘  ADB 网络 串口等等。

那么我们优先推进使用 网络方式，网络也有很多，有TFTP传输，有nfs传输，有SFTP传输，其中nfs传输需要内核支持 nfs文件系统，SFTP需要根文件系统支持 openssh组件服务，那么最终我们还是选用tftp服务。

### 使用tftp网络服务

1. 首先，需要你的ubuntu系统支持 tftp服务，已经配置并且安装好，然后讲编译出来的 helloword程序 拷贝到 tftp目录下。

```bash
book@virtual-machine:~$ cp helloword ~/tftpboot/
book@virtual-machine:~$ ls ~/tftpboot/helloword
/home/book/tftpboot/helloword
book@virtual-machine:~$
```

2. 进入到开发板内，首先让开发板可以获取到IP地址，并且可以和 ubuntu系统ping通(这里指的是编译helloword主机)，之后我们在开发板上 获取 helloword 应用程序，并执行。

```bash
# udhcpc
udhcpc: started, v1.35.0
udhcpc: broadcasting discover
udhcpc: broadcasting select for 192.168.1.47, server 192.168.1.1
udhcpc: lease of 192.168.1.47 obtained from 192.168.1.1, lease time 86400
deleting routers
adding dns 192.168.1.1
# tftp -g -r helloword 192.168.1.133
# ls
helloword

```

如上所示，我的ubuntu主机IP地址是 192.168.1.133 ，所以使用tftp 从 ubuntu获取helloword 程序，获取速度根据网速而定。



### 使用usb adb方式

* 后面我们将会介绍如何使用 usb otg  adb命令传输文件。





## 运行

下载好程序以后，需要使用chmod +x 命令来给程序添加可执行权限，之后 我们就可以执行 这个helloword应用了。

```bash
# chmod +x helloword
# ./helloword
hello word!
#
```

