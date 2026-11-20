---
title: "念力宅的游戏AI周报：2026-11-19"
date: 2026-11-19
author: VibOtaku
tags: ai agents game-ai newsletter
lang: zh
translation_key: ai-in-games-weekly-2026-11-19
---

**2026-11-12 - 2026-11-19**

## 本周精彩研究

- 游戏智能体开始连着推理服务和数据管线一起交付。Gamer 用数万小时的键鼠轨迹做预训练，再把多轮强化学习与一个支持无限上下文的云边推理引擎配在一起；Game2Player 则让环境建造者和玩家共同生产训练集。
- “代码造世界、视频模型来渲染”这条路线，两侧都有了基准测试。VibePrevisBench 评的是 Blender 场景本身，WorldDraft 评的是渲染结果与场景是否对齐，而反复出现的失败是：智能体看过自己的渲染，仍然不去改。
- 规则、存档和物体持久化正在作为显式结构进入生成式世界模型。WorldRuler 把规则定义和物体状态存进一张能随存档读档一起恢复的图里，这正是“生成了一段视频”和“玩家可以行动的世界”之间的差别。
- 交互延迟开始有解。FastFrames 在 1.3B 参数上做到每帧 36 毫秒，Code World Model 2.0 的四步因果渲染器在 30 分钟的驾驶 rollout 中保持稳定。
- 评测正在转向“参考在产物出现之前就已固定”：用起始视角几何评场景刚性，用重跑代码评场景重建，用引擎状态评 GTA V 内的任务成败。

## 推荐阅读

| 文章 | 推荐度 | 重点 |
| --- | --- | --- |
| Gamer: A Generalist Vision-Language-Action Foundation Model for Long-Horizon Real-Time Game Agent | SSS | 面向 3D 开放世界的端到端智能体，键鼠预训练、稀疏思考、多轮强化学习与为无限上下文设计的云边服务层组合在一起。 |
| VibePrevisBench: Evaluating Agentic Coding for 4D Scene Generation in Blender | SSS | 242 条简述生成可执行的 Blender 场景；11 个编程智能体都能摆好基础机位，卡在困难的空间请求上，且没有一个会修掉自己渲染里看得见的错误。 |
| GTAGym: AAA Game Tasks Environment for Evaluation and Reinforcement Learning | SSS | 103 个 GTA V 任务用引擎状态验证，另配面向强化学习的任务生成器；在生成任务上做 LoRA 训练，把原生任务成功率从 15% 提到 23%。 |
| WorldRuler: Interactive World Modeling under Configurable Rules | SS | 一张可执行的 3D 规则图，能随存档读档一起恢复，未出现在画面中的物体进入视野时也会正确呈现规则效果。 |
| Game2Player: Scaling Gameplay Experience for Visual Agents through Environment-Player Co-Development | SS | 建造智能体暴露网页游戏内部状态并写出可验证任务，玩家侧同时产生数据；568 个游戏、3,436 个任务喂给按进度加权的 SFT，9B 模型在 PlayerBench 上达到 31.2%。 |
| Code World Model 2.0: Towards Infinite Controllable Video Generation | SS | 把双向创作模型与四步因果流式渲染器放到统一的代理接口下，30 分钟赛车 rollout 保持稳定。 |
| WorldDraft: If Code Builds the World, Can Video Models Just Render It? | SS | 432 组严格配对用例把世界保持与视觉实现分开评，发现草稿脚手架越丰富，几何越稳、外观还原反而越差。 |
| FastFrames: Video Generation at Interactive Latency | A | 把交互延迟追溯到 VAE 解码与 KV cache 更新，去掉这两项后 1.3B 参数在单张 B200 上做到每帧 36 毫秒。 |
| StrideWorld: Benchmarking Scene Rigidity and Action Consistency in Interactive Video World Model | A | 参考在运动发生之前固定，结果发现几乎所有开源模型在相机移动时都会让场景变形。 |
| Actor, Judge, or Teacher? Understanding Decision Models in Long-Horizon Agents | A | 行动很差的决策模型，在被问“这一步有没有帮助”时依然教得很好，这个信号能把小策略训过自己的上限。 |
| Continuation-Guided LLM Evolution of Program Policies and Goal Compilers in Craftax | A | 反馈级联加一个学出来的目标编译器，为一个 roguelike 产出零推理成本的代码策略，并能迁移到更长的任务。 |

## 文章小记

### 1. Gamer: A Generalist Vision-Language-Action Foundation Model for Long-Horizon Real-Time Game Agent

- **推荐度:** SSS（必读）
- **链接:** [arXiv](https://arxiv.org/abs/2611.18397) / [Hugging Face](https://huggingface.co/papers/2611.18397) / [GitHub](https://github.com/RainBowLuoCS/Gamer) / [项目主页](https://mihoyo-gamer.github.io/)
- **念力宅的话:** 多数游戏智能体由视觉模型、规划器和手写动作层拼起来，每道接缝都要付出延迟、丢掉状态。Gamer 用跨度覆盖模拟游戏与具身环境的设备级键鼠轨迹训练单一策略，再加入稀疏思考的指令微调削减冗余推理，并用多轮强化学习对准长会话最常坏的三件事：有限上下文窗口、累积的 rollout 误差、稀疏奖励。服务侧被放在同等重要的位置：云边引擎同时托管多个游戏环境，执行可以越过上下文窗口继续，线上延迟压到实时可玩的程度。论文报告的是在数个 3D 开放世界中具备与熟练玩家相当的运行效率，在常规 2D 游戏基准上也有竞争力。读它的重点是管线形状而不是某个分数：先用普通输入轨迹预训练，再在长交互上做强化学习，最后把推理引擎当作智能体的一部分而不是事后补丁。最后这一点，多数工作室会认成自己的瓶颈。2026-11-19 核查：Hugging Face 140 个赞；仓库 1,006 star、22 fork、6 个未关闭 issue，最后推送 2026-11-16。公开目录里目前是 `streaming-vllm` 推理路径加素材和许可证，服务侧代码可用，训练代码尚未放出。

### 2. VibePrevisBench: Evaluating Agentic Coding for 4D Scene Generation in Blender

- **推荐度:** SSS（必读）
- **链接:** [arXiv](https://arxiv.org/abs/2611.15789) / [Hugging Face](https://huggingface.co/papers/2611.15789) / [GitHub](https://github.com/vibeprevisbench/vibeprevisbench) / [项目主页](https://vibeprevisbench.github.io/)
- **念力宅的话:** 灰模预演正在变成视频世界模型的控制通道：渲染出的代理视频固定布局、运动和机位，生成器负责填充外观。这个安排只有在代理本身正确时才成立，而按文字简述搭出代理，现在已经属于编程智能体的活。VibePrevisBench 给出 242 条简述，覆盖剧本、照片、续写和参考视频四类输入，要求交付可执行的 Blender 场景以及它渲染出的代理视频。评分分两路：基于规则的 Scene Check 重跑代码，把导出场景的九个维度与隐藏的真值事实对比；基于 VLM 的 Screen Check 判断通过检查的事实能否在视频里被认出来。11 个前沿编程智能体跑下来，三个规律成立：多模态理解把队伍拉开，差距集中在空间与机位运动；基本运镜和摆位人人都会，困难请求几乎全军覆没；智能体会检查自己的渲染，却不修正看到的问题。对生产使用来说，最后一个最贵，因为它意味着闭环在视觉上关上了、在语义上没关上。任何让智能体写场景的工具，都该把“逐条对照简述验证”当作缺失环节，这也正是论文的建议。2026-11-19 核查：Hugging Face 87 个赞；仓库 18 star、0 fork、0 个未关闭 issue，最后推送 2026-11-19。

### 3. GTAGym: AAA Game Tasks Environment for Evaluation and Reinforcement Learning

- **推荐度:** SSS（必读）
- **链接:** [arXiv](https://arxiv.org/abs/2611.13028) / [Hugging Face](https://huggingface.co/papers/2611.13028) / [GitHub](https://github.com/ljbro/GTAGym)
- **念力宅的话:** 商业游戏是好的智能体试验场，却是不好的基准，因为成败通常靠人盯着屏幕判断。GTAGym 把这个判断机械化了。GTAVBench 从 GTA V 主线任务重建出 9 个任务族的 103 个在线任务，再加 47 个基于图像素材的射击与狙击任务，全部经由一个 Web 服务接入：接受截图与键鼠动作，用引擎状态校验结果。六个前沿 LLM 先在文字指令下评测，再在更难的 GTAVBench-AGI 分支上评测，该分支的指令只出现在第一张截图里，历史由框架自动压缩。最好配置达到 70.9%，空中导航、逃脱和拦截仍然预难。真正值得游戏团队细读的是强化学习那一半：生成器在用户给定范围和选项集内、按固定模板采样任务参数，批量产出带自动奖励与终止信号的 3D 和图像任务；用生成的对步行进任务对 Qwen3.8-27B 做 LoRA 强化学习，域内表现提升，同时迁移到域外类别，把原生 GTA V 任务从 15% 推到 23%。23% 这个数字诚实地说出了当下的位置，而从生成任务迁移到人工设计任务，才是这套基础设施站得住的理由。2026-11-19 核查：Hugging Face 14 个赞；仓库 13 star、0 fork、0 个未关闭 issue，最后推送 2026-11-12。

### 4. WorldRuler: Interactive World Modeling under Configurable Rules

- **推荐度:** SS（强烈推荐）
- **链接:** [arXiv](https://arxiv.org/abs/2611.24869) / [Hugging Face](https://huggingface.co/papers/2611.24869) / [GitHub](https://github.com/AlayaLab/WorldRuler) / [项目主页](https://alaya-lab.github.io/WorldRuler/)
- **念力宅的话:** 只复现物理规律的世界模型给出的是步行模拟器。游戏之间靠规则区分，而规则恰恰是生成式视频模型没有表征的部分。WorldRuler 把规则定义、依赖关系和物体状态组织成一张可执行的 3D 规则图，并把规则执行、视频生成、感知反馈连成闭环。两个问题被点名并给出解法。一是规则与状态持久化：规则图与世界状态一起保存和恢复，读档后的世界仍然遵守此前建立的规则。二是规则到视觉的落实：规则效果要作用在还没出现过的物体上，靠动态物体绑定和已存状态约束物体首次出现时的生成。训练环节加入物体属性状态替换与反事实重建，让属性可控，同时限制对无关内容的干扰。RuleVerse 数据集从 GTA V 和 Habitat 采集标注，包含“规则在目标物体可见之前就已生效”这类情况，而这正是多数生成管线会做错的地方。如果正在做带任何游戏逻辑味道的生成式世界原型，这里提出的存档读档要求，会决定设计能不能扛住一次真实的玩家会话。2026-11-19 核查：Hugging Face 20 个赞；仓库 10 star、0 fork、0 个未关闭 issue，最后推送 2026-11-19。

### 5. Game2Player: Scaling Gameplay Experience for Visual Agents through Environment-Player Co-Development

- **推荐度:** SS（强烈推荐）
- **链接:** [arXiv](https://arxiv.org/abs/2611.24154) / [Hugging Face](https://huggingface.co/papers/2611.24154) / [GitHub](https://github.com/game2player/Game2Player) / [项目主页](https://game2player.github.io/)
- **念力宅的话:** 从视频学打游戏要反推动作，从试玩学打游戏要有能校验的任务。Game2Player 把环境工程塞进数据采集里，两个问题一起解决。建造智能体改造现成的网页游戏、暴露内部状态，并依据状态定义进度度量和成功判定；编程玩家和 VLM 玩家去完成任务，把问题回报给建造者修，于是试玩既是环境验证也是轨迹生产。收获是 568 个已验证的网页游戏、3,436 个任务、15.7K 条轨迹、307 小时游玩，其中 50 个游戏留作 PlayerBench。训练用按进度加权的 SFT，把权重放在真正推动任务的步骤上；Game2Player-9B 在 PlayerBench 上把进度从 14.3% 提到 31.2%，在外部 GameWorld 基准上从 30.0% 提到 53.1%。9B 模型能做到这个区间，说明瓶颈在标签质量和任务结构，不在模型规模，而建造者这个循环，任何工作室都可以对着自己的关卡编辑器搭一遍。2026-11-19 核查：Hugging Face 26 个赞；仓库 24 star、1 fork、3 个未关闭 issue，最后推送 2026-11-19。

### 6. Code World Model 2.0: Towards Infinite Controllable Video Generation

- **推荐度:** SS（强烈推荐）
- **链接:** [arXiv](https://arxiv.org/abs/2611.22925) / [Hugging Face](https://huggingface.co/papers/2611.22925) / [GitHub](https://github.com/inclusionAI/CodeWorldModel) / [项目主页](https://www.inclusion-ai.org/CodeWorldModel/)
- **念力宅的话:** 从演示用的渲染器到一个能摆到玩家面前的渲染器，隔着两个工程问题：代理输入粗细差别极大，流式推理必须几步因果内出结果，而且不能在自生成历史上漂移。Invoker-Render 两个都接。一个双向创作模型和一个四步因果流式模型共享同一套条件接口，输入是结构代理、外观参考和语言，训练语料约 200 小时、多粒度配对。给精细几何就守住几何，给粗代理就自行补出合理的局部运动，应用可以在运行时选择输入细节。漂移交给 Progressive Forcing：先训因果预测器，用训练视频里的历史把它蒸馏成四步渲染器，再转入连续自生成 rollout 继续训练。最值得记住的不是榜单分数，而是 30 分钟赛车 rollout 里场景与主体外观都保持住了，这才是会话级产品在意的量。代理与渲染分离还有一层好处：游戏状态留在代码里，策划能改，而不是关在潜空间里。2026-11-19 核查：Hugging Face 17 个赞；仓库 145 star、4 fork、1 个未关闭 issue，最后推送 2026-11-20。

### 7. WorldDraft: If Code Builds the World, Can Video Models Just Render It?

- **推荐度:** SS（强烈推荐）
- **链接:** [arXiv](https://arxiv.org/abs/2611.16217) / [Hugging Face](https://huggingface.co/papers/2611.16217) / [GitHub](https://github.com/KevinHuge/WorldDraft) / [项目主页](https://haroldchen19.github.io/WorldDraft-site/)
- **念力宅的话:** 一旦由编程智能体写出 4D 世界，视频模型就不再是世界生成器，而是渲染器，渲染器有两种义务：守住规定的运动学与相机轨迹，以及随着时间持续推进文本指定的属性绑定。WorldDraft 就用 432 组严格配对用例测这两件事，覆盖 48 个可执行 4D 世界家族、六类动态挑战，用 3×3 因子设计把草稿粒度与外观对齐方式正交交叉，动力学保持不变。12 个模型的结果里三条结论成立。世界保持与视觉实现互相拉扯。草稿脚手架越丰富，结构越稳，外观还原反而越差，这跟“条件越多越安全”的直觉相反。源线索冲突时会泄漏，参考与草稿矛盾时泄漏率最高到 64%。作者提出的 WorldDraft-Primer 是在冻结渲染器上做视觉提示，两个目标一起改善，不需要重训。对工作室而言，可操作的结论是：调外观参考，别盲目打磨代理，代理打磨过头会赔掉渲染。2026-11-19 核查：Hugging Face 18 个赞；仓库 18 star、1 fork、0 个未关闭 issue，最后推送 2026-11-13。

### 8. FastFrames: Video Generation at Interactive Latency

- **推荐度:** A（值得一读）
- **链接:** [arXiv](https://arxiv.org/abs/2611.23226) / [Hugging Face](https://huggingface.co/papers/2611.23226) / [GitHub](https://github.com/apple-aiml-research/ml-fastframes) / [项目主页](https://apple-aiml-research.github.io/ml-fastframes/)
- **念力宅的话:** 自回归视频扩散是交互式世界模型最自然的底座，现在的速度却离“可交互”差着数量级。FastFrames 先做性能剖析再改架构，发现通常被当成实现细节的 VAE 解码和 KV cache 更新，占了端到端延迟的一半以上。两项都换成轻量替代后，1.3B 模型在单张 B200 上做到每帧 36 毫秒，即每秒 28 帧响应，画质与慢近 8 倍的模型持平；14B 版本在八卡节点上每帧 45 毫秒，真实感明显更好。论文还展示了相机条件与物理条件的实时交互。可迁移的是剖析结论：任何在服务流式世界模型的团队，都在为解码和缓存记账先付一次钱，再为去噪付第二次，而这两项可以不动模型就压下去。2026-11-19 核查：Hugging Face 29 个赞；仓库 79 star、5 fork、0 个未关闭 issue，最后推送 2026-11-18。

### 9. StrideWorld: Benchmarking Scene Rigidity and Action Consistency in Interactive Video World Model

- **推荐度:** A（值得一读）
- **链接:** [arXiv](https://arxiv.org/abs/2611.24408) / [Hugging Face](https://huggingface.co/papers/2611.24408) / [GitHub](https://github.com/chuangwang123/strideworld) / [排行榜](https://huggingface.co/spaces/strideworld-release/strideworld-leaderboard)
- **念力宅的话:** 交互式世界模型的评测，经常拿被测视频自己生成的参考来打分：从自身帧重建场景，或按指令路径拟合一条自由缩放的相机轨迹。这类参考会把逐渐变形吸收掉，指标报成功，世界在镜头背后悄悄换了形状。StrideWorld 让参考在运动发生之前就固定。场景刚性对着起始视角的真值几何来量。动作一致性按步幅来量，即一条指令产生的相机位移，先从模型自己的最初几步测出来，再用它检验后续步骤。140 个起始视角、多个开源模型跑下来，除一个之外所有模型在相机移动时都让场景变形，偏移最高达到同路径刚体渲染的 7.3 倍。单条指令按住时步速还算稳，组合路线就会卡住，多数走完的路线终点偏离模型自身步幅预测的位置。只有两个按估计场景深度缩放指令的模型把路线落回预测附近，这是团队这周就能用上的具体修法；在少量渲染片段上微调，也能消掉大部分中途卡顿。2026-11-19 核查：Hugging Face 13 个赞；仓库 2 star、0 fork、0 个未关闭 issue，最后推送 2026-11-18。

### 10. Actor, Judge, or Teacher? Understanding Decision Models in Long-Horizon Agents

- **推荐度:** A（值得一读）
- **链接:** [arXiv](https://arxiv.org/abs/2611.17167) / [Hugging Face](https://huggingface.co/papers/2611.17167) / [GitHub](https://github.com/ZJU-REAL/decision-agents) / [项目主页](https://zjureal.com/decision-agents/)
- **念力宅的话:** 决策模型输出带概率的类型化选择，于是一个模型可以选动作、评动作，也可以给每个动作打分去训练策略。以往工作都把它当行动者来评判。这篇问的是：行动很差的模型还能不能教。答案会改变智能体管线的分工方式。在 ALFWorld 上，三个决策模型和三个通用 LLM 作为行动者，全都被一个更省事的选择从必须先完成的子目标上拉走；给它们看完整轨迹再当评委，同一批模型又能正确归功于通往子目标的那些步骤，而把提问从“任务会不会成功”换成“这一步有没有帮助”，才把有用动作和无用动作分开。作为教师，JEV 把 Qwen2.5-1.5B 策略在 GRPO 下从 72.8% 提到 82.9%（ALFWorld），从 56.8% 提到 71.4%（WebShop）。这个设计直接适用于游戏智能体：回合结束后的评判能看到行动当下必须预判的后果，所以用评估者做信用分配，往往比改进产出数据的行动者更便宜。2026-11-19 核查：Hugging Face 38 个赞；仓库 49 star、0 fork、0 个未关闭 issue，最后推送 2026-11-14。

### 11. Continuation-Guided LLM Evolution of Program Policies and Goal Compilers in Craftax

- **推荐度:** A（值得一读）
- **链接:** [arXiv](https://arxiv.org/abs/2611.19537) / [Hugging Face](https://huggingface.co/papers/2611.19537) / [GitHub](https://github.com/vicgalle/craftax-program-policies)
- **念力宅的话:** 用代码表达的策略，对上线到游戏里的 AI 有一个诱人之处：推理成本为零，因为循环里不跑模型。代价出现在长程行为上，局部改代码能让回合中段变好、结尾变差。这篇搭了一条反馈级联，从回合回报推进到分层目标再到信念更新，每一层都能从模拟器密集打分。在 Craftax 这个部分可观测、程序生成的 roguelike 里，只用回合级反馈做精修，抵达最终关的比例停在 1% 以下；换成级联之后，50% 的运行能到最终阶段，28% 能打败最终 Boss。更有意思的是后半段的工程结果：控制器被拆成持有模型、记忆和技能的环境解释器，与一段规则程序，然后由编程智能体优化一个目标编译器，把结构化任务映射成新的规则程序。五次只用短任务训练的编译搜索，零样本迁移到长任务，既不做任务特调，也不在推理时调用 LLM。这是一个可复用的产物：每个游戏家族一个便宜、可审计的策略生成器，而不是每个目标训一个模型。2026-11-19 核查：Hugging Face 5 个赞；仓库 1 star、0 fork、0 个未关闭 issue，最后推送 2026-11-08。

## 参考链接

- Gamer arXiv: https://arxiv.org/abs/2611.18397
- Gamer Hugging Face 页面: https://huggingface.co/papers/2611.18397
- Gamer GitHub: https://github.com/RainBowLuoCS/Gamer
- Gamer 项目主页: https://mihoyo-gamer.github.io/
- VibePrevisBench arXiv: https://arxiv.org/abs/2611.15789
- VibePrevisBench Hugging Face 页面: https://huggingface.co/papers/2611.15789
- VibePrevisBench GitHub: https://github.com/vibeprevisbench/vibeprevisbench
- VibePrevisBench 项目主页: https://vibeprevisbench.github.io/
- GTAGym arXiv: https://arxiv.org/abs/2611.13028
- GTAGym Hugging Face 页面: https://huggingface.co/papers/2611.13028
- GTAGym GitHub: https://github.com/ljbro/GTAGym
- WorldRuler arXiv: https://arxiv.org/abs/2611.24869
- WorldRuler Hugging Face 页面: https://huggingface.co/papers/2611.24869
- WorldRuler GitHub: https://github.com/AlayaLab/WorldRuler
- WorldRuler 项目主页: https://alaya-lab.github.io/WorldRuler/
- Game2Player arXiv: https://arxiv.org/abs/2611.24154
- Game2Player Hugging Face 页面: https://huggingface.co/papers/2611.24154
- Game2Player GitHub: https://github.com/game2player/Game2Player
- Game2Player 项目主页: https://game2player.github.io/
- Code World Model 2.0 arXiv: https://arxiv.org/abs/2611.22925
- Code World Model 2.0 Hugging Face 页面: https://huggingface.co/papers/2611.22925
- Code World Model 2.0 GitHub: https://github.com/inclusionAI/CodeWorldModel
- Code World Model 2.0 项目主页: https://www.inclusion-ai.org/CodeWorldModel/
- WorldDraft arXiv: https://arxiv.org/abs/2611.16217
- WorldDraft Hugging Face 页面: https://huggingface.co/papers/2611.16217
- WorldDraft GitHub: https://github.com/KevinHuge/WorldDraft
- WorldDraft 项目主页: https://haroldchen19.github.io/WorldDraft-site/
- FastFrames arXiv: https://arxiv.org/abs/2611.23226
- FastFrames Hugging Face 页面: https://huggingface.co/papers/2611.23226
- FastFrames GitHub: https://github.com/apple-aiml-research/ml-fastframes
- FastFrames 项目主页: https://apple-aiml-research.github.io/ml-fastframes/
- StrideWorld arXiv: https://arxiv.org/abs/2611.24408
- StrideWorld Hugging Face 页面: https://huggingface.co/papers/2611.24408
- StrideWorld GitHub: https://github.com/chuangwang123/strideworld
- StrideWorld 排行榜: https://huggingface.co/spaces/strideworld-release/strideworld-leaderboard
- Actor, Judge, or Teacher? arXiv: https://arxiv.org/abs/2611.17167
- Actor, Judge, or Teacher? Hugging Face 页面: https://huggingface.co/papers/2611.17167
- Actor, Judge, or Teacher? GitHub: https://github.com/ZJU-REAL/decision-agents
- Actor, Judge, or Teacher? 项目主页: https://zjureal.com/decision-agents/
- Continuation-Guided LLM Evolution of Program Policies arXiv: https://arxiv.org/abs/2611.19537
- Continuation-Guided LLM Evolution of Program Policies Hugging Face 页面: https://huggingface.co/papers/2611.19537
- Continuation-Guided LLM Evolution of Program Policies GitHub: https://github.com/vicgalle/craftax-program-policies
