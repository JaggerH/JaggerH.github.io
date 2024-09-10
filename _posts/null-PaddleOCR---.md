---
id: 2b389a93-748a-4ecc-a4ab-79bb59825db7
title: PaddleOCR的安装
created_time: 2024-06-06T13:38:00.000Z
last_edited_time: 2024-06-08T03:52:00.000Z
tags:
  - Conda
status: Ready
layout: post

---

这个地方我做了很多尝试，终于用上了GPU，结果给大家。

方法还是用Conda，使用Docker镜像有30G，在我的部署中也不能解决GPU的问题，我的实验结果是，如果在Docker中使用`nvidia-smi` 指令，其实显示的CUDA版本也还是宿主机的，在运行的时候会提示GPU not set properly，最后也还是用的CPU。

但是Conda也有一些坑需要踩，首先是你需要安装的东西：

*   CUDA：从官网下载

*   CUNVV：Conda

*   Paddle：pip

*   PaddleOCR：pip

> 💡 这几个是顺序关系，而且需要非常关注各个软件的版本，2-4都可以使用以下的environment.yml安装

在我的安装过程中，CUDA版本是v12.5

```yaml
name: paddleocr

channels:
  - defaults
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/Paddle/
dependencies:
  - paddlepaddle-gpu==2.6.1.post120 # 必须指定post120为cuda版本号
  - cudatoolkit=11.6
  - pip
```

有几个是需要跟你的版本进行绑定的，一个是`CUNVV`的版本，一个是\*\*`paddlepaddle-gpu`\*\* **的版本**

`CUNVV`的版本可以参照这个<https://docs.nvidia.com/deploy/cuda-compatibility/>，我的解决方案比较野蛮，就是直接用最新的。

```bash
conda search cudatoolkit
```

因为我的CUDA版本是12.5，所以我指定\*\*`paddlepaddle-gpu==2.6.1.post120`\*\* ，我理解大版本号要保持一致，我尝试一下过使用\*\*`paddlepaddle-gpu==2.6.1`\*\* ，用不了。

```bash
conda env create -f environment.yml
conda activate paddleocr
```

> 💡 在这里我还碰到过一个隐藏BUG，我安装之后一直用不了，后来发现在`powershell` 中激活环境根本没切换，你可以使用`Get-Command python`来验证，我的案例中`cmd`可以，不过这倒是不影响程序运行。

到这一步也没有完全解决问题

我碰到的问题是

*   找不到CNVV，因为CNVV安装的位置没有在系统路径中

*   提示了一个环境错误

```bash
import os
import sys

os.environ["KMP_DUPLICATE_LIB_OK"] = "TRUE"
cudnn_dir = os.path.join(os.path.dirname(sys.executable), r"Library\bin")
if cudnn_dir not in sys.path:
    os.environ["PATH"] = os.pathsep.join([os.environ["PATH"], cudnn_dir])
    sys.path.append(cudnn_dir)
```

> [**飞桨PaddlePaddle-源于产业实践的开源深度学习平台**](https://www.paddlepaddle.org.cn/)\
> 飞桨致力于让深度学习技术的创新与应用更简单。具有以下特点：同时支持动态图和静态图，兼顾灵活性和效率；精选应用效果最佳算法模型并提供官方支持；真正源于产业实践，提供业界最强的超大规模并行深度学习能力；推理引擎一体化设计，提供训练到多端推理的无缝对接；唯一提供系统化技术服务与支持的深度学习平台\
> <https://www.paddlepaddle.org.cn/>
