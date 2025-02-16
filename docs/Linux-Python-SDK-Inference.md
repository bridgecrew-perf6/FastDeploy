# 简介

本文档介绍FastDeploy中的模型SDK，在**Intel x86_64 / NVIDIA GPU Linux Python** 环境下： （1)图像推理部署步骤； （2）介绍模型推流全流程API，方便开发者了解项目后二次开发。
其中Linux C++请参考[Linux CPP环境下的推理部署](./Linux-CPP-SDK-Inference.md)文档。

<!--ts-->

* [简介](#简介)

* [环境准备](#环境准备)
  
  * [1. SDK下载](#1-sdk下载)
  * [2. Python环境](#2-python环境)
  * [3. 安装依赖](#3-安装依赖)
    * [3.1 安装paddlepaddle](#31-安装paddlepaddle)
    * [3.2 安装EasyEdge Python Wheel 包](#32-安装easyedge-python-wheel-包)

* [快速开始](#快速开始)
  
  * [1. 文件结构说明](#1-文件结构说明)
  * [2. 测试Demo](#2-测试demo)
    * [2.1 预测图像](#21-预测图像)

* [预测API流程详解](#预测api流程详解)
  
  * [1. 基础流程](#1-基础流程)
  * [2. 初始化](#2-初始化)
  * [3. SDK参数配置](#3-sdk参数配置)
  * [4. 预测图像](#4-预测图像)

* [FAQ](#faq)
  
  <!--te-->

# 环境准备

## 1. SDK下载

根据开发者模型、部署芯片、操作系统需要，在图像界面[飞桨开源模型](https://ai.baidu.com/easyedge/app/openSource)或[GIthub](https://github.com/PaddlePaddle/FastDeploy)中选择对应的SDK进行下载。解压后SDK目录结构如下：

```shell
EasyEdge-Linux-x86-[部署芯片]
├── RES  # 模型文件资源文件夹，可替换为其他模型
├── README.md
├── cpp  # C++ SDK
└── python  # Python SDK
```

## 2. Python环境

> 当前SDK仅支持Python 3.5, 3.6, 3.7

使用如下命令获取已安装的Python版本号。如果本机的版本不匹配，建议使用[pyenv](https://github.com/pyenv/pyenv)、[anaconda](https://www.anaconda.com/)等Python版本管理工具对SDK所在目录进行配置。

```shell
$python3 --version
```

接着使用如下命令确认pip的版本是否满足要求，要求pip版本为20.2.2或更高版本。详细的pip安装过程可以参考[官网教程](https://pip.pypa.io/en/stable/installation/)。

```shell
$python3 -m pip --version
```

## 3. 安装依赖

### 3.1 安装paddlepaddle

根据具体的部署芯片（CPU/GPU）安装对应的PaddlePaddle的whl包。

`x86_64 CPU` 平台可以使用如下命令进行安装：

```shell
python3 -m pip install paddlepaddle==2.2.2 -i https://mirror.baidu.com/pypi/simple 
```

NVIDIA GPU平台的详细安装教程可以参考[官网Paddle安装教程](https://www.paddlepaddle.org.cn/install/quick?docurl=/documentation/docs/zh/install/pip/linux-pip.html)。  

> 使用 NVIDIA GPU 预测时，必须满足：  
> 
> 1. 机器已安装 cuda, cudnn  
>    2. 已正确安装对应 cuda 版本的paddle 版本  
>    3. 通过设置环境变量`FLAGS_fraction_of_gpu_memory_to_use`设置合理的初始内存使用比例

### 3.2 安装EasyEdge Python Wheel 包

在`python`目录下，安装特定Python版本的EasyEdge Wheel包。对`x86_64 CPU` 或 `x86_64 Nvidia GPU平台 `可以使用如下命令进行安装，具体名称以 Python SDK 包中的 whl 为准。

```shell
python3 -m pip install -U BaiduAI_EasyEdge_SDK-{SDK版本号}-cp{Python版本号}-cp{Python版本号}m-linux_x86_64.whl
```

`armv8 CPU平台`可以使用如下命令进行安装：

```shell
python3 -m pip install -U BaiduAI_EasyEdge_SDK-{版本号}-cp36-cp36m-linux_aarch64.whl
```

# 快速开始

## 1. 文件结构说明

Python SDK文件结构如下：

```shell
EasyEdge-Linux-x86--[部署芯片]
├──...
├──python  # Linux Python SDK
    ├── # 特定Python版本的EasyEdge Wheel包, 二次开发可使用
    ├── BaiduAI_EasyEdge_SDK-1.2.8-cp35-cp35m-linux_x86_64.whl 
    ├── BaiduAI_EasyEdge_SDK-1.2.8-cp36-cp36m-linux_x86_64.whl
    ├── BaiduAI_EasyEdge_SDK-1.2.8-cp37-cp37m-linux_x86_64.whl
    ├── infer_demo # demo体验完整文件
    │   ├──  demo_xxx.py # 包含前后处理的端到端推理demo文件
    │   └──  demo_serving.py # 提供http服务的demo文件
    ├── tensor_demo # tensor in/out demo文件
    │   └──  demo_xxx.py
```

## 2. 测试Demo

> 模型资源文件默认已经打包在开发者下载的SDK包中， 默认为`RES`目录。

### 2.1 预测图像

使用infer_demo文件夹下的demo文件。

```bash
python3 demo_x86_cpu.py {模型RES文件夹}  {测试图片路径}
```

运行效果示例：

<div align=center><img src="https://user-images.githubusercontent.com/54695910/175854068-28d27c0a-ef83-43ee-9e89-b65eed99b476.jpg" width="400"></div>

```shell
2022-06-14 14:40:16 INFO [EasyEdge] [demo_nvidia_gpu.py:38] 140518522509120: Init paddlefluid engine...
2022-06-14 14:40:20 INFO [EasyEdge] [demo_nvidia_gpu.py:38] 140518522509120: Paddle version: 2.2.2
{'confidence': 0.9012349843978882, 'index': 8, 'label': 'n01514859 hen'}
```

可以看到，运行结果为`index：8，label：hen`，通过imagenet [类别映射表](https://gist.github.com/yrevar/942d3a0ac09ec9e5eb3a)，可以找到对应的类别，即 'hen'，由此说明我们的预测结果正确。

# 预测API流程详解

本章节主要结合前文的Demo示例来介绍推理API，方便开发者学习并将运行库嵌入到开发者的程序当中，更详细的API请参考`infer_demo/demo_xx_xx.py`文件，查看下面的Python代码中的step注释说明。

## 1. 基础流程

> ❗注意，请优先参考SDK中自带demo的使用流程和说明。遇到错误，请优先参考文件中的注释、解释、日志说明。

`infer_demo/demo_xx_xx.py`

```python
# 引入EasyEdge运行库
import BaiduAI.EasyEdge as edge

# 创建并初始化一个预测Progam；选择合适的引擎
pred = edge.Program()
pred.init(model_dir={RES文件夹路径}, device=edge.Device.CPU, engine=edge.Engine.PADDLE_FLUID) # x86_64 CPU
# pred.init(model_dir=_model_dir, device=edge.Device.GPU, engine=edge.Engine.PADDLE_FLUID) # x86_64 Nvidia GPU
# pred.init(model_dir=_model_dir, device=edge.Device.CPU, engine=edge.Engine.PADDLE_LITE) # armv8 CPU

# 预测图像
res = pred.infer_image({numpy.ndarray的图片})

# 关闭结束预测Progam
pred.close()
```

`infer_demo/demo_serving.py`

```python
import BaiduAI.EasyEdge as edge
from BaiduAI.EasyEdge.serving import Serving

# 创建并初始化Http服务
server = Serving(model_dir={RES文件夹路径}, license=serial_key)

# 运行Http服务
# 请参考同级目录下demo_xx_xx.py里:
# pred.init(model_dir=xx, device=xx, engine=xx, device_id=xx)
# 对以下参数device\device_id和engine进行修改
server.run(host=host, port=port, device=edge.Device.CPU, engine=edge.Engine.PADDLE_FLUID) # x86_64 CPU
# server.run(host=host, port=port, device=edge.Device.GPU, engine=edge.Engine.PADDLE_FLUID) # x86_64 Nvidia GPU
# server.run(host=host, port=port, device=edge.Device.CPU, engine=edge.Engine.PADDLE_LITE) # armv8 CPU
```

## 2. 初始化

- 接口

```python
    def init(self,
          model_dir,
          device=Device.CPU,
          engine=Engine.PADDLE_FLUID,
          config_file='conf.json',
          preprocess_file='preprocess_args.json',
          model_file='model',
          params_file='params',
          label_file='label_list.txt',
          infer_cfg_file='infer_cfg.json',
          device_id=0,
          thread_num=1
          ):
       """
       Args:
           model_dir: str
           device: BaiduAI.EasyEdge.Device，比如：Device.CPU
           engine: BaiduAI.EasyEdge.Engine， 比如： Engine.PADDLE_FLUID
           config_file: str
           preprocess_file: str
           model_file: str
           params_file: str
           label_file: str 标签文件
           infer_cfg_file: 包含预处理、后处理信息的文件
        device_id: int 设备ID
           thread_num: int CPU的线程数

       Raises:
           RuntimeError, IOError
       Returns:
           bool: True if success
       """
```

若返回不是True，请查看输出日志排查错误原因。

## 3. SDK参数配置

使用 CPU 预测时，可以通过在 init 中设置 thread_num 使用多线程预测。如：

```python
pred.init(model_dir=_model_dir, device=edge.Device.CPU, engine=edge.Engine.PADDLE_FLUID, thread_num=4)
```

使用 GPU 预测时，可以通过在 init 中设置 device_id 指定需要的GPU device id。如：

```python
pred.init(model_dir=_model_dir, device=edge.Device.GPU, engine=edge.Engine.PADDLE_FLUID, device_id=0)
```

## 4. 预测图像

- 接口

```python
    def infer_image(self, img,
                   threshold=0.3,
                   channel_order='HWC',
                   color_format='BGR',
                   data_type='numpy')
        """

       Args:
           img: np.ndarray or bytes
           threshold: float
               only return result with confidence larger than threshold
           channel_order: string
               channel order HWC or CHW
           color_format: string
               color format order RGB or BGR
           data_type: string
               仅在图像分割时有意义。 'numpy' or 'string'
               'numpy': 返回已解析的mask
               'string': 返回未解析的mask游程编码

       Returns:
           list

       """
```

| 字段         | 类型                   | 取值        | 说明                       |
| ---------- | -------------------- | --------- | ------------------------ |
| confidence | float                | 0~1       | 分类或检测的置信度                |
| label      | string               |           | 分类或检测的类别                 |
| index      | number               |           | 分类或检测的类别                 |
| x1, y1     | float                | 0~1       | 物体检测，矩形的左上角坐标 (相对长宽的比例值) |
| x2, y2     | float                | 0~1       | 物体检测，矩形的右下角坐标（相对长宽的比例值）  |
| mask       | string/numpy.ndarray | 图像分割的mask |                          |

***关于矩形坐标***

x1 * 图片宽度 = 检测框的左上角的横坐标

y1 * 图片高度 = 检测框的左上角的纵坐标

x2 * 图片宽度 = 检测框的右下角的横坐标

y2 * 图片高度 = 检测框的右下角的纵坐标

可以参考 demo 文件中使用 opencv 绘制矩形的逻辑。

***结果示例***

i) 图像分类

```json
{
    "index": 736,
    "label": "table",
    "confidence": 0.9
}
```

ii) 物体检测

```json
{
    "index": 8,
    "label": "cat",
    "confidence": 1.0,
    "x1": 0.21289,
    "y1": 0.12671,
    "x2": 0.91504,
    "y2": 0.91211,
}
```

iii) 图像分割

```json
{
    "name": "cat",
    "score": 1.0,
    "location": {
        "left": ..., 
        "top": ..., 
        "width": ...,
        "height": ...,
    },
    "mask": ...
}
```

mask字段中，data_type为`numpy`时，返回图像掩码的二维数组

```text
{
  {0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
  {0, 0, 0, 1, 1, 1, 0, 0, 0, 0},
  {0, 0, 0, 1, 1, 1, 0, 0, 0, 0},
  {0, 0, 0, 1, 1, 1, 0, 0, 0, 0},
  {0, 0, 0, 1, 1, 1, 0, 0, 0, 0},
  {0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
}
其中1代表为目标区域，0代表非目标区域
```

data_type为`string`时，mask的游程编码，解析方式可参考 [demo](https://github.com/Baidu-AIP/EasyDL-Segmentation-Demo)。


# FAQ

1. 执行infer_demo文件时，提示your generated code is out of date and must be regenerated with protoc >= 3.19.0
   
   进入当前项目，首先卸载protobuf

```shell
python3 -m pip uninstall protobuf
```

       安装低版本protobuf

```shell
python3 -m pip install protobuf==3.19.0
```
