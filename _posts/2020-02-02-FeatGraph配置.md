---
layout:     post
title:      资料
subtitle:   调研，当仓库用
date:       2020-01-31
author:     BY, Amadeus979
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Blog
---

# bench_vanilla_spmm

## 命令行

```shell
python bench_vanilla_spmm.py --dataset data/reddit_csr_float32.npz --feat-len 64 --target cuda
```

## bash或pycharm的Terminal中配置

命令行（bash或pycharm的Ternminal中，不论是否打开conda）

```shell
/home/xry/anaconda3/envs/tvm-build/bin/python /home/xry/Project/FeatGraph/benchmark/bench_vanilla_spmm.py --dataset data/reddit_csr_float32.npz --feat-len 64 --target cuda
```

但实际上pycharm中需要有一些其他配置，包括tvm和该项目的python包的环境变量，和系统cuda等的环境变量

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/20210202223428.png)

查看具体Path

![](https://raw.githubusercontent.com/Amadeus979/CloudImage/main/img/20210202223630.png)

发现PATH自动放到下面去了

完整的环境变量为：

```shell
PYTHONPATH=/home/xry/Project/tvm/python:/home/xry/Project/FeatGraph/python:/home/xry/anaconda3/envs/tvm-build/bin/python:/usr/bin/python:;LD_LIBRARY_PATH=/usr/local/cuda-11.0/lib64;PATH=/usr/local/cuda-11.0/bin
```

Parameters为：

```shell
--dataset data/reddit_coo_float32.npz --feat-len 64 --target cuda
```

解释器路径为：

```shell
/home/xry/Project/FeatGraph/benchmark/bench_vanilla_sddmm.py
```

工作目录：

```shell
/home/xry/Project/FeatGraph/benchmark
```

选中Excution中的Run with Python Console，需要完整的PATH（上面就剩一个cuda11.0的路径了）

此时完整的环境变量是：

```shell
PYTHONPATH=/home/xry/Project/tvm/python:/home/xry/Project/FeatGraph/python:/home/xry/anaconda3/envs/tvm-build/bin/python:/usr/bin/python:;LD_LIBRARY_PATH=/usr/local/cuda-11.0/lib64;PATH=/usr/local/cuda-11.0/bin:/home/xry/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

输出：

```shell
/home/xry/anaconda3/envs/tvm-build/bin/python /home/xry/IDE/Jetbrains/pycharm-2020.3.2/plugins/python/helpers/pydev/pydevconsole.py --mode=client --port=42861
import sys; print('Python %s on %s' % (sys.version, sys.platform))
sys.path.extend(['/home/xry/Project/FeatGraph'])
PyDev console: starting.
Python 3.8.5 (default, Sep  4 2020, 07:30:14) 
[GCC 7.3.0] on linux
import os
os.environ['PYTHONPATH'] = '/home/xry/Project/tvm/python:/home/xry/Project/FeatGraph/python:/home/xry/anaconda3/envs/tvm-build/bin/python:/usr/bin/python:'
os.environ['LD_LIBRARY_PATH'] = '/usr/local/cuda-11.0/lib64'
os.environ['PATH'] = '/usr/local/cuda-11.0/bin:/home/xry/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin'
runfile('/home/xry/Project/FeatGraph/benchmark/bench_vanilla_spmm.py', args=['--dataset', 'data/reddit_csr_float32.npz', '--feat-len', '64', '--target', 'cuda'], wdir='/home/xry/Project/FeatGraph/benchmark')
csr
num_cuda_blocks: 64
num_threads_per_cuda_block: 32
average time of 5 runs: 128.5084812 ms
num_cuda_blocks: 64
num_threads_per_cuda_block: 64
average time of 5 runs: 101.7300082 ms
num_cuda_blocks: 256
num_threads_per_cuda_block: 32
average time of 5 runs: 98.2328608 ms
num_cuda_blocks: 256
num_threads_per_cuda_block: 64
average time of 5 runs: 63.09703559999999 ms
num_cuda_blocks: 1024
num_threads_per_cuda_block: 32
average time of 5 runs: 87.4512484 ms
num_cuda_blocks: 1024
num_threads_per_cuda_block: 64
average time of 5 runs: 70.344957 ms
num_cuda_blocks: 4096
num_threads_per_cuda_block: 32
average time of 5 runs: 83.373778 ms
num_cuda_blocks: 4096
num_threads_per_cuda_block: 64
average time of 5 runs: 78.0573878 ms
num_cuda_blocks: 16384
num_threads_per_cuda_block: 32
average time of 5 runs: 82.54708 ms
num_cuda_blocks: 16384
num_threads_per_cuda_block: 64
average time of 5 runs: 74.6629498 ms
num_cuda_blocks: 65536
num_threads_per_cuda_block: 32
average time of 5 runs: 82.18908780000001 ms
num_cuda_blocks: 65536
num_threads_per_cuda_block: 64
average time of 5 runs: 71.3538984 ms

```

因为其是直接将PATH赋值，因此需要完整的环境变量。