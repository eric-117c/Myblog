---
title: 大模型扫盲计划——总体规划
published: 2026-08-28
pinned: true
description: 这里打算做一个大模型扫盲的长期博客
image: ""
tags:
  - AI
  - 大模型
  - 入门教程
category: 大模型扫盲
draft: false
author: 山吹
comment: true
date: 2026-08-28
---

# 我准备如何写一套 AI 扫盲系列：从硬件、数学到大模型与前沿研究

如果要真正理解今天的人工智能，仅仅会调用几个 API，或者知道 Transformer、GPT、Agent 这些名词，其实远远不够。

一个现代 AI 系统背后同时涉及计算机体系结构、GPU、线性代数、概率论、优化算法、机器学习、神经网络、Transformer、大模型训练、分布式系统以及推理优化等大量知识。

很多 AI 教程的问题在于，它们往往从某一个中间位置开始。

有的从 Python 开始，有的从机器学习算法开始，有的直接从 Transformer 开始，还有的干脆从“如何调用大模型 API”开始。

这些内容当然有价值，但如果没有完整的知识链，就很容易出现一种情况：

> 每个概念都好像认识，但不知道它们为什么会连接在一起。

因此，我准备写一套完整的 AI 扫盲系列。

这个系列不会只介绍“AI 怎么用”，而是尝试回答一个更加完整的问题：

> **从一颗芯片开始，一直到一个大语言模型生成一句话，中间究竟发生了什么？**

整套内容会沿着这样一条主线展开：

```
计算机硬件
    ↓
矩阵计算
    ↓
数学基础
    ↓
机器学习
    ↓
神经网络
    ↓
深度学习
    ↓
Attention
    ↓
Transformer
    ↓
大语言模型
    ↓
多模态模型
    ↓
强化学习与推理模型
    ↓
AI 系统优化
    ↓
Agent 与未来架构
```

我的目标不是写一本严格意义上的数学教材或者是一套单纯的工程教程。

我更希望最终得到的是一张完整的 **AI 知识地图**。

---

## 一、为什么我要从硬件开始讲 AI

很多人工智能课程都会从数学或者 Python 开始。

但我认为，如果希望真正理解现代 AI，硬件其实是一个非常重要的起点。

因为今天深度学习的发展，本身就在很大程度上受到计算硬件的影响。

例如，我们经常听到这些概念：

- GPU
    
- CUDA
    
- Tensor Core
    
- FP16
    
- BF16
    
- FP8
    
- 显存
    
- KV Cache
    
- FlashAttention
    
- Tensor Parallel
    
- Pipeline Parallel
    

这些并不是和 AI 算法完全独立的“工程细节”。

恰恰相反，它们会直接影响模型架构和算法设计。

例如 Transformer 中最重要的操作之一就是矩阵乘法。

一个简单的神经网络层可以写成：

$$
y=Wx+b
$$

在数学上，这只是一次线性变换。

但是当矩阵的规模变成：

```
4096 × 4096
8192 × 8192
甚至更大
```

并且一个模型中存在几十层甚至上百层这样的操作时，问题就从纯数学问题变成了计算机系统问题。

于是就会进一步涉及：

```
矩阵乘法
↓
GPU 并行计算
↓
显存带宽
↓
Cache
↓
Arithmetic Intensity
↓
Memory Bound / Compute Bound
```

继续往后，就能理解：

> 为什么 FlashAttention 会出现？

> 为什么 KV Cache 会成为大模型推理的重要瓶颈？

> 为什么需要 FP16、BF16 和 FP8？

> 为什么现代大模型往往必须使用多张 GPU？

这就是为什么我会从最基础的计算机结构开始。

---

# 二、第一部分：计算机与 AI 硬件

这一部分计划写大约 6 篇文章。

内容大致包括：

### 1. CPU 到底是怎么计算的

从最底层开始理解：

```
晶体管
↓
逻辑门
↓
加法器
↓
寄存器
↓
ALU
↓
CPU
```

我没那个能力去做到教材的水平。

主要是去回答：

> 一条数学公式最后究竟是怎么变成硬件上的计算的？

---

### 2. 内存到底是什么

这一篇会介绍：

- Register
    
- Cache
    
- RAM
    
- Storage
    

