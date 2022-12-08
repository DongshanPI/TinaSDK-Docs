# Linux_NPU部署工具安装指导
## 1 前言

### 1.1 读者对象

本文档（本指南）主要适用于以下人员：

• 技术支持工程师

• 软件开发工程师

• AI 应用案客户

## 2 正文

### 2.1 工具列表

![image-20221208103336436](${media}/image-20221208103336436.png)

<center>图2-1: npu_toolchain</center>

• Verisilicon_Tool_Acuity_Toolkit_6.0.14_Binary_Whl_src_20211228.tgz: 模型部署工具，提供了命令行和python 脚本两种界面协助客户将模型部署到芯原NPU 

上.acuity tools做的工作包括网络导入，优化，训练，量化以及推理.

• Verisilicon_Tool_VivanteIDE_v5.5.0_CL421327_Linux_Windows_SDK_6.4.x_dev_6.4.8_21Q3_CL423100B_IDE 工具，用于PC 侧的模型仿真验证，以及Profile

性能分析, 比如模型带宽，帧率等等。业务流上，IDE 依赖于acuity toolkit，比如IDE 需要acuity toolkit 导入过程中创建的C CODE 工程进行仿真。工具角度，

acuity toolkit 依赖于IDE 提供的一些支持库才能运行。

### 2.2 下载

可通过全志提供的NUP部署工具包获得：[https://netstorage.allwinnertech.com:5001/sharing/ZIruS49kj](https://netstorage.allwinnertech.com:5001/sharing/ZIruS49kj)

### 2.3 安装

#### 2.3.1 安装IDE 仿真工具

IDE 仿真工具用于离线仿真。该IDE 工具分为Window 版本和Linux 版本。为方便开发，建议安装Linux 版本。因此，需要先准备Ubuntu 的环境（建议使用Ubuntu 16.04LTS、18.04LTS、20.04LTS）。这两个版本都在一个压缩包中，解压IDE 仿真工具包：

```
$ tar xvf Verisilicon_Tool_VivanteIDE_v5.5.0_CL421327_Linux_Windows_SDK_6.4.x_dev_6.4.8_21Q3_CL423100B_20211228.tgz
```

![image-20221208103728986](${media}/image-20221208103728986.png)

<center>图2-2: npu_ext</center>

先解压，解压后可以看到Linux 版本的安装文件：

Vivante_IDE-5.5.0_CL421327-Linux-x86_64-12-27-2021-15.31.40-plus-W-p6.4.x_dev_6.4.8_21Q3_CL423100BInstall

执行该文件开始安装，会弹出安装向导，选择“Yes”。

```
$ ./Vivante_IDE-5.5.0_CL421327-Linux-x86_64-12-27-2021-15.31.40-plus-W-p6.4.x_dev_6.4.8_21Q3_CL423100B-Install
```

![image-20221208103810303](${media}/image-20221208103810303.png)

<center>图2-3: npu_install_1</center>

根据安装向导，逐步安装。

![image-20221208103845079](${media}/image-20221208103845079.png)

<center>图2-4: npu_install_2</center>

![image-20221208103859562](${media}/image-20221208103859562.png)

<center>图2-5: npu_install_3</center>

自己指定安装的目录。

![image-20221208103915416](${media}/image-20221208103915416.png)

<center>图2-6: npu_install_3</center>

License file 暂时可不指定。正式使用时，需要申请License。

![image-20221208103934699](${media}/image-20221208103934699.png)

<center>图2-7: npu_install_4</center>

![image-20221208103948506](${media}/image-20221208103948506.png)

<center>图2-8: npu_install_5</center>

![image-20221208104025782](${media}/image-20221208104025782.png)

<center>图2-9: npu_install_6</center>

![image-20221208104043386](${media}/image-20221208104043386.png)

<center>图2-10: npu_install_3</center>

安装完成后，在控制台中，执行path/to/VivanteIDE5.4.0/ide/vivanteide5.4.0打开IDE 软件，界面如下：

![image-20221208104259333](${media}/image-20221208104259333.png)

<center>图2-11: npu_ide_ok</center>

关于IDE 的使用，可以参考文档VivanteIDE_User_Guide.pdf。

#### 2.3.2 Lincese 申请

仿真工具IDE 需要Lincese 才能使用全部的功能(acuity tools 不需要安装license)，到芯原官网，填写必要的信息，申请一个可用的License, 合法的License 会通过

邮件发送到你的邮箱，根据之前的经验，周期是非常短的。

License 申请地址：https://www.verisilicon.com/cn/VIPAcuityIDELicenseRequest

申请页面说明：

![image-20221208104341187](${media}/image-20221208104341187.png)

<center>图2-12: npu_lic_req</center>

填写相关信息，包括服务器中的MAC 地址，等待邮件发送HostID-*.lic 文件，需要注意的是，license 貌似只和你这里所填写的信息有关，和工具版本没多大关

系，这次升级工具使用的license仍然是5.4.0 IDE 所使用的同一个license 文件。

填写相关信息，包括服务器中的MAC 地址，等待邮件发送HostID-*.lic 文件，需要注意的是，license 貌似只和你这里所填写的信息有关，和工具版本没多大关

系，这次升级工具使用的license仍然是5.4.0 IDE 所使用的同一个license 文件。

2.3.3 安装License

在安装过程中，遇到安装License 页时，可以先直接跳过这一步，等安装过程结束后，在IDE 环境下通过Help->Install License 入口安装Lincese 文件.

![image-20221208104432501](${media}/image-20221208104432501.png)

<center>图2-13: npu_install_lic</center>

#### 2.3.4 安装模型转换工具acuity tools

先解压模型转换工具压缩包, 解压后，得到各个版本的模型转换工具包。

```
$ tar xvf Verisilicon_Tool_Acuity_Toolkit_5.21.1_Binary_Whl_src_20210331.tgz
```

解压后，选择Vivante_acuity_toolkit_binary_6.0.14_20211227_ubuntu18.04.tgz 继续解压

```
$ tar xvf Vivante_acuity_toolkit_binary_6.0.14_20211227_ubuntu18.04.tgz -C /path/to/destination
```

根据需要，当前使用的是binary 版本，所以选择Vivante_acuity_toolkit_binary_6.0.14_20211227_ubuntu18.04.tgz。

关于模型转换工具的使用可以参考文档Vivante.VIP.ACUITY.Toolkit.User.Guide-v0.92-20210922.pdf 。

![image-20221208104613015](${media}/image-20221208104613015.png)

<center>图2-14: npu_acuity</center>

解压后，得到acuity-toolkit-binary-6.0.14，为方便后面配置环境变量，可以将该文件夹放到与Verisilicon IDE 同级目录。

之后，编辑.bashrc，增加以下两行，导出环境变量(具体路径要依据你的安装目录）：

```
export ACUITY_PATH=/path/to/VeriSilicon/$ACUITY_TOOLS_METHOD/bin/
export VIV_SDK=/path/to/VeriSilicon/VivanteIDE5.5.0/cmdtools/
```

之后在控制台端直接执行source ~/.bashrc，安装完成。

#### 2.3.5 测试acuity tools

在控制台中，执行pegasus help出现以下打印即可：

![image-20221208104922729](${media}/image-20221208104922729.png)

