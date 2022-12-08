# Linux_NPU_Lenet模型之从训练到端侧部署

## 1 前言

### 1.1 读者对象

本文档（本指南）主要适用于以下人员：
• 技术支持工程师
• 软件开发工程师
• AI 应用案客户

### 1.2 约定

#### 1.2.1 符号约定

本文中可能出现的符号如下：

警告
**警告**
技巧

1. 技巧
2. 小常识

说明
**说明**



## 2 正文

### 2.1 NPU 开发简介

• 支持int8/uint8/int16 量化精度，运算性能可达1TOPS.

• 相较于GPU 作为AI 运算单元的大型芯片方案，功耗不到GPU 所需要的1%.

• 可直接导入Caffe, TensorFlow, Onnx, TFLite，Keras, Darknet, pyTorch 等模型格式.

• 提供AI 开发工具：支持模型快速转换、支持开发板端侧转换API、支持TensorFlow, TFLite, Caffe, ONNX, Darknet, pyTorch 等模型.

• 提供AI 应用开发接口：提供NPU 跨平台API.

• 部署工具支持离线量化(后训练量化,PTQ) 和量化感知训练(QAT) 两类模型的导入，对于QAT训练得到的.tflite,onnx 格式模型，在模型import 阶段会根据原生模型

中的量化描述生成量化表文件。

### 2.2 开发流程

NPU 开发完整的流程如下图所示：

![image-20221207184620961](${media}/image-20221207184620961.png)

<center>图2-1: npu_1.png</center>

## 3 Lenet 模型简介

Lenet 是一系列网络的合称，包括Lenet1 - Lenet5，由Yann LeCun 等人在1990 年《Handwritten Digit Recognition with a Back-Propagation Network》中提

出，是卷积神经网络的HelloWorld。这里就以lenet 为例介绍AI model 在tina 平台上部署从训练到端侧运行的全部过程。

细分环节包括, 模型训练，模型导入，模型量化，模型推理，模型导出，模型仿真，模型profile，模型端侧部署几个部分. 用一幅图表示如下：

![image-20221207185412148](${media}/image-20221207185412148.png)

<center>图3-1: npu 部署流程</center>

接下来从头开始介绍。

### 3.1 模型训练

本例中使用keras 框架编写并训练lenet5 网络，训练完成后，导出h5 格式的模型文件,acuity tools 原生支持H5 格式.

模型结构：

![image-20221207185520650](${media}/image-20221207185520650.png)

<center>图3-2: lenet5 模型结构</center>

训练完成后，观察精度是否满足训练目标要求，本例精度达到了%97, 可以拿来部署说明问题:

![image-20221207185544323](${media}/image-20221207185544323.png)

输出保存模型为lenet.h5,

![image-20221207185621907](${media}/image-20221207185621907.png)

<center>图3-4: lenet 模型文件</center>

使用netron 查看模型结构:

![image-20221207185721712](${media}/image-20221207185721712.png)

<center>图3-5: lenet 模型结构查看</center>

至此，我们的原生模型已经产生，接下来就可以进行PC 侧以及端侧的部署了，下一个环节是模型导入.

### 3.2 模型导入

在进行导入操作前，先看一下部署目录的结构：

如下图所示，其中data 目录的图像来源于mnist 数据集，作用是用来作为后训练量化的数据输入，用于给量化算法提供输入参考，从而获知实际场景的数据输入分

布.dataset.txt 则是对data目录的引用，工具会通过dataset.txt 文件查找到data 目录中的每张图片。

![image-20221207185811861](${media}/image-20221207185811861.png)

<center>图3-6: 部署目录结构</center>

数据集：

![image-20221207185838762](${media}/image-20221207185838762.png)

<center>图3-7: 量化参考图像数据集</center>

### 3.3 导入模型

使用芯原提供的acuity tool 中的pegasus 工具进行模型的导入.

```
pegasus import onnx --model yolact-sim.onnx --output-model yolact-sim.json --output-data yolact-sim.data
```

导入模型的目的是将开放模型转换为符合VIP 模型网络描述文件(.json) 和权重文件(.data)

![image-20221207185921978](${media}/image-20221207185921978.png)

<center>图3-8: npu_import</center>
接下来进行lenet.h5 模型导入.