并解释现代计算机中的 Memory Hierarchy。

例如：

```
Register
  ↓
L1 Cache
  ↓
L2 Cache
  ↓
L3 Cache
  ↓
DRAM
  ↓
SSD
```

这一部分最终会服务于一个重要概念：

> 在现代 AI 中，搬运数据有时候比计算本身更加昂贵。

这也是理解 FlashAttention、KV Cache 以及很多推理优化技术的基础。

---

### 3. 为什么 AI 使用 GPU

CPU 很强，为什么深度学习却主要运行在 GPU 上？

这一篇会介绍：

- CPU
    
- GPU
    
- SIMD
    
- SIMT
    
- CUDA Core
    
- Tensor Core
    

然后解释：

> 深度学习本质上包含大量可以并行执行的矩阵运算。

因此 GPU 非常适合这种计算模式。

---

### 4. AI 为什么需要低精度计算

这一篇会介绍：

- FP32
    
- FP16
    
- BF16
    
- FP8
    
- INT8
    
- INT4
    

以及 Quantization。

重点解释一个看似反直觉的问题：

> 为什么减少数字精度，有时候不会明显降低模型能力？

然后引出：

```
模型压缩
显存占用
推理速度
Tensor Core
```

---

### 5. 一次矩阵乘法到底发生了什么

这一篇会把数学和硬件正式连接起来。

从：

[  
C = AB  
]

开始。

一直讲到：

```
Matrix Multiplication
↓
Tile
↓
Thread
↓
Warp
↓
GPU Kernel
```

最终让读者第一次真正理解：

> 一个神经网络到底是怎样跑在 GPU 上的。

---

### 6. AI 为什么会受到内存限制

最后介绍：

- FLOPS
    
- Memory Bandwidth
    
- Arithmetic Intensity
    
- Memory Bound
    
- Compute Bound
    

这会成为后面介绍：

- FlashAttention
    
- vLLM
    
- PagedAttention
    
- KV Cache
    

的重要铺垫。

---

# 三、第二部分：AI 真正需要的数学

数学部分我不准备写成完整的高等数学或者线性代数课程。

因为 AI 学习中一个非常常见的问题就是：

> 学了大量数学，却不知道这些数学到底在哪里用。

所以这里会采用另一种方式：

**只介绍真正会在 AI 中反复出现的数学。**

计划大约写 6 篇。

---

## 1. 向量、矩阵和 Tensor 到底是什么

首先介绍：

```
Scalar
Vector
Matrix
Tensor
```

然后介绍：

- Dot Product
    
- Matrix Multiplication
    
- Transpose
    

最终连接到神经网络中最常见的一条公式：

[  
y = Wx+b  
]

---

## 2. 线性代数为什么贯穿整个 AI

这一篇会进一步介绍：

- Linear Transformation
    
- Basis
    
- Projection
    
- Rank
    
- Eigenvalue
    
- SVD
    

但我不会单纯介绍公式。

例如介绍 Matrix Rank 时，会直接告诉读者：

```
Matrix Rank
↓
Low Rank
↓
Low Rank Approximation
↓
LoRA
```

这样以后讲 LoRA 时，就可以直接回到这里。

---

## 3. 微积分与 Gradient Descent

这一篇介绍：

- Derivative
    
- Partial Derivative
    
- Gradient
    
- Chain Rule
    

然后进入机器学习最核心的公式之一：

\theta_t-\eta\nabla L  
]

也就是 Gradient Descent。

---

## 4. 概率论

介绍：

- Random Variable
    
- Probability Distribution
    
- Expectation
    
- Variance
    
- Conditional Probability
    
- Bayes Rule
    

最终为机器学习和语言模型建立概率基础。

---

## 5. 信息论

这一篇会重点介绍：

- Entropy
    
- Cross Entropy
    
- KL Divergence
    

因为 Cross Entropy 几乎贯穿整个现代深度学习。

最终需要解释：

> 为什么训练语言模型，本质上可以变成一个 Cross Entropy 问题？

---

## 6. Optimization

最后介绍常见优化算法：

```
Gradient Descent
↓
SGD
↓
Momentum
↓
Adam
↓
AdamW
```

同时介绍：

- Learning Rate
    
- Weight Decay
    
