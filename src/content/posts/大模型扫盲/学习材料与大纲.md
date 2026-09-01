---
date: 2026-08-29
title: List for AI learnings
published: 2026-08-29
pinned: false
description: This page would show a bunch of lessons and books for AI learnings
image: ""
tags:
  - AI
  - "#参考书"
  - "#好课推荐"
category: 大模型扫盲
draft: false
author: 山吹
comment: true
---

# 我的 AI 学习材料路线

| 主线章节              | 主要学习材料                                             | 补充材料                                                                       |
| ----------------- | -------------------------------------------------- | -------------------------------------------------------------------------- |
| AI 硬件 / GPU 基础    | **NVIDIA CUDA C++ Programming Guide**              | NVIDIA Deep Learning Performance Guide、NVIDIA GPU Architecture Whitepapers |
| 数学基础              | **Mathematics for Machine Learning**               | 3Blue1Brown：Essence of Linear Algebra / Calculus                           |
| 机器学习              | **Stanford CS229**                                 | _Pattern Recognition and Machine Learning_、StatQuest                       |
| 深度学习              | **MIT 6.S191**                                     | _Deep Learning_、Dive into Deep Learning                                    |
| NLP / Transformer | **Stanford CS224N**                                | _Attention Is All You Need_、_Speech and Language Processing_               |
| LLM / 语言模型训练      | **Stanford CS336: Language Modeling from Scratch** | nanoGPT、Hugging Face LLM Course                                            |
| AI Systems / 推理优化 | **Stanford CS336 Systems 部分**                      | FlashAttention、vLLM / PagedAttention、Triton Documentation                  |
| 视觉 / 多模态          | **Stanford CS231n**                                | ViT、CLIP、DDPM 等原论文                                                         |
| 强化学习 / 后训练        | **Berkeley CS285**                                 | PPO、RLHF、DPO 等原论文                                                          |
| AI 前沿             | **论文原文 + arXiv**                                   | OpenReview、各实验室 Technical Report / Blog                                    |
# 链接整理