```
pegasus import keras --model lenet.h5 --output-data lenet.data --output-model lenet.json
```

执行成功后，可以看到目录中多了lenet.json 和lenet.data 文件，它们分别是符合芯原格式的模型结构文件和模型权重文件.

![image-20221207185955890](${media}/image-20221207185955890.png)

<center>图3-9: 模型导入</center>

导入阶段，pegasus 工具也会对模型结构进行解析并输出：

![image-20221207190015642](${media}/image-20221207190015642.png)

<center>图3-10: 模型结构描述</center>

导入阶段到这里还没有完，我们需要生成对网络的输入输出描述文件，这也是acuity tool 工具要求的，输入输出描述是YML 格式的文本文件，后面我们将通过修

改YML 文件来对模型参数，输入/输出tensor 格式等信息进行配置.

#### 3.3.1 创建input/output YML 文件

YML 文件对网络的输入和输出进行描述，比如输入图像的形状，归一化系数(均值，零点)，图像格式，输出tensor 的输出格式，后处理方式等等，命令如下：

```
pegasus generate inputmeta --model lenet.json --input-meta-output lenet-inputmeta.yml pegasus
generate postprocess-file --model lenet.json --postprocess-file-output lenet-postprocess-file.yml
```

命令成功执行后，目录中又多了两个文件，分别是input 和output YML 文件。

![image-20221207190103043](${media}/image-20221207190103043.png)

<center>图3-11: YML 文件生成</center>

至此，模型导入阶段的工作才算全部完成，从这里开始，模型已经被转成了芯原格式的模型文件。后续步骤已经和原生模型没有太大关系了.

在执行下一步的模型量化前，我们需要修改input yml 文件的scale，mean 参数，使其和训练时的参数保持一致.

训练代码中的均值和scale 为：

![image-20221207190126855](${media}/image-20221207190126855.png)

<center>图3-12: 训练代码中的均值和scale</center>

根据代码，均值mean = 0, scale 为1/255 = 0.0039.

修改YML 为对应值：

![image-20221207190147405](${media}/image-20221207190147405.png)

<center>图3-13: 修改input YML Scale 参数</center>

### 3.4 模型量化

此款NPU 支持uint8,int8,int16 三种量化类型，基于推理速度和精度折衷的考虑，我们用int8，非对称量化模式，量化命令如下：

```
pegasus quantize --model lenet.json --model-data lenet.data --batch-size 1 --device CPU --withinput-meta lenet-inputmeta.yml --rebuild --model-quantize lenet.quantilize --quantizer asymmetric_affine --qtype uint8
```

命令执行后，工程目录下可以看到新创建的量化表文件文件

![image-20221207190243780](${media}/image-20221207190243780.png)

<center>图3-14: 量化表文件</center>
### 3.5 模型预推理

```
pegasus inference --model lenet.json --model-data lenet.data --batch-size 1 --dtype quantized --model-quantize lenet.quantilize --device CPU --with-input-meta lenet-inputmeta.yml --postprocess-file lenet-postprocess-file.yml --iterations 10
```

参数中，--batch-size 1表示每次推理处理1 张图像，我们dataset 目录中包含了10 张图像，所以要运行10 次处理完毕，这也是后面参数--iterations 10的作用.

成功运行后，目录中多出了20 个文本格式的.tensor 文件，文件中保存的是每张图像的输入和输出的tensor 数据，默认情况下，只对输入输出tensor 进行保存，

你可以通过下面命令将每层的tensor 都保存下来:

```
pegasus dump --model lenet.json --model-data lenet.data --with-input-meta lenet-inputmeta.yml
```

推理结束后创建的tensor 文件：

![image-20221207190401783](${media}/image-20221207190401783.png)

我们从命令的输出来看，也可以看出推理是否正确, 我们以前六个为例，可以看到top5 输出中，每次概率最高的分别是0,1,2,3,4 ….，和我们dataset.txt 文件实际

输入的图像数据顺序是相符的。

![image-20221208090925317](${media}/image-20221208090925317.png)

<center>图3-16: 部署推理过程输出</center>

输出tensor 则为最后一层的softmax 输出，也就是分别为数字0-9 的概率，以第9 张图像的输出tensor 为例，概率最大的是9, 如下图：

