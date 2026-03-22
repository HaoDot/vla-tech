# RT-2: Vision-Language-Action Models Transfer Web Knowledge to Robotic Control

## 基本信息
- **论文**: RT-2: Vision-Language-Action Models Transfer Web Knowledge to Robotic Control
- **链接**: https://arxiv.org/abs/2307.15818
- **发表**: CoRL 2023（Oral）
- **机构**: Google DeepMind

## Tokenizer 设计概述

RT-2 是首批将大规模预训练视觉语言模型（VLM）直接用于机器人动作预测的工作之一，开创了 VLA 范式。其核心 Tokenizer 策略极为简洁：将每个动作维度的连续值均匀离散化为 256 个 bin，每个 bin 对应一个整数标签（0–255），再将这些整数直接"注入"到语言模型的现有词汇表中（复用最不常用的 token ID）。这样，机器人动作就被表示为一串普通的"语言 token"，模型无需任何结构改动即可同时处理语言和动作。

## 技术细节

**动作离散化方案**：RT-2 为机器人每个动作维度（如末端执行器的 x、y、z 位置，滚转、俯仰、偏航角，以及夹爪开合）分别执行独立的均匀分箱（uniform discretization）。每个维度的取值范围被等分为 256 个区间，动作值落入哪个区间就编码为对应的整数（0 到 255）。最终一个时间步的动作被表示为 7 个（或更多）独立的离散 token，按固定顺序拼接成 token 序列。

**词汇表复用**：RT-2 不引入任何新 token，而是直接将动作整数值映射到现有语言模型词汇表中出现频率最低的那些 token（如"\<256\>"等）。这一设计保留了预训练 LLM 的全部参数，使得 web-scale 预训练知识得以直接迁移到机器人控制任务中，且无需修改 tokenizer 结构。

**输入输出格式**：输入为图像帧（经过视觉编码器 PaLI/PaLM 处理）+ 自然语言指令；输出为动作 token 序列（例如"x_token y_token z_token roll_token pitch_token yaw_token gripper_token"）。整个流水线以 next-token prediction 方式自回归生成动作 token，与标准语言建模完全一致。

**涌现能力**：RT-2 的实验表明，通过将动作表示为语言 token，模型能够"涌现"出零样本泛化能力，例如理解语义概念（将"垃圾桶旁边的饮料"与可能的危险关联）并做出正确操作决策。这说明语言模型中隐含的常识知识可以通过动作 token 的方式被有效激活。

## 与语言模型的整合方式

RT-2 直接在 PaLI-X（55B）和 PaLM-E（12B）两个大型 VLM 上进行联合微调，微调数据混合了 web 数据（图文对）和机器人演示数据（图像+动作 token）。模型在推理时以标准自回归方式一次生成一个动作 token，直到生成完整的动作序列。由于动作 token 已被融入语言词汇表，整个训练和推理流程对 LLM 框架完全透明，无需任何额外的动作解码头或特殊损失函数。

## 优缺点分析

### 优点
- **架构极简**：动作直接复用语言 token，无需任何额外模块，实现了真正意义上的统一建模
- **知识迁移强**：大规模 web 预训练知识可直接迁移，赋予模型零样本泛化和语义推理能力
- **涌现能力**：展现出超出训练数据分布的泛化行为（emergent capabilities）
- **开创性**：定义了 VLA 的基本范式，影响了后续所有 VLA 工作

### 缺点/局限
- **精度受限**：每维度仅 256 个 bin，对需要高精度连续控制的任务（如灵巧操作）存在量化误差
- **时序独立性**：每个时间步的动作 token 独立预测，不能隐式建模动作序列的时间依赖性（无 action chunking）
- **推理较慢**：大模型（55B）推理延迟较高，难以实现高频控制
- **模型规模大**：55B 参数模型训练和部署成本极高
- **词汇表污染**：复用原有 token 可能对语言理解产生轻微干扰（尽管实验中影响有限）

## 对比其他 Tokenizer

RT-2 是 VLA 领域动作 Tokenizer 的"开山之作"，奠定了"将动作离散化为语言 token"的基础范式：

- **vs. OpenVLA**：OpenVLA 沿用了 RT-2 的均匀分箱思路，但进行了开源化并增加了 256 个新 token（而非复用现有 token），同时支持 action chunking
- **vs. FASTer/FAST**：FAST 和 FASTer 认为逐维度独立分箱忽略了动作的时序结构，提出了更高效的频域或神经网络压缩方案
- **vs. π0**：π0 完全抛弃了离散化 Tokenizer，改用连续流匹配（flow matching）来处理动作的连续性和多模态分布
- **vs. SpatialVLA**：SpatialVLA 在分箱基础上引入了空间感知的自适应网格（Adaptive Action Grids），增强了空间理解能力

RT-2 的均匀分箱方案因其简洁性而影响深远，但其局限性也直接推动了后续更精细 Tokenizer 方案的发展。
