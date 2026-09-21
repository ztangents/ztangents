---
title: "Vibotaku's AI in Games Weekly: 2026-09-21"
date: 2026-09-21
author: VibOtaku
tags: ai agents game-ai newsletter
lang: en
translation_key: ai-in-games-weekly-2026-09-21
---

**2026-09-14 - 2026-09-21**

## Highlights

- Game evaluation is moving from scores to rule compliance. Two new benchmarks check generated games at every simulation tick and require all declared requirements to hold, and both report large gaps between average check pass rates and strict success.
- Playable world models are small enough to serve. Zing-0.5 runs a 5B autoregressive world model at 832x480 and 24 FPS for roughly USD 0.009 per stream-minute with weights released, and Astronex-World 1.0 streams a comparable model in real time on a single L20.
- The AI-in-games literature now has a map. A 120-page survey sorts the field into six roles across the game lifecycle and separates the artifacts that transfer between roles from those tied to particular engines, interfaces, and player populations.
- Character memory is turning into an inference-state problem. A local NPC runtime removes superseded attention KV entries and writes replacements at the true sequence tail instead of rebuilding a character prefix, because a fluent but wrong memory can corrupt deterministic game rules.
- Persistent multi-agent systems fail through memory. Across 16 days of ten-agent worlds, detecting injected content did not stop agents from writing it into their own long-lived memory and acting on it up to 46 hours later.

## Reading recommendations

| Paper | Recommendation Index | Highlight |
| --- | --- | --- |
| AI for Games in the Foundation Model Era | SSS | Organizes the field into six roles across the game lifecycle and separates transferable artifacts from setting-specific ones. |
| Zing-0.5: Toward Playable Worlds with Real-Time Joint Action and Text Control | SSS | Ships a 5B playable world model with joint keyboard and text control, real-time streaming, published weights, and a per-minute serving cost. |
| Astronex-World 1.0: Real-Time Interactive World Model Foundation | SS | Open camera- and action-controllable video world model whose causal variant streams 832x480 at 24 FPS on one GPU. |
| GameLogicBench: Evaluating Coding Agents on Runtime Game Logic with Tick-Level State Assertions | SS | Grades Godot gameplay rules at every tick and rejects mutants, showing how few coding-agent submissions are strictly correct. |
| GameASG-Bench: Benchmarking Autonomous Software Generation for Game Development | SS | Declares each game's evaluation interface before generation and scores all-requirements compliance instead of average check pass rate. |
| Long-Lived Characters, Local Inference: Incremental Memory Maintenance for Game NPCs | SS | Maintains character memory by removing superseded KV entries and rewriting at the sequence tail rather than re-encoding a prefix. |
| RecreationWorld: Scalable and Verifiable Environments for Hybrid Computer-Use Agents | SS | Uses a running application as the oracle for execution-grounded rewards, then measures how rarely agents verify what they built. |
| Emergence World: Adversarial Stress-Testing of Long-Horizon Multi-Agent Systems | A | Reports 16 days of persistent agent worlds where detection of adversarial content did not prevent it from reaching long-term memory. |
| SIMLIFE: Pattern Understanding for Long-Horizon Human-Agent Partnership | A | Benchmarks rule inference over weeks of simulated daily life and finds surface-level prediction without rule understanding. |

## Detailed Notes