![image-20221208090956262](${media}/image-20221208090956262.png)

<center>图3-17: softmax 输出</center>

输入则是输入图像正则化后的浮点数据：

![image-20221208091015981](${media}/image-20221208091015981.png)

<center>图3-18: 输入tensor</center>

至此模型预推理阶段结束，进入下一步模型导出阶段.

### 3.6 导出代码和NBG 文件

导出代码的命令如下，两次命令的区别只有选项--pack-nbg-unify和--pack-nbg-viplite，其余完全相同。其中--pack-nbg-unify生成的是仿真侧的代码，而--pack-

nbg-viplite则会生成在端侧运行部署的代码，两条命令分别执行一遍。

```
pegasus export ovxlib --model lenet.json --model-data lenet.data --dtype quantized --model-quantize
lenet.quantilize --batch-size 1 --save-fused-graph --target-ide-project 'linux64' --with-inputmeta
lenet-inputmeta.yml --postprocess-file lenet-postprocess-file.yml --output-path ovxlib/lenet/
lenet --pack-nbg-unify --optimize "VIP9000PICO_PID0XEE" --viv-sdk ${VIV_SDK}
```

```
pegasus export ovxlib --model lenet.json --model-data lenet.data --dtype quantized --model-quantize
lenet.quantilize --batch-size 1 --save-fused-graph --target-ide-project 'linux64' --with-inputmeta
lenet-inputmeta.yml --postprocess-file lenet-postprocess-file.yml --output-path ovxlib/lenet/
lenet --pack-nbg-viplite --optimize "VIP9000PICO_PID0XEE" --viv-sdk ${VIV_SDK}
```

执行结束后，工程代码和NBG 文件都已经生成了:

![image-20221208091107463](${media}/image-20221208091107463.png)

<center>图3-19: 导出模型和工程代码</center>

导出的NBG 文件可以投入到NPU 中运行.

ovxlib/lenet/工程用于仿真和profile.

ovxlib/lenet_nbg_viplite/可以用部署在端侧运行.

ovxlib/lenet_nbg_unify 和ovxlib/lenet_nbg_unify_ovx 暂时可以不用理会，没有用处。

接下来，进入模型仿真环节.

### 3.7 模型仿真

上文说到，ovxlib/lenet/是用于仿真和profile 分析的工程，下面我们启动IDE 运行仿真.

#### 3.7.1 启动IDE

启动IDE 的命令如下:

```
~/VeriSilicon/VivanteIDE5.5.0/ide/vivanteide5.5.0
```

启动时，首先选择一个工作目录用于保存仿真工程：

![image-20221208091310996](${media}/image-20221208091310996.png)

<center>图3-20: IDE 仿真</center>

#### 3.7.2 导入ovxlib/lenet 工程

IDE 中，选择File->Import->General选项卡->Existing Projects into Workspace

![image-20221208091730108](${media}/image-20221208091730108.png)

<center>图3-21: npu_prj_import</center>

之后选择模型导出阶段创建的工程目录ovxlib/lenet/

![image-20221208091748054](${media}/image-20221208091748054.png)

<center>图3-22: npu_prj_ok</center>

这里强烈建议选中Copy projects into workspace, 这样我们的仿真工程将会拷贝一份到IDE 工作空间中，保证与导出空间隔离.

之后点击Finsih，结束导入过程. 导入后的工作空间如下所示：

![image-20221208091811642](${media}/image-20221208091811642.png)

<center>图3-23: npu_sim_ok</center>

#### 3.7.3 编译工程

执行菜单命令Project->Build All，先将仿真工程编译一下，看有没有犯低级错误(比如导错工程了等等.):

![image-20221208091838854](${media}/image-20221208091838854.png)

<center>图3-24: npu_compile_ok</center>

编译OK，生成了可执行lenet 文件，我们进入下一步.

#### 3.7.4 配置仿真参数

执行菜单命令Run->Debug Configurations...在选项卡中，双击OpenVX Application，即可出现下图的输出，默认情况下Search Project 和Browser 按钮窗口会被

正确设置成如下图的样子，如果没有，请按照上面编译的结果正确选择工程和应用路径。

这个选项卡最最重要的设置是Program arguments，不同的网络根据输入输出个数的不同，输入也不尽相同，lenet 按照如下的方法设置:

![image-20221208091913430](${media}/image-20221208091913430.png)

<center>图3-25: npu_argu</center>

其中lenet.export.data 是量化权重，它是在模型导出阶段生成在ovxlib/lenet 工程中的，工程导入阶段已经自动拷贝到IDE 仿真工程下，不需要手工拷贝，而0.jpg 

则需要手动拷贝到IDE工程下:

![image-20221208091937601](${media}/image-20221208091937601.png)

<center>图3-26: npu_arg_copy</center>

输入除了图片，也可以是模型推理阶段生成的输入tensor, 仿真程序会根据后缀名自动运行到不同的处理分支，保证处理结果都是对的。

点击Apply, 之后就可以开始正式仿真了.

#### 3.7.5 仿真

点击工具栏Run 按钮，弹出对话框，直接点击Run 触发仿真.

![image-20221208092001120](${media}/image-20221208092001120.png)

<center>图3-27: npu_run</center>

lenet 是很小的网络，仿真很快结束，输出如下, 根据下面控制台的输出可以看到，我们给的参数图像是0.jpg，而仿真输出的top5 结果表明，推理结果为0 的概率

为%99.0023, 符合预期.

![image-20221208092059915](${media}/image-20221208092059915.png)

<center>图3-28: npu_sim_res</center>

至此，模型仿真结束，进入模型profile.

### 3.8 模型Profile

模型profile 可以帮助分析网络的整体运行效率，带宽，帧率以及各层的处理性能，是分析算法精度，性能瓶颈等问题的利器。

点击工具栏运行旁边的Profile按钮即可触发Profile 操作，同样在选项卡中选择Profile 按钮继续.

![image-20221208092134981](${media}/image-20221208092134981.png)

<center>图3-29: npu_profile_con</center>

之后点击Resume 继续:

![image-20221208092154631](${media}/image-20221208092154631.png)

<center>图3-30: npu_profile_in</center>

Profile 结束后, IDE 输出如下：

![image-20221208092325476](${media}/image-20221208092325476.png)

<center>图3-31: npu_lenet_profile</center>

简单分析一下Profile 的信息，左下角和仿真结果输出相同，为推理top5 的结果，中间的是各层的运行情况统计，包括每层的硬件处理单元，读写带宽，对于由

PPU 计算的层，比如softmax层，还会有指令数统计等等。右下角的则是网络的整体运行性能分析，包括网络整体的读写带宽，处理帧率，时钟数，等等信息。更

具体地分析，请参考芯原文档。

接下来，我们到了最后一步，我们前面所作地一切工作地目的，就是要将网络部署在端侧，真正地在板子上跑起来。我们开始端侧部署地介绍:

### 3.9 端侧部署

前面仿真阶段，仿真结束后，工程中生成了两个bin 文件是我们部署验证要用到的，分别是input_0.dat 和output0_10_1.dat, 它们都是二进制格式的文件。

input_0.dat 是网络第一层的输入，output0_10_1.dat 是网络最后一层的输出，由于根据仿真结果说明，这两笔数据都是正确的，可以作为golden 数据和端侧的

运行结果进行对比，如果在同样的input 下，端侧跑出的output tensor 和output_0._10_1.dat 是binary identical 的，那就说明，端侧部署是正确的。



理清了逻辑，我们开始动手操作，首先在仿真工程下认识一下这两个.dat, 下图左框中的蓝底文件:

![image-20221208092431537](${media}/image-20221208092431537.png)

<center>图3-32: goldentensor</center>

接下来，我们用到模型导出阶段生成的另一个工程，ovxlib/lenet_nbg_viplite 工程.

#### 3.9.1 交叉编译ovxlib/lenet_nbg_viplite 工程

我们发布的Tina SDK 将会包含NPU 的开发SDK，NPU 的开发SDK 结构如下图所示：

![image-20221208092502665](${media}/image-20221208092502665.png)

<center>图3-33: npu sdk</center>

如果您拿到了tina sdk，NPU 的KMD 模块已经存在于tina sdk 对应的linux 内核代码中，而用户态SDK, 也就是图中的npu runtime library(UMD Driver)，则在

SDK 的package/allwinner/目录下以库的形式存在。