- Learning Rate Scheduler
    

到这里，训练神经网络所需要的数学工具基本就准备完成了。

---

# 四、第三部分：机器学习

这一部分大约安排 6 篇。

首先回答一个最重要的问题：

> 什么叫“机器学习”？

整个过程其实可以抽象成：

```
Dataset
↓
Model
↓
Prediction
↓
Loss
↓
Optimization
↓
Better Model
```

在这一部分中，我会介绍几个经典算法。

---

## 1. Linear Regression

这是最适合第一次理解机器学习的算法。

可以完整看到：

```
Model
Loss
Gradient
Optimization
```

整个流程。

---

## 2. Logistic Regression

进一步进入分类问题。

同时介绍：

- Sigmoid
    
- Softmax
    
- Cross Entropy
    

这些概念以后都会在神经网络和大模型中不断出现。

---

## 3. Decision Tree

介绍：

- Decision Tree
    
- Random Forest
    
- Gradient Boosting
    

这一篇的重要意义是告诉读者：

> AI 并不等于神经网络。

---

## 4. SVM

介绍：

- Margin
    
- Support Vector
    
- Kernel
    

帮助理解传统机器学习的另一种思想。

---

## 5. Clustering

介绍：

- K-Means
    
- Unsupervised Learning
    

---

## 6. PCA

介绍：

- Dimensionality Reduction
    
- Principal Component
    

并重新连接到之前学过的线性代数。

---

# 五、第四部分：神经网络与深度学习

进入这里以后，现代 AI 的主线就真正开始了。

计划大约写 7 篇。

---

## 1. 从 Perceptron 到神经网络

介绍：

```
Neuron
Layer
Activation
Weight
Bias
```

然后构建一个最简单的 MLP。

---

## 2. Backpropagation

这会是整个系列最重要的文章之一。

因为很多人知道：

> 神经网络通过反向传播训练。

但真正理解 Backpropagation 的人并没有想象中那么多。

这一篇会从 Computational Graph 开始。

最终手推一个两层神经网络。

---

## 3. 为什么神经网络可以学习复杂函数

介绍：

- Representation Learning
    
- Feature Learning
    
- Nonlinearity
    
- Universal Approximation
    

---

## 4. CNN

从传统图像处理进入：

```
Convolution
Kernel
Feature Map
Pooling
```

然后简要介绍：

```
LeNet
↓
AlexNet
↓
VGG
↓
ResNet
```

---

## 5. RNN 与 LSTM

这一篇的真正目的并不是学习 RNN。

而是为了回答：

> 为什么后来必须出现 Transformer？

---

## 6. Embedding

介绍：

- Word Embedding
    
- Word2Vec
    
- Vector Space
    
- Semantic Similarity
    

这也是进入现代 NLP 的桥梁。

---

## 7. 深度学习训练完整流程

把前面所有知识组合起来：

```
Dataset
↓
Batch
↓
Forward
↓
Loss
↓
Backward
↓
Optimizer
↓
Update
```

同时介绍：

- Epoch
    
- Batch Size
    
- Dropout
    
- Normalization
    
- Initialization
    
- Regularization
    

---

# 六、第五部分：Transformer 与大语言模型

这一部分会是整个系列的核心。

计划至少写 9 篇。

---

## 1. Transformer 之前的 NLP

首先介绍：

```
N-Gram
↓
Word2Vec
↓
RNN
↓
LSTM
↓
Seq2Seq
```

这样读者才能理解：

> Transformer 到底解决了什么问题。

---

## 2. Attention

Attention 会单独写一篇非常详细的文章。

核心公式：

softmax  
\left(  
\frac{QK^T}{\sqrt{d_k}}  
\right)V  
]

但是不会直接从公式开始。

而是从问题出发：

> 一个词应该怎样知道自己需要关注前面的哪些词？

然后逐渐推导出：

```
Query
Key
Value
```

---

## 3. Transformer

随后完整拆解：

```
Token
↓
Embedding
↓
Position
↓
Multi-Head Attention
↓
Residual
↓
LayerNorm
↓
MLP
↓
Transformer Block
```

最后组合成完整 Transformer。

---

## 4. Tokenizer

介绍：

- Token
    
- Vocabulary
    
