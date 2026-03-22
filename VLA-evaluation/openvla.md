# OpenVLA: An Open-Source Vision-Language-Action Model

## 基本信息
- **论文**: OpenVLA: An Open-Source Vision-Language-Action Model
- **链接**: https://arxiv.org/abs/2406.09246
- **发表**: CoRL 2024（Conference on Robot Learning）
- **机构**: Stanford University、UC Berkeley、Toyota Research Institute 等，作者：Moo Jin Kim, Karl Pertsch, Siddharth Karamcheti, Ted Xiao, Ashwin Balakrishna, Suraj Nair, Rafael Rafailov, Chelsea Finn, Sergey Levine 等

## 核心贡献
OpenVLA 提出了一个 **7B 参数的开源 Vision-Language-Action（VLA）模型**，在大规模机器人演示数据（Open X-Embodiment 数据集的 970k+ 真实机器人轨迹）上预训练，旨在填补 VLA 研究中**开源缺失**的空白。其评测方法论是 VLA 领域的重要参考：系统性地评估了在 BridgeData V2（真实机器人）和 WidowX 操控任务上的性能，并对比了 RT-2-X 等闭源大型 VLA，验证了开源模型在真实机器人任务中的竞争力。同时探索了 LoRA 等参数高效微调方法用于新任务适配。

## 方法/测评设计详解

**模型架构**：OpenVLA 以 Prismatic VLM（基于 LLaVa-1.5 框架，使用 DINOv2 + SigLIP 双视觉编码器与 Llama-2 7B 语言模型）作为骨干，将机器人动作 token 化为离散词汇表 token（256 bins per action dimension），通过自回归语言建模方式预测动作序列，无需改变语言模型原有架构。

**训练数据与预训练策略**：OpenVLA 在 Open X-Embodiment 数据集的筛选子集上训练，包含来自 22 种机器人形态、21 家机构的约 970,000 条真实机器人轨迹。训练数据经过去重和质量过滤，最终选取 BridgeData V2 和来自谷歌 robot 的数据作为主要训练集。使用 16 块 A100 GPU，训练时长约 2 周。

**评测方法论——真实机器人评估**：OpenVLA 采用严格的**真实机器人评估协议**，在 WidowX 机器人上执行 BridgeData V2 的测试集任务，涵盖：
- **分布内任务（In-distribution）**：训练时见过的任务场景和物体
- **分布外任务（Out-of-distribution）**：新颜色、新物体、新背景等视觉泛化测试
- **指令泛化**：对未见语言表述的鲁棒性
评测时固定评估轮次（通常每个任务 50 次 trials），计算 Success Rate。

**微调评估**：系统性测试了多种微调方法在下游任务上的效率与效果：
- **Full fine-tuning**：完整参数微调
- **LoRA**：低秩适配，大幅减少微调参数量（仅需 ~0.5% 参数）
- **Sandwich LoRA**：在视觉和语言编码器上均施加 LoRA
- **Frozen backbone + MLP head**：冻结预训练参数，只训练动作头

**对比基线**：在相同任务和协议下，对比 RT-2-X（540B 参数，闭源）、Octo（93M，开源）、Diffusion Policy、BC-Z 等基线，验证 OpenVLA 的竞争力。

## 主要任务与场景

**BridgeData V2 真实机器人任务（WidowX 机械臂）**：
- Put eggplant into yellow bowl（将茄子放入黄色碗）
- Put carrot on plate（将胡萝卜放到盘子上）
- Stack blocks（叠放积木）
- Open/close microwave（开关微波炉）
- Pour water（倒水）
- Pick up toy（拾取玩具）

**泛化测试维度**：
- 物体颜色/形状变化
- 背景桌面纹理变化
- 容器位置变化
- 新颖物体（unseen objects）

**核心指标**：
- Task Success Rate（任务成功率 per task）
- Overall Success Rate（所有任务的平均成功率）
- Few-shot fine-tuning efficiency（微调样本效率曲线）

## 优缺点分析

### 优点
- **完全开源**：模型权重、训练代码、评测代码全部开源，推动 VLA 研究民主化
- **强大的预训练基础**：在 970k+ 真实机器人轨迹上训练，跨越多机器人形态，泛化能力强
- **参数高效微调**：LoRA 微调方案使新任务适配成本极低（仅需数百条 demo、单块 GPU）
- **与闭源大模型竞争**：7B 参数模型在多个真实任务上接近甚至超过 RT-2-X（540B）
- **评测方法严谨**：详细报告每个任务的成功率、标准差，具有高可复现性

### 局限性
- **动作离散化损失精度**：将连续动作 token 化为 256 bins 会引入量化误差，对精细操控任务有影响
- **推理速度较慢**：7B LLM 自回归推理频率约 6Hz，低于许多实时控制需求（通常需要 10-50Hz）
- **训练成本仍高**：尽管微调成本低，但从头预训练需要大量 GPU 资源
- **仿真评估不足**：主要依赖真实机器人评估，缺乏仿真基准的系统对比
- **单视角图像**：依赖单个固定相机视角，对遮挡和视角变化鲁棒性有限

## 被引用情况 / 影响力

OpenVLA 是 VLA 领域**最具影响力的开源模型之一**，发表后迅速成为新 VLA 研究的基准对比对象。其开放性推动了大量后续工作，包括 OpenVLA-OFT（改进的微调版本）、π0、GR-2 等。其评测协议（BridgeData V2 + WidowX 真实机器人任务）也成为评估 VLA 真实机器人性能的常见参照。截至 2025 年，GitHub 仓库 (openvla/openvla) star 数超过 1600，论文引用量超过 400 次，是 VLA 开源生态的基础性工作。
