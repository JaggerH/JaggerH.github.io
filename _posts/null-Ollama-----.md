---
id: 7590fe75-2a28-4084-a792-ba7b4b43444f
title: Ollama本地知识库
created_time: 2024-05-28T14:26:00.000Z
last_edited_time: 2024-06-05T12:55:00.000Z
tags:
  - Ollama
status: Ready
layout: post
_thumbnail: /assets/img/384127954_1010009116964035_2906046058102555347_n_igHW5U2G.jpg

---

> [**🚀 Getting Started | Open WebUI**](https://docs.openwebui.com/getting-started/)\
> How to Install 🚀\
> <https://docs.openwebui.com/getting-started/>

> [![image](/assets/img/384127954_1010009116964035_2906046058102555347_n_igHW5U2G.jpg) **Meta Llama**](https://llama.meta.com/)\
> Meta Llama is the next generation of our open source large language model, available for free for research and commercial use.\
> <https://llama.meta.com/>

> [![image](/assets/img/card-base-2_hu06b1a92291a380a0d2e0ec03dab66b2f_17642_filter_7508709088536350108_xuvWvXn7.png) **Docker Compose 快速部署**](https://doc.fastgpt.in/docs/development/docker/)\
> 使用 Docker Compose 快速部署 FastGPT\
> <https://doc.fastgpt.in/docs/development/docker/>

> [![image](/assets/img/theme-image_MN0mwsqm.png) **Document loaders | 🦜️🔗 LangChain**](https://python.langchain.com/v0.1/docs/modules/data_connection/document_loaders/)\
> Head to Integrations for documentation on built-in document loader integrations with 3rd-party tools.\
> <https://python.langchain.com/v0.1/docs/modules/data_connection/document_loaders/>

> [![image](/assets/img/og_2iSKdyhH) **Notebooks - Unstructured**](https://docs.unstructured.io/examplecode/notebooks)\
> Notebooks contain complete working sample code for end to end solutions.\
> <https://docs.unstructured.io/examplecode/notebooks>

# 项目的结构

```mermaid
graph LR
    A[检索] --> B[下载PDF]
    B --> C[读取PDF内容]
    C --> D[embedding存入]
    D --> E[LLM汇总相关信息]

```

*   检索

    检索部分考虑到使用Google API的价格，所以还是本地Selenium

*   下载PDF

    *   通过巨潮信息的接口下载

    *   或者通过网页信息聚合下载

*   读取PDF

    *   PDF是文字的：使用unstructured批量处理，返回文字

    *   PDF是纯粹图片的：使用PaddleOCR进行读取，因为PaddleOCR在Windows上安装环境问题很多，包括在Docker镜像的GPU调用上也存在很多问题，但是多次调试没弄好，暂时就用CPU吧

        [Docker Hub | Paddle/PaddleOCR](https://hub.docker.com/r/paddlecloud/paddleocr/tags?page=\&page_size=\&ordering=\&name=)

*   embedding存入

    目前使用官网教程，FAISS
