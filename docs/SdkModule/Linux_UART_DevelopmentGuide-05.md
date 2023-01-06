## 5 模块使用范例

此 demo 程序是打开一个串口设备，然后侦听这个设备，如果有数据可读就读出来并打印。设备名称、侦听的循环次数都可以由参数指定。

```
#include <stdio.h>      /*标准输入输出定义*/
#include <stdlib.h>     /*标准函数库定义*/
#include <unistd.h>     /*Unix 标准函数定义*/
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>       /*文件控制定义*/
#include <termios.h>     /*PPSIX 终端控制定义*/
#include <errno.h>       /*错误号定义*/
#include <string.h>

enum parameter_type {
    PT_PROGRAM_NAME = 0,
    PT_DEV_NAME,
    PT_CYCLE,
    PT_NUM
};
 
#define DBG(string, args...) \
    do { \
        printf("%s, %s()%u---", __FILE__, __FUNCTION__, __LINE__); \
        printf(string, _00args); \
        printf("\n"); \
    } while (0)

void usage(void)
{
    printf("You should input as: \n");
    printf("\t select_test [/dev/name] [Cycle Cnt]\n");
}

int OpenDev(char *name)
{
    int fd = open(name, O_RDWR ); //| O_NOCTTY | O_NDELAY
    if (-1 == fd)
	    DBG("Can't Open(%s)!", name);

    return fd;
}

/**
 * @brief 设置串口通信速率
 * @param fd 类型 int 打开串口的文件句柄
 * @param speed 类型 int 串口速度
 * @return void
 */
void set_speed(int fd, int speed)
{
    int i;
    int status;
    struct termios Opt = {0};
    int speed_arr[] = { B38400, B19200, B9600, B4800, B2400, B1200, B300,
    	B38400, B19200, B9600, B4800, B2400, B1200, B300, };
    int name_arr[] = {38400, 19200, 9600, 4800, 2400, 1200, 300, 38400,
    	19200, 9600, 4800, 2400, 1200, 300, };

    tcgetattr(fd, &Opt);

    for ( i= 0; i < sizeof(speed_arr) / sizeof(int); i++) {
        if (speed == name_arr[i])
        break;
    }

    tcflush(fd, TCIOFLUSH);
    cfsetispeed(&Opt, speed_arr[i]);
    cfsetospeed(&Opt, speed_arr[i]);

    Opt.c_lflag &= ~(ICANON | ECHO | ECHOE | ISIG); /*Input*/
    Opt.c_oflag &= ~OPOST; /*Output*/

    status = tcsetattr(fd, TCSANOW, &Opt);
    if (status != 0) {
        DBG("tcsetattr fd");
        return;
    }
    tcflush(fd, TCIOFLUSH);
}

/**
 *@brief 设置串口数据位，停止位和效验位
 *@param fd 类型 int 打开的串口文件句柄
 *@param databits 类型 int 数据位 取值 为 7 或者8
 *@param stopbits 类型 int 停止位 取值为 1 或者2
 *@param parity 类型 int 效验类型 取值为N,E,O,,S
 */
int set_Parity(int fd,int databits,int stopbits,int parity)
{
    struct termios options;

    if ( tcgetattr(fd, &options) != 0) {
        perror("SetupSerial 1");
        return -1;
    }
    options.c_cflag &= ~CSIZE;

    switch (databits) /*设置数据位数*/
    {
        case 7:
            options.c_cflag |= CS7;
            break;
        case 8:
            options.c_cflag |= CS8;
            break;
        default:
            fprintf(stderr,"Unsupported data size\n");
            return -1;
    }
    
    switch (parity)
    {
        case 'n':
        case 'N':
            options.c_cflag &= ~PARENB; /* Clear parity enable */
            options.c_iflag &= ~INPCK; /* Enable parity checking */
            break;
        case 'o':
        case 'O':
            options.c_cflag |= (PARODD | PARENB); /* 设置为奇效验*/
            options.c_iflag |= INPCK; /* Disnable parity checking */
            break;
        case 'e':
        case 'E':
            options.c_cflag |= PARENB; /* Enable parity */
            options.c_cflag &= ~PARODD; /* 转换为偶效验*/
            options.c_iflag |= INPCK; /* Disnable parity checking */
            break;
        case 'S':
        case 's': /*as no parity*/
            options.c_cflag &= ~PARENB;
            options.c_cflag &= ~CSTOPB;break;
        default:
            fprintf(stderr,"Unsupported parity\n");
            return -1;
    }
    
    /* 设置停止位*/
    switch (stopbits)
    {
        case 1:
            options.c_cflag &= ~CSTOPB;
            break;
        case 2:
            options.c_cflag |= CSTOPB;
            break;
        default:
            fprintf(stderr,"Unsupported stop bits\n");
            return -1;
    }
    
    /* Set input parity option */
    if (parity != 'n')
    	options.c_iflag |= INPCK;
    tcflush(fd,TCIFLUSH);
    options.c_cc[VTIME] = 150; /* 设置超时15 seconds*/
    options.c_cc[VMIN] = 0;    /* Update the options and do it NOW */
    if (tcsetattr(fd,TCSANOW,&options) != 0)
    {
        perror("SetupSerial 3");
        return -1;
    }
    return 0;
}

void str_print(char *buf, int len)
{
    int i;

    for (i=0; i<len; i++) {
        if (i%10 == 0)
            printf("\n");

        printf("0x%02x ", buf[i]);
    }
    printf("\n");
}

int main(int argc, char **argv)
{
    int i = 0;
    int fd = 0;
    int cnt = 0;
    char buf[256];

    int ret;
    fd_set rd_fdset;
    struct timeval dly_tm; // delay time in select()

    if (argc != PT_NUM) {
        usage();
        return -1;
    }

    sscanf(argv[PT_CYCLE], "%d", &cnt);
    if (cnt == 0)
        cnt = 0xFFFF;

    fd = OpenDev(argv[PT_DEV_NAME]);
    if (fd < 0)
        return -1;

    set_speed(fd,19200);
    if (set_Parity(fd,8,1,'N') == -1) {
        printf("Set Parity Error\n");
        exit (0);
    }

    printf("Select(%s), Cnt %d. \n", argv[PT_DEV_NAME], cnt);
    while (i<cnt) {
        FD_ZERO(&rd_fdset);
        FD_SET(fd, &rd_fdset);

        dly_tm.tv_sec = 5;
        dly_tm.tv_usec = 0;
        memset(buf, 0, 256);

        ret = select(fd+1, &rd_fdset, NULL, NULL, &dly_tm);
        // DBG("select() return %d, fd = %d", ret, fd);
        if (ret == 0)
            continue;

        if (ret < 0) {
            printf("select(%s) return %d. [%d]: %s \n", argv[PT_DEV_NAME], ret, errno,
            strerror(errno));
            continue;
        }

        i++;
        ret = read(fd, buf, 256);

        printf("Cnt%d: read(%s) return %d.\n", i, argv[PT_DEV_NAME], ret);
        str_print(buf, ret);
    }

    close(fd);
    return 0;
}

```