| 主线章节              | 学习材料                                     | 链接                                                                                                                                         | 推荐度    | 备注                                 |
| ----------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ------ | ---------------------------------- |
| AI 硬件 / GPU 基础    | NVIDIA CUDA C++ Programming Guide        | [https://docs.nvidia.com/cuda/cuda-c-programming-guide/](https://docs.nvidia.com/cuda/cuda-c-programming-guide/)                           | ★★★★★  | 最贴近 AI GPU、线程模型、内存层级、Kernel        |
| AI 硬件 / GPU 基础    | NVIDIA Deep Learning Performance Guide   | [https://docs.nvidia.com/deeplearning/performance/](https://docs.nvidia.com/deeplearning/performance/)                                     | ★★★★★  | 很适合补 AI 性能、Tensor Core、带宽和吞吐       |
| AI 硬件 / GPU 基础    | NVIDIA GPU Architecture Whitepapers      | [https://www.nvidia.com/en-us/data-center/resources/](https://www.nvidia.com/en-us/data-center/resources/)                                 | ★★★★☆  | 想深入 Hopper、Blackwell 等架构时再看        |
| 数学基础              | Mathematics for Machine Learning         | [https://mml-book.github.io/](https://mml-book.github.io/)                                                                                 | ★★★★★  | 最适合作为主数学教材                         |
| 数学基础              | 3Blue1Brown – Essence of Linear Algebra  | [https://www.3blue1brown.com/topics/linear-algebra](https://www.3blue1brown.com/topics/linear-algebra)                                     | ★★★★★  | 线性代数直觉非常强                          |
| 数学基础              | 3Blue1Brown – Essence of Calculus        | [https://www.3blue1brown.com/topics/calculus](https://www.3blue1brown.com/topics/calculus)                                                 | ★★★★★  | 微积分、梯度、链式法则直觉很好                    |
| 数学基础              | StatQuest                                | [https://www.youtube.com/@statquest](https://www.youtube.com/@statquest)                                                                   | ★★★★☆  | 概率、统计、ML 查漏补缺很好用                   |
| 机器学习              | Stanford CS229                           | [https://cs229.stanford.edu/](https://cs229.stanford.edu/)                                                                                 | ★★★★★  | 机器学习主线首选                           |
| 机器学习              | Pattern Recognition and Machine Learning | [https://www.microsoft.com/en-us/research/people/cmbishop/prml-book/](https://www.microsoft.com/en-us/research/people/cmbishop/prml-book/) | ★★★★☆  | 偏理论，不建议一开始全啃                       |
| 机器学习              | Probabilistic Machine Learning           | [https://probml.github.io/pml-book/](https://probml.github.io/pml-book/)                                                                   | ★★★★☆  | 概率机器学习体系非常完整                       |
| 深度学习              | MIT 6.S191                               | [https://introtodeeplearning.com/](https://introtodeeplearning.com/)                                                                       | ★★★★★  | 节奏快，适合深度学习入门主线                     |
| 深度学习              | Dive into Deep Learning                  | [https://d2l.ai/](https://d2l.ai/)                                                                                                         | ★★★★★  | 理论和代码结合非常好                         |
| 深度学习              | Deep Learning – Goodfellow               | [https://www.deeplearningbook.org/](https://www.deeplearningbook.org/)                                                                     | ★★★★☆  | 经典教材，适合查理论                         |
| NLP / Transformer | Stanford CS224N                          | [https://web.stanford.edu/class/cs224n/](https://web.stanford.edu/class/cs224n/)                                                           | ★★★★★  | NLP → Attention → Transformer 主线很好 |
| NLP / Transformer | Attention Is All You Need                | [https://arxiv.org/abs/1706.03762](https://arxiv.org/abs/1706.03762)                                                                       | ★★★★★  | Transformer 必读                     |
| NLP / Transformer | Speech and Language Processing           | [https://web.stanford.edu/~jurafsky/slp3/](https://web.stanford.edu/~jurafsky/slp3/)                                                       | ★★★★★  | NLP 理论参考非常强                        |
| LLM / 语言模型训练      | Stanford CS336                           | [https://cs336.stanford.edu/](https://cs336.stanford.edu/)                                                                                 | ★★★★★+ | 整套路线里最值得认真啃的课之一                    |
| LLM / 语言模型训练      | Hugging Face LLM Course                  | [https://huggingface.co/learn/llm-course/](https://huggingface.co/learn/llm-course/)                                                       | ★★★★☆  | 工程实践和生态很好                          |
| LLM / 语言模型训练      | nanoGPT                                  | [https://github.com/karpathy/nanoGPT](https://github.com/karpathy/nanoGPT)                                                                 | ★★★★★  | 很适合真正搞懂 GPT 训练代码                   |
| LLM / 语言模型训练      | minGPT                                   | [https://github.com/karpathy/minGPT](https://github.com/karpathy/minGPT)                                                                   | ★★★★☆  | 比 nanoGPT 更偏教学                     |
| AI Systems / 推理优化 | Stanford CS336 Systems 部分                | [https://cs336.stanford.edu/](https://cs336.stanford.edu/)                                                                                 | ★★★★★  | GPU、FLOPs、Memory、Parallelism 都覆盖   |
| AI Systems / 推理优化 | FlashAttention                           | [https://arxiv.org/abs/2205.14135](https://arxiv.org/abs/2205.14135)                                                                       | ★★★★★  | Attention 系统优化必读                   |
| AI Systems / 推理优化 | vLLM / PagedAttention                    | [https://arxiv.org/abs/2309.06180](https://arxiv.org/abs/2309.06180)                                                                       | ★★★★★  | LLM Serving 核心论文                   |
| AI Systems / 推理优化 | Triton Documentation                     | [https://triton-lang.org/](https://triton-lang.org/)                                                                                       | ★★★★★  | 学 AI Kernel 很合适                    |
| AI Systems / 推理优化 | CUDA Best Practices Guide                | [https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/)                     | ★★★★☆  | CUDA 性能优化的重要补充                     |
| 视觉 / 多模态          | Stanford CS231n                          | [https://cs231n.stanford.edu/](https://cs231n.stanford.edu/)                                                                               | ★★★★★  | CV、ViT、多模态视觉基础主线                   |
| 视觉 / 多模态          | Vision Transformer                       | [https://arxiv.org/abs/2010.11929](https://arxiv.org/abs/2010.11929)                                                                       | ★★★★★  | ViT 必读                             |
| 视觉 / 多模态          | CLIP                                     | [https://arxiv.org/abs/2103.00020](https://arxiv.org/abs/2103.00020)                                                                       | ★★★★★  | 理解现代 VLM 的关键论文                     |
| 视觉 / 多模态          | DDPM                                     | [https://arxiv.org/abs/2006.11239](https://arxiv.org/abs/2006.11239)                                                                       | ★★★★★  | Diffusion 基础论文                     |
| 强化学习 / 后训练        | Berkeley CS285                           | [https://rail.eecs.berkeley.edu/deeprlcourse/](https://rail.eecs.berkeley.edu/deeprlcourse/)                                               | ★★★★★  | RL 主线首选                            |
| 强化学习 / 后训练        | PPO                                      | [https://arxiv.org/abs/1707.06347](https://arxiv.org/abs/1707.06347)                                                                       | ★★★★★  | RLHF / Reasoning RL 基础             |
| 强化学习 / 后训练        | InstructGPT                              | [https://arxiv.org/abs/2203.02155](https://arxiv.org/abs/2203.02155)                                                                       | ★★★★★  | 理解 RLHF 很重要                        |
| 强化学习 / 后训练        | DPO                                      | [https://arxiv.org/abs/2305.18290](https://arxiv.org/abs/2305.18290)                                                                       | ★★★★★  | 偏好优化非常值得读                          |
| 参数高效微调            | LoRA                                     | [https://arxiv.org/abs/2106.09685](https://arxiv.org/abs/2106.09685)                                                                       | ★★★★★  | PEFT 基础                            |
| 参数高效微调            | QLoRA                                    | [https://arxiv.org/abs/2305.14314](https://arxiv.org/abs/2305.14314)                                                                       | ★★★★★  | 量化微调经典                             |
| Scaling           | Scaling Laws for Neural Language Models  | [https://arxiv.org/abs/2001.08361](https://arxiv.org/abs/2001.08361)                                                                       | ★★★★★  | 理解“模型为什么越做越大”                      |
| Scaling           | Chinchilla                               | [https://arxiv.org/abs/2203.15556](https://arxiv.org/abs/2203.15556)                                                                       | ★★★★★  | 数据量与模型规模关系必读                       |
| 新架构               | Mamba                                    | [https://arxiv.org/abs/2312.00752](https://arxiv.org/abs/2312.00752)                                                                       | ★★★★☆  | 理解 Transformer 替代路线                |
| AI 前沿             | arXiv                                    | [https://arxiv.org/](https://arxiv.org/)                                                                                                   | ★★★★★  | 最新论文主入口                            |
| AI 前沿             | OpenReview                               | [https://openreview.net/](https://openreview.net/)                                                                                         | ★★★★★  | ICLR、NeurIPS 等论文和评审很好用             |
| AI 前沿             | Papers with Code                         | [https://paperswithcode.com/](https://paperswithcode.com/)                                                                                 | ★★★★☆  | 找论文、代码、Benchmark 很方便               |