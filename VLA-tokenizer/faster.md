# FASTer: Toward Efficient Autoregressive Vision Language Action Modeling via Neural Action Tokenization

## 基本信息
- **论文**: FASTer: Toward Efficient Autoregressive Vision Language Action Modeling via Neural Action Tokenization
- **链接**: https://arxiv.org/abs/2512.04952 | https://openreview.net/forum?id=k6nTUFoqeT
- **发表**: ICLR 2026（OpenReview k6nTUFoqeT），2025年12月
- **机构**: 复旦大学、上海人工智能实验室等

## Tokenizer 设计概述

FASTer 提出了一种基于神经网络的动作 Tokenizer（FASTerVQ），将连续动作序列（action chunk）编码为单通道图像，通过 Vector Quantization（VQ）压缩为少量离散 token。相比传统的逐维度均匀分箱（per-dimension uniform binning），FASTerVQ 能够捕获动作序列的全局时空依赖性，大幅提高重建保真度的同时实现更高的压缩比。整体框架 FASTerVLA 将该 Tokenizer 与自回归 VLA 策略无缝集成，通过 block-wise autoregressive decoding 和 lightweight action expert 进一步加速推理。

## 技术细节

**动作编码方式**：FASTerVQ 将一个 action chunk（多步动作序列）视为单通道"图像"，利用卷积编码器对其进行特征提取，捕捉时间步之间和各动作维度之间的联合依赖关系。这一思路与传统方法将每个维度独立分箱的做法根本不同，能够保留动作序列的结构信息。

**Vector Quantization 压缩**：编码后的特征通过 VQ codebook 量化为离散 token，整个 action chunk 被压缩成极少量（通常为 1 个或若干个）离散 token。这种极高的压缩比使得推理时所需的自回归解码步数大幅减少，从而显著提升推理速度。

**FASTerVLA 的自回归策略**：在 FASTerVQ 的基础上，FASTerVLA 采用 block-wise autoregressive decoding 方式，将语言指令、视觉观测与压缩后的动作 token 统一进行自回归建模。同时引入轻量级 action expert（adapter），专门负责解码动作 token，降低对主干 LLM 的计算负担。

**跨任务/跨机器人泛化**：实验显示，FASTerVQ 在 cross-task 和 cross-embodiment 场景下均展现出强泛化性，token 利用率高，重建质量优异。FASTerVLA 在多个仿真和真实世界 benchmark 上超越了此前 state-of-the-art VLA 模型，在推理速度和任务成功率两方面同时取得提升。

## 与语言模型的整合方式

FASTerVLA 以预训练的视觉语言模型（VLM/LLM）为骨干，将视觉 token、语言指令 token 与 FASTerVQ 压缩生成的动作 token 一同输入自回归 Transformer 进行联合建模。动作 token 的词汇表来自 VQ codebook，被视为与语言 token 同等地位的离散符号。推理时，模型自回归地生成动作 token，再通过 FASTerVQ 的解码器还原为连续动作序列，实现端到端的感知-决策闭环。

## 优缺点分析

### 优点
- **极高压缩比**：将整个 action chunk 压缩为极少量 token，大幅减少自回归解码步数，推理速度快
- **全局时空建模**：编码器能够捕获动作序列跨时间步和跨维度的联合依赖，重建质量优于逐维度分箱方法
- **跨机器人泛化**：在 cross-embodiment 场景表现出色，token 利用率高
- **端到端可学习**：Tokenizer 为神经网络，可与策略网络联合训练，避免手工设计分箱的局限性

### 缺点/局限
- **Tokenizer 需要额外训练**：VQ Tokenizer 的训练引入额外开销，需要足够多样的动作数据才能学到通用 codebook
- **VQ 量化误差**：Vector Quantization 存在信息损失，对精度要求极高的任务（如微米级操作）可能带来性能瓶颈
- **模型复杂度提升**：与简单分箱方法相比，架构更复杂，调试和部署门槛更高
- **仅在特定 benchmark 验证**：真实世界场景下的大规模验证尚不充分

## 对比其他 Tokenizer

| Tokenizer 方法 | 方式 | 压缩比 | 时序依赖 |
|---|---|---|---|
| RT-2 / OpenVLA 均匀分箱 | 逐维度独立分箱为离散整数 | 低（每步每维1 token）| 无 |
| FAST（DCT 频域压缩） | 频域变换后 BPE 编码 | 中（约16 token/chunk）| 弱 |
| **FASTer (FASTerVQ)** | 神经网络 VQ 编码 | 高（1~少量 token/chunk）| 强 |
| π0 Flow Matching | 不离散化，连续流匹配 | N/A | 强 |

FASTer 相比 FAST（DCT 方案）进一步提高了压缩比，且 Tokenizer 为端到端可学习的神经网络，重建质量更高。相比 RT-2/OpenVLA 的简单分箱，FASTer 显著降低了解码步数，同时保留了动作序列的时空结构信息。
