# MPP_Sample 类别
## 6 类别

本章节主要介绍 MPP sample 的分类情况。 

参考 MPP Middleware sample 目录下的文档 SampleManual.md ，目前主要分成以下几类： 

• 视频

• 音频 

• ISE 和 EIS 

• 视频显示 

• G2D 

• CE 

• UVC 和 UAC 

• 多媒体文件

• AI demo 

• 其他 

【视频】

• sample_driverVipp 演示直接调用 linux 内核驱动获取 frame 

• sample_virvi 视频采集 

• sample_virvi2vo 演示视频采集预览 

• sample_virvi2vo_zoom 演示缩放 

• sample_vi_reset 演示 mpi_vi 组件的 reset 流程

• sample_isposd 测试 isp 相关

• sample_vin_isp_test 通过调用 isp 接口抓图的 demo 

• sample_region 测试 osd

• sample_venc 视频编码

• sample_venc2muxer 演示视频编码和封装 mp4

• sample_virvi2venc 演示采集到编码

• sample_timelapse 演示缩时录影 

• sample_virvi2venc2muxer 演示采集到编码到封装 mp4

• sample_multi_vi2venc2muxer 演示多路编码 

• sample_rtsp 演示视频编码后的 rtsp 传输

• sample_CodecParallel 演示同编同解 

• sample_vdec 视频解码

• sample_demux2vdec 演示解封装和解码

• sample_demux2vdec_saveFrame 从原始文件中分离出视频数据帧并解码生成 yuv 文件 

• sample_demux2vdec2vo 演示解码显示 

• sample_vencQpMap 演示视频编码 QPMAP 模式 

• sample_OnlineVenc 在线编码 

• sample_vencGdcZoom 编码 GDC 数字变焦 

• sample_takePicture 演示单拍和连拍 

• sample_recorder 演示四路录制编码封装或者预览显示 

• sample_vencRecreate 演示动态配置编码格式、帧率、码率等功能 

【音频】 

• sample_ai 演示音频采集 

• sample_ao 演示音频输出 

• sample_aoSync 演示采用同步的方式 send pcm frame。而 sample_ao 是采用异步的方式 

• sample_ao_resample_mixer 演示读取 pcm 数据，然后播放声音，从耳机口输出声音 

• sample_ao2ai_aec 回声消除，初版 

• sample_ao2ai_aec_rate_mixer 回声消除，初版基础上测试打开音频输出重采样和混音功能 后，回声消除是否正常 

• sample_aec 回声消除，修改版 

• sample_aenc 音频编码 

• sample_ai2aenc 音频采集和编码 

• sample_ai2aenc2muxer 音频采集、编码和封装 

• sample_select 演示多路音频编码，用 select() 方式获取编码码流 

• sample_adec 音频解码 

• sample_adec2ao 音频解码输出 

• sample_demux2adec 从原始文件中分离出音频数据帧并解码 

• sample_demux2adec2ao 从原始文件中分离出音频数据帧并解码输出音频 

【ISE 和 EIS】 

• sample_fish 演示单目鱼眼功能 

• sample_virvi2fish2venc 演示鱼眼图像畸形校正后做编码 

• sample_virvi2fish2vo 演示鱼眼图像畸形校正后预览 

• sample_virvi2eis2venc 演示防抖功能 

• sample_ise_dzoom 演示 ise 缩放功能 

• sample_gdc_dzoom 演示 gdc 缩放功能 

【视频显示】 

• sample_vo 演示视频 YUV 预览 

• sample_UILayer 验证 UILayer 的格式

【G2D】 

• sample_g2d 演示直接用 G2D 处理 YUV 文件做透明叠加、旋转和镜像、缩放等 

• sample_vi_g2d 演示从 VI 获取视频帧，用 G2D 做旋转、裁剪、缩放等处理后送 VO 显示 

【CE】 

• sample_twinchn_virvi2venc2ce 两路 ce 加解密测试 

• sample_virvi2venc2ce 单路 ce 加解密测试 

【UVC 和 UAC】 

• sample_uvc2vdec_vo 做主，mjpeg 解码显示 

• sample_uvc2vdenc2vo 做主，mjpeg 解码显示 

• sample_uvc2vo 做主，yuv 显示 

• sample_uvc_vo 做主，yuv 显示 

• sample_uvcout 做从，mjpeg 编码输出 

• sample_uac 演示 uac 测试 

• sample_usbcamera 测试 UVC、UAC 复合设备 

【多媒体文件】 

• sample_demux 解复用 mp4 文件 

• sample_file_repair 修复 mp4 文件 

【AI demo】 

• sample_odet_demo 目标检测演示示例 

• sample_RegionDetect 演示区域检测 

【其他】 

• sample_glog 演示 glog 库的用法 

• sample_hello 演示 MPP helloworld 程序 

• sample_pthread_cancel 测试 pthread_cancel() 

• sample_motor 电机测试 

• sample_sound_controler 语音识别 

• sample_ai_Demo_0.9.2 智能算子 demo 

• sample_ai_Demo_0.9.6 智能算子 demo

• sample_thumb 缩略图 

• sample_nna 人形检测和人脸检测 

• sample_MotionDetect 移动侦测 

• sample_PersonDetect 人形检测 

• sample_directIORead 演示使用 directIO 方式读文件
