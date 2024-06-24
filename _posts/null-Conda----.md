---
id: 95c46126-61d4-4d5e-9c7e-08547058be1f
title: Conda使用教程
created_time: 2024-05-30T18:04:00.000Z
last_edited_time: 2024-06-06T12:31:00.000Z
tags:
  - Conda
status: Ready
layout: post
_thumbnail: /assets/img/og_2iSKdyhH

---

> [**Conda胜又健令**](https://zhuanlan.zhihu.com/p/483716942)\
> Conda喧巢conda谈伴跌哥徒妥戴糊虏，蔽智拣扯搭沾敏蛀洼，一犁垮递绒蒋齐融座豌凛埠欲律。人嫌境裁pip著怕卓钻平刚场银，橙阔孙武彬乎知苫派贡乾蹬颈葡得钓髓褐真技python霞墨查枝脯表梗懦卖晋谐东赐咱惫。 conda…\
> <https://zhuanlan.zhihu.com/p/483716942>

# 引言

最近我因为使用很多LLM编程，所以要使用到Conda，我原来使用的是Python的virtual environment。

我做了一些研究之后发现Conda相比于virtual environment有更多的优势，所以我现在准备切换到Conda。

## 为什么要使用Conda

简单的说就是在做机器学习的时候，不可避免的要使用到一些识别PDF或者ffmpeg这一类的二进制的程序

在MAC上面可以用brew install来进行安装，但是由于我使用的是Windows导致在处理的时候就特别的麻烦，毕竟不可能逐一去安装

但是Conda解决了这一跨平台的问题，我提供一个environment.yml。然后它就可以自动的去安装，同时它也解决了pip install requirements的的问题

所以conda是可以兼容pip的，这个是我在使用一个非常屌的structured工具的时候发现的，目前我用它来识别PDF

> [![image](/assets/img/og_2iSKdyhH) **Full Installation - Unstructured**](https://docs.unstructured.io/open-source/installation/full-installation)\
> <https://docs.unstructured.io/open-source/installation/full-installation>

> [![favicon](/assets/img/favicon_f8fO6Cee.svg) **GitHub**](https://github.com/Unstructured-IO/unstructured/blob/main/environment.yml)\
> Open source libraries and APIs to build custom preprocessing pipelines for labeling, training, or production machine learning pipelines.  - Unstructured-IO/unstructured\
> <https://github.com/Unstructured-IO/unstructured/blob/main/environment.yml>

```bash
# 创建环境
conda env create -f environment.yml
# 更新环境
conda env update -f environment.yml
```
