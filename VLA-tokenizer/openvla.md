# OpenVLA: An Open-Source Vision-Language-Action Model

## 基本信息
- **论文**: OpenVLA: An Open-Source Vision-Language-Action Model
- **链接**: https://arxiv.org/abs/2406.09246
- **发表**: CoRL 2024
- **机构**: Stanford University, UC Berkeley, Toyota Research Institute (TRI), Google DeepMind, MIT

## Tokenizer 设计概述

OpenVLA 采用与 RT-2 类似的**逐维度均匀离散化（per-dimension uniform discretization）**策略，将每个动作维度的连续值离散化为 256 个等宽 bin，并为每个 bin 创建专属的新 token（共 256 个额外 token 加入词汇表）。与 RT-2 的区别在于，OpenVLA 显式扩展了词汇表并重新训练了 token embedding，而非复用原有语言 token，使动作 token 具有独立的语义空间。作为 7B 参数的开源 VLA，OpenVLA 在多任务泛化上超越了闭源的 RT-2-X（55B）。

## 技术细节

**动作离散化方法**：对于机器人末端执行器的每个控制维度（通常为 7 维：Δx, Δy, Δz, Δroll, Δpitch, Δyaw, gripper），OpenVLA 统计训练数据中该维度的取值分布，然后将其范围均匀划分为 256 个等宽区间（bins）。每个 bin 分配一个唯一的整数 ID（0–255），并对应词汇表中新增的一个专属 token（例如 `<ACT_0>`，`<ACT_1>`，…，`<ACT_255>`）。

**词汇表扩展**：相比 RT-2 复用低频 token 的做法，OpenVLA 正式扩展了 Llama 2 的词汇表，新增 256 个动作专用 token，并对这些 token 的 embedding 进行端到端训练。这样可以避免动作语义与原始语言 token 语义之间的混淆，同时使得动作 token 的 embedding 能够更专注地学习动作空间的几何结构。

**模型骨干**：OpenVLA 基于 Prismatic-7B VLM 构建，视觉编码器融合了 DINOv2 和 SigLIP 的预训练特征（双编码器融合），文本骨干为 Llama 2-7B。训练数据来自 Open X-Embodiment 数据集的 970k 条真实机器人演示，覆盖多种机器人平台和操作任务。

**推理与解码**：推理时，给定图像和语言指令，模型自回归地生成 7 个动作 token（每维度一个），再通过反离散化（de-binning，取 bin 中点值）还原为连续动作值。OpenVLA 支持通过 LoRA 等低秩适配方法在消费级 GPU 上高效微调。

## 与语言模型的整合方式

OpenVLA 将视觉 token（由 DINOv2 + SigLIP 双编码器生成）和语言 token（指令文本）拼接后输入 Llama 2 骨干，最后通过 token prediction head 预测动作 token。整个流程复用了标准 LLM 的 next-token prediction 损失，不需要单独的动作解码头。微调时支持全参数微调或 LoRA，后者可在单张 A100（40GB）上训练。OpenVLA 完全开源（模型权重、训练代码、数据处理流程），便于社区复现和二次开发。

## 优缺点分析

### 优点
- **开源生态**：模型权重、代码、训练流程完全开放，极大降低了 VLA 研究门槛
- **强泛化性**：在 29 个任务上以 7B 参数超越闭源的 RT-2-X（55B），数据效率更高
- **高效微调**：支持 LoRA，可在消费级 GPU 上微调，适配新任务、新机器人
- **双编码器视觉**：DINOv2 + SigLIP 融合特征比单编码器更鲁棒

### 缺点/局限
- **量化精度有限**：每维度 256 个 bin，无法精确表示高精度连续动作（如毫米级操作）
- **时序依赖缺失**：逐时间步预测，不建模动作序列的时序结构，不支持原生 action chunking
- **推理速度慢**：7B 参数的自回归解码延迟较高，难以支持高频控制（>10Hz）
- **维度独立假设**：各动作维度独立分箱，忽略维度间的相关性（如耦合的旋转运动）
- **分箱方案静态**：均匀分箱不考虑动作值的实际分布，对稀疏或偏态分布的动作维度效果欠佳

## 对比其他 Tokenizer

OpenVLA 是当前开源 VLA 领域最重要的 baseline 之一，其 Tokenizer 方案直接继承自 RT-2 并做了若干改进：

- **vs. RT-2**：OpenVLA 显式扩展词汇表（新增 256 token）而非复用现有 token，语义更干净；规模更小（7B vs 55B），更高效
- **vs. FASTer/FAST**：FAST 指出均匀分箱在高频机器人数据上性能欠佳，提出频域压缩；FASTer 进一步用神经网络 VQ 提升压缩比和时序建模能力
- **vs. π0**：π0 完全放弃离散化，采用 flow matching 直接建模连续动作分布，适合灵巧操作
- **vs. SpatialVLA**：SpatialVLA 引入 Ego3D Position Encoding 和自适应动作网格，在空间感知上超越 OpenVLA 的静态均匀分箱
- **vs. RoboVLMs**：RoboVLMs 系统性对比了离散 vs 连续动作表示，为 OpenVLA 的分箱方案提供了消融分析框架
