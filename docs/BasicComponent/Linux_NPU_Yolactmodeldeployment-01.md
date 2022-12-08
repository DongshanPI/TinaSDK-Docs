# Linux_NPU_Yolact模型部署

## 1 前言
### 1.1 读者对象

本文档（本指南）主要适用于以下人员：

• 技术支持工程师
• 软件开发工程师
• AI 应用案客户


## 2 正文

### 2.1 NPU 开发简介

• 支持int8/uint8/int16 量化精度，运算性能可达1TOPS.

• 相较于GPU 作为AI 运算单元的大型芯片方案，功耗不到GPU 所需要的1%.

• 可直接导入Caffe, TensorFlow, Onnx, TFLite，Keras, Darknet, pyTorch 等模型格式.

• 提供AI 开发工具：支持模型快速转换、支持开发板端侧转换API、支持TensorFlow, TF Lite, Caffe, ONNX, Darknet, pyTorch 等模型.

• 提供AI 应用开发接口：提供NPU 跨平台API.

### 2.2 开发流程

NPU 开发完整的流程如下图所示：

![image-20221208101012970](${media}/image-20221208101012970.png)

<center>图2-1: npu_1.png</center>

### 2.3 获取YOLACT 原始模型

YOLACT 模型获取方式有很多种，可以基于项目https://github.com/dbolya/yolact 产生onnx 格式的yolact 模型，本文档假设你已经产生了yolact.onnx 模型

### 2.4 模型部署工作目录结构

yolact-sim.onnx 有120M，还是蛮大的.

![image-20221208101053708](${media}/image-20221208101053708.png)

<center>图2-2: npu_workspace</center>

### 2.5 导入模型

```
pegasus import onnx --model yolact-sim.onnx --output-model yolact-sim.json --output-data yolact-sim.data
```

导入模型的目的是将开放模型转换为符合VIP 模型网络描述文件(.json) 和权重文件(.data)

![image-20221208101310404](${media}/image-20221208101310404.png)

<center>图2-3: npu_import</center>

### 2.6 创建YML 文件

YML 文件对网络的输入和输出进行描述，比如输入图像的形状，归一化系数(均值，零点)，图像格式，输出tensor 的输出格式，后处理方式等等，命令如下：

```
pegasus generate inputmeta --model yolact-sim.json --input-meta-output yolact-sim-inputemeta.yml
```

```
pegasus generate postprocess-file --model yolact-sim.json --postprocess-file-output yolact-simpostprocess-file.yml
```

![image-20221208101601819](${media}/image-20221208101601819.png)

<center>图2-4: npu_yml</center>

修改input meta 文件中的的scale 参数为0.0039(1/256)，和yolov3 的改法完全一致。

![image-20221208101624198](${media}/image-20221208101624198.png)

<center>图2-5: npu_scale</center>

### 2.7 量化

量化命令：

```
pegasus quantize --model yolact-sim.json --model-data yolact-sim.data --batch-size 1 --device CPU --with-input-meta yolact-sim-inputemeta.yml --rebuild --model-quantize yolact-sim.quantilize --quantizer asymmetric_affine --qtype uint8
```

命令执行后，创建了量化表文件

![image-20221208101701439](${media}/image-20221208101701439.png)

<center>图2-6: npu_quantilize</center>

### 2.8 预推理

```
pegasus inference --model yolact-sim.json --model-data yolact-sim.data --batch-size 1 --dtype quantized --model-quantize yolact-sim.quantilize --device CPU --with-input-meta yolact-siminputemeta.yml --postprocess-file yolact-sim-postprocess-file.yml
```

![image-20221208101802000](${media}/image-20221208101802000.png)

<center>图2-7: npu_inf</center>

### 2.9 导出代码和NBG 文件

```
pegasus export ovxlib --model yolact-sim.json --model-data yolact-sim.data --dtype quantized --model-quantize yolact-sim.quantilize --batch-size 1 --save-fused-graph --target-ide-project 'linux64' --with-input-meta yolact-sim-inputemeta.yml --postprocess-file yolact-sim-postprocess-file.yml --output-path ovxlib/yolact/yolact --pack-nbg-unify --optimize "VIP9000PICO_PID0XEE" --vivsdk${VIV_SDK}
```

```
pegasus export ovxlib --model yolact-sim.json --model-data yolact-sim.data --dtype quantized --model-quantize yolact-sim.quantilize --batch-size 1 --save-fused-graph --target-ide-project 'linux64' --with-input-meta yolact-sim-inputemeta.yml --postprocess-file yolact-sim-postprocess-file.yml --output-path ovxlib/yolact/yolact --pack-nbg-viplite --optimize "VIP9000PICO_PID0XEE" --vivsdk${VIV_SDK}
```

![image-20221208101926523](${media}/image-20221208101926523.png)

<center>图2-8: npu_export</center>

生成的NBG 文件, 可以部署到端侧:

![image-20221208101945583](${media}/image-20221208101945583.png)

<center>图2-9: npu_nbg</center>

### 2.10 模型算力测量

```
pegasus measure --model yolact-sim.json
```

![image-20221208102008519](${media}/image-20221208102008519.png)

<center>图2-10: npu_measure</center>

对比yolov3 的32.99G MACC 算力和yolov3-tiny 的2.79G macc 的算力需求，YOLACT 明显对算力的需求大很多. 题外话,1TOPS=1000GOPS, 对于yolov3-tiny 来

说，我们以3G 的算力需求来计算，就是0.003T，在500M 频率下，NPU 的理论算力是0.5T, 所以，YOLOV3 的帧率为：0.5T/0.003T=166 帧。这个值和仿真值是

比较接近的.

### 2.11 后处理验证

yolact 的后处理C 代码在如下位置：

![image-20221208102042389](${media}/image-20221208102042389.png)

<center>图2-11: npu_gerit</center>

编译后处理模型:

![image-20221208102206195](${media}/image-20221208102206195.png)

<center>图2-12: npu_post</center>

out 目录生成了yolact 的后处理程序：

![image-20221208102225653](${media}/image-20221208102225653.png)

<center>图2-13: npu_postprocess</center>

yolact 模型有五个输出层,inference 阶段生成了5 个output tensor.

![image-20221208102753573](${media}/image-20221208102753573.png)

<center>图2-14: npu_tensor</center>

tensor之间的对应关系,0是location, 1是confidence, 2是mask，3暂时无用，4是maskmaps.

output0_4_19248_1.dat <---->iter_0_attach_Concat_Concat_256_out0_0_out0_1_19248_4.tensor

output1_81_19248_1.dat<---->iter_0_attach_Softmax_Softmax_260_out0_1_out0_1_19248_81.tensor

output2_32_19248_1.dat<---->iter_0_attach_Concat_Concat_258_out0_2_out0_1_19248_32.tensor

output3_4_19248.dat <---->iter_0_attach_Initializer_769_out0_3_out0_19248_4.tensor

output4_32_138_138_1.dat<---->iter_0_attach_Transpose_Transpose_165_out0_4_out0_1_138_138_32.tensor

执行后处理

![image-20221208102833160](${media}/image-20221208102833160.png)

<center>图2-15: npu_exepost</center>

后处理结果：

![image-20221208102858625](${media}/image-20221208102858625.png)

<center>图2-16: npu_yolact</center>

图中识别除了两只dog, 有两个狗的信息，其他都能很好的输出结果，dog 70.7% 不应该出现的，大概率是量化导致输出结果精度精度降低。

修改归一化参数后重新部署，得到的结果如下，可以看到，之前的问题消失：

![image-20221208102920884](${media}/image-20221208102920884.png)

<center>图2-17: npu_yolact_ok</center>

至此，yoloact 模型导入完成.

### 2.12 结束

