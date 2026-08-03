---
title: "念力宅的游戏AI周报：2026-08-03"
date: 2026-08-03
author: VibOtaku
tags: ai agents game-ai newsletter
lang: zh
translation_key: ai-in-games-weekly-2026-08-03
---

**2026-07-27 - 2026-08-03**

## 本周精彩研究

- GUI agent 正在变成完整的工作流 agent。Qwen-UI-Agent 把移动端控制、桌面操作、浏览器任务、CLI 执行和主动服务放进同一个训练与评测栈里。
- World-action model 正在从视频预测走向规划。World Action Planner 把想象 rollout 当成搜索空间，用来处理组合任务和变化后的场景布局。
- 第一视角数据生成要真的服务长程 agent，关键在于同时保持场景锚点、相机几何和末端执行器动作的一致性。
- 3D 世界生成开始认真处理可导航性。Genie Sim PanoWorld 通过一个轨迹可控的全景视频阶段，把单张 360 度全景图变成可以自由漫游的 3D Gaussian 场景。
- 安全研究正在追上 world model。新的 survey 把投毒、传感器欺骗、prompt injection、轨迹攻击和记忆污染都放进 embodied agent 的生命周期风险里。

## 推荐阅读

| 文章 | 推荐度 | 重点 |
| --- | --- | --- |
| Qwen-UI-Agent Technical Report: Toward Next-Generation Real-World Centric Foundation GUI Agents | SSS | 一个很接近产品形态的 GUI agent 栈：真实设备移动端训练、GUI 加 CLI 混合动作、超过 100 步的 online RL，以及很强的社区热度。 |
| World Action Planner: Generalizable Decision-Making with Action-Conditioned World Models | SSS | 用 VLM 规划和 action-conditioned world model rollout 在执行前修正动作，面向组合任务和 zero-shot 操作任务。 |
| EgoGenesis: Egocentric World-Action Modeling with Online Anchored Projective Memory and Action-3D RoPE | SS | 生成动作对齐的第一视角 rollout，并报告了把合成轨迹加入少量真实示范后，真实机器人 OOD 成功率的提升。 |
| Genie Sim PanoWorld: An Infinite Indoor 3D World Generation Pipeline via Panoramic Scene Modeling and Simulation | SS | 从一张 360 度全景图出发，通过显式轨迹控制的全景视频，生成可以自由漫游的 3D Gaussian 室内场景。 |
| Security of World-Model-Based Embodied AI: A Lifecycle of Threats, Defenses, and Evaluation | A | 把 world model 当成安全边界来分析，覆盖数据、grounding、imagination、评估、执行、记忆和工具。 |

## 文章小记

### 1. Qwen-UI-Agent Technical Report: Toward Next-Generation Real-World Centric Foundation GUI Agents

- **推荐度:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.28227) / [Hugging Face](https://huggingface.co/papers/2607.28227) / [GitHub](https://github.com/Tongyi-MAI/MAI-UI) / [Project page](https://tongyi-mai.github.io/Qwen-UI-Agent/)
- **念力宅's Note:** Qwen-UI-Agent 很值得从 agent 基础设施角度读一遍，即使你的团队做的是游戏。它把 GUI 控制当成长程运行时问题来处理，覆盖移动端、桌面、网页、DeepSearch 和 shell 命令，并且一次模型决策可以输出 batched actions。报告称训练使用超过 100 步的 online RL，并行环境超过 10,000 个；同时还有一个数据飞轮，让 agent 生成任务、诊断失败并决定下一轮迭代。结果也很硬：MobileWorld 82.1%，MobileWorld-Real 92.2%，AndroidDaily 97.5%，OSWorld-Verified 79.5%，WebArena 73.6%，ScreenSpot-Pro 81.5%。我检查时，GitHub repo 有 1.9k stars、181 forks、57 commits，最近一次提交在 Aug 2。对游戏 agent 团队来说，最值得借鉴的是 harness 思路：把 UI 动作、工具、状态化工作流、verifier 和 online rollout 放在一起，而不是把 agent 当成一次性点屏幕的模型。

### 2. World Action Planner: Generalizable Decision-Making with Action-Conditioned World Models

- **推荐度:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.27599) / [Hugging Face](https://huggingface.co/papers/2607.27599) / [GitHub](https://github.com/XiangchengZhang/world-action-planner) / [Project page](http://worldactionplanner.github.io/)
- **念力宅's Note:** World Action Planner 的形态很适合游戏 agent：planner 先提出动作，在 world model 里想象结果，然后搜索或优化，最后才真正执行。系统把 VLM 和 pose-image conditioned world model 结合起来，测试组合泛化、布局变化，以及只用硬编码 grasp 和 release primitive 的 zero-shot Robosuite 任务。项目页展示了 planner 如何用 imagined rollout 处理转场动作、目标物移动、碰撞风险和立方体精确对齐。我检查时 repo 还很早期，有 6 stars、3 commits，以及 July 29 的 README 更新，但代码路径已经公开。放到游戏里，这对应的是 NPC 在消耗昂贵环境步数之前，先在想象中试几种计划。

### 3. EgoGenesis: Egocentric World-Action Modeling with Online Anchored Projective Memory and Action-3D RoPE

- **推荐度:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.28243) / [Project page](https://egogenesis.github.io/)
- **念力宅's Note:** EgoGenesis 盯住了合成经验里最容易坏掉的部分：时间维度上的动作对齐。它用 Online Anchored Projective Memory 保存第一帧的 3D 场景锚点，同时刷新最近状态；再用 Action-3D RoPE 把末端执行器动作放进 camera-aware 3D 坐标里。项目页报告了 210K 个 source-balanced training clips，相比 RoPE baseline 在第 80 帧时 depth error 降低 79.3%、camera error 降低 78.3%。把 400 条生成轨迹加入 400 条真实轨迹后，OOD 成功率在单臂任务上从 77% 到 84%，双臂任务上从 53% 到 70%。对游戏和 embodied agent 来说，这里的教训很直接：合成 rollout 要有稳定的物体身份、度量动作控制和持续场景记忆，才可能成为有用的训练数据。

### 4. Genie Sim PanoWorld: An Infinite Indoor 3D World Generation Pipeline via Panoramic Scene Modeling and Simulation

- **推荐度:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.26646)
- **念力宅's Note:** Genie Sim PanoWorld 是一篇场景生成论文，但抓住了一个很实用的问题：从单张 360 度全景图出发，先生成轨迹可控的全景视频，再把它提升成可以实时自由视角漫游的 3D Gaussian 场景。论文使用 NavMesh 规划的 SE(3) roaming trajectory、dense geometry-warped conditioning、长短轨迹混合训练，以及基于 shortcut model 的一致性目标，用四步 CFG-free denoising 生成视频。对虚拟世界来说，这不只是生成一张好看的图。它输出的是带有明确路径、可以导航的资产，更接近 simulation、QA 和 NPC 训练管线真正需要的东西。

### 5. Security of World-Model-Based Embodied AI: A Lifecycle of Threats, Defenses, and Evaluation

- **推荐度:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.28226)
- **念力宅's Note:** 这篇 survey 有用，是因为它把 world model 周围的攻击面讲清楚了，而不是把它当成 planner 里的黑盒组件。论文按生命周期讨论威胁，包括数据构建、representation learning、state grounding、imagination、轨迹评估、执行、adaptation、memory 和 tools。这对游戏 agent 也很现实。被污染的 world model 可能让 agent 信错 affordance，想象出一条穿过危险区域的安全路径，接受被投毒的反馈，或者把错误记忆带进后续计划。实践上的 takeaway 是按生命周期阶段评估 world-model failure：数据要查 provenance，状态要做 robust grounding，想象未来要带 uncertainty，动作前要有 trajectory gating，反馈和记忆更新要能审计。

## References

- Qwen-UI-Agent arXiv: https://arxiv.org/abs/2607.28227
- Qwen-UI-Agent Hugging Face page: https://huggingface.co/papers/2607.28227
- Qwen-UI-Agent project page: https://tongyi-mai.github.io/Qwen-UI-Agent/
- Qwen-UI-Agent GitHub: https://github.com/Tongyi-MAI/MAI-UI
- World Action Planner arXiv: https://arxiv.org/abs/2607.27599
- World Action Planner Hugging Face page: https://huggingface.co/papers/2607.27599
- World Action Planner project page: http://worldactionplanner.github.io/
- World Action Planner GitHub: https://github.com/XiangchengZhang/world-action-planner
- EgoGenesis arXiv: https://arxiv.org/abs/2607.28243
- EgoGenesis project page: https://egogenesis.github.io/
- Genie Sim PanoWorld arXiv: https://arxiv.org/abs/2607.26646
- Security of World-Model-Based Embodied AI arXiv: https://arxiv.org/abs/2607.28226
