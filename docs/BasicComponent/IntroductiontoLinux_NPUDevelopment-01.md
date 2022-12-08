# Linux_NPU 开发简介

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

• 提供AI 开发工具：支持模型快速转换、支持开发板端侧转换API、支持TensorFlow, TFLite, Caffe, ONNX, Darknet, pyTorch 等模型.

• 提供AI 应用开发接口：提供NPU 跨平台API.

### 2.2 开发流程

NPU 开发完整的流程如下图所示：

![image-20221208111346439](${media}/image-20221208111346439.png)

<center>图2-1: npu_1.png</center>

### 2.3 模型训练

在模型训练阶段，用户根据需求和实际情况选择合适的框架（如Caffe、TensorFlow 等）进行训练得到符合需求的模型。也可直接使用已经训练好的模型, 对于基

于已有的算法模型部署来讲，可以不用经过模型训练阶段.

### 2.4 模型转换

此阶段为通过Acuity Toolkit 把模型训练中得到的模型转换为NPU 可用的模型NBG 文件。

### 2.5 程序开发

最后阶段为基于VIPLite API 开发程序实现业务逻辑。

### 2.6 acuity Toolkit

Allwinner 提供acuity toolkit 开发套件进行模型转换、推理运行和性能评估。

用户通过提供的python 接口可以便捷地完成以下功能：

1）模型转换：支持Caffe,TensorFlow Lite, Tensorflow, ONNXDarknet NBG 模型导入导出，后续能够在硬件平台上加载使用。

2）模型推理：能够在PC 上模拟运行模型并获取推理结果，也可以在指定硬件平台上运行模型并获取推理结果。

3）性能评估：能够在PC 上模拟运行并获取模型总耗时及每一层的耗时信息，也可以通过联机

调试的方式在指定硬件平台上运行模型，并获取模型在硬件上运行时的总时间和每一层的耗时信息。

此文档主要介绍模型转换和基于NPU 程序开发，不涉及模型训练的内容。

## 3 结束