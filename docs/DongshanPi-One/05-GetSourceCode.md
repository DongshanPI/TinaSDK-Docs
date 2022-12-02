# 获取配套sdk源码
如果您感觉跟着文档不知道怎么操作，可以直接查看我们提前录制好的手把手教学视频。
<iframe width="800" height="600"
  src="//player.bilibili.com/player.html?aid=379890992&bvid=BV11Z4y1Q7pL&cid=462871023&page=1">
</iframe>

## 配置开发环境
> 我们的开发环境基于ubuntu-18.04系统 为了避免爬坑浪费您的时间 请尽量和我们保持一致。
### 安装编译需要的lib & tool
接下来安装相应的编译工具等：针对SDK编译需要安装一些tool，否则会编译失败。
``` shell
book@100ask:~$ sudo apt-get install libc6-dev-i386
book@100ask:~$ sudo apt-get install lib32z1 lib32ncurses5
book@100ask:~$ sudo apt-get install libuuid1:i386
book@100ask:~$ sudo apt-get install cmake
book@100ask:~$ sudo apt-get install libncurses5-dev libncursesw5-dev
book@100ask:~$ sudo apt install bc
book@100ask:~$ sudo apt-get install xz-utils
book@100ask:~$ sudo apt-get install automake
book@100ask:~$ sudo apt-get install libtool
book@100ask:~$ sudo apt-get install libevdev-dev
book@100ask:~$ sudo apt-get install pkg-config
book@100ask:~$ sudo apt-get install openssh-server
book@100ask:~$ sudo apt-get install repo
```

或者使用下面这条命令一次全部下载上面的tools：

```shell
sudo apt-get install -y libc6-dev-i386 lib32z1 lib32ncurses5 libuuid1:i386 cmake libncurses5-dev libncursesw5-dev bc xz-utils automake libtool libevdev-dev pkg-config openssh-server repo
```

安装完了即可使用,如上这些需要安装的tool && lib是必须的，因为编译过程中会用到相关内容。这边就不一一说明是哪个错误，可以尝试先不安装编译来查看相关错误，为了方便，在编译之前一次性安装即可。
如果默认sh不是bash，需要将sh改成bash：
``` shell
book@100ask:~$  sudo rm /bin/sh
book@100ask:~$  sudo ln -s /bin/bash /bin/sh
```
## 获取源码
   源码我们使用repo工具来统一管理多个git仓库，方便一键获取和更新，之后使用如下命令进行源码获取，等待获取完成即可
``` shell
book@100ask:~$ git clone https://e.coding.net/codebug8/repo.git
book@100ask:~$ mkdir DongshanPiOne-TAKOYAKI  && cd  DongshanPiOne-TAKOYAKI
book@100ask:~/DongshanPiOne-TAKOYAKI$ ../repo/repo init -u  https://gitee.com/weidongshan/manifests.git -b linux-sdk -m  SSD202D/dongshanpi-one_takoyaki_dlc00v030.xml --no-repo-verify
book@100ask:~/DongshanPiOne-TAKOYAKI$ ../repo/repo sync -j4
```
获取成功后的源码目录结构如下
```shell
book@virtual-machine:~/DongshanPiOne-TAKOYAKI$ tree -L 1
.
├── boot   //使用的是uboot-2015  主要用于烧写引导kernel启动。
├── DevelopmentEnvConf  //自动配置开发环境的一些脚本文件
├── gcc-arm-8.2-2018.08-x86_64-arm-linux-gnueabihf   //交叉编译工具链。
├── kernel  //使用4.9.88版本的内核
├── project  //根文件系统和应用相关源码,主要包含了 编译构建根文件系统 APP 并自动打包生成合适的格式用来烧写。
└── sdk  
```

### 配置编译环境

#### 设置交叉编译工具链
> 交叉编译工具链在嵌入式Linux开发中起到一个非常关键的作用，必须要进行配置，否则就无法进行后续的操作。
> 
> **注意**：这里指定的  PATH=$PATH:/home/book 路径为演示主机的家目录绝对路径 您需要根据自己的家目录绝对路径进行配置，家目录绝对路径可以在终端下使用 cd ~ && pwd 获取。

