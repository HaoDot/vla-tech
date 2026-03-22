# RLBench: The Robot Learning Benchmark & Learning Environment

## 基本信息
- **论文**: RLBench: The Robot Learning Benchmark & Learning Environment
- **链接**: https://arxiv.org/abs/1909.12271
- **发表**: IEEE Robotics and Automation Letters (RAL) + ICRA 2020
- **机构**: Imperial College London（伦敦帝国理工学院），作者：Stephen James, Zicong Ma, David Rovick Arrojo, Andrew J. Davison

## 核心贡献
RLBench 是一个大规模、多样化的机器人学习仿真基准，提供**100 个完全独特、人工精心设计的操控任务**，覆盖从简单目标触达到复杂多阶段任务的广泛难度范围。其独特之处在于：每个任务均内置运动规划器（motion planner），可自动生成**无限数量的演示数据**，从而为强化学习、模仿学习、少样本学习等多种研究方向提供支撑。RLBench 的设计目标是为机器人学习社区提供一个统一、可扩展的评测平台。

## 方法/测评设计详解

**仿真环境基础**：RLBench 基于 PyRep（Lua 绑定的 V-REP/CoppeliaSim）构建，使用 7-DoF Franka Panda 机械臂配合并行式夹爪（Franka gripper）。仿真环境提供高质量的物理仿真，支持接触力学、刚体动力学等物理效果。

**无限演示生成机制**：RLBench 最核心的技术贡献是其**基于 waypoint 的运动规划演示生成系统**。每个任务在创建时由设计者指定一系列关键路径点（waypoints），运动规划器（基于 RRT 等算法）自动规划连接这些路径点的可行轨迹，结合随机初始化（物体位置、颜色等），可以生成理论上无限数量的 demonstration。这一机制消除了依赖人工遥操作收集数据的瓶颈。

**多模态感知支持**：RLBench 为每个任务提供丰富的多模态观测：
- **视觉观测**：来自肩部立体相机（over-the-shoulder stereo camera）的 RGB、深度图和语义分割掩码
- **末端感知**：eye-in-hand 单目相机的 RGB、深度和分割信息
- **本体感知**：关节角度、关节速度、末端执行器位姿、夹爪状态

**可扩展的任务提交系统**：RLBench 设计了开放的任务创建工具链，允许研究者创建新任务（包括运动规划 demo）并提交到 RLBench 任务仓库，持续扩展基准的任务多样性。验证工具确保新任务的质量和一致性。

**首个大规模少样本挑战**：利用海量演示数据，RLBench 提出了机器人领域**首个大规模少样本学习挑战（large-scale few-shot challenge）**，使用新任务上的少量演示评估策略的快速适应能力。

## 主要任务与场景

**100 个任务的难度分级（示例）**：

*简单任务（Single-stage）*：
- reach_target：触达目标球体
- pick_up_cup：拾取杯子
- open_drawer：打开指定抽屉
- push_button：按下按钮
- open_door：开门

*中等任务（Multi-stage）*：
- stack_blocks：叠放积木
- place_cups_in_holder：将杯子放入支架
- put_groceries_in_cupboard：将食品放入橱柜
- open_oven：打开烤箱门

*复杂任务（Long-horizon）*：
- place_hanger_on_rack：将衣架放到衣架杆上
- put_tray_in_oven：打开烤箱并放入托盘
- sweep_to_dustpan_of_size：将碎屑扫入指定大小的撮箕
- setup_checkers：摆放跳棋棋盘

**核心评测指标**：
- Task Success Rate（任务成功率）
- Few-shot Success Rate（少样本成功率，N-shot）
- Multi-task Performance（多任务联合训练的平均成功率）

## 优缺点分析

### 优点
- **任务多样性无与伦比**：100 个精心设计的任务，覆盖日常操控技能的全谱系
- **无限演示生成**：运动规划器自动生成 demonstration，彻底解决数据稀缺问题
- **多模态感知完备**：RGB、深度、分割、本体感知一应俱全，支持多感知融合研究
- **可扩展架构**：开放的任务提交系统使基准持续成长
- **多研究方向支持**：同时支持 RL、IL、多任务学习、少样本学习等多种学习范式

### 局限性
- **纯仿真环境**：与真实 Franka Panda 存在 sim-to-real gap，真实部署性能无保证
- **运动规划生成的 demo 质量有限**：自动生成的轨迹不如人类遥操作数据自然，可能引入非最优行为模式
- **单一机器人形态**：仅支持 Franka Panda，无法测试跨机器人泛化
- **评测标准化不足**：不同论文使用不同任务子集和评测协议，横向比较困难
- **高 GPU/CPU 需求**：高质量仿真对计算资源要求较高

## 被引用情况 / 影响力

RLBench 是机器人操控领域**引用最广泛的仿真基准之一**，截至 2025 年引用量超过 1500 次。它被大量视觉-语言-动作（VLA）和操控策略研究采用，包括 PerAct、Act3D、GNFactor、RVT、ARM 等重要工作。RLBench 特别在**多任务操控（multi-task manipulation）**研究中扮演核心角色，是该方向的事实标准基准。其 GitHub 仓库 (stepjam/RLBench) 已获得 2000+ star，持续更新维护，任务库也在社区贡献下不断扩展。
