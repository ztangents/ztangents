---
title: "念力宅的游戏AI周报：2026-09-28"
date: 2026-09-28
author: VibOtaku
tags: ai agents game-ai newsletter
lang: zh
translation_key: ai-in-games-weekly-2026-09-28
---

**2026-09-21 - 2026-09-28**

## 本周精彩研究

- 游戏评测体系正在被重新设计，以便在模型持续进步后依然有效。GameHorizon 发布了 5,000 小时带标注的 3A 游戏数据，三层指令金字塔配合分步在线评测轨道；Kaggle 的 Game Arena 则用国际象棋、德州扑克和狼人杀做模型对战，让排名不会因为模型变强而失去区分度。
- 规则保真度是生成式世界缺失的那一层。GameDirector 把机制、状态和 NPC 战术交给一个 agent 导演模块，像素渲染仍由视频世界模型负责，在同一批游戏上报告 99.6% 的机制保真度，端到端基线只有 21–34%。
- 世界模型研究正在向物理先验与可用记忆收敛。物体恒存性与固体性获得 150 个 Blender 生成器组成的课程和 300 题考卷；WorldCrafter 则把多视角历史压缩成由目标视角决定的记忆 token，支撑分钟级的持续探索。
- 线上运营的游戏智能体已经有真实数据。PUBG Ally 与真实玩家收集了近 3.9 万场对局数据，调查覆盖 141 个国家，并完整记录了模型压缩、运行时护栏、记忆脱敏等上线前的必备工程。
- 记忆与环境设计是长期任务智能体的瓶颈。在查询时整理记忆的做法在三个智能体基准上都优于写入时蒸馏的方案；新的评测开始考核持久世界中的协作与记忆，而不再只看单任务成功率。

## 推荐阅读

| 文章 | 推荐度 | 重点 |
| --- | --- | --- |
| GameHorizon Suite: Multi-Horizon Data and Evaluation in Gameplay | SSS | 发布 5,000 小时带多时间跨度指令的 3A 游戏数据，并提供可复现的离线与分步在线评测。 |
| Training Object Permanence in World Models | SSS | 用 150 个 Blender 任务生成器覆盖物体恒存性与固体性，发布 150 万条训练语料与 300 题考卷，训出的 16B 模型在其接口类别中排名第一。 |
| PUBG Ally: A Conversational Embodied Agent as an AI Teammate | SSS | 记录了一个已上线的语音 LLM 队友，包含 3.9 万场真实对局训练数据与 141 个国家的玩家调查结果。 |
| WorldCrafter: Consistent Video World Model with Implicit 3D-aware Memory | SS | 把长期历史存成可被相机查询的记忆 token，让流式探索在回访与视角切换后依然保持一致。 |
| GameDirector: Decoupling Gameplay Logic from Rendering for Player-Configurable Game World Models | SS | 把规则与状态转移从视频生成器中移出，交给导演智能体，在三款游戏上达到 99.6% 机制保真度。 |
| Game Arena: Strategic LLM Evaluation in Competitive Environments | SS | Kaggle 的平台让前沿模型在象棋、扑克与狼人杀中对战，扑克部分跑了 90 万手牌并使用 Elo 式评分。 |
| HappyWorld-Bench | SS | 用视频、空间、具身三条轨道加人类 A/B Elo 评估世界模型，指出生成世界在交互中失效的位置。 |
| AgentWorld: Benchmarking Long-Horizon Collaboration of Multi-agent LLMs | SS | 在 MMORPG 沙盒中运行 100 个标注任务，3–20 个角色能力不对称，并提出衡量团队实际有效贡献的因果指标。 |
| Just-in-Time Memory: Learning to Curate Task-Adaptive Memory for LLM Agents | A | 保留原始轨迹、在查询时整理记忆，在 ALFWorld、WebShop 与 τ²-bench 上超过写入时记忆方案。 |
| Synthesizing Reactive Character Behaviors for Continuous Games via Programmatic Policy Search | A | 搜索可读可改的程序来驱动游戏角色，由编程智能体提出结构、枚举器补全细节。 |
| Unity Insight: A Production Code--Asset Index for LLM Coding Agents in Unity Projects | A | 为 Unity 工程建立代码与资源联合索引，在项目专属问题上把智能体 token 消耗降低 53%、耗时降低 52%。 |

## 文章小记

### 1. GameHorizon Suite: Multi-Horizon Data and Evaluation in Gameplay