- BPE
    
- WordPiece
    
- SentencePiece
    
- Byte-level Tokenization
    

回答：

> 为什么大模型看到的并不是“文字”？

---

## 5. GPT 为什么可以生成文本

介绍 Autoregressive Language Model：

\prod_t  
P(x_t|x_{<t})  
]

然后介绍：

- Logits
    
- Softmax
    
- Temperature
    
- Top-k
    
- Top-p
    

最终完整解释：

> GPT 是如何一个 Token 一个 Token 地生成一句话的？

---

## 6. BERT、GPT 和 T5

介绍三种主流架构：

```
Encoder Only
Decoder Only
Encoder-Decoder
```

并解释它们各自适合解决什么问题。

---

## 7. Scaling Law

从这里开始真正进入“大模型”。

介绍：

- Model Size
    
- Dataset Size
    
- Compute
    

以及 Scaling Law。

然后讨论一个重要问题：

> 为什么模型做大以后，能力会不断提升？

---

## 8. 一个 LLM 是怎样训练出来的

从原始数据开始：

```
Internet Data
↓
Cleaning
↓
Deduplication
↓
Tokenizer
↓
Pretraining
↓
SFT
↓
Preference Optimization
↓
Evaluation
↓
Deployment
```

让读者第一次看到完整的大模型生命周期。

---

## 9. 从零实现一个 MiniGPT

这一篇会尽量用一个小型 PyTorch 项目实现：

- Tokenizer
    
- Embedding
    
- Attention
    
- Transformer
    
- Training
    
- Generation
    

最终真正训练一个可以生成文本的小模型。

---

# 七、第六部分：现代生成式 AI

Transformer 以后，AI 开始逐渐从语言扩展到视觉、语音和视频。

这一部分计划大约 6 篇。

---

## 1. GAN

简要介绍生成模型历史。

主要介绍 Generator 与 Discriminator。

---

## 2. Diffusion Model

解释一个非常有意思的思想：

```
Image
↓
Noise
↓
More Noise
↓
Pure Noise
```

训练模型学习反向过程：

```
Noise
↓
Less Noise
↓
Image
```

然后理解 Stable Diffusion 等模型的基础原理。

---

## 3. Vision Transformer

解释：

> 为什么图片也可以被 Transformer 处理？

将图片切成：

```
Patch
↓
Visual Token
↓
Transformer
```

---

## 4. CLIP

介绍如何让：

```
Image
```

和：

```
Text
```

进入同一个 Embedding Space。

这也是理解现代多模态模型非常重要的一步。

---

## 5. VLM

进一步解释：

```
Image
↓
Vision Encoder
↓
Projection
↓
LLM
↓
Text
```

介绍现代 Vision Language Model 的基本结构。

---

## 6. Reinforcement Learning

介绍：

- Agent
    
- Environment
    
- State
    
- Action
    
- Reward
    
- Policy
    

然后介绍：

- Policy Gradient
    
- PPO
    

为后面的 RLHF 和 Reasoning Model 做准备。

---

# 八、第七部分：现代大模型训练与推理

这一部分会更加偏向工程与研究。

也是我认为这套系列与普通 AI 科普最容易形成区别的地方。

计划大约 6 篇。

---

## 1. SFT、RLHF、DPO 与 Reasoning

介绍完整 Post-training 流程。

例如：

```
Pretraining
↓
Supervised Fine-Tuning
↓
Preference Learning
↓
RL / DPO
↓
Reasoning
```

---

## 2. LoRA 与 QLoRA

这里会重新回到之前介绍的：

```
Matrix Rank
↓
Low Rank Approximation
↓
LoRA
```

形成一次完整知识回链。

---

## 3. MoE

介绍 Mixture of Experts：

```
Token
↓
Router
↓
Expert A
Expert B
Expert C
...
```

然后回答：

> 为什么一个拥有上千亿参数的模型，不一定每次推理都使用全部参数？

---

## 4. FlashAttention 与 KV Cache

这一篇会把整个系列最开始的硬件知识重新连接回来。

例如：

```
Attention
↓
Memory Access
↓
HBM
↓
SRAM
↓
Tiling
↓
FlashAttention
```

同时介绍 KV Cache。

---

## 5. vLLM 与 PagedAttention

