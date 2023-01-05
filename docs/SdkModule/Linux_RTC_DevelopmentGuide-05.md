## 5 模块使用范例

此demo 程序是打开一个RTC 设备，然后设置和获取RTC 时间以及设置闹钟功能。

```
1 #include <stdio.h> /*标准输入输出定义*/
2 #include <stdlib.h> /*标准函数库定义*/
3 #include <unistd.h> /*Unix 标准函数定义*/
4 #include <sys/types.h>
5 #include <sys/stat.h>
6 #include <fcntl.h> /*文件控制定义*/
7 #include <linux/rtc.h> /*RTC支持的CMD*/
8 #include <errno.h> /*错误号定义*/
9 #include <string.h>
10
11 #define RTC_DEVICE_NAME "/dev/rtc0"
12
13 int set_rtc_timer(int fd)
14 {
15 struct rtc_time rtc_tm = {0};
16 struct rtc_time rtc_tm_temp = {0};
17
18 rtc_tm.tm_year = 2020 - 1900; /* 需要设置的年份，需要减1900 */
19 rtc_tm.tm_mon = 11 - 1; /* 需要设置的月份,需要确保在0-11范围*/
20 rtc_tm.tm_mday = 21; /* 需要设置的日期*/
21 rtc_tm.tm_hour = 10; /* 需要设置的时间*/
22 rtc_tm.tm_min = 12; /* 需要设置的分钟时间*/
23 rtc_tm.tm_sec = 30; /* 需要设置的秒数*/
24
25 /* 设置RTC时间*/
26 if (ioctl(fd, RTC_SET_TIME, &rtc_tm) < 0) {
27 printf("RTC_SET_TIME failed\n");
28 return -1;
29 }
30
31 /* 获取RTC时间*/
32 if (ioctl(fd, RTC_RD_TIME, &rtc_tm_temp) < 0) {
33 printf("RTC_RD_TIME failed\n");
34 return -1;
35 }
36 printf("RTC_RD_TIME return %04d-%02d-%02d %02d:%02d:%02d\n",
37 rtc_tm_temp.tm_year + 1900, rtc_tm_temp.tm_mon + 1, rtc_tm_temp.tm_mday,
38 rtc_tm_temp.tm_hour, rtc_tm_temp.tm_min, rtc_tm_temp.tm_sec);
39 return 0;
40 }
41
42 int set_rtc_alarm(int fd)
43 {
44 struct rtc_time rtc_tm = {0};
45 struct rtc_time rtc_tm_temp = {0};
46
47 rtc_tm.tm_year = 0; /* 闹钟忽略年设置*/
48 rtc_tm.tm_mon = 0; /* 闹钟忽略月设置*/
49 rtc_tm.tm_mday = 0; /* 闹钟忽略日期设置*/
50 rtc_tm.tm_hour = 10; /* 需要设置的时间*/
51 rtc_tm.tm_min = 12; /* 需要设置的分钟时间*/
52 rtc_tm.tm_sec = 30; /* 需要设置的秒数*/
53
54 /* set alarm time */
55 if (ioctl(fd, RTC_ALM_SET, &rtc_tm) < 0) {
56 printf("RTC_ALM_SET failed\n");
57 return -1;
58 }
59
60 if (ioctl(fd, RTC_AIE_ON) < 0) {
61 printf("RTC_AIE_ON failed!\n");
62 return -1;
63 }
64
65 if (ioctl(fd, RTC_ALM_READ, &rtc_tm_temp) < 0) {
66 printf("RTC_ALM_READ failed\n");
67 return -1;
68 }
69
70 printf("RTC_ALM_READ return %04d-%02d-%02d %02d:%02d:%02d\n",
71 rtc_tm_temp.tm_year + 1900, rtc_tm_temp.tm_mon + 1, rtc_tm_temp.tm_mday,
72 rtc_tm_temp.tm_hour, rtc_tm_temp.tm_min, rtc_tm_temp.tm_sec);
73 return 0;
74 }
75
76 int main(int argc, char *argv[])
77 {
78 int fd;
79 int ret;
80
81 /* open rtc device */
82 fd = open(RTC_DEVICE_NAME, O_RDWR);
83 if (fd < 0) {
84 printf("open rtc device %s failed\n", RTC_DEVICE_NAME);
85 return -ENODEV;
86 }
87
88 /* 设置RTC时间*/
89 ret = set_rtc_timer(fd);
90 if (ret < 0) {
91 printf("set rtc timer error\n");
92 return -EINVAL;
93 }
94
95 /* 设置闹钟*/
96 ret = set_rtc_alarm(fd);
97 if (ret < 0) {
98 printf("set rtc alarm error\n");
99 return -EINVAL;
100 }
101
102 close(fd);
103 return 0;
104 }
```

