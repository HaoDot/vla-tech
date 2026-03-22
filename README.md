# VLA Tech Survey

> 系统整理 VLA（Vision-Language-Action）模型的**测评方法**与 **Tokenizer 技术**，持续更新中。

---

## 📋 目录
- [一、VLA 测评](#一vla-测评)
  - [仿真环境测评](#仿真环境测评)
  - [真机测评](#真机测评)
- [二、VLA Tokenizer](#二vla-tokenizer)
  - [离散化方法](#离散化方法)
  - [连续值方法](#连续值方法)

---

## 一、VLA 测评

### 仿真环境测评

| 名称 | 年份 | 任务数 | 核心特点 | 论文 | 笔记 |
|------|------|--------|----------|------|------|
| CALVIN | 2021 | 34 | 语言条件长时域操控，4个场景 | [arXiv](https://arxiv.org/abs/2112.03227) | [笔记](VLA-evaluation/calvin.md) |
| RLBench | 2020 | 100 | 大规模多任务，Gym接口 | [arXiv](https://arxiv.org/abs/1909.12271) | [笔记](VLA-evaluation/rlbench.md) |
| LIBERO | 2023 | 130 | 知识迁移，5个任务套件 | [arXiv](https://arxiv.org/abs/2306.03310) | [笔记](VLA-evaluation/libero.md) |
| SIMPLER | 2024 | - | 仿真代理真实评估，sim-to-real对齐 | [OpenReview](https://openreview.net/forum?id=LZh48DTg71) | [笔记](VLA-evaluation/simpler.md) |

### 真机测评

| 名称 | 年份 | 机器人数 | 核心特点 | 论文 | 笔记 |
|------|------|----------|----------|------|------|
| Open X-Embodiment | 2023 | 22种 | 跨形态大规模数据集，RT-X基础 | [arXiv](https://arxiv.org/abs/2310.08864) | [笔记](VLA-evaluation/open-x-embodiment.md) |
| OpenVLA | 2024 | - | 开源7B VLA，标准化评测协议 | [arXiv](https://arxiv.org/abs/2406.09246) | [笔记](VLA-evaluation/openvla.md) |

### 离线测评

> 目前 VLA 领域尚缺乏成熟的离线评测方法，主流方案仍以在线仿真/真机测评为主。SIMPLER 是离线评测的重要探索，通过构建与真实场景高度对齐的仿真环境实现低成本评测。

---

## 二、VLA Tokenizer

### 核心方法对比

| 方法 | 模型 | 类型 | 词表大小 | 核心思想 | 笔记 |
|------|------|------|----------|----------|------|
| 均匀分箱 | RT-2 | 离散 | 256 bins | 复用语言token，最简实现 | [笔记](VLA-tokenizer/rt2.md) |
| 扩展词表 | OpenVLA | 离散 | 256 bins | 专用action token，开源7B | [笔记](VLA-tokenizer/openvla.md) |
| FASTer | FASTer | 离散(频域) | 可变 | 频域VQ编码，极高压缩比 | [笔记](VLA-tokenizer/faster.md) |
| Flow Matching | π0 | 连续 | N/A | 无离散化，连续动作流 | [笔记](VLA-tokenizer/pi0.md) |
| 空间网格 | SpatialVLA | 离散(空间) | 自适应 | Ego3D位置编码+动作网格 | [笔记](VLA-tokenizer/spatialvla.md) |
| 消融对比 | RoboVLMs | 多种 | - | 系统比较离散vs连续方案 | [笔记](VLA-tokenizer/robovlms.md) |

### 离散化方法
- **RT-2 风格**：将连续动作值均匀分成256个bin，映射到语言模型词表中已有的token（如"0"到"255"）。简单有效，但精度受限。
- **FASTer**：使用神经网络VQ编码器在频域对动作轨迹压缩，大幅减少token数量，提升长序列效率。

### 连续值方法
- **π0 / Flow Matching**：不做离散化，直接用扩散/流匹配模型生成连续动作，精度更高，适合灵巧操作。
- **SpatialVLA**：引入3D空间感知，构建自适应动作网格，兼顾空间精度与token效率。

---

## 📊 可视化总览

👉 [查看 HTML 可视化页面](https://haodot.github.io/vla-tech/)

---

## 🔄 持续更新

欢迎 PR 补充更多论文！格式参考已有 markdown 文件。
