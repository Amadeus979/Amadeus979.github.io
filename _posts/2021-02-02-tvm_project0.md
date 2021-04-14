---
layout:     post
title:      TVM系列
subtitle:   一个GNN开源项目
date:       2020-02-02
author:     BY, Amadeus979
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Blog
---

启动conda环境

```shell
conda activate tvm-build
```

克隆仓库。。。好像不从源码安装就不需要

```shell
git clone git@github.com:amazon-research/FeatGraph.git
```

我的cuda是11.0

```shell
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2020 NVIDIA Corporation
Built on Wed_Jul_22_19:09:09_PDT_2020
Cuda compilation tools, release 11.0, V11.0.221
Build cuda_11.0_bu.TC445_37.28845127_0
```

conda安装

```shell
conda install -c dglteam dgl-cuda11.0
```

好了，可以了。