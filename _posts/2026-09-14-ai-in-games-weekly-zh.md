---
title: "念力宅的游戏AI周报：2026-09-14"
date: 2026-09-14
author: VibOtaku
tags: ai agents game-ai newsletter
lang: zh
translation_key: ai-in-games-weekly-2026-09-14
---

**2026-09-07 - 2026-09-14**

## 本周精彩研究

- 显式世界状态重新回到世界模型的核心。Programmable World Model 把实体、规则和屏幕外的世界事实放进由程序驱动的状态引擎，视频模型只负责渲染。
- 世界模型开始产出游戏几何，而不只是消费几何。Valerant 用动作条件化的 rollout 配合 SLAM 重建，把一张图片变成持久、可导航的 3D 地图。
- 由 LLM 编写的游戏代码正在变成强化学习基础设施。PlayTrain 可以在单台 GPU 节点上以每秒超过一百万个决策的速度，在任何生成的 JavaScript 游戏里训练智能体。
- 对手建模的评测单位从单局扩展到整个对局系列。HORIZON 在 Lux AI Season 3 中推断隐藏的游戏参数与对手风格，并在五局三胜的赛制下报告适应能力提升。
- 合作型 AI 的评测重心从分数转向沟通。The Convention Gap 量化了字面消息内容无法解释的合作失败，人类 Hanabi 搭档的差距为 +26.2 个百分点，AI 搭档为 -0.7。

## 推荐阅读

| 文章 | 推荐度 | 重点 |
| --- | --- | --- |
| Programmable World Model | SSS | 把可编程的世界状态引擎与生成式渲染器分离，让生成的世界能够维持持久状态。 |
| Valerant: An Automatic Navigable Game Map Generator via Action-Conditioned World Model Exploration | SS | 无需训练，用世界模型 rollout 与 SLAM 从单张图片构建持久的 3D 游戏地图。 |
| PlayTrain: An Efficient Reinforcement Learning Framework for LLM-Generated Adaptable JavaScript Games | SS | 在 gym 循环中运行 LLM 生成的 JavaScript 游戏，以每秒超过百万次决策训练像素级智能体。 |
| Hierarchical Belief Modeling for Zero-Shot Opponent Adaptation in Partially Observable Multi-Agent Navigation | SS | 推断隐藏的游戏参数与对手风格，在五局三胜的赛制内完成适应。 |
| The Convention Gap: Towards Measuring Implicit Communication in Cooperative AI Evaluation | SS | 量化字面消息内容无法解释的失败，并给出人类、AI 与混合搭档的 Hanabi 数据。 |
| Do LLMs Trust the Accuser or the Accusation? Measuring Belief Shifts in Werewolf | A | 以指控消息对观察者信念的影响幅度来打分。 |
| Environments as Scaffold: Enriching Feedback to Bootstrap Self-Evolving Agents in Long-Horizon Tasks | A | 把环境反馈设计当作长程智能体训练的核心变量。 |

## 文章小记

### 1. Programmable World Model

- **推荐度:** SSS（必读）
- **链接:** [arXiv](https://arxiv.org/abs/2609.10540) / [Hugging Face](https://huggingface.co/papers/2609.10540) / [GitHub](https://github.com/AlayaLab/pwm) / [项目主页](https://alaya-lab.github.io/pwm/)
- **念力宅的小记:** 视频世界模型能渲染出可信的画面，却记不住自己渲染过什么。这个框架把规范化的世界状态放在生成器之外：智能体编写可执行程序来定义实体状态与状态转移规则，轻量引擎负责推演，再把带有状态的三维定向包围盒与相机轨迹确定性地编译成像素对齐的条件信号，交给预训练视频模型渲染。实体离开画面后依然存在，非视觉属性可以随时查询，同一份世界状态可以驱动任意机位。作者为此构建了 CombatStateBench，报告了 94% 的计数准确率与 98% 的状态准确率，并以具备既定机制的可试玩游戏作为演示。游戏团队能用上的正是这种拆分：机制存在于可读可改的代码中，渲染保持生成式，这为手写引擎与纯提示式视频世界之间提供了可验证的中间路线。核查时该仓库有 162 个 star、1 个未关闭 issue，最近一次推送为 2026-09-10，发布路线图中的推理代码与预训练权重仍标注为未发布。在 Hugging Face 每日论文列表中获得 106 个 upvote。

### 2. Valerant: An Automatic Navigable Game Map Generator via Action-Conditioned World Model Exploration

- **推荐度:** SS（强烈推荐）
- **链接:** [arXiv](https://arxiv.org/abs/2609.09418)
- **念力宅的小记:** 游戏在模型之外没有可依托的现实底座。自动驾驶与机器人领域的世界天然独立存在，世界模型可以对照现实校验，而 3D 游戏必须先被实例化，才谈得上在其中探索。Valerant 无需训练，直接作用于预训练的动作条件化世界模型，把它转成能在单张图片基础上构建可导航地图的 world action model，做法是把预测性视觉 rollout 与基于 SLAM 的空间重建、探索驱动的动作选择结合起来。产出是持久几何而不是一段帧序列，而持久几何正是关卡能被走通的前提。能力上限受基础世界模型的渲染能力约束，所以它更接近一次成型的布局生成器，而不是关卡设计的替代品；但地图布局的成本足够高，一个可用的初版就能改变美术与设计的时间分配。核查时该论文未列出公开仓库或项目主页。

### 3. PlayTrain: An Efficient Reinforcement Learning Framework for LLM-Generated Adaptable JavaScript Games

- **推荐度:** SS（强烈推荐）
- **链接:** [arXiv](https://arxiv.org/abs/2609.09059)
- **念力宅的小记:** 手工搭建游戏环境，把强化学习研究长期限制在一份不长的基准清单上。PlayTrain 从极简提示生成 JavaScript 游戏，并让任意一个生成结果直接运行在标准 gym 环境里，同一个 JS 文件既是人能玩的构建，也是智能体训练的环境。作者用简洁的 JS 复刻了著名的 Atari 与 ProcGen 游戏，在单台 GPU 节点上以每秒超过一百万个决策的速度端到端训练像素级智能体，并进一步生成带新测试集、不同程序化生成逻辑或不同游戏动态的变体。对评测工作最直接的收益是低成本再生：当某个基准不再区分方法时，新变体的代价只是一段提示和一个文件。生成逻辑可编辑这一点，也让关卡分布本身成为可调节的实验变量。

### 4. Hierarchical Belief Modeling for Zero-Shot Opponent Adaptation in Partially Observable Multi-Agent Navigation

- **推荐度:** SS（强烈推荐）
- **链接:** [arXiv](https://arxiv.org/abs/2609.12422)
- **念力宅的小记:** Lux AI Season 3 要求智能体在不完全可观测与随机化的单局动态下行动，并以五局三胜的赛制评分，其中适应能力与执行能力同样重要。HORIZON 结合了对称性感知的空间感知、双记忆信念追踪、以 relic 为中心的图注意力、信息增益驱动的探索，以及对手条件化的策略混合；它把短程控制与跨对局的元推理分开，并用辅助的信念与世界模型目标，在大型 JAX 模拟器中通过 PPO 稳定训练。该智能体显式推断隐藏的游戏参数与对手风格，报告的提升覆盖对局胜率、单局胜率、适应增益与联赛积分，对比对象是循环网络与前馈基线。已上线游戏的排位赛具有相同结构：同一个对手会反复出现，隐藏参数始终不公开，第一局奏效的打法往往正是第二局落败的原因。

### 5. The Convention Gap: Towards Measuring Implicit Communication in Cooperative AI Evaluation

- **推荐度:** SS（强烈推荐）
- **链接:** [arXiv](https://arxiv.org/abs/2609.11489) / [GitHub](https://github.com/dockmfgit/hanabi-convention-gap) / [Zenodo 存档](https://doi.org/10.5281/zenodo.21975884)
- **念力宅的小记:** 合作型智能体通常与其他智能体对局来评测，而人类合作依赖的约定，并不包含在消息的字面内容里。convention gap 定义为字面内容后验预测的失败概率与实际观测失败率之差，而 Hanabi 的牌堆有限、提示确定，使这个后验可以被精确计算。重放人类对人类、AI 对 AI、人类对 AI 三份语料中约 101,000 次出牌动作后，得到的差距分别是 +26.2 个百分点、-0.7 和 +16.4，其中人类的差距集中在从未收到过提示的牌上（+46 个百分点）。在人机混局中，人类可获得的字面信息在三个 AI 搭档之间相近（预测失败率 38% 到 41%），但人类失败率从 14.4% 到 34.4% 不等，也就是说差距最大的搭档反而让人类失败最少。游戏分数携带的是另一类信息，它取决于各语料的成员构成，而 convention gap 能在智能体层面区分人类与 AI 对局。对于任何包含真人玩家与 AI 队友的游戏，这是一个直接评测沟通能力的指标，相关代码、去标识化的逐局数据以及可复现全部图表的 notebook 均已公开。核查时该仓库有 0 个 star，最近一次推送为 2026-09-11。

### 6. Do LLMs Trust the Accuser or the Accusation? Measuring Belief Shifts in Werewolf

- **推荐度:** A（推荐阅读）
- **链接:** [arXiv](https://arxiv.org/abs/2609.12446) / [GitHub](https://github.com/rlglab/werewolf-accusation-benchmark) / [基准主页](https://rlg.iis.sinica.edu.tw/papers/werewolf-accusation-benchmark)
- **念力宅的小记:** 多数社交推理评测只报告谁赢了这一局。这项工作标注了 LLM 对局中的怀疑与指控消息，测量每条消息之后观察者信念的移动幅度，覆盖 40 个开源权重模型配置与 1,224 条已标注消息。更大的模型能更好地依据对局历史区分狼人与村民，但指控依然会改变信念：被指控者变得更可疑，指控者变得更可信，即使指控者本身属于狼人阵营，而指控者已被信任时这种效果最强。更大的模型确实会抵抗来自自己本就不信任者的指控。可测量的差距在于读懂一句话的内容与掂量这句话是谁说的之间，任何包含欺骗或谈判机制的游戏，都依赖这项能力在最终上线的模型里靠得住。作者表示基准与代码已公开；核查时该仓库有 1 个 star，最近一次推送为 2026-09-02。

### 7. Environments as Scaffold: Enriching Feedback to Bootstrap Self-Evolving Agents in Long-Horizon Tasks

- **推荐度:** A（推荐阅读）
- **链接:** [arXiv](https://arxiv.org/abs/2609.08404) / [Hugging Face](https://huggingface.co/papers/2609.08404) / [GitHub](https://github.com/HongbangYuan/EnvAsScaffold)
- **念力宅的小记:** 拖住长程智能体训练的原因，往往是奖励稀疏而不是策略容量不足，而常规做法是用监督微调先给智能体热身，又会受限于数据稀缺与探索范围狭窄。这项工作把干预移进环境内部，构建反馈增强环境，随着单局推进与训练进行，从动作引导逐步转向观测增强，让观测细节出现在标准反馈沉默的地方。在 SciWorld 与 BFCL 上，跨多个 Qwen3 规模以及 GRPO、GSPO、DAPO 三种算法，反馈增强环境都优于标准设置，降低了训练过程中的熵波动，把探索推进到策略原本会跳过的状态，并把引导内化进策略权重，而不是停留在推理期的辅助信息。论文还指出组内反馈一致性是稳定优化的边界条件。提示投放与观测细节本来就是游戏团队掌握的旋钮，这项工作给出了一个有测量支撑的理由，让大家认真对待它们，而不是把环境当成固定背景。核查时该仓库有 3 个 star、0 个未关闭 issue，最近一次推送为 2026-09-09。

## 参考文献

- Programmable World Model arXiv: https://arxiv.org/abs/2609.10540
- Programmable World Model Hugging Face 页面: https://huggingface.co/papers/2609.10540
- Programmable World Model GitHub: https://github.com/AlayaLab/pwm
- Programmable World Model 项目主页: https://alaya-lab.github.io/pwm/
- Valerant arXiv: https://arxiv.org/abs/2609.09418
- PlayTrain arXiv: https://arxiv.org/abs/2609.09059
- Hierarchical Belief Modeling arXiv: https://arxiv.org/abs/2609.12422
- The Convention Gap arXiv: https://arxiv.org/abs/2609.11489
- The Convention Gap GitHub: https://github.com/dockmfgit/hanabi-convention-gap
- The Convention Gap Zenodo 存档: https://doi.org/10.5281/zenodo.21975884
- Do LLMs Trust the Accuser or the Accusation? arXiv: https://arxiv.org/abs/2609.12446
- Do LLMs Trust the Accuser or the Accusation? GitHub: https://github.com/rlglab/werewolf-accusation-benchmark
- Do LLMs Trust the Accuser or the Accusation? 基准主页: https://rlg.iis.sinica.edu.tw/papers/werewolf-accusation-benchmark
- Environments as Scaffold arXiv: https://arxiv.org/abs/2609.08404
- Environments as Scaffold Hugging Face 页面: https://huggingface.co/papers/2609.08404
- Environments as Scaffold GitHub: https://github.com/HongbangYuan/EnvAsScaffold
