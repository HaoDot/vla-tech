# RoboVLMs: What Matters in Building Vision-Language-Action Models for Generalist Robot Policies

## 基本信息
- **论文**: Towards Generalist Robot Policies: What Matters in Building Vision-Language-Action Models
- **链接**: https://arxiv.org/abs/2412.14058
- **发表**: Nature Machine Intelligence, 2025年2月（arXiv 2024年12月）
- **机构**: 清华大学、字节跳动、上海AI实验室、上海交通大学等

## Tokenizer 设计概述

RoboVLMs 不是单一模型，而是一个**系统性消融研究框架**，通过控制变量实验深入探讨了构建 VLA 的各项关键设计选择，包括骨干网络选择、动作表示（离散 vs 连续）、输入历史观测、跨机器人数据使用等。在 Tokenizer 方面，RoboVLMs 对比了离散化动作 token（分箱方案）与连续动作回归（直接输出连续值）两种范式，并系统分析了不同设计在多个 benchmark 上的表现，为 VLA 社区提供了迄今最全面的设计指南。

## 技术细节

**四大核心问题**：RoboVLMs 围绕以下四个问题展开：
1. **Why**：VLA 相比其他通用机器人策略（如 Diffusion Policy）有何优势？
2. **Which**：选择哪种 VLM 骨干（探索了 8 种不同骨干，包括 PaLI、InstructBLIP、KosMos 等）？
3. **How**：如何制定动作预测的形式（离散 vs 连续，history observable vs not）？
4. **When**：何时加入跨机器人（cross-embodiment）数据有益？

**动作表示对比**：在 Tokenizer 设计上，RoboVLMs 系统对比了：
- **离散化（Discrete）**：将每维度动作均匀分箱为 256 bins，复用或扩展语言 token，与 RT-2/OpenVLA 方案一致
- **连续输出（Continuous）**：在 LLM 之上添加一个连续回归头（MLP），直接预测 MSE 损失下的连续动作值
- **混合方案**：部分维度离散、部分维度连续

**关键发现（Findings）**：
- VLA（基于预训练 VLM 微调）在鲁棒性、泛化性和数据效率上均优于非 VLA 的通用策略
- 骨干网络的选择对性能影响显著：更强的视觉-语言预训练（如 KosMos-2）通常带来更好的操作性能
- 离散化动作 token 与连续回归在不同任务上各有优势：离散化在多任务泛化上较强，连续回归在精度上更优
- 使用历史观测帧（多帧输入）通常比单帧更好
- 跨机器人数据的加入时机和方式影响显著，需根据任务分布选择

**评测 Benchmark**：CALVIN（仿真，语言条件多任务）、SimplerEnv（仿真，RT-2 任务复现）、ByteDance Robot Benchmark（真实世界操作）。

## 与语言模型的整合方式

RoboVLMs 探索了多种 VLM 骨干（PaLI、InstructBLIP、KosMos-2 等）与动作预测头的组合方式。在离散化方案中，动作 token 被追加到 LLM 的 output token 序列，沿用 next-token prediction；在连续方案中，则在 LLM 最后一层 hidden state 之上添加 MLP 回归头，以 MSE 损失训练。研究发现，强 VLM 骨干（尤其是具有良好空间推理能力的）对机器人操作任务更为关键，而动作预测形式（离散/连续）的影响因任务而异。

## 优缺点分析

### 优点
- **系统性消融**：迄今最全面的 VLA 设计空间探索，为社区提供了可复现的设计指南
- **多骨干对比**：同时评测 8 种 VLM 骨干，澄清了"哪种预训练更重要"的问题
- **开源**：代码、模型权重、实验结果均已公开（github.com/Robot-VLAs/RoboVLMs）
- **发表在 Nature MI**：经过严格同行评审，结论可信度高

### 缺点/局限
- **非端到端最优模型**：作为消融研究框架，各配置并非为最优性能设计，单模型性能不如专门优化的 VLA（如 π0）
- **计算资源需求大**：系统性消融需要大量实验，复现成本高
- **任务分布局限**：主要在 CALVIN 和 SimplerEnv 上验证，对精细操作任务（如灵巧手）的覆盖有限
- **动作 Tokenizer 创新不足**：在 Tokenizer 设计上主要是对比分析，并未提出新颖方案

## 对比其他 Tokenizer

RoboVLMs 的价值在于为 Tokenizer 设计提供了**客观的基准比较**：

- **离散化 token（RT-2/OpenVLA 方案）vs 连续回归**：结论是任务依赖，无绝对优劣；离散化在泛化任务上有优势，连续回归在精度任务上更好
- **对 FASTer/FAST 的启示**：RoboVLMs 的分析揭示了简单均匀分箱的局限，间接支持了 FAST 等更高效 Tokenizer 的动机
- **对 π0 的补充**：RoboVLMs 的消融表明，仅仅改变 Tokenizer 不够，VLM 骨干的质量同样至关重要
- **对 SpatialVLA 的背景支持**：RoboVLMs 揭示空间理解能力的重要性，为 SpatialVLA 的空间感知设计提供了实证依据
