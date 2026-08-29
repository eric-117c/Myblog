---
title: CUDA Intro
published: 2026-08-29
pinned: false
description: We will briefly go through the essential part of the AI hardware
image: ""
tags:
  - AI
  - "#hardware"
category: ""
draft: false
author: 山吹
comment: true
date: 2026-08-29
---

# AI需要什么样的硬件
大模型很重要的一环就是神经网络，而神经网络长这样
![CNN.png](https://tu.shanchui.cc/file/blog/wenzhang/1787988485149_CNN.png)
我们先不去解释为什么是这样，我们很容易提炼出他每一层之间的运算都是类似这样的：

$$
y=Wx+b
$$

其中y, W, x, b 都是矩阵，这些运算会涉及到海量的加法和乘法。

比如，输入 $x$ 是 $100\times5$ 的矩阵，系数 $W$ 是  $25\times100$ 的矩阵，相应的 $b$ 和 $y$ 是 $25\times5$

用最naive的数学办法，这里面就涉及设计到 $25\times5\times100=12500$ 次乘法和$25\times5\times99+25\times5=12500$次加法

计算机计算式子和人类不一样，这会经历一个把数据从磁盘搬运到内存，从内存提取运算，然后再送回内存，最后送回磁盘的过程。

这里面决定运算速度的要素简单来讲分为搬运数据（如memory latency，memory bandwidth）的速度和计算的速度（Clock Frequency，Core Count， IPC， FLOPS）

这样的计算任务有个特点：
1. 数据海量
2. 具体的数值计算非常简单

CPU的单核性能高，缓存小，并发数低就不适合这种任务，反观GPU尽管单核性能不算高，但是缓存高，并发高，还有专门为向量设计的硬件架构。

所以GPU会很适合这种神经网络乃至AI任务。

# GPU的硬件结构
GPU(graphics processing unit)是 L2 cache 和众多SM(streaming multiprocess)的集合。SM在GPU中重组为GPC(graphics processing cluster),每个GPC下面含有多个SM。SM会包含众多functional units执行计算任务，一个 register file 和一块 unified data cache (包含了可调配比例的 L1 cache 和 shared memory)。

同时在计算机里面GPU并非单独存在而是和CPU，DRAM部件等紧密联系的存在。GPU一般通过PCIe与CPU传输数据，与GPU DRAM 则由 memory controller管理
![gpu-cpu-system-diagram.png](https://tu.shanchui.cc/file/blog/wenzhang/1787989766016_gpu-cpu-system-diagram.png)


你会觉得上面的东西很绕，我们从搬运数据进来运算的角度就能看清这些东西。

首先，数据从磁盘触发，进入 system DRAM 也就是服务器的内存，内存搬运到L3，再从PCIe总线搬运到GPU DRAM，接下来就是走经过L2进入每个SM的Unified Data Cache，SM把数据从Register File 提上来最后进行计算，再把结果反过来走。
## Grid,Block & Thread


