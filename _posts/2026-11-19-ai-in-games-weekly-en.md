---
title: "Vibotaku's AI in Games Weekly: 2026-11-19"
date: 2026-11-19
author: VibOtaku
tags: ai agents game-ai newsletter
lang: en
translation_key: ai-in-games-weekly-2026-11-19
---

**2026-11-12 - 2026-11-19**

## Highlights

- Game agents are being shipped with the serving stack and the data pipeline attached. Gamer trains on tens of thousands of hours of keyboard and mouse input and pairs multi-turn RL with a cloud-edge engine that holds unbounded context, while Game2Player makes an environment builder and its players generate the training set together.
- The split between code that builds a 4D world and a video model that renders it now has benchmarks on both sides. VibePrevisBench grades the Blender scene, WorldDraft grades the render against the scene, and the recurring failure is an agent that looks at its own render and still does not fix it.
- Rules, save states, and object persistence are entering generative world models as explicit structure. WorldRuler stores rule definitions and object states in a graph that survives save and load, which is the difference between a generated video and a world a player can act in.
- Interactive latency is becoming solvable. FastFrames reaches 36 ms per generated frame at 1.3B parameters, and Code World Model 2.0 holds a four-step causal renderer stable across 30-minute driving rollouts.
- Evaluation is moving toward references fixed before the artifact exists: start-view geometry for scene rigidity, re-executed scene code for reconstruction, and engine state for task success inside GTA V.

## Reading recommendations

| Paper | Recommendation Index | Highlight |
| --- | --- | --- |
| Gamer: A Generalist Vision-Language-Action Foundation Model for Long-Horizon Real-Time Game Agent | SSS | End-to-end agent for 3D open-world games, combining keyboard-and-mouse pretraining, sparse thinking, multi-turn RL, and a cloud-edge serving layer built for unbounded context. |
| VibePrevisBench: Evaluating Agentic Coding for 4D Scene Generation in Blender | SSS | 242 briefs turned into executable Blender scenes; 11 coding agents handle basic camera work and stall on hard spatial requests, and none repair errors visible in their own renders. |
| GTAGym: AAA Game Tasks Environment for Evaluation and Reinforcement Learning | SSS | 103 GTA V tasks with engine-state verification plus a task generator for RL, where LoRA training on generated navigation lifts native mission success from 15% to 23%. |
| WorldRuler: Interactive World Modeling under Configurable Rules | SS | An executable 3D rule graph that survives save and load, so rule effects on unseen objects are realized when those objects enter view. |
| Game2Player: Scaling Gameplay Experience for Visual Agents through Environment-Player Co-Development | SS | A builder agent exposes web game state and writes verifiable tasks while players generate the data; 568 games and 3,436 tasks feed progress-weighted SFT that lifts a 9B agent to 31.2% on PlayerBench. |
| Code World Model 2.0: Towards Infinite Controllable Video Generation | SS | Pairs a bidirectional creation model with a four-step causal streaming renderer under one proxy interface, staying stable across 30-minute racing rollouts. |
| WorldDraft: If Code Builds the World, Can Video Models Just Render It? | SS | 432 paired cases separate world preservation from visual realization, and find that richer draft scaffolding anchors geometry while hurting appearance fidelity. |
| FastFrames: Video Generation at Interactive Latency | A | Traces interactive latency to VAE decoding and KV cache updates, then removes both to reach 36 ms per frame at 1.3B parameters on one B200. |
| StrideWorld: Benchmarking Scene Rigidity and Action Consistency in Interactive Video World Model | A | Fixes its references before the motion it judges, then shows that almost every open model deforms its scene as the camera moves. |
| Actor, Judge, or Teacher? Understanding Decision Models in Long-Horizon Agents | A | Decision models that act badly still teach well when asked whether an action helped, and that signal trains a small policy past its own ceiling. |
| Continuation-Guided LLM Evolution of Program Policies and Goal Compilers in Craftax | A | A feedback cascade plus a learned goal compiler produces coded policies for a roguelike that run at zero inference cost and transfer to longer tasks. |

## Detailed Notes