- **推荐度：** SSS（必读）
- **链接：** [arXiv](https://arxiv.org/abs/2609.25001) / [Hugging Face](https://huggingface.co/papers/2609.25001) / [GitHub](https://github.com/TencentARC/GameHorizon) / [项目主页](https://gamehorizon-suite.github.io/)
- **念力宅的笔记：** 游戏评测常见的做法是每款游戏只跑几次回合，这样的对比噪声很大，也看不出长目标到底在哪一步失败。GameHorizon 直接把这些基础设施建设好：数据集包含 100 位资深玩家在 21 款 3A 游戏中的 5,000 小时录像，4,571 段 60fps 视频、4.11 亿次键鼠事件，平均每 2.63 秒就有一条新的短时间跨度指令。指令按三层金字塔组织，分别对应 1–5 秒的操作、1–2 分钟的目标和 5–8 分钟的策略，总计 618 万条标注。评测分为两条轨道：离线轨道有三个主任务，每个 1,000 道选择题，另有十个各 200 题的诊断变体；在线轨道有 10 个因果任务和 10 个主题任务，拆成 62 个可验证子任务，每个子任务最多 400 次模型调用，因此失败可以定位到具体步骤。44 个具备问答能力的模型平均正确率 64.7%，随机基线是 25%，分数区间从 44.6% 到 80.2%，难度梯度清晰：动作预测 57.3%、目标分解 65.1%、跨时间跨度一致性 71.6%。给纯视觉模型补上中长跨度指令后，未来动作规划提升 7.2 个百分点，这也是把指令留在数据里、而不是用逆动力学模型反推动作的最有力论据。核查时间 2026-09-28：Hugging Face 130 个赞；代码仓库 328 stars、1 fork、1 个 open issue，最近推送 2026-09-22。

### 2. Training Object Permanence in World Models

- **推荐度：** SSS（必读）
- **链接：** [arXiv](https://arxiv.org/abs/2609.28654) / [Hugging Face](https://huggingface.co/papers/2609.28654) / [GitHub](https://github.com/hokindeng/object-permanence) / [项目主页](https://www.object-permanence.world/)
- **念力宅的笔记：** 生成出来的世界仍然会以玩家一眼就能发现的方式崩坏：物体被遮挡后消失，回来时出现在不可能的位置，或者直接穿过墙壁。WROP 把这两种行为——物体恒存性和固体性——当作世界模型其他能力的基础，因为允许物体互相穿透的模型根本渲染不出有效的碰撞或支撑移除。构建方式刻意贴合认知科学：150 个手写 Blender 生成器分属六个任务族，速度、光照、相机角度等参数随机化，任务结构保持不变，每个生成器产出 1 万条样本，合计 150 万条训练语料。评测使用固定 300 题考卷和 20 位评审的盲测成对比较，作者训练的 16B 续写模型 PWM-WROP 在续写类模型中以 Elo 1679.5 排名第一，仅次于两个商用参考图生视频系统之间统计上的平局（1723.6）。数据、考卷、模型答案、权重和原生 PyTorch 训练栈均已发布，整套方案可以复现，而不只是被报告。核查时间 2026-09-28：Hugging Face 209 个赞；代码仓库 271 stars、7 forks、0 个 open issue，最近推送 2026-09-26。

### 3. PUBG Ally: A Conversational Embodied Agent as an AI Teammate

- **推荐度：** SSS（必读）
- **链接：** [arXiv](https://arxiv.org/abs/2609.29837) / [Hugging Face](https://huggingface.co/papers/2609.29837)
- **念力宅的笔记：** 这是少见的讲上线而不是讲演示的游戏智能体报告。Ally 在 PUBG: BATTLEGROUNDS 中作为语音队友和玩家一起作战，这要求双速架构：语言模型智能体通过受控接口读取战场信息、理解玩家语音、维护上下文，并给出高层行动选择；更快的一层控制模块负责移动、交战与恢复。语音必须与动作同步，每个决策还要落在实时对局的延迟预算内。团队收集了近 3.9 万场真实玩家与 Ally 同场对局的数据，发现离线评测与玩家偏好经常不一致，评测标准本身需要靠玩家反馈和偏好比较反复修订。部署部分的价值在于范围完整：模型压缩、上下文压缩、面向安全性的定向训练、运行时护栏和记忆脱敏，都是面向玩家智能体上线前的必备条件。在 141 个国家的被调查玩家中，经对局记录确认真正与 Ally 同场游玩过的受访者里，愿意推荐 Ally 的正向反馈比负向反馈高出 25.1 个百分点。核查时间 2026-09-28：Hugging Face 11 个赞；未列出公开仓库或项目主页。

### 4. WorldCrafter: Consistent Video World Model with Implicit 3D-aware Memory

- **推荐度：** SS（强烈推荐）
- **链接：** [arXiv](https://arxiv.org/abs/2609.24984) / [Hugging Face](https://huggingface.co/papers/2609.24984) / [GitHub](https://github.com/TencentARC/WorldCrafter) / [项目主页](https://drexubery.github.io/WorldCrafter)
- **念力宅的笔记：** 流式生成的世界，记忆预算是以 token 计的，这笔预算怎么花，决定了场景换个角度回看时还能不能站得住。WorldCrafter 学习了一种可被相机查询的隐式 3D 记忆：目标视角决定多视角证据如何被压缩进固定数量、与该视角对应的 token，整个流程不依赖显式深度对应关系。记忆编码器与位姿条件读出模块和视频生成器联合训练，再把这份记忆与近期时序上下文、少步蒸馏结合，就能从单张图片或一段文字提示开始做流式场景探索。收益体现在分钟级探索中的长期一致性与相机控制精度，静态和动态场景都有效。对交互式世界来说，这件事的通俗版本就是：玩家转身走开再回来，房间还是那个房间。核查时间 2026-09-28：Hugging Face 156 个赞；代码仓库 408 stars、11 forks、2 个 open issue，最近推送 2026-09-28。

### 5. GameDirector: Decoupling Gameplay Logic from Rendering for Player-Configurable Game World Models

- **推荐度：** SS（强烈推荐）
- **链接：** [arXiv](https://arxiv.org/abs/2609.25652) / [项目主页](https://jimntu.github.io/gamedirector/)
- **念力宅的笔记：** 扣血、技能释放、战斗结算、终止条件，全部依赖精确的状态转移，而这正是端到端游戏世界模型容易出事的地方：像素不是执行规则的好场所。GameDirector 把问题拆开，让一个 agent 导演模块解读画面观察、更新游戏状态、战术性地控制 NPC 并执行游戏规则，再把这些决策翻译成文字提示，交给视频世界模型渲染。玩家可以像开发者那样配置角色、初始状态和规则。在三款商业作品、13 个角色上，框架报告 99.6% 的机制保真度，端到端基线为 21–34%，Boss 的决策质量相对最强基线提升 39.9%。这个设计也指向一个面向创作者的产品形态：世界由玩家配置，硬规则保持成立，视觉层保持生成式。核查时间 2026-09-28：项目主页 `jimntu.github.io/gamedirector` 可访问；未列出 Hugging Face 论文页或代码仓库。

### 6. Game Arena: Strategic LLM Evaluation in Competitive Environments

- **推荐度：** SS（强烈推荐）
- **链接：** [arXiv](https://arxiv.org/abs/2609.31473) / [Hugging Face](https://huggingface.co/papers/2609.31473) / [项目主页](https://www.kaggle.com/game-arena)
- **念力宅的笔记：** 静态基准会随着模型变强而失效，游戏天梯则相反：对手随模型一起变强，天花板一直在动。Kaggle 的 Game Arena 首批开放三个环境，按信息结构挑选：国际象棋是完全信息，单挑无限注德州扑克是不完全信息与信念更新，8 人狼人杀则是带自然语言交流的欺骗与社会推理。排名方式分别为象棋使用 Bradley-Terry 拟合的 Elo 式评分、扑克使用每百手盈利大盲（BB/100）、狼人杀使用博弈论评估。首轮比赛有来自五家前沿实验室的十个模型参加，扑克部分共 90 万手牌，每个模型 18 万手，比早先的扑克基准高出两个数量级。o3 在 9 组对局中赢或平了 8 组，策略分析显示 Grok 4 在小盲位高达 95.0% 的抢盲频率与其 +27.1 BB/100 相匹配。对需要为游戏智能体选型的团队来说，这里的价值在方法论：足够多的对局保证区分度，以及一套可以持续加入新游戏的机制。核查时间 2026-09-28：Hugging Face 4 个赞；项目主页 kaggle.com/game-arena 可访问。

### 7. HappyWorld-Bench

- **推荐度：** SS（强烈推荐）
- **链接：** [arXiv](https://arxiv.org/abs/2609.24308) / [Hugging Face](https://huggingface.co/papers/2609.24308) / [GitHub](https://github.com/LivingFutureLab/HappyWorldBench)
- **念力宅的笔记：** 生成世界最容易测的是画面质量，最能说明能不能玩的恰恰不是它。HappyWorld-Bench 围绕六项世界能力（生成式构建到统一世界建模）组织评测，并落到三条轨道：视频世界模型 1,138 条提示、空间世界模型 300 个场景、具身模型 254 个测试用例，用 A/B 竞技场获得人类 Elo 评分，同时给出衡量行为正确性的自动化指标。结果是一份生成世界在交互中失效的地图：视频模型在长回合与回访中一致性下滑，空间系统最多只有 70.14% 的放置准确率和 73.33% 的编辑执行率，具身候选模型难以在多步动作中保持状态，也难以在动作条件与物理规则改变后正确响应。要做交互式功能选型的团队正好需要这组数字，因为它们描述的正是玩家在最初几秒的惊艳之后会看到的东西。核查时间 2026-09-28：Hugging Face 51 个赞；代码仓库 9 stars、0 forks、0 个 open issue，最近推送 2026-09-21。

### 8. AgentWorld: Benchmarking Long-Horizon Collaboration of Multi-agent LLMs

- **推荐度：** SS（强烈推荐）
- **链接：** [arXiv](https://arxiv.org/abs/2609.31590) / [Hugging Face](https://huggingface.co/papers/2609.31590) / [项目主页](https://agentworld.io/)
- **念力宅的笔记：** 多智能体评测通常要么让模型互相对打，要么把个人成绩平均之后称之为合作。AgentWorld 在 MMORPG 沙盒里问的是另一个问题：100 个人工标注任务加 100 个增广变体，50 轮以上交互，3–20 个能力和角色都不对称的智能体，团队的努力到底有多少真正推动了结果？每个智能体独立行动，看不到彼此的内部状态，协作只能通过沟通、联合规划和资源共享发生。作者提出的 Causal Collaboration Effectiveness 指标追踪动作之间的因果依赖，衡量有多少投入真正起了作用。四个前沿模型中最高的任务成功率只有 52.0%，失败模式对任何做过同伴或小队 AI 的人来说都很眼熟：沟通中断、角色混乱、无法在多轮之间维持共享计划。多个角色长期共存的世界，正是这些失败会出现在玩家游戏里的场景。核查时间 2026-09-28：Hugging Face 6 个赞；项目主页 agentworld.io 可访问；论文被 COLM 2026 接收。

### 9. Just-in-Time Memory: Learning to Curate Task-Adaptive Memory for LLM Agents

- **推荐度：** A（推荐阅读）
- **链接：** [arXiv](https://arxiv.org/abs/2609.27334) / [Hugging Face](https://huggingface.co/papers/2609.27334)
- **念力宅的笔记：** 记忆系统通常在写入时就决定什么值得保留，把一条已完成的轨迹蒸馏成反思、工作流或技能，之后再按相似度检索。这意味着必须在未来查询出现之前做判断，训练也难以进行，因为一次存储决策的价值可能要到很多个任务之后才显现。JitMem 把顺序倒过来：保留原始轨迹，在读取时、也就是当前任务已知时再做整理，从检索到的轨迹中合成一份贴合当前需求的精简内容。因为这份内容就在同一个任务里被使用，整理器可以直接用即时的任务成败来训练。在 ALFWorld、WebShop 和 τ²-bench 上，它相对最强基线分别提升 16.2、16.3 和 3.9 个绝对成功率点，而且未经训练的整理器就已能与学习式写入时系统相比，说明读取时整理的框架本身贡献了大部分收益。对经验横跨各种异质情境的 NPC 或同伴智能体来说，把整理推迟到需要时再做，可以避免永久性地压掉只在某一种情境里重要的细节。核查时间 2026-09-28：Hugging Face 47 个赞；未列出代码仓库或项目主页。

### 10. Synthesizing Reactive Character Behaviors for Continuous Games via Programmatic Policy Search

- **推荐度：** A（推荐阅读）
- **链接：** [arXiv](https://arxiv.org/abs/2609.24025)
- **念力宅的笔记：** 上线游戏里的角色行为仍大量依靠手写的行为树、状态机和脚本，而强化学习往往产出训练成本高、又不好修改的控制器。这项工作直接在一种面向连续空间游戏策略的领域特定语言上搜索，语言围绕反应式几何决策设计，包含方向最大化这类高阶构造，把连续行为空间离散成可枚举的程序结构，同时用一组合成反模式剪掉冗余形式而不损失行为覆盖。搜索把自底向上的符号枚举与编程智能体的自上而下引导结合起来：智能体提出高层策略结构，再调用枚举器补全局部槽位，作者称之为 agentic sketching。在覆盖 14 款连续类游戏（从经典控制任务到多智能体足球）的基准上，纯粹枚举常常比单用编程智能体更高效，而两者结合明显优于任一单独方式。产出是设计师可以阅读和修改的程序，这也是程序化策略搜索能成为创作工具、而不是研究产物的关键。论文将在 SIGGRAPH Asia 2026 发表。核查时间 2026-09-28：未列出 Hugging Face 论文页或代码仓库。

### 11. Unity Insight: A Production Code--Asset Index for LLM Coding Agents in Unity Projects

- **推荐度：** A（推荐阅读）
- **链接：** [arXiv](https://arxiv.org/abs/2609.27585)
- **念力宅的笔记：** 在游戏引擎仓库里，真正重要的关系往往在代码之外：一次玩法改动可能同时涉及 C# 脚本、prefab、场景和 ScriptableObject，它们通过存在 `.meta` 文件和 YAML 资源里的 Unity GUID 连接。Shell 工具和只索引代码的方案回答不了随之而来的跨文件问题，智能体只能把上下文花在重新发现项目本来就知道的结构上。Unity Insight 在代码与资源联合图上建立持久、与智能体集成的索引，自 2026-07-28 起已随 Tuanjie 引擎的智能体 CLI 产品 Tuanjie Codely 上线。在两款 Unity 游戏的 28 个项目专属问题上、使用同一模型与 harness 的配对实验中，有索引的智能体比通用探索智能体少消耗 53% 的 token、少花 52% 的墙钟时间，配对符号检验 p < 0.004。检索工具是否贴合引擎工程约定，很大程度上决定了智能体辅助能否在生产级游戏代码库里真正落地。核查时间 2026-09-28：未列出 Hugging Face 论文页或公开代码仓库。

## 参考链接

- GameHorizon Suite arXiv：https://arxiv.org/abs/2609.25001
- GameHorizon Suite Hugging Face 页面：https://huggingface.co/papers/2609.25001
- GameHorizon Suite GitHub：https://github.com/TencentARC/GameHorizon
- GameHorizon Suite 项目主页：https://gamehorizon-suite.github.io/
- Training Object Permanence in World Models arXiv：https://arxiv.org/abs/2609.28654
- Training Object Permanence in World Models Hugging Face 页面：https://huggingface.co/papers/2609.28654
- Training Object Permanence in World Models GitHub：https://github.com/hokindeng/object-permanence
- Training Object Permanence in World Models 项目主页：https://www.object-permanence.world/
- PUBG Ally arXiv：https://arxiv.org/abs/2609.29837
- PUBG Ally Hugging Face 页面：https://huggingface.co/papers/2609.29837
- WorldCrafter arXiv：https://arxiv.org/abs/2609.24984
- WorldCrafter Hugging Face 页面：https://huggingface.co/papers/2609.24984
- WorldCrafter GitHub：https://github.com/TencentARC/WorldCrafter
- WorldCrafter 项目主页：https://drexubery.github.io/WorldCrafter
- GameDirector arXiv：https://arxiv.org/abs/2609.25652
- GameDirector 项目主页：https://jimntu.github.io/gamedirector/
- Game Arena arXiv：https://arxiv.org/abs/2609.31473
- Game Arena Hugging Face 页面：https://huggingface.co/papers/2609.31473
- Game Arena 项目主页：https://www.kaggle.com/game-arena
- HappyWorld-Bench arXiv：https://arxiv.org/abs/2609.24308
- HappyWorld-Bench Hugging Face 页面：https://huggingface.co/papers/2609.24308
- HappyWorld-Bench GitHub：https://github.com/LivingFutureLab/HappyWorldBench
- AgentWorld arXiv：https://arxiv.org/abs/2609.31590
- AgentWorld Hugging Face 页面：https://huggingface.co/papers/2609.31590
- AgentWorld 项目主页：https://agentworld.io/
- Just-in-Time Memory arXiv：https://arxiv.org/abs/2609.27334
- Just-in-Time Memory Hugging Face 页面：https://huggingface.co/papers/2609.27334
- Synthesizing Reactive Character Behaviors arXiv：https://arxiv.org/abs/2609.24025
- Unity Insight arXiv：https://arxiv.org/abs/2609.27585
