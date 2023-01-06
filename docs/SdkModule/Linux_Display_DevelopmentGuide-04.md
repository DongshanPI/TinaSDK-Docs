## 4 显示输出设备操作说明

Disp2 支持多种的显示输出设备，LCD、TV、HDMI。开启显示输出设备有几种方式，第一种是在sys_config 或dts 中配置[disp] 的初始化参数，显示模块在加载时

将会根据配置初始化选择的显示输出设备；第二种是在kernel 启动后，调用驱动模块的ioctl 接口去开启或关闭指定的输出设备，以下是操作的说明：

• 开启或切换到某个具体的显示输出设备，ioctl(DISP_DEVICE_SWITCH…），参数设置为特定的输出设备类型，DISP_OUTPUT_TYPE_LCD/TV/HDMI。

• 关闭某个设备，ioctrl(DISP_DEVICE_SWITCH…)，参数设置为DISP_OUTPUT_TYPE_NONE。

## 5 接口参数更改说明

sunxi 平台支持disp1 和disp2。

<center>表5-1: disp1 与disp2 区别</center>

| 项目\平台          | disp2                                              | disp1                     |
| ------------------ | -------------------------------------------------- | ------------------------- |
| 图层标识           | 以disp, chennel, layer_id 唯一标识                 | 以disp, layer_id 唯一标识 |
| 图层开关           | 将开关当成图层参数设置DISP_LAYER_SET_CONFIG 中     | 独立图层开关接口          |
| 图层size           | 每个分量都需要设置1 个size                         | 一个buffer 只有1          |
| 图层align          | 针对每个分量需要设置其align，单位为byte。          | 无                        |
| 图层Crop           | 为64 位定点小数，高32 位为整数，低32位为小数       | 为32 位参数，不支持小数   |
| YUV MB 格式支持    | 不再支持                                           | 支持                      |
| PALETTE 格式支持   | 不再支持                                           | 支持                      |
| 单色模式(无buffer) | 支持                                               | 不支持                    |
| Pipe 选择          | Pipe 对用户透明，用户无需选择，只需要配置channel   | 用户设置                  |
| zorder             | 用户设置，保证zorder 不重复，从0 到N-1             | 用户不能设置              |
| 设置图层信息接口   | 一次可设置多个图层的信息，增加一个图层信息数目参数 | 一次设置1 个图层信息      |