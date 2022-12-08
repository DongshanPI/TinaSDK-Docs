# Linux_NPU混合量化部署

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

![image-20221208105945861](${media}/image-20221208105945861.png)

<center>图2-1: npu_1.png</center>

本篇以yolov5s 模型为例，来说明混合量化的具体步骤.

### 2.3 浮点部署

浮点部署的目的是获取golden 数据，目的是可以和后面混合量化得到的数据比较相似度，来衡量混合量化的效果。

```
pegasus.py import onnx --model yolov5s.onnx --output-data yolov5s.data --output-model yolov5s.json

pegasus.py generate inputmeta --model yolov5s.json --input-meta-output yolov5s-inputmeta.yml

pegasus.py generate postprocess-file --model yolov5s.json --postprocess-file-output yolov5s-postprocess-file.yml

pegasus.py inference --model yolov5s.json --model-data yolov5s.data --batch-size 1 --dtype float32 --device CPU --with-input-meta yolov5s-inputmeta.yml --postprocess-file yolov5s-postprocess-file.yml

pegasus.py export ovxlib --model yolov5s.json --model-data yolov5s.data --dtype float32 --batch-size 1 --save-fused-graph --target-ide-project 'linux64' --with-input-meta yolov5s-inputmeta.yml --postprocess-file yolov5s-postprocess-file.yml --output-path ovxlib/yolov5s/yolov5sprj --pack-nbg-unify --optimize "VIP9000PICO_PID0XEE" --viv-sdk ${VIV_SDK}
```

要注意在第三步完成之后，需要将input yml 文件的mean 和scale 参数修改为符合网络实际的训练时的参数，对于yolov5s 来讲，scale 需要修改为0.0039.

![image-20221208110132347](${media}/image-20221208110132347.png)

<center>图2-2: scale</center>

结束后，最终得到了输出层的golden tensor:

```
iter_0_attach_Concat_Concat_303_out0_0_out0_1_25200_85.tensor
```

![image-20221208110156068](${media}/image-20221208110156068.png)

<center>图2-3: tensor</center>

### 2.4 混合量化部署

前两步操作相同：

```
pegasus.py import onnx --model yolov5s.onnx --output-data yolov5s.data --output-model yolov5s.json

pegasus.py generate inputmeta --model yolov5s.json --input-meta-output yolov5s-inputmeta.yml

pegasus.py generate postprocess-file --model yolov5s.json --postprocess-file-output yolov5s-postprocess-file.yml
```

之后修改归一化系数，均值，方差(scale).

![image-20221208110441010](${media}/image-20221208110441010.png)

<center>图2-4: normallize</center>

#### 2.4.1 PCQ+int8 量化

```
pegasus.py quantize --model yolov5s.json --model-data yolov5s.data --batch-size 1 --device CPU --with-input-meta yolov5s-inputmeta.yml --rebuild --model-quantize yolov5s.quantize --quantizer perchannel_symmetric_affine --qtype int8
```

此步骤中得到量化表文件yolov5s.quantize，

```
pegasus.py quantize --model yolov5s.json --model-data yolov5s.data --device CPU --withinput-meta yolov5s-inputmeta.yml --hybrid --model-quantize yolov5s.quantize --quantizer perchannel_symmetric_affine --qtype int8
```

![image-20221208110532874](${media}/image-20221208110532874.png)

<center>图2-5: quantilize</center>

YOLOV5S 精度问题的主要原因是，后处理部分也加入到网络中执行了，后处理不太适合量化， 量化精度损失很大，如下图所示：

![image-20221208110559683](${media}/image-20221208110559683.png)

<center>图2-6: output</center>

permute 下面的层都属于后处理的部分，这部分量化精度损失特别大, 需要进行混合量化。

#### 2.4.2 混合量化

修改默认的yolov5s.quantilize 文件，将permute 下面需要混合量化的层加入进来，进行int16 量化。

```
customized_quantize_layers:
Sigmoid_Sigmoid_202_21: dynamic_fixed_point-i16
Initializer_342_62: dynamic_fixed_point-i16
Slice_Slice_207_61: dynamic_fixed_point-i16
Initializer_344_48: dynamic_fixed_point-i16
Mul_Mul_209_47: dynamic_fixed_point-i16
Sub_Sub_211_32: dynamic_fixed_point-i16
Initializer_346_33: dynamic_fixed_point-i16
Add_Add_213_17: dynamic_fixed_point-i16
Mul_Mul_215_8: dynamic_fixed_point-i16
Initializer_348_18: dynamic_fixed_point-i16
Slice_Slice_220_49: dynamic_fixed_point-i16
Initializer_355_50: dynamic_fixed_point-i16
Mul_Mul_222_34: dynamic_fixed_point-i16
Initializer_460_20: dynamic_fixed_point-i16
Pow_Pow_223_19: dynamic_fixed_point-i16
Mul_Mul_224_9: dynamic_fixed_point-i16
Concat_Concat_230_5: dynamic_fixed_point-i16
Slice_Slice_229_10: dynamic_fixed_point-i16
Reshape_Reshape_232_2: dynamic_fixed_point-i16
Sigmoid_Sigmoid_237_26: dynamic_fixed_point-i16
Initializer_385_66: dynamic_fixed_point-i16
Slice_Slice_242_65: dynamic_fixed_point-i16
Mul_Mul_244_52: dynamic_fixed_point-i16
Initializer_387_53: dynamic_fixed_point-i16
Sub_Sub_246_37: dynamic_fixed_point-i16
Add_Add_248_22: dynamic_fixed_point-i16
Initializer_389_38: dynamic_fixed_point-i16
Mul_Mul_250_11: dynamic_fixed_point-i16
Initializer_391_23: dynamic_fixed_point-i16
Initializer_461_35: dynamic_fixed_point-i16
Initializer_398_55: dynamic_fixed_point-i16
Slice_Slice_255_54: dynamic_fixed_point-i16
Mul_Mul_257_39: dynamic_fixed_point-i16
Pow_Pow_258_24: dynamic_fixed_point-i16
Mul_Mul_259_12: dynamic_fixed_point-i16
Initializer_464_25: dynamic_fixed_point-i16
Slice_Slice_264_13: dynamic_fixed_point-i16
Concat_Concat_265_6: dynamic_fixed_point-i16
Reshape_Reshape_267_3: dynamic_fixed_point-i16
Concat_Concat_303_1: dynamic_fixed_point-i16
attach_Concat_Concat_303/out0_0: dynamic_fixed_point-i16
Sigmoid_Sigmoid_272_31: dynamic_fixed_point-i16
Initializer_428_70: dynamic_fixed_point-i16
Slice_Slice_277_69: dynamic_fixed_point-i16
Mul_Mul_279_57: dynamic_fixed_point-i16
Initializer_430_58: dynamic_fixed_point-i16
Sub_Sub_281_42: dynamic_fixed_point-i16
Initializer_432_43: dynamic_fixed_point-i16
Add_Add_283_27: dynamic_fixed_point-i16
Mul_Mul_285_14: dynamic_fixed_point-i16
Initializer_434_28: dynamic_fixed_point-i16
Initializer_441_45: dynamic_fixed_point-i16
Slice_Slice_290_59: dynamic_fixed_point-i16
Mul_Mul_292_44: dynamic_fixed_point-i16
Initializer_468_30: dynamic_fixed_point-i16
Mul_Mul_294_15: dynamic_fixed_point-i16
Slice_Slice_299_16: dynamic_fixed_point-i16
Concat_Concat_300_7: dynamic_fixed_point-i16
Reshape_Reshape_302_4: dynamic_fixed_point-i16
Pow_Pow_293_29: dynamic_fixed_point-i16
```

![image-20221208110656474](${media}/image-20221208110656474.png)

<center>图2-7: mix</center>

![image-20221208110708470](${media}/image-20221208110708470.png)

<center>图2-8: mix</center>

#### 2.4.3 执行混合量化

```
pegasus.py quantize --model yolov5s.json --model-data yolov5s.data --device CPU --withinput-meta yolov5s-inputmeta.yml --hybrid --model-quantize yolov5s.quantize --quantizer perchannel_symmetric_affine --qtype int8
```

![image-20221208110746513](${media}/image-20221208110746513.png)

<center>图2-9: hybrid</center>

![image-20221208110818811](${media}/image-20221208110818811.png)

<center>图2-10: diff 变化</center>

执行后, 可以看到量化层输出的变化。

### 2.5 推理

```
pegasus.py inference --model yolov5s.json --model-data yolov5s.data --batch-size 1 --dtype quantized --model-quantize yolov5s.quantize --device CPU --with-input-meta yolov5sinputmeta.yml --postprocess-file yolov5s-postprocess-file.yml
```

### 2.6 模型导出

```
pegasus.py export ovxlib --model yolov5s.quantize.json --model-data yolov5s.data --dtype quantized --model-quantize yolov5s.quantize --batch-size 1 --save-fused-graph --targetide-project 'linux64' --with-input-meta yolov5s-inputmeta.yml --postprocess-file yolov5s-postprocess-file.yml --output-path ovxlib/yolov5s/yolov5sprj --pack-nbg-unify --optimize "VIP9000PICO_PID0XEE" --viv-sdk ${VIV_SDK}
```

### 2.7 相似度对比

将前面生成的golden tensor 和此时生成的输出tensor 对比余弦相似度：

```
python /home/caozilong/VeriSilicon/acuity-toolkit-whl-6.6.1/bin/tools/
compute_tensor_similarity.py ./iter_0_attach_Concat_Concat_303_out0_0_out0_1_25200_85.
tensor ../wendang/iter_0_attach_Concat_Concat_303_out0_0_out0_1_25200_85.tensor
```

![image-20221208111010297](${media}/image-20221208111010297.png)

<center>图2-11: diff 变化</center>

可以看到余弦相似度还是非常高的，达到了0.999912, 混合量化部署步骤到此结束。

## 3 结束

