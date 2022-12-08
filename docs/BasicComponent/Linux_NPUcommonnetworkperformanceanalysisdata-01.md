# Linux_NPU常见网络性能分析数据

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

![image-20221208105235547](${media}/image-20221208105235547.png)

<center>图2-1: npu_1.png</center>

### 2.3 常见网络benchmark

![image-20221208105336553](${media}/image-20221208105336553.png)

<center>图2-2: NPU benchmark</center>

以上数据是裸机程序跑网络的数据，并未考虑到方案中的其它应用。

### 2.4 内存分析数据

方案应用场景中的内存消耗数据分析.

代码和数据部分的占用，包括KMD 和UMD 本身占用的空间大小, 大约180k.

![image-20221208105505832](${media}/image-20221208105505832.png)

<center>图2-3: code 占用大小</center>

Yolov3 模型的内存数据统计，运行时消耗约48M 内存。

![image-20221208105619800](${media}/image-20221208105619800.png)

<center>图2-4: yolov3 内存统计</center>

yolov3-tiny 模型的内存数据统计，运行时消耗月6.8M 内存。

![image-20221208105638438](${media}/image-20221208105638438.png)

<center>图2-5: yolov3-tiny 内存统计</center>

帧率，带宽等数据待补充.

## 3 结束