# Open X-Embodiment: Robotic Learning Datasets and RT-X Models

## 基本信息
- **论文**: Open X-Embodiment: Robotic Learning Datasets and RT-X Models
- **链接**: https://arxiv.org/abs/2310.08864
- **发表**: ICRA 2024（Best Paper Award Finalist）
- **机构**: Open X-Embodiment Collaboration — 来自 Google DeepMind、Stanford、Berkeley、CMU、UMich 等 21 家机构的大规模协作

## 核心贡献
Open X-Embodiment（OXE）构建了机器人学习领域**规模最大的开源真实机器人数据集**，包含来自 22 种机器人形态（embodiments）、21 家机构的超过 **100 万条真实机器人操控轨迹**，覆盖 500+ 种操控技能。在此基础上，论文训练了 RT-X 模型系列（RT-1-X 和 RT-2-X），验证了**跨形态预训练可以提升目标机器人的任务性能**，为机器人基础模型（foundation model for robotics）的研究奠定了数据基础。

## 方法/测评设计详解

**数据集构建**：OXE 数据集是多机构协作的产物，各机构贡献各自机器人平台（Franka、Kuka、UR5、WidowX、Google Robot、Spot 等）的操控数据。为实现跨数据集的统一，团队设计了 **Open Embodiment Robotics Utilities（RLDS）** 标准化数据格式，将所有机器人轨迹统一表示为 `(observation, action, reward, discount, step_metadata)` 序列，解决了不同机器人形态间动作空间、传感器配置差异的问题。

**跨形态学习假设验证**：论文的核心研究问题是：在一种机器人上收集的数据是否能帮助另一种机器人学习？为此，团队采用受控实验设计，在相同的 RT-1 和 RT-2 架构下，对比：
- 仅用目标机器人数据训练（single-embodiment）
- 加入 OXE 全量数据训练（cross-embodiment，得到 RT-1-X 和 RT-2-X）
通过在真实机器人上的系统性评估，验证跨形态预训练对下游任务的正向迁移效果。

**真实机器人评估协议**：OXE 采用严格的**真实世界评估**（无仿真代理），在多个机器人平台上执行标准化操控任务：
- 评估机器人包括：Google Robot（手持夹爪）、WidowX、Franka Panda
- 评估场景：桌面拾取放置、开关抽屉/微波炉、叠放物体、倒水等
- 每个任务评估固定轮次（通常 25-50 次），报告 success rate
- 对比条件：不同数据混合策略、不同模型规模、有无跨形态预训练

**数据混合策略研究**：OXE 系统研究了如何最优地混合来自不同机器人的数据，发现加权采样（根据机器人形态和任务多样性）优于简单的均匀混合，为后续多源数据训练提供了实践指导。

## 主要任务与场景

**覆盖的机器人形态（22种，部分示例）**：
- Google Robot（GR1 手持夹爪）
- Franka Panda（多种变体）
- WidowX（BridgeData V2）
- Kuka LBR iiwa
- Sawyer
- UR5（多种变体）
- Boston Dynamics Spot
- 双臂系统

**代表性任务类别（500+ 技能）**：
- Pick & Place（拾取放置各类物体）
- Articulated Object Manipulation（关节物体操控：开关抽屉、门、烤箱）
- Tool Use（工具使用：扫地、倒水）
- Object Sorting（物体分类整理）
- Stack/Unstack（叠放/拆分物体）
- Wiping（擦拭清洁）

**RT-X 评估任务（真实 Google Robot 上）**：
- Knock object off table（将物体推落桌面）
- Pick object up（拾取特定物体）
- Place object into container（放入容器）
- Open/close fridge（开关冰箱）

**核心评测指标**：
- Task Success Rate（每个任务的成功率）
- Relative Improvement（vs. single-embodiment baseline 的提升百分比）
- Generalization to Unseen Objects/Environments（泛化测试成功率）

## 优缺点分析

### 优点
- **数据规模前所未有**：100万+ 真实机器人轨迹、22种形态，是迄今最大的开源机器人操控数据集
- **证明跨形态迁移有效**：实验验证了使用来自不同机器人的数据可以提升目标机器人性能，为跨形态学习奠定实证基础
- **标准化数据格式**：RLDS 格式统一了异构数据，推动了机器人数据生态建设
- **推动开源生态**：所有数据集完全开源，极大地降低了机器人基础模型研究的数据门槛
- **真实机器人评估**：所有评估均在真实机器人上进行，具有直接的实际意义

### 局限性
- **数据质量参差不齐**：来自多机构的数据在质量、任务复杂性、语言标注等方面存在较大差异
- **跨形态改进有限**：RT-2-X 的跨形态提升幅度（约 3-6%）有限，且效果对数据混合比例敏感
- **动作空间不统一**：不同机器人的动作空间维度和含义不同，统一表示仍然困难
- **长尾分布问题**：数据量在不同机器人形态和任务类型上极不均衡
- **语言标注质量不一**：部分数据集语言指令简单，无法充分利用语言模型的语义理解能力
- **评估任务覆盖不全面**：主要在 Google Robot 上评估，其他形态上的迁移效果验证较少

## 被引用情况 / 影响力

Open X-Embodiment 是机器人学习领域**最具影响力的数据集论文之一**，已成为 VLA 领域大规模预训练的**标准数据基础**。截至 2025 年，论文引用量超过 800 次，几乎所有主流 VLA 模型（OpenVLA、π0、RoboVLMs、Octo 等）均使用 OXE 数据集进行预训练或评估。RLDS 数据格式也成为机器人学习社区的标准数据格式。该工作不仅提供了数据，更为"以数据为中心的机器人基础模型"研究范式提供了范本，影响深远。