接上文，我们将ovxlib/lenet_nbg_viplite 拷贝出来，和Tina 中NPU runtime library 并列放在一个目录下，按照下面的内容编写makefile 文件（SDK 中将会包含

一个demo makefile).

![image-20221208092527322](${media}/image-20221208092527322.png)

<center>图3-34: 工程目录结构</center>

![image-20221208092609567](${media}/image-20221208092609567.png)

<center>图3-35: 工程makefile</center>

工程目录中，sdk 目录内容是NPU 的用户态运行库，也就是UMD 驱动，而lenet_nbg_viplite目录中的内容，则是模型部署阶段产生的lenet_nbg_viplite 工程加上

makefile 的结果。

接下来就可以交叉编译测试工程了，在lenet_nbg_viplite 目录，执行make clean && make ,执行结束后，在out 目录将会生成可以在端侧跑的lenet 测试程序。

![image-20221208092632515](${media}/image-20221208092632515.png)

<center>图3-36: npu_elf_res</center>

交叉编译到此结束，接下来准备测试工程目录。

#### 3.9.2 准备测试工程目录

测试工程目录的内容包括，模型NBG 文件，lenet 可执行程序，两个UMD 动态库以及仿真阶段生成的input_0.dat，一共5 个文件，如下图所示：

![image-20221208092713183](${media}/image-20221208092713183.png)

<center>图3-37: npu_test_env</center>

至此，测试目录已准备OK，下面开始准备端侧验证平台。

#### 3.9.3 准备端侧验证环境

首先，NPU 运行的大是你的端侧存下下面的设备节点/dev/vipcore，否则，说明SDK 配置是错误的，请寻求我们的协助。

![image-20221208092735339](${media}/image-20221208092735339.png)

<center>图3-38: npu_device</center>

将前面建立的测试目录保存到tf 卡中，我们用TF 卡作为媒介验证, 将TF 卡插到端侧平台，之后执行命令

```
mount -t vfat /dev/mmcblkxxx /mnt/sdcard
```

将其挂载到/mnt/sdcard 目录，之后，进入lenet-test 验证目录，执行命令

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/mnt/sdcard/lenet-test
```

保证运行库可以被正确的连接到。

之后就可以正式测试了，执行测试命令

```
./lenet network_binary.nb input_0.dat
```

输出如下：

![image-20221208092904086](${media}/image-20221208092904086.png)

<center>图3-39: result</center>

注意最后一行，可以看到测试目录下多了一个文件，output0_10_1.dat，他就是网络输出的结果。

### 3.10 验证

我们得到了tensor 的端侧运行结果，将他和仿真生成的.dat 做对比，预期情况应该是binary identical 的。

![image-20221208092937875](${media}/image-20221208092937875.png)

<center>图3-40: npu_cmp_res</center>

结果Binary Identical 的，和我们的预期一致.

#### 3.10.1 验证tensor

根据前面网络结构的描述，网络的最后一层是softmax,softmax 层输出概率值为浮点数，在芯原的NPU 设计中，存在三类计算单元，分别是TP,NNE 和PPU，这里

面只有PPU 支持浮点计算，所以softmax 层要在PPU 上运行的。

我们看一下NBG 文件中的输出层信息, 如下图所示，可以看到输出为浮点FP16，没有量化，根据上面验证beyondcompare 对比，我们也可以看出，tensor 输出

一共20 个字节。我们来计算一下，lenet 分类网络一共识别十类目标，每一类的概率为FP16，所以计算起来正好是20 个字节，由此我们知道了output tensor 的

结构。

![image-20221208093034585](${media}/image-20221208093034585.png)

<center>图3-41: nbinfo</center>

既然知道了结构，我们就可以将运行产生的output tensor 转换为概率打印出来，由于32 位机上不支持FP16 的格式，所以需要将其转换为符合ieee754 的float32 

格式，核心转换代码如下:

![image-20221208093112338](${media}/image-20221208093112338.png)

<center>图3-42: npu_fp16</center>

输出如下：

![image-20221208093127325](${media}/image-20221208093127325.png)

<center>图3-43: npu_fp32</center>

对比仿真阶段的输出，得到的TOP5 输出概率完全一样。说明我们的部署以及后处理是正确的。

至此，部署加验证过程全部结束~！