* 永久生效
如需永久修改，请修改用户配置文件。
注意：如果不会使用vim命令，可以使用图形化的编辑工具，执行：gedit  ~/.bashrc
``` shell
book@100ask:~$ vim  ~/.bashrc
``` 
在行尾添加或修改，加上下面几行：
``` shell
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
export PATH=$PATH:/home/book/DongshanPiOne-TAKOYAKI/gcc-arm-8.2-2018.08-x86_64-arm-linux-gnueabihf/bin
```
设置完毕后，要执行 `source  ~/.bashrc ` 命令使其生效，这条命令是加载这些设置的环境变量。

* 临时生效
注意：手工执行"export"命令设置环境变量，该设置只对当前终端有效(另开一个终端需要再次设置).
``` shell
book@100ask:~$ export ARCH=arm
book@100ask:~$ export CROSS_COMPILE=arm-linux-gnueabihf-
book@100ask:~$ export PATH=$PATH:/home/book/DongshanPiOne-TAKOYAKI/gcc-arm-8.2-2018.08-x86_64-arm-linux-gnueabihf/bin
```

* 验证交叉编译工具链
在终端下执行 arm-linux-gnueabihf-gcc -v命令 有如下类似的信息输出 即表明正常。
``` shell
book@100ask:~$ arm-linux-gnueabihf-gcc -v
Using built-in specs.
COLLECT_GCC=arm-linux-gnueabihf-gcc
COLLECT_LTO_WRAPPER=/home/book/DongshanPiOne-TAKOYAKI/gcc-arm-8.2-2018.08-x86_64-arm-linux-gnueabihf/bin/../libexec/gcc/arm-linux-gnueabihf/8.2.1/lto-wrapper
Target: arm-linux-gnueabihf
Configured with: /tmp/dgboter/bbs/bc-b1-2-11--rhe6x86_64/buildbot/rhe6x86_64--arm-linux-gnueabihf/build/src/gcc/configure --target=arm-linux-gnueabihf --prefix= --with-sysroot=/arm-linux-gnueabihf/libc --with-build-sysroot=/tmp/dgboter/bbs/bc-b1-2-11--rhe6x86_64/buildbot/rhe6x86_64--arm-linux-gnueabihf/build/build-arm-linux-gnueabihf/install//arm-linux-gnueabihf/libc --enable-gnu-indirect-function --enable-shared --disable-libssp --disable-libmudflap --disable-libsanitizer --enable-checking=release --enable-languages=c,c++,fortran --with-gmp=/tmp/dgboter/bbs/bc-b1-2-11--rhe6x86_64/buildbot/rhe6x86_64--arm-linux-gnueabihf/build/build-arm-linux-gnueabihf/host-tools --with-mpfr=/tmp/dgboter/bbs/bc-b1-2-11--rhe6x86_64/buildbot/rhe6x86_64--arm-linux-gnueabihf/build/build-arm-linux-gnueabihf/host-tools --with-mpc=/tmp/dgboter/bbs/bc-b1-2-11--rhe6x86_64/buildbot/rhe6x86_64--arm-linux-gnueabihf/build/build-arm-linux-gnueabihf/host-tools --with-isl=/tmp/dgboter/bbs/bc-b1-2-11--rhe6x86_64/buildbot/rhe6x86_64--arm-linux-gnueabihf/build/build-arm-linux-gnueabihf/host-tools --with-arch=armv7-a --with-fpu=neon --with-float=hard --with-arch=armv7-a --with-pkgversion='GNU Toolchain for the A-profile Architecture 8.2-2018-08 (arm-rel-8.23)'
Thread model: posix
gcc version 8.2.1 20180802 (GNU Toolchain for the A-profile Architecture 8.2-2018-08 (arm-rel-8.23))
book@100ask:~$
```

设置完成交叉编译工具链 接下来就可以开始编译烧写了。
