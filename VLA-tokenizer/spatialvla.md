# SpatialVLA: Exploring Spatial Representations for Visual-Language-Action Model

## 基本信息
- **论文**: SpatialVLA: Exploring Spatial Representations for Visual-Language-Action Model
- **链接**: https://arxiv.org/abs/2501.15830
- **发表**: Robotics: Science and Systems (RSS) 2025
- **机构**: 上海人工智能实验室、上海交通大学、中国科学技术大学等

## Tokenizer 设计概述

SpatialVLA 在标准 VLA 的动作离散化基础上提出了两项关键创新：**Ego3D Position Encoding**（将深度/3D 空间信息注入视觉观测）和 **Adaptive Action Grids**（自适应离散化动作网格）。Adaptive Action Grids 摒弃了均匀分箱的假设，根据机器人的实际动作分布和空间结构，以自适应方式划分动作空间，使得 Tokenizer 既能精确表示高频动作区域，又能有效覆盖稀疏区域，从而更好地泛化至不同机器人平台和任务场景。

## 技术细节

**Ego3D Position Encoding**：SpatialVLA 的视觉输入不仅包含 RGB 图像，还通过 Ego3D Position Encoding 将每个图像 patch 的三维空间坐标（基于深度估计或相机模型）编码为位置特征，与视觉 token 拼接后输入模型。这使得 VLA 能够感知物体的三维位置和机器人末端执行器在空间中的绝对/相对位置，解决了传统 2D 视觉 token 缺乏深度感知的问题。

**Adaptive Action Grids（自适应动作网格）**：在动作 Tokenizer 设计上，SpatialVLA 提出用"动作网格（Action Grid）"替代独立的逐维度均匀分箱。具体而言：
- 在预训练阶段，通过对训练数据的动作分布进行聚类分析，学习出一组覆盖主要动作模式的自适应离散化网格节点（grid points）
- 每个时间步的动作被映射到最近的网格节点，并以该节点的 ID 作为动作 token
- 在微调阶段，预学习的动作网格可以**再离散化（re-discretize）**，以适配新机器人平台的动作分布，实现跨机器人迁移

**预训练规模**：SpatialVLA 在 1.1 Million 条真实世界机器人演示上预训练（来自 Open X-Embodiment 等数据集），覆盖多种机器人环境和任务，学习通用的空间动作知识。

**零样本泛化与微调**：预训练后，SpatialVLA 可以 zero-shot 执行大量任务。通过 re-discretize 动作网格，SpatialVLA 还可以快速微调适配新场景，实验表明其 in-distribution 泛化和 out-of-distribution 适应性均优于 OpenVLA 等基线。

## 与语言模型的整合方式

SpatialVLA 基于预训练 VLM（论文中使用 InternVL 作为骨干），将 Ego3D 增强的视觉 token 和语言指令 token 一起输入 Transformer。动作 token（自适应网格 ID）以标准的 next-token prediction 方式由模型自回归生成，与 RT-2/OpenVLA 的集成方式类似，但 token 的含义由均匀 bin 变为空间自适应的网格节点。这种改变不需要修改 LLM 骨干的结构，仅需替换动作 token 的 embedding 层和动作 tokenizer/detokenizer 模块。

## 优缺点分析

### 优点
- **空间感知增强**：Ego3D Position Encoding 赋予模型真正的三维空间理解能力，显著提升涉及精确空间定位的操作任务性能
- **自适应分箱**：Adaptive Action Grids 比均匀分箱更精确地覆盖实际动作分布，减少量化误差
- **跨机器人迁移**：动作网格的 re-discretize 机制使得跨平台微调更高效，适配新机器人只需重新学习网格
- **RSS 2025 认可**：经过顶级机器人学会议评审，技术可靠
- **代码开源**：完整代码和模型权重已公开

### 缺点/局限
- **依赖深度信息**：Ego3D 需要深度图或可靠的深度估计，在无深度传感器的场景（如单目相机）中效果可能受限
- **动作网格需预学习**：与均匀分箱相比，自适应网格需要额外的聚类分析步骤，增加了预处理复杂度
- **时序建模有限**：仍是逐步动作预测，与 FASTer/π0 的 action chunking 相比时序建模能力较弱
- **需要训练数据的动作分布**：自适应网格依赖数据统计，对新奇分布的动作可能需要重新调整

## 对比其他 Tokenizer

SpatialVLA 的创新点在于将"**空间感知**"引入动作 Tokenizer：

| 方法 | 动作离散化 | 空间感知 | 跨机器人迁移 |
|---|---|---|---|
| RT-2 | 均匀分箱，逐维度 | 无（2D 视觉）| 差 |
| OpenVLA | 均匀分箱，扩展词表 | 无（2D 视觉）| 中 |
| **SpatialVLA** | **自适应动作网格** | **Ego3D 位置编码** | **强（re-discretize）** |
| FASTer | 神经网络 VQ，极高压缩 | 无 | 强（跨具身测试）|
| π0 | 无离散化（连续 flow matching）| 无（2D 视觉）| 强（大规模数据）|

SpatialVLA 是目前将三维空间信息与动作 Tokenizer 设计最紧密结合的工作，其 Adaptive Action Grids 方法对于需要精确空间操作的任务（如物体抓取、放置）具有明显优势。与 FASTer 的神经网络压缩和 π0 的连续方法相比，SpatialVLA 保持了离散 token 的可解释性，同时通过空间感知提升了动作表示质量。