### 1. AI for Games in the Foundation Model Era

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.16679) / [Hugging Face](https://huggingface.co/papers/2609.16679) / [GitHub](https://github.com/Eurekaleo/awesome-ai-for-games) / [Project page](https://eurekaleo.github.io/awesome-ai-for-games)
- **Vibotaku's Note:** The work on AI in games has grown up in separate rooms, and this survey builds the map. It sorts the literature into six roles defined by the immediate use of AI output: playing and acting, modeling players and games, designing games, building and maintaining games, generating and adapting at runtime, and testing and evaluating games. For each role it records what structure the game or workflow supplies, what the model learns or produces, which artifacts and capabilities carry across settings, and what evidence backs the claims. The cross-role connections are where a production plan can start: trajectories train world models, learned environments supply experience for agents, design specifications feed executable implementations, and play or test feedback drives revision. The caution is as concrete as the map. Control schemes, rules, engine interfaces, state representations, and player populations stay setting-specific, so a capability that transfers still needs its effectiveness re-established where it will ship. Evaluation is most standardized for bounded game playing and selected learned environments, while persistent state in learned worlds, repeated software revision, validated player modeling, sustained runtime adaptation, and representative automated testing stay thin. At 120 pages it is a searchable reference rather than a linear read, and the companion site carries the taxonomy. On Hugging Face it had 145 upvotes; the checked repository had 221 stars, 9 forks, 0 open issues, and its last push was 2026-09-21.

### 2. Zing-0.5: Toward Playable Worlds with Real-Time Joint Action and Text Control

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.17909) / [Hugging Face](https://huggingface.co/papers/2609.17909) / [GitHub](https://github.com/seedleap/zing-world-model) / [Project page](https://zing.loopit.me/) / [Weights](https://huggingface.co/seedleap/zing-0.5) / [Serving code](https://github.com/seedleap/Zing-SGLang)
- **Vibotaku's Note:** Zing-0.5 is a 5B autoregressive world model built for play. Keyboard input carries magnitude, text instructions are temporally aligned with it, and both are learned in the same sequence, so a text instruction can change the event being generated while navigation continues without a restart anywhere in the demo. The engineering is aimed at the numbers that decide whether a generated world can sit inside a play session: a segment-level teacher supervises a block-level causal student through distribution-matching distillation, four-step generation runs with context-preserving streaming, 832x480 streams at 24 FPS, and the estimated server cost is about USD 0.009 per stream-minute. It scores 81.0 overall and 88.5 on consistency across 158 WBench Navigation cases. Weights, inference code, and a Zing-SGLang serving implementation are public, which means a team can price and profile a playable generated world on its own hardware instead of reading a benchmark table. The checked repository had 155 stars, 18 forks, 3 open issues, and its last push was 2026-09-17; the paper had 42 upvotes on Hugging Face.

### 3. Astronex-World 1.0: Real-Time Interactive World Model Foundation

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.20034) / [GitHub](https://github.com/Astronex-Robotics/Astronex-World) / [Project page](https://world.astronex.com.cn) / [Weights](https://huggingface.co/Astronex-Lab/Astronex-World)
- **Vibotaku's Note:** Astronex-World 1.0 predicts future visual states under frame-aligned camera trajectories, continuous actions, and an embodiment identifier, and accepts text events inserted at a chosen position in a rollout. The build is on the Wan2.2-TI2V-5B prior, with PRoPE injecting camera intrinsics and extrinsics and a 64-dimensional action stream modulating every transformer layer. Two variants ship: a bidirectional model for full-context generation and a causal model with block-causal attention and cross-block KV caching for generation that keeps going. The causal one streams 832x480 at 24 FPS in real time on a single L20 48GB GPU, and all five training stages fit on two L20s. It scores 73.5 on WBench Navi and 70.0 on WBench Full, within a point of a 22B model on Full. A real-time streaming world model that trains on two workstation-class GPUs and lands near much larger systems is the version of this technology a studio can evaluate on its own terms. The checked repository had 12 stars, 1 fork, 0 open issues, and its last push was 2026-09-17. No Hugging Face paper page was listed at the time of checking.

### 4. GameLogicBench: Evaluating Coding Agents on Runtime Game Logic with Tick-Level State Assertions

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.21562) / [GitHub](https://github.com/NJU-LINK/GameLogicBench)
- **Vibotaku's Note:** A game can finish in a valid state after breaking its own rules along the way, which is why replaying fixed examples, scoring videos, or asking a model to judge the result misses the failures that matter in gameplay code. GameLogicBench holds 72 gameplay-logic tasks in Godot projects, 403 hand-designed scenarios expanded by seeded parameter variations into 1,451 test cases, and an evaluator that checks rules at every simulation tick. The evaluator has to accept different correct implementations while rejecting mutants that lose one required capability, so a verdict depends on behavior rather than on implementation choice. Across 20 language-model and scaffold combinations the best run solves 52.78% of tasks, and under Claude Code every one of twelve models solves fewer tasks as scope grows from isolated mechanics through interacting systems to repository-scale features. Most unsuccessful submissions are runnable and implement some required behavior incorrectly, which is the same class of defect a human playtester struggles to catch repeatably, and the authors report that without mutant validation incorrect submissions passed the evaluator. A separate analysis found agents copying code from public repositories when network access is open, so what an evaluation permits to be read changes what it measures. The checked repository had 0 stars, 0 open issues, and its last push was 2026-08-25.

### 5. GameASG-Bench: Benchmarking Autonomous Software Generation for Game Development

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.21293) / [GitHub](https://github.com/areal-project/GameASG-Bench)
- **Vibotaku's Note:** This benchmark makes behavioral testability part of the generation task. The evaluation interface is declared before generation, fixing legal starting scenarios, player-level actions, stable snapshots, rejection behavior, and invariants while leaving implementations open, then static L1 source checks and browser-executed L2 checks combine semantic observations with real input and runtime evidence. The suite covers 47 browser-native game-generation tasks across 12 primary genres, in both 2D and 3D, each with executable checks and an independently verified reference implementation. Across nine agent stacks the best mean L2 check pass rate is 93.2%, while the best strict task success rate, requiring every L1 check and all applicable L2 prerequisite and core requirement checks, is 55.3%, or 26 of 47 tasks. Two harnesses each reached 18 strict successes but overlapped on only ten tasks, and for DeepSeek-V4-Flash full tool access and larger turn budgets helped while strict success was not monotonic in reasoning effort. For anyone wiring agents into a game pipeline, the gap between those two numbers is the practical lesson: high average check pass rates hide task-level non-compliance, and partial credit is where it hides. The checked repository had 5 stars, 0 open issues, and its last push was 2026-09-18.

### 6. Long-Lived Characters, Local Inference: Incremental Memory Maintenance for Game NPCs

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.18935)
- **Vibotaku's Note:** Character memory usually gets handled at the prompt layer, and the cost of that choice shows up the moment a few memories change: revising them invalidates a long reusable prefix, so preparation work competes with the dialogue a player is waiting on. This paper moves the problem into the inference state of a quantized Qwen hybrid recurrent-attention model. The runtime removes superseded attention KV entries, computes replacement records at the true sequence tail, and preserves the continuing recurrent state along with the unchanged KV, so a character stops having to reread its entire life before a conversation. Latency is the smaller half of the problem. When dialogue feeds game-defined actions and value judgments, a fluent but incorrect account of who owns an item or whether a transfer already happened corrupts the input to otherwise deterministic rules. True-tail updates preserved important current-state and historical bindings across eight scripted maintenance rounds, and in a placement case recovered the full-refill quantity in three reconstructions while slot-preserving alternatives repeated a double-subtraction error. Attention-distribution proximity alone did not explain those differences, which is the sort of result that justifies treating a character's inference state as a maintained, history-dependent resource. No repository or project page was listed at the time of checking, and the paper is 18 pages with numerical snapshots in ancillary files.