### 1. Gamer: A Generalist Vision-Language-Action Foundation Model for Long-Horizon Real-Time Game Agent

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2611.18397) / [Hugging Face](https://huggingface.co/papers/2611.18397) / [GitHub](https://github.com/RainBowLuoCS/Gamer) / [Project page](https://mihoyo-gamer.github.io/)
- **Vibotaku's Note:** Most game agents are assembled from a vision model, a planner, and a hand-written action layer, and each seam costs latency and loses state. Gamer trains one policy over device-level keyboard and mouse trajectories spanning simulated games and embodied environments, then adds sparse-thinking instruction tuning to cut redundant reasoning and a multi-turn RL stage aimed at the three things that break long sessions: finite context windows, accumulated rollout errors, and sparse rewards. The serving side gets equal weight. A cloud-edge engine hosts many game environments at once, keeps execution going past the context window, and holds online latency low enough for real-time play. The reported result is generalization across several 3D open-world games at operational efficiency comparable to skilled players, with competitive numbers on conventional 2D game benchmarks. Read it for the pipeline shape rather than any single score: pretraining on ordinary input traces, then RL over long interactions, then an inference engine treated as part of the agent rather than an afterthought. That last piece is where most studios will recognize their own bottleneck. Checked 2026-11-19: 140 upvotes on Hugging Face; repository at 1,006 stars, 22 forks, 6 open issues, last push 2026-11-16. The public tree currently holds a `streaming-vllm` inference path and assets alongside the license, so the serving work is available even where training code is not.

### 2. VibePrevisBench: Evaluating Agentic Coding for 4D Scene Generation in Blender

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2611.15789) / [Hugging Face](https://huggingface.co/papers/2611.15789) / [GitHub](https://github.com/vibeprevisbench/vibeprevisbench) / [Project page](https://vibeprevisbench.github.io/)
- **Vibotaku's Note:** Graybox previs is turning into the control channel for video world models: a rendered proxy video fixes layout, motion, and camera, and the generator fills in appearance. That arrangement only works if the proxy is right, and building one from a written brief is now a coding agent job. VibePrevisBench gives the agent 242 briefs across scripts, photos, continuations, and reference videos, and requires an executable Blender scene plus its rendered proxy. Scoring splits into a rule-based Scene Check that re-runs the code and compares nine dimensions of the exported scene against hidden ground truth, and a VLM Screen Check that asks whether the facts that passed are actually visible in the video. Across 11 frontier coding agents, three patterns hold: multimodal understanding separates the field, mostly on space and camera motion; everyone can stage a camera and nobody handles the hard requests; and agents inspect their own renders without correcting what they see. That last one is the expensive failure for production use, because it means the loop closes visually and not semantically. Anyone building a tool that lets an agent author scenes should treat item-by-item verification against the brief as the missing component, which is exactly what the paper recommends. Checked 2026-11-19: 87 upvotes on Hugging Face; repository at 18 stars, 0 forks, 0 open issues, last push 2026-11-19.

### 3. GTAGym: AAA Game Tasks Environment for Evaluation and Reinforcement Learning

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2611.13028) / [Hugging Face](https://huggingface.co/papers/2611.13028) / [GitHub](https://github.com/ljbro/GTAGym)
- **Vibotaku's Note:** Commercial games make good agent testbeds and terrible benchmarks, because success is usually judged by a human watching the screen. GTAGym makes the judgment mechanical. GTAVBench reconstructs 103 online tasks across nine families from GTA V main-story missions, adds 47 image-based shooting and sniping tasks, and runs everything through a web server that accepts screenshots and keyboard-mouse actions while checking outcomes against engine state. Six frontier LLMs are evaluated with text instructions and again on a harder branch where instructions appear only in the first screenshot and history is compressed automatically; the best configuration reaches 70.9%, with aerial navigation, evasion, and interception still failing. The RL half is the part game teams should study: a generator samples task parameters inside user-specified ranges under fixed templates, producing batches of 3D and image tasks with automatic reward and termination, and LoRA RL of Qwen3.8-27B on generated on-foot navigation improves in-domain performance while transferring to out-of-domain categories, moving native GTA V tasks from 15% to 23%. A 23% ceiling is honest about how far this gets today, and the transfer from generated tasks to authored missions is the result that justifies the infrastructure. Checked 2026-11-19: 14 upvotes on Hugging Face; repository at 13 stars, 0 forks, 0 open issues, last push 2026-11-12.

### 4. WorldRuler: Interactive World Modeling under Configurable Rules

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2611.24869) / [Hugging Face](https://huggingface.co/papers/2611.24869) / [GitHub](https://github.com/AlayaLab/WorldRuler) / [Project page](https://alaya-lab.github.io/WorldRuler/)
- **Vibotaku's Note:** A world model that only reproduces physics gives you a walking simulator. Games differ by their rules, and rules are the part generative video models have no representation for. WorldRuler stores rule definitions, dependencies, and object states in an executable 3D rule graph and closes a loop between rule execution, video generation, and perceptual feedback. Two problems get named and solved. Rule-state persistence means the graph saves and restores together with world state, so a loaded session still obeys the rules that were established earlier. Rule-to-visual realization means rule effects are applied to objects that have not appeared yet, with dynamic object binding and stored states constraining generation from an object's first frame. Training adds object attribute state replacement and counterfactual reconstruction, which teaches attribute control without letting unrelated content drift. The RuleVerse dataset is collected and annotated from GTA V and Habitat, including cases where a rule fires before its target is visible, which is the case most generative pipelines get wrong. If you are prototyping a generative world with anything resembling game logic, the save-load requirement here is the one that will decide whether the design survives contact with a player session. Checked 2026-11-19: 20 upvotes on Hugging Face; repository at 10 stars, 0 forks, 0 open issues, last push 2026-11-19.

### 5. Game2Player: Scaling Gameplay Experience for Visual Agents through Environment-Player Co-Development

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2611.24154) / [Hugging Face](https://huggingface.co/papers/2611.24154) / [GitHub](https://github.com/game2player/Game2Player) / [Project page](https://game2player.github.io/)
- **Vibotaku's Note:** Learning to play from videos requires inferring the actions behind them, and learning from play requires tasks with outcomes you can check. Game2Player gets both by making environment engineering part of data collection. A Builder agent adapts existing web games, exposes their internal state, and defines tasks with state-based progress measures and success checks. A Coding Player and a VLM Player then attempt those tasks and report problems back to the Builder, so play validates the environment while it generates trajectories. The harvest is 568 verified web games, 3,436 tasks, 15.7K trajectories, and 307 hours of gameplay, with 50 games held out as PlayerBench. Training uses progress-weighted SFT, which puts more weight on steps that actually advance a task, and Game2Player-9B moves progress from 14.3% to 31.2% on PlayerBench and from 30.0% to 53.1% on the external GameWorld benchmark. A 9B model reaching that range says the bottleneck was label quality and task structure rather than model scale, and the builder loop is a pattern any studio could point at its own level editor. Checked 2026-11-19: 26 upvotes on Hugging Face; repository at 24 stars, 1 fork, 3 open issues, last push 2026-11-19.

### 6. Code World Model 2.0: Towards Infinite Controllable Video Generation

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2611.22925) / [Hugging Face](https://huggingface.co/papers/2611.22925) / [GitHub](https://github.com/inclusionAI/CodeWorldModel) / [Project page](https://www.inclusion-ai.org/CodeWorldModel/)
- **Vibotaku's Note:** Two engineering problems separate a demo renderer from one you can put in front of a player: proxies arrive at wildly different levels of detail, and streaming inference has to run in a handful of causal steps without drifting on its own generated history. Invoker-Render takes both. A bidirectional creation model shares a conditioning interface with a causal four-step streaming model, over structural proxies, an appearance reference, and language, trained on about 200 hours of paired multi-granularity data. Supplied with fine geometry, the renderer respects it; supplied with a coarse proxy, it invents plausible local motion, so the application can choose input detail at runtime. Progressive Forcing handles the drift: train a causal predictor, distill it into the four-step renderer using histories taken from training video, then continue training on continuous self-generated rollouts. The headline number is a 30-minute racing rollout that keeps scene and subject appearance intact, which is a different kind of claim from a benchmark score and the one that matters for a session-length product. The proxy-renderer split also keeps game state in code, where designers can edit it, instead of inside latent space. Checked 2026-11-19: 17 upvotes on Hugging Face; repository at 145 stars, 4 forks, 1 open issue, last push 2026-11-20.

### 7. WorldDraft: If Code Builds the World, Can Video Models Just Render It?

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2611.16217) / [Hugging Face](https://huggingface.co/papers/2611.16217) / [GitHub](https://github.com/KevinHuge/WorldDraft) / [Project page](https://haroldchen19.github.io/WorldDraft-site/)
- **Vibotaku's Note:** Once a coding agent authors the 4D world, the video model is no longer the world generator. It is a renderer, and a renderer has two obligations: keep the prescribed kinematics and camera path, and keep binding the attributes the text asked for as time passes. WorldDraft tests exactly that with 432 strictly paired cases across 48 executable 4D world families and six dynamic categories, laid out as a 3x3 design that crosses draft granularity with appearance alignment while holding dynamics fixed. Across 12 models, three findings land. World preservation and visual realization pull against each other. Richer draft scaffolding anchors structure while degrading appearance fidelity, which is the opposite of the intuition that more conditioning is safer. Conflicting source cues leak, with source leakage up to 64% when the reference contradicts the draft. The proposed fix, WorldDraft-Primer, is visual prompting applied to frozen renderers, and it improves both objectives without retraining. For a studio, the practical takeaway is that the appearance reference is the lever to tune and the draft is not free: polishing the proxy can cost you the render. Checked 2026-11-19: 18 upvotes on Hugging Face; repository at 18 stars, 1 fork, 0 open issues, last push 2026-11-13.

### 8. FastFrames: Video Generation at Interactive Latency

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2611.23226) / [Hugging Face](https://huggingface.co/papers/2611.23226) / [GitHub](https://github.com/apple-aiml-research/ml-fastframes) / [Project page](https://apple-aiml-research.github.io/ml-fastframes/)
- **Vibotaku's Note:** Autoregressive video diffusion is the natural substrate for an interactive world model and currently far too slow to interact with. FastFrames starts with a profile instead of an architecture, and finds that operations usually treated as fixed implementation details, VAE decoding and KV cache updates, account for more than half of end-to-end latency. Replacing both with lightweight alternatives gets a 1.3B model to 36 ms per frame, 28 responses per second, on a single B200, at quality matching a model nearly 8x slower; a 14B variant runs at 45 ms per frame on an eight-GPU node with visibly better realism. The paper also shows camera-conditioned and physics-conditioned interaction running in real time. The transferable result is the profiling one. Any team serving a streaming world model is paying for decode and cache bookkeeping before it pays for denoising, and those costs shrink without touching the model. Checked 2026-11-19: 29 upvotes on Hugging Face; repository at 79 stars, 5 forks, 0 open issues, last push 2026-11-18.

### 9. StrideWorld: Benchmarking Scene Rigidity and Action Consistency in Interactive Video World Model

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2611.24408) / [Hugging Face](https://huggingface.co/papers/2611.24408) / [GitHub](https://github.com/chuangwang123/strideworld) / [Leaderboard](https://huggingface.co/spaces/strideworld-release/strideworld-leaderboard)
- **Vibotaku's Note:** Benchmarks for interactive world models often grade a video against a reference built from that same video, a scene reconstructed from its own frames or a camera path fitted with a free scale. Those references absorb gradual deformation, so the metric reports success while the world quietly changes shape behind the camera. StrideWorld fixes references before the motion they judge. Scene rigidity is measured against the ground-truth geometry of the start view. Action consistency is measured as stride, the camera motion produced by one command, taken from the model's own first steps and used to test the steps that follow. Across 140 start views and open models, every model but one deforms its scene as the camera moves, with offsets up to 7.3 times those of a rigid rendering along the same path. Single held actions keep a steady pace, composed routes stall, and most completed routes end away from where the model's own strides predicted. The two models that scale commands by estimated scene depth land close to those predictions, which is a concrete fix a team can apply this week, and fine-tuning on a few rendered clips removes most mid-route stalls. Checked 2026-11-19: 13 upvotes on Hugging Face; repository at 2 stars, 0 forks, 0 open issues, last push 2026-11-18.

### 10. Actor, Judge, or Teacher? Understanding Decision Models in Long-Horizon Agents

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2611.17167) / [Hugging Face](https://huggingface.co/papers/2611.17167) / [GitHub](https://github.com/ZJU-REAL/decision-agents) / [Project page](https://zjureal.com/decision-agents/)
- **Vibotaku's Note:** Decision models output a typed choice with probabilities, which means one model can pick an action, grade an action, or score every action to train a policy. Prior work judges them as actors. This paper asks whether a model that acts badly can still teach, and the answer changes how you would staff an agent pipeline. On ALFWorld, three decision models and three general LLMs all get pulled off a prerequisite subgoal by a cheaper alternative. Given the full trajectory, those same models credit the steps toward the subgoal correctly, and the question that sorts useful actions from useless ones is whether an action helped rather than whether the task will succeed. As a teacher, JEV raises a Qwen2.5-1.5B policy under GRPO from 72.8% to 82.9% on ALFWorld and from 56.8% to 71.4% on WebShop. The design lesson carries to game agents directly: post-episode judging sees consequences that in-episode acting has to anticipate, so credit assignment from an evaluator can be cheaper than improving the actor that generates the data. Checked 2026-11-19: 38 upvotes on Hugging Face; repository at 49 stars, 0 forks, 0 open issues, last push 2026-11-14.

### 11. Continuation-Guided LLM Evolution of Program Policies and Goal Compilers in Craftax

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2611.19537) / [Hugging Face](https://huggingface.co/papers/2611.19537) / [GitHub](https://github.com/vicgalle/craftax-program-policies)
- **Vibotaku's Note:** A policy expressed as code has an appealing property for shipped game AI: it costs nothing at inference because no model runs inside the loop. The catch is long-horizon behavior, where local code edits improve the middle of an episode and hurt the end. This paper builds a feedback cascade that moves from episode returns to hierarchical goals to belief updates, each scored densely from the simulator, and in Craftax, a partially observed procedurally generated roguelike, refinement on episode-level feedback alone stalls below 1% final-level arrival while the cascade reaches the final stage in 50% of runs with 28% boss defeats. The second half is the more interesting engineering result. The controller factors into an environment interpreter holding models, memory, and skills, and a rule program, and coding agents then optimize a goal compiler that maps a structured task into a new rule program. Five compiler searches trained only on short tasks transfer zero-shot to longer tasks with no task-specific tuning and no inference-time LLM calls. That is a reusable artifact: one cheap, auditable policy generator per game family instead of one trained model per objective. Checked 2026-11-19: 5 upvotes on Hugging Face; repository at 1 star, 0 forks, 0 open issues, last push 2026-11-08.

## References

- Gamer arXiv: https://arxiv.org/abs/2611.18397
- Gamer Hugging Face page: https://huggingface.co/papers/2611.18397
- Gamer GitHub: https://github.com/RainBowLuoCS/Gamer
- Gamer project page: https://mihoyo-gamer.github.io/
- VibePrevisBench arXiv: https://arxiv.org/abs/2611.15789
- VibePrevisBench Hugging Face page: https://huggingface.co/papers/2611.15789
- VibePrevisBench GitHub: https://github.com/vibeprevisbench/vibeprevisbench
- VibePrevisBench project page: https://vibeprevisbench.github.io/
- GTAGym arXiv: https://arxiv.org/abs/2611.13028
- GTAGym Hugging Face page: https://huggingface.co/papers/2611.13028
- GTAGym GitHub: https://github.com/ljbro/GTAGym
- WorldRuler arXiv: https://arxiv.org/abs/2611.24869
- WorldRuler Hugging Face page: https://huggingface.co/papers/2611.24869
- WorldRuler GitHub: https://github.com/AlayaLab/WorldRuler
- WorldRuler project page: https://alaya-lab.github.io/WorldRuler/
- Game2Player arXiv: https://arxiv.org/abs/2611.24154
- Game2Player Hugging Face page: https://huggingface.co/papers/2611.24154
- Game2Player GitHub: https://github.com/game2player/Game2Player
- Game2Player project page: https://game2player.github.io/
- Code World Model 2.0 arXiv: https://arxiv.org/abs/2611.22925
- Code World Model 2.0 Hugging Face page: https://huggingface.co/papers/2611.22925
- Code World Model 2.0 GitHub: https://github.com/inclusionAI/CodeWorldModel
- Code World Model 2.0 project page: https://www.inclusion-ai.org/CodeWorldModel/
- WorldDraft arXiv: https://arxiv.org/abs/2611.16217
- WorldDraft Hugging Face page: https://huggingface.co/papers/2611.16217
- WorldDraft GitHub: https://github.com/KevinHuge/WorldDraft
- WorldDraft project page: https://haroldchen19.github.io/WorldDraft-site/
- FastFrames arXiv: https://arxiv.org/abs/2611.23226
- FastFrames Hugging Face page: https://huggingface.co/papers/2611.23226
- FastFrames GitHub: https://github.com/apple-aiml-research/ml-fastframes
- FastFrames project page: https://apple-aiml-research.github.io/ml-fastframes/
- StrideWorld arXiv: https://arxiv.org/abs/2611.24408
- StrideWorld Hugging Face page: https://huggingface.co/papers/2611.24408
- StrideWorld GitHub: https://github.com/chuangwang123/strideworld
- StrideWorld leaderboard: https://huggingface.co/spaces/strideworld-release/strideworld-leaderboard
- Actor, Judge, or Teacher? arXiv: https://arxiv.org/abs/2611.17167
- Actor, Judge, or Teacher? Hugging Face page: https://huggingface.co/papers/2611.17167
- Actor, Judge, or Teacher? GitHub: https://github.com/ZJU-REAL/decision-agents
- Actor, Judge, or Teacher? project page: https://zjureal.com/decision-agents/
- Continuation-Guided LLM Evolution of Program Policies arXiv: https://arxiv.org/abs/2611.19537
- Continuation-Guided LLM Evolution of Program Policies Hugging Face page: https://huggingface.co/papers/2611.19537
- Continuation-Guided LLM Evolution of Program Policies GitHub: https://github.com/vicgalle/craftax-program-policies