解释大模型 Serving 中的显存管理。

这里可以类比操作系统中的：

```
Virtual Memory
Page
Paging
```

再解释 PagedAttention。

---

## 6. Speculative Decoding

介绍：

```
Small Model
↓
Predict Several Tokens
↓
Large Model Verify
↓
Accept / Reject
```

以及为什么这能够减少大模型自回归推理的延迟。

---

# 九、最后一部分：AI 的未来

主线最后一篇不会尝试预测：

> AGI 什么时候出现？

这种问题。

而是整理当前 AI 正在探索哪些方向。

例如：

```
Modern AI
│
├── Large Language Model
│
├── Multimodal
│   ├── Image
│   ├── Audio
│   └── Video
│
├── Reasoning
│   ├── RL
│   └── Test-Time Compute
│
├── Agent
│   ├── Tool Use
│   ├── Memory
│   └── Planning
│
├── Efficient AI
│   ├── Quantization
│   ├── Distillation
│   └── Speculative Decoding
│
└── New Architecture
    ├── MoE
    ├── SSM
    └── Mamba
```

这一篇会成为整个系列进入“持续更新阶段”的入口。

---

# 十、我准备读哪些资料

整个过程中不会依赖某一本教材。

而会同时使用三类资料：

## 第一类：教材

硬件部分：

- 《Computer Organization and Design》
    
- 《Computer Architecture: A Quantitative Approach》
    

数学部分：

- 《Mathematics for Machine Learning》
    

机器学习部分：

- 《Pattern Recognition and Machine Learning》
    
- 《Probabilistic Machine Learning》
    

深度学习部分：

- 《Deep Learning》
    
- 《Dive into Deep Learning》
    

---

## 第二类：公开课程

大致学习顺序计划如下：

```
Nand2Tetris / CS61C
        ↓
3Blue1Brown
        ↓
Stanford CS229
        ↓
MIT 6.S191
        ↓
Stanford CS231n
        ↓
Stanford CS224N
        ↓
Stanford CS336
        ↓
Berkeley CS285
```

其中不会要求自己完整修完每一门课。

更多时候是：

> 写到哪里，就学习对应章节。

例如准备写 CNN：

就看 CS231n。

准备写 Transformer：

就看 CS224N。

准备写 LLM Training：

重点看 CS336。

这样效率会高很多。

---

# 十一、论文应该怎么读

我不准备一开始就每天刷大量最新论文。

因为没有基础知识时，看最新论文的收益实际上很低。

首先会建立一条 AI 发展的“论文主线”。

例如：

```
Perceptron
↓
Backpropagation
↓
LeNet
↓
AlexNet
↓
ResNet
↓
Word2Vec
↓
Seq2Seq
↓
Attention
↓
Attention Is All You Need
↓
BERT
↓
GPT
↓
Scaling Laws
↓
ViT
↓
CLIP
↓
LoRA
↓
DDPM
↓
FlashAttention
↓
DPO
↓
Mamba
↓
Reasoning Models
```

第一阶段只需要认真理解二十篇左右真正重要的论文。

之后再逐渐进入最新研究。

---

# 十二、每一篇文章应该怎么写

我希望整个系列尽量保持统一结构。

例如：

```
0. 为什么需要这个技术？

1. 它试图解决什么问题？

2. 最简单的方法是什么？

3. 数学原理是什么？

4. 算法具体怎么工作？

5. 手算一个简单例子

6. 用 Python / PyTorch 实现

7. 这种方法有什么问题？

8. 后来的算法怎么改进？

9. 今天它还在哪里使用？

10. 推荐论文与延伸阅读
```

我尤其希望避免一种写法：

> Attention 定义如下。

然后直接扔出一行公式。

更好的方式是：

```
RNN 很难处理长距离关系
↓
能不能让每个 Token 直接查看其他 Token？
↓
那应该看谁？
↓
需要计算相关性
↓
Query 与 Key
↓
相关性决定 Value 的权重
↓
Attention
```

换句话说：

> **先讲问题，再讲解决方案，最后才讲公式。**

---

# 十三、整个系列最重要的是“知识回链”

整个博客不会只是几十篇独立文章。

真正理想的效果是形成大量知识链接。

例如：