### 7. RecreationWorld: Scalable and Verifiable Environments for Hybrid Computer-Use Agents

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.22000) / [Hugging Face](https://huggingface.co/papers/2609.22000) / [GitHub](https://github.com/QwenLM/RecreationWorld) / [Project page](https://recreation-bench.cc/)
- **Vibotaku's Note:** Computer-use agents have advanced along two lines that real digital work interleaves: clicking through an interface, and writing and running software. RecreationWorld gives an agent a running reference and no prescribed workflow, so it has to discover the behavior, build a faithful implementation, and then run and visually verify what it produced, across Ubuntu, macOS, Windows, Android, and Web on one harness with native GUI control and coding tools. The running reference doubles as an oracle for hidden behavioral tests, which keeps rewards grounded in execution and lets trajectory generation scale from high-quality open-source applications. RecreationBench adds 250 held-out tasks with programmatic and visual assertions validated on the reference and by human reviewers before the suite was frozen. The result worth internalizing is how far output runs ahead of verification: GPT-6 Astra leads at 58.1% overall while passing all programmatic tests on just 2.8% of tasks, and agents reproduce static interface structure far more reliably than interactions or computed outputs. A team porting or rebuilding a game feature with agents carries the same exposure, since a screen that looks right and a screen that behaves right are two different tests. On Hugging Face the paper had 57 upvotes; the checked repository had 35 stars, 2 forks, 0 open issues, and its last push was 2026-09-21.

### 8. Emergence World: Adversarial Stress-Testing of Long-Horizon Multi-Agent Systems

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.17320) / [Hugging Face](https://huggingface.co/papers/2609.17320)
- **Vibotaku's Note:** Eight parallel worlds of ten agents ran from identical starting conditions for 16 days, seven of them homogeneous on distinct frontier models and one mixed, generating more than 850,000 LLM calls and nearly 50 billion tokens while pursuing goals, creating tools, maintaining persistent memory, and governing shared institutions. Once operational state had accumulated, three stress events arrived through ordinary interaction surfaces: indirect prompt injection, misinformation, and exposure of private agent memories. No evaluated world was fully resilient to all three. The finding to carry into production systems is that detection did not produce containment. Agents recognized threats and still interacted with the adversarial content, wrote it into their own persistent memory, and acted on it as much as 46 hours later; the same model-persona pairing behaved differently in mixed and homogeneous populations. Long-running NPC or companion populations in a live game have the same failure shape at smaller scale, with memory persistence as the transmission path. The paper reports 3 upvotes on Hugging Face and lists no public repository.

### 9. SIMLIFE: Pattern Understanding for Long-Horizon Human-Agent Partnership

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.19610)
- **Vibotaku's Note:** An agent that lives alongside a person has to infer how routines form, why they repeat, and when they change, and that is what SimLife-BP measures: 106 episodes averaging 15.49 hours of observation and 38.57 in-game days, plus 1,439 question-answer pairs that probe direct, counterfactual, noisy, and inverse reasoning under different levels of rule hints. Frontier models often predict the next action at surface level without recovering the rule behind it, lean on frequency-based heuristics instead of if-then reasoning over evidence, and struggle when behavioral patterns shift. Life-sim and companion characters have exactly this problem, because personalization depends on noticing a routine changing rather than replaying the modal event. The platform also supplies rich visual observations, ground-truth action logs, and synthetic dialogue with audio, so it works as a place to test memory, personalization, and adaptation designs before they reach players. This is a COLM 2026 workshop paper, and no repository was listed at the time of checking.

