## 7 状态

本章节主要介绍当前各 MPP sample 的状态，即：分别在 Tina 和 Melis 各平台方案上的支持情况。

###  7.1 Tina 各平台方案上 MPP sample 支持情况

说明 

​	标注 “Y” 则表示该 sample 支持在该平台方案上测试，未标注则表示不支持。

| MPP sample                   | V853 | V833 | V536 | V533 | V316 |
| ---------------------------- | ---- | ---- | ---- | ---- | ---- |
| 总数                         | 61   | 54   | 39   | 45   | 33   |
| 视频                         | 26   | 19   | 13   | 17   | 11   |
| sample_driverVipp            | Y    | Y    |      | Y    |      |
| sample_virvi                 | Y    | Y    | Y    | Y    | Y    |
| sample_virvi2vo              | Y    | Y    | Y    | Y    | Y    |
| sample_virvi2vo_zoom         | Y    | Y    |      |      |      |
| sample_vi_reset              | Y    | Y    |      | Y    |      |
| sample_isposd                | Y    | Y    |      | Y    |      |
| sample_vin_isp_test          | Y    | Y    |      | Y    |      |
| sample_region                | Y    | Y    | Y    | Y    | Y    |
| sample_venc                  | Y    | Y    | Y    | Y    | Y    |
| sample_venc2muxer            | Y    | Y    | Y    | Y    | Y    |
| sample_virvi2venc            | Y    | Y    | Y    | Y    | Y    |
| sample_timelapse             | Y    | Y    | Y    | Y    |      |
| sample_virvi2venc2muxer      | Y    | Y    | Y    | Y    | Y    |
| sample_multi_vi2venc2muxer   | Y    | Y    |      |      |      |
| sample_rtsp                  | Y    | Y    | Y    | Y    |      |
| sample_CodecParallel         | Y    | Y    |      |      |      |
| sample_vdec                  | Y    | Y    | Y    | Y    | Y    |
| sample_demux2vdec            | Y    | Y    | Y    | Y    | Y    |
| sample_demux2vdec_saveFrame  | Y    | Y    | Y    | Y    | Y    |
| sample_demux2vdec2vo         | Y    | Y    | Y    | Y    | Y    |
| sample_vencQpMap             | Y    |      |      |      |      |
| sample_OnlineVenc            | Y    |      |      |      |      |
| sample_vencGdcZoom           | Y    |      |      |      |      |
| sample_takePicture           | Y    |      |      |      |      |
| sample_recorder              | Y    |      |      |      |      |
| 音频                         | 14   | 14   | 9    | 10   | 9    |
| sample_ai                    | Y    | Y    | Y    | Y    | Y    |
| sample_ao                    | Y    | Y    | Y    | Y    | Y    |
| sample_aoSync                | Y    | Y    |      |      |      |
| sample_ao_resample_mixer     | Y    | Y    |      |      |      |
| sample_ao2ai_aec             | Y    | Y    |      | Y    |      |
| sample_ao2ai_aec_rate_mixer  | Y    | Y    |      |      |      |
| sample_aec                   | Y    | Y    |      |      |      |
| sample_aenc                  | Y    | Y    | Y    | Y    | Y    |
| sample_ai2aenc               | Y    | Y    | Y    | Y    | Y    |
| sample_ai2aenc2muxer         | Y    | Y    | Y    | Y    | Y    |
| sample_select                | Y    | Y    | Y    | Y    | Y    |
| sample_adec                  | Y    | Y    | Y    | Y    | Y    |
| sample_demux2adec            | Y    | Y    | Y    | Y    | Y    |
| sample_demux2adec2ao         | Y    | Y    | Y    | Y    | Y    |
| ISE 和 EIS                   | 0    | 4    | 6    | 4    | 5    |
| sample_fish                  |      | Y    | Y    | Y    | Y    |
| sample_virvi2fish2venc       |      | Y    | Y    | Y    | Y    |
| sample_virvi2fish2vo         |      | Y    | Y    | Y    | Y    |
| sample_virvi2eis2venc        |      | Y    | Y    | Y    | Y    |
| sample_ise_dzoom             |      |      | Y    |      |      |
| sample_gdc_dzoom             |      |      | Y    |      | Y    |
| 视频显示                     | 2    | 2    | 1    | 2    | 1    |
| sample_vo                    | Y    | Y    | Y    | Y    | Y    |
| sample_UILayer               | Y    | Y    |      | Y    |      |
| G2D                          | 2    | 2    | 1    | 1    | 1    |
| sample_g2d                   | Y    | Y    | Y    | Y    | Y    |
| sample_vi_g2d                | Y    | Y    |      |      |      |
| CE                           | 2    | 2    | 0    | 2    | 0    |
| sample_twinchn_virvi2venc2ce | Y    | Y    |      | Y    |      |
| sample_virvi2venc2ce         | Y    | Y    |      | Y    |      |
| UVC 和 UAC                   | 7    | 5    | 5    | 5    | 5    |
| sample_uvc2vdec_vo           | Y    | Y    | Y    | Y    | Y    |
| sample_uvc2vdenc2vo          | Y    | Y    | Y    | Y    | Y    |
| sample_uvc2vo                | Y    | Y    | Y    | Y    | Y    |
| sample_uvc_vo                | Y    | Y    | Y    | Y    | Y    |
| sample_uvcout                | Y    | Y    | Y    | Y    | Y    |
| sample_uac                   | Y    |      |      |      |      |
| sample_usbcamera             | Y    |      |      |      |      |
| 多媒体文件                   | 2    | 2    | 2    | 1    | 1    |
| sample_demux                 | Y    | Y    | Y    | Y    | Y    |
| sample_file_repair           | Y    | Y    | Y    |      |      |
| AI demo                      | 2    | 0    | 0    | 0    | 0    |
| sample_odet_demo             | Y    |      |      |      |      |
| sample_RegionDetect          | Y    |      |      |      |      |
| 其他                         | 6    | 3    | 2    | 4    | 0    |
| sample_glog                  | Y    | Y    | Y    | Y    |      |
| sample_hello                 | Y    | Y    | Y    | Y    |      |
| sample_pthread_cancel        | Y    | Y    |      |      |      |
| sample_motor                 |      |      |      | Y    |      |
| sample_sound_controler       |      |      |      | Y    |      |
| sample_ai_Demo_0.9.2         |      |      |      |      |      |
| sample_ai_Demo_0.9.6         |      |      |      |      |      |
| sample_thumb                 |      |      |      |      |      |
| sample_nna                   |      |      |      |      |      |
| sample_MotionDetect          | Y    |      |      |      |      |
| sample_PersonDetect          | Y    |      |      |      |      |
| sample_directIORead          | Y    |      |      |      |      |

### 7.2 Melis 各平台方案上 MPP sample 支持情况

说明 

​	标注 “Y” 则表示该 sample 支持在该平台方案上测试，未标注则表示不支持。

| MPP sample                   | V459 |
| ---------------------------- | ---- |
| 总数                         | 21   |
| 视频                         | 9    |
| sample_driverVipp            | Y    |
| sample_virvi                 | Y    |
| sample_virvi2vo              | Y    |
| sample_virvi2vo_zoom         | Y    |
| sample_vi_reset              |      |
| sample_isposd                |      |
| sample_vin_isp_test          |      |
| sample_region                | Y    |
| sample_venc                  |      |
| sample_venc2muxer            |      |
| sample_virvi2venc            | Y    |
| sample_timelapse             |      |
| sample_virvi2venc2muxer      | Y    |
| sample_multi_vi2venc2muxer   | Y    |
| sample_rtsp                  |      |
| sample_CodecParallel         |      |
| sample_vdec                  |      |
| sample_demux2vdec            |      |
| sample_demux2vdec_saveFrame  |      |
| sample_demux2vdec2vo         | Y    |
| 音频                         | 9    |
| sample_ai                    | Y    |
| sample_ao                    | Y    |
| sample_ao_resample_mixer     |      |
| sample_ao2ai_aec             | Y    |
| sample_ao2ai_aec_rate_mixer  |      |
| sample_aec                   | Y    |
| sample_aenc                  | Y    |
| sample_ai2aenc               |      |
| sample_ai2aenc2muxer         | Y    |
| sample_select                |      |
| sample_adec                  | Y    |
| sample_demux2adec            |      |
| sample_demux2adec2ao         | Y    |
| sample_aoSync                | Y    |
| ISE 和 EIS                   | 1    |
| sample_fish                  |      |
| sample_virvi2fish2venc       |      |
| sample_virvi2fish2vo         | Y    |
| sample_virvi2eis2venc        |      |
| sample_ise_dzoom             |      |
| sample_gdc_dzoom             |      |
| 视频显示                     | 0    |
| sample_vo                    |      |
| sample_UILayer               |      |
| G2D                          | 1    |
| sample_g2d                   |      |
| sample_vi_g2d                | Y    |
| CE                           | 0    |
| sample_twinchn_virvi2venc2ce |      |
| sample_virvi2venc2ce         |      |
| UVC                          | 0    |
| sample_uvc2vdec_vo           |      |
| sample_uvc2vdenc2vo          |      |
| sample_uvc2vo                |      |
| sample_uvc_vo                |      |
| sample_uvcout                |      |
| 多媒体文件                   | 00   |
| sample_demux                 |      |
| sample_file_repair           |      |
| 其他                         | 1    |
| sample_glog                  |      |
| sample_hello                 |      |
| sample_pthread_cancel        |      |
| sample_motor                 |      |
| sample_sound_controler       |      |
| sample_ai_Demo_0.9.2         |      |
| sample_ai_Demo_0.9.6         |      |
| sample_thumb                 | Y    |
| sample_nna                   |      |