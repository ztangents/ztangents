---
title: "念力宅的游戏AI周报：2026-07-27"
date: 2026-07-27
author: VibOtaku
tags: ai agents game-ai newsletter
lang: zh
translation_key: ai-in-games-weekly-2026-07-27
---

**2026-07-20 - 2026-07-27**

## 本周精彩研究

- 交互式世界模型正在接近产品约束：低延迟、身份一致性、动作反馈，以及单张桌面 GPU 的运行目标。
- 3D 智能体评测开始转向可执行产物。SceneActBench 要求智能体编辑 Blender 场景，并用几何指标评分，这比文本问答更接近游戏生产。
- 具身模型开始更重视接触点、度量 3D grounding，以及跨机器人身体的共享动作空间。这些思路也能直接迁移到 NPC 交互和重物理玩法。
- 智能体可靠性研究仍在反复回到记忆和验证。AREX 的核心是把已验证证据和未解决约束保留在长研究循环里。
- 空间推理评测正在把像素当作一种答案接口。对于需要指点、画路径、选择区域或操作场景状态的智能体，这一点很重要。

## 推荐阅读

| 文章 | 推荐度 | 重点 |
| --- | --- | --- |
| ABot-World-0: Infinite Interactive World Rollout on a Single Desktop GPU | SSS | 在单张 RTX 5090 上以最高 16 FPS 流式生成动作条件控制的 720p 世界 rollout，并开放代码、权重和 HF Space。 |
| SceneActBench: Can Agents Act on the 3D Scenes They See? | SSS | 让 VLM 智能体在 Blender 中完成可执行 3D 任务，再用隐藏几何指标评分最终产物。 |
| RynnBrain 1.1: Towards More Capable and Generalizable Embodied Foundation Model | SS | 加入接触点预测、原生 3D grounding，以及横跨三种机器人平台的共享 VLA 动作接口。 |
| AREX: Towards a Recursively Self-Improving Agent for Deep Research | SS | 用递归验证和学习出来的上下文更新工具，避免长程研究智能体丢失已检查证据。 |
| Show, Don't Tell: Evaluating Spatial Cognition in Generative Pixels Rather Than LLM Text | A | 允许图像生成模型直接用像素回答空间任务，再把答案解析回 benchmark 指标。 |

## 文章小记

### 1. ABot-World-0: Infinite Interactive World Rollout on a Single Desktop GPU

- **Reading Priority:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.19191) / [Hugging Face](https://huggingface.co/papers/2607.19191) / [GitHub](https://github.com/amap-cvlab/ABot-World) / [Project page](https://abot-world.amap.com/) / [HF Space](https://huggingface.co/spaces/acvlab/abot-world-interactive) / [Model](https://huggingface.co/acvlab/ABot-World-0-5B-LF)
- **念力宅小记:** ABot-World-0 值得读，因为它把世界模型当成一个交互式 runtime 问题，而不是视频片段生成问题。论文报告了键盘控制的漫游和第三人称角色交互，用 reference-character memory 保持身份一致性，并在单张 RTX 5090 上实现最高 16 FPS 的 720p 流式生成，action-to-first-frame 延迟 1.2 秒，峰值显存约 19 GiB。检查时，开源仓库有 1.3k stars、35 forks、20 commits，并且 7 月 27 日仍有提交。HF 模型页显示上月下载 1,067 次，公开 Space 也让控制循环更容易观察。对游戏智能体团队来说，这是一种很实用的原型形态：动作采集、rollout 稳定性、运行预算和部署路径都在同一个系统里。

### 2. SceneActBench: Can Agents Act on the 3D Scenes They See?

- **Reading Priority:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.22393) / [Hugging Face](https://huggingface.co/papers/2607.22393) / [GitHub](https://github.com/Feinaldo2/SceneActBench) / [Project page](https://feinaldo2.github.io/sceneactbench-project-page/)
- **念力宅小记:** SceneActBench 问的是 3D 智能体最该回答的问题：模型能不能真正对场景采取行动，而不仅是描述场景？这个 benchmark 覆盖 layout、camera、articulated objects、reconstruction、dynamics 五类任务，包含 210 个 source instances 和 520 个 task cases。智能体观察图像或采样视频帧，通过共享 Blender 工具接口行动，并提交 JSON 或 GLB 产物，由隐藏几何指标评分。论文报告的 11 个 proprietary VLM configurations 的 overall 分数范围是 38.6 到 50.2，说明 3D 控制策略还有很大空间。检查时，GitHub 仓库有 7 stars、6 commits，并在 7 月 22 日扩展了 README；社区信号还很早，但这个 harness 正是游戏团队应该想要的评测形态。

### 3. RynnBrain 1.1: Towards More Capable and Generalizable Embodied Foundation Model

- **Reading Priority:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.17977) / [Hugging Face](https://huggingface.co/papers/2607.17977) / [GitHub](https://github.com/alibaba-damo-academy/RynnBrain) / [Project page](https://alibaba-damo-academy.github.io/RynnBrain) / [HF collection](https://huggingface.co/collections/Alibaba-DAMO-Academy/rynnbrain-11)
- **念力宅小记:** RynnBrain 1.1 是机器人论文，但它和游戏智能体的关系很直接。它训练了 2B、9B、122B-A10B 三个尺度的模型家族，面向具身感知、空间推理、定位、规划、接触点预测和原生 3D grounding。VLA 版本使用共享的 81 维动作空间，并通过 embodiment-specific masking 适配 Unitree G1、Astribot-S1 和 Tianji-Wuji。项目页报告，在一个 controlled comparison 中，RynnBrain-VLA 达到 91.28 percent process score 和 86.67 percent success；使用 joint multi-task and multi-embodiment training 后提升到 94.14 percent 和 91.67 percent。检查时，仓库有 844 stars、81 forks、66 commits，并在 7 月 22 日更新了 README。对 NPC、同伴角色和重操作模拟来说，最值得借鉴的是 contact-aware grounding。

### 4. AREX: Towards a Recursively Self-Improving Agent for Deep Research

- **Reading Priority:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.21461) / [Hugging Face](https://huggingface.co/papers/2607.21461) / [GitHub](https://github.com/VectorSpaceLab/arex-model) / [Project page](https://vectorspacelab.github.io/arex-model/) / [HF collection](https://huggingface.co/collections/BAAI/arex) / [Application](https://arex-research.com/)
- **念力宅小记:** AREX 不是游戏论文，但它的循环很适合长程游戏智能体和工具智能体。智能体在 research、verification、state update 和下一轮 targeted pass 之间交替。它学习出来的 context-update tool 会把已验证证据、未解决约束和下一步计划压缩进紧凑状态。任务解谜智能体、live-ops 助手、自动 QA 智能体、build-debug loop 都需要这种形态：记住什么已经被证明，而不是每次重新读完整历史。检查时，这篇论文是 7 月 24 日 Hugging Face 当日 #1 Paper，有 141 upvotes；GitHub 仓库有 18 stars 和 7 月 24 日提交，HF collection 中列出了 AREX-Turbo 和 AREX-Base 模型。

### 5. Show, Don't Tell: Evaluating Spatial Cognition in Generative Pixels Rather Than LLM Text

- **Reading Priority:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.21072) / [Hugging Face](https://huggingface.co/papers/2607.21072) / [Project page](https://zju-omniai.github.io/ProVisE/) / [GitHub](https://github.com/ZJU-OmniAI/ProVisE) / [Dataset](https://huggingface.co/datasets/wx91726/SpatialGen-Bench)
- **念力宅小记:** ProVisE 提醒了一个很实际的评测问题：有些空间答案本来就应该用图像空间里的绘制、标注或选择来表达。它提出的 SpatialGen-Bench 包含 470 个 samples、14 个 subtasks、四个 capability levels，并覆盖区域、路径、next-view choice 等答案形式。Agentic builder 会构建并验证任务专用的 visual-answer protocols，再把生成图像解析成可用于既有指标的结构化预测。检查时，Hugging Face 页面有 35 upvotes，链接的 GitHub 仓库有 18 stars。对游戏智能体来说，这指向了更好的评测接口：导航、affordance、战术站位和编辑器动作往往不适合只用坐标或文字回答。

## References

- ABot-World-0: https://arxiv.org/abs/2607.19191
- ABot-World-0 Hugging Face page: https://huggingface.co/papers/2607.19191
- ABot-World GitHub: https://github.com/amap-cvlab/ABot-World
- ABot World Studio: https://abot-world.amap.com/
- ABot-World Interactive HF Space: https://huggingface.co/spaces/acvlab/abot-world-interactive
- ABot-World model page: https://huggingface.co/acvlab/ABot-World-0-5B-LF
- SceneActBench: https://arxiv.org/abs/2607.22393
- SceneActBench Hugging Face page: https://huggingface.co/papers/2607.22393
- SceneActBench GitHub: https://github.com/Feinaldo2/SceneActBench
- SceneActBench project page: https://feinaldo2.github.io/sceneactbench-project-page/
- RynnBrain 1.1: https://arxiv.org/abs/2607.17977
- RynnBrain 1.1 Hugging Face page: https://huggingface.co/papers/2607.17977
- RynnBrain GitHub: https://github.com/alibaba-damo-academy/RynnBrain
- RynnBrain project page: https://alibaba-damo-academy.github.io/RynnBrain
- RynnBrain 1.1 HF collection: https://huggingface.co/collections/Alibaba-DAMO-Academy/rynnbrain-11
- AREX: https://arxiv.org/abs/2607.21461
- AREX Hugging Face page: https://huggingface.co/papers/2607.21461
- AREX GitHub: https://github.com/VectorSpaceLab/arex-model
- AREX project page: https://vectorspacelab.github.io/arex-model/
- AREX HF collection: https://huggingface.co/collections/BAAI/arex
- AREX application: https://arex-research.com/
- Show, Don't Tell: https://arxiv.org/abs/2607.21072
- Show, Don't Tell Hugging Face page: https://huggingface.co/papers/2607.21072
- ProVisE project page: https://zju-omniai.github.io/ProVisE/
- ProVisE GitHub: https://github.com/ZJU-OmniAI/ProVisE
- SpatialGen-Bench dataset: https://huggingface.co/datasets/wx91726/SpatialGen-Bench