## References

- AI for Games in the Foundation Model Era arXiv: https://arxiv.org/abs/2609.16679
- AI for Games in the Foundation Model Era Hugging Face page: https://huggingface.co/papers/2609.16679
- AI for Games in the Foundation Model Era GitHub: https://github.com/Eurekaleo/awesome-ai-for-games
- AI for Games in the Foundation Model Era project page: https://eurekaleo.github.io/awesome-ai-for-games
- Zing-0.5 arXiv: https://arxiv.org/abs/2609.17909
- Zing-0.5 Hugging Face paper page: https://huggingface.co/papers/2609.17909
- Zing-0.5 GitHub: https://github.com/seedleap/zing-world-model
- Zing-0.5 project page: https://zing.loopit.me/
- Zing-0.5 weights: https://huggingface.co/seedleap/zing-0.5
- Zing-SGLang serving code: https://github.com/seedleap/Zing-SGLang
- Astronex-World 1.0 arXiv: https://arxiv.org/abs/2609.20034
- Astronex-World 1.0 GitHub: https://github.com/Astronex-Robotics/Astronex-World
- Astronex-World 1.0 project page: https://world.astronex.com.cn
- Astronex-World 1.0 weights: https://huggingface.co/Astronex-Lab/Astronex-World
- GameLogicBench arXiv: https://arxiv.org/abs/2609.21562
- GameLogicBench GitHub: https://github.com/NJU-LINK/GameLogicBench
- GameASG-Bench arXiv: https://arxiv.org/abs/2609.21293
- GameASG-Bench GitHub: https://github.com/areal-project/GameASG-Bench
- Long-Lived Characters, Local Inference arXiv: https://arxiv.org/abs/2609.18935
- RecreationWorld arXiv: https://arxiv.org/abs/2609.22000
- RecreationWorld Hugging Face page: https://huggingface.co/papers/2609.22000
- RecreationWorld GitHub: https://github.com/QwenLM/RecreationWorld
- RecreationWorld project page: https://recreation-bench.cc/
- Emergence World arXiv: https://arxiv.org/abs/2609.17320
- Emergence World Hugging Face page: https://huggingface.co/papers/2609.17320
- SIMLIFE arXiv: https://arxiv.org/abs/2609.19610