```
Matrix Multiplication
↓
Neural Network
↓
Transformer
↓
Attention
↓
GPU Memory
↓
FlashAttention
```

另一条路线：

```
Linear Algebra
↓
Matrix Rank
↓
SVD
↓
Low Rank Approximation
↓
LoRA
```

还有：

```
Probability
↓
Maximum Likelihood
↓
Cross Entropy
↓
Language Modeling
↓
Next Token Prediction
↓
GPT
```

甚至：

```
Operating System
↓
Virtual Memory
↓
Paging
↓
KV Cache Management
↓
PagedAttention
↓
vLLM
```

当这样的链接越来越多以后，整个博客就会逐渐从“文章合集”变成真正的：

> **AI Knowledge Graph。**

---

# 十四、最终准备写多少篇

目前我的规划是：

|部分|文章数量|
|---|---|
|AI 导论|2|
|计算机与硬件|6|
|数学|6|
|机器学习|6|
|深度学习|7|
|Transformer / LLM|9|
|多模态与生成模型|6|
|大模型系统与前沿|6|
|合计|**48 篇**|

因此第一阶段的目标就是：

# 48 篇主线文章

但这并不意味着整个系列写完 48 篇以后就结束。

主线内容应该尽量保持稳定。

例如：

- Linear Algebra
    
- Backpropagation
    
- Transformer
    
- Attention
    
- GPU
    
- Scaling Law
    

这些内容不会因为某个新模型发布就失效。

而模型更新和最新论文则会单独建立：

# AI 前沿

栏目。

例如未来可能不断出现：

```
论文解读
模型架构分析
新推理算法
新多模态模型
Agent
Video Model
AI Compiler
Inference Optimization
Scaling Research
```

因此最终博客会形成两套体系：

```
AI 从零开始
│
└── 48 篇稳定主线

AI 前沿
│
├── Paper
├── Model
├── System
└── Research
```

前者负责建立知识。

后者负责跟踪变化。

---

# 十五、不会按照目录顺序开始写

虽然整个目录从硬件开始，但真正发布文章时，我并不准备严格按照：

```
01
02
03
04
05
...
```

一路写下去。

因为这样非常容易出现一个问题：

写了十几篇以后，整个博客仍然停留在基础阶段。

所以第一批我准备先建立整个 AI 的“骨架”。

优先完成：

1. AI 到底是什么
    
2. 一个大模型的一生
    
3. CPU 与 GPU
    
4. 矩阵到底是什么
    
5. Gradient Descent
    
6. Neural Network
    
7. Backpropagation
    
8. Attention
    
9. Transformer
    
10. GPT 如何生成文字
    
11. LLM 如何训练
    
12. ChatGPT 类系统是怎样构成的
    

写完这 12 篇以后，整个网站实际上已经拥有一条：

```
Hardware
↓
Math
↓
Neural Network
↓
Transformer
↓
LLM
```

的完整主线。

然后再不断向里面补充细节。

例如 Transformer 页面以后可以不断长出新的知识节点：

```
Transformer
│
├── Attention
│   ├── Softmax
│   ├── Matrix Multiplication
│   └── FlashAttention
│
├── Embedding
│   ├── Word2Vec
│   └── Tokenizer
│
├── MLP
│   └── Neural Network
│
└── Training
    ├── Backpropagation
    ├── Adam
    └── GPU
```

最终逐渐形成完整的知识网络。

---

# 结语

如果让我用一句话总结这套系列的目标，我希望它最终能够回答这样一个问题：

> **如果一个完全不了解人工智能的人从第一篇开始读，他能不能一路理解到今天的大语言模型、VLM、Reasoning Model、Agent 以及 AI 推理系统？**

我希望答案是可以。

所以这套系列不会只讲算法，也不会只讲代码。

它会尝试把：

```
硬件
数学
算法
模型
系统
工程
论文
```

真正串成一条完整的线。

从最底层的：

> 一个数字在计算机里是怎么被计算的？

一路走到：

> 一个拥有数百亿甚至更多参数的模型，为什么能够理解文字、图片，并产生具有一定推理能力的回答？

这大概就是我想写这套 AI 扫盲系列最主要的原因。

---
感谢GPT老师对总览写作的大力支持
