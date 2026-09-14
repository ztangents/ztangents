---
title: "Vibotaku's AI in Games Weekly: 2026-09-14"
date: 2026-09-14
author: VibOtaku
tags: ai agents game-ai newsletter
lang: en
translation_key: ai-in-games-weekly-2026-09-14
---

**2026-09-07 - 2026-09-14**

## Highlights

- Explicit world state is coming back into world models. Programmable World Model keeps entities, rules, and off-screen facts in a program-driven state engine and uses a video model as the renderer.
- World models are starting to produce game geometry instead of only consuming it. Valerant turns one image into a persistent, navigable 3D map by combining action-conditioned rollouts with SLAM reconstruction.
- LLM-written game code is turning into RL infrastructure. PlayTrain plays and trains agents on any generated JavaScript game at over a million decisions per second on a single GPU node.
- Opponent modeling is being measured across a match series rather than a single episode. HORIZON infers hidden game parameters and opponent style in Lux AI Season 3 and reports adaptation gains over best-of-five matches.
- Cooperation benchmarks are moving from score to communication. The Convention Gap measures the cooperation failure left unexplained by literal message content, and human Hanabi pairs show a +26.2 percentage point gap against -0.7 for AI pairs.

## Reading recommendations

| Paper | Recommendation Index | Highlight |
| --- | --- | --- |
| Programmable World Model | SSS | Separates a programmable world-state engine from a generative renderer so generated worlds keep persistent state. |
| Valerant: An Automatic Navigable Game Map Generator via Action-Conditioned World Model Exploration | SS | Builds persistent 3D game maps from a single image using world-model rollouts and SLAM, without training. |
| PlayTrain: An Efficient Reinforcement Learning Framework for LLM-Generated Adaptable JavaScript Games | SS | Runs LLM-generated JavaScript games in a gym loop and trains pixel-based agents at over 1M decisions per second. |
| Hierarchical Belief Modeling for Zero-Shot Opponent Adaptation in Partially Observable Multi-Agent Navigation | SS | Infers hidden game parameters and opponent style to adapt within a best-of-five match structure. |
| The Convention Gap: Towards Measuring Implicit Communication in Cooperative AI Evaluation | SS | Quantifies the failure that literal message content cannot explain, with Hanabi numbers for human, AI, and mixed pairs. |
| Do LLMs Trust the Accuser or the Accusation? Measuring Belief Shifts in Werewolf | A | Scores accusation messages by how far they move an observer's beliefs about who is a wolf. |
| Environments as Scaffold: Enriching Feedback to Bootstrap Self-Evolving Agents in Long-Horizon Tasks | A | Treats environment feedback design as the training lever for long-horizon agents. |

## Detailed Notes

### 1. Programmable World Model

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.10540) / [Hugging Face](https://huggingface.co/papers/2609.10540) / [GitHub](https://github.com/AlayaLab/pwm) / [Project page](https://alaya-lab.github.io/pwm/)
- **Vibotaku's Note:** Video world models render convincing frames and then forget what they rendered. This framework keeps the canonical world state outside the generator: an agent writes executable programs that define entity states and state-transition rules, a lightweight engine advances them, and state-augmented 3D oriented bounding boxes plus the camera trajectory are compiled deterministically into pixel-aligned conditioning for a pretrained video model acting as renderer. Entities persist off-screen, non-visual attributes stay queryable, and one world state drives any camera. The authors built CombatStateBench for this setting and report 94% Count Accuracy and 98% State Accuracy, with playable games that have predefined mechanics as the demonstration. The split is what a game team can use: mechanics live in code you can read and patch while rendering stays generative, which gives a testable middle ground between a hand-built engine and a prompt-only video world. The checked repository had 162 stars, 1 open issue, and its last push was 2026-09-10, and the release roadmap still lists inference code and pretrained weights as unpublished. On the Hugging Face daily papers listing it had 106 upvotes.

### 2. Valerant: An Automatic Navigable Game Map Generator via Action-Conditioned World Model Exploration

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.09418)
- **Vibotaku's Note:** A game has no substrate outside the model. In driving and robotics the world exists independently, so a world model can be checked against reality, while a 3D game has to be instantiated before anything can be explored in it. Valerant runs training-free on a pretrained action-conditioned world model and converts it into a world action model that builds a navigable map from a single image, coupling predictive visual rollouts with SLAM-based spatial reconstruction and exploration-driven action selection. What comes out is persistent geometry rather than a rollout of frames, which is the property a level needs before it can be walked through. The capability is bounded by whatever the base world model can render, so this reads as a first-pass layout generator rather than a replacement for level design, and map layout is expensive enough that a usable first pass changes where art and design time goes. No public repository or project page was listed at the time of checking.

### 3. PlayTrain: An Efficient Reinforcement Learning Framework for LLM-Generated Adaptable JavaScript Games

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.09059)
- **Vibotaku's Note:** Building game environments by hand has kept reinforcement learning research on a short list of benchmarks. PlayTrain generates JavaScript games from a minimal prompt and runs any of them inside a standard gym environment, so one JS file is both the build a person can play and the environment an agent trains in. The authors clone well-known Atari and ProcGen titles in simple JS and train pixel-based agents end to end at over one million agent-decisions per second on a single GPU node, then generate modified versions with new test sets, different procedural generation logic, or changed dynamics. For evaluation work the useful consequence is cheap regeneration: when a benchmark stops separating methods, a new variant costs a prompt and a file. Editable generation logic also turns level distribution itself into an experimental variable.

### 4. Hierarchical Belief Modeling for Zero-Shot Opponent Adaptation in Partially Observable Multi-Agent Navigation

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.12422)
- **Vibotaku's Note:** Lux AI Season 3 makes agents act under partial observability and randomized episode dynamics, and scores them across a best-of-five series that rewards adaptation as much as execution. HORIZON combines symmetry-aware spatial perception, dual memory belief tracking, relic-centric graph attention, information-gain-driven exploration, and an opponent-conditioned policy mixture, keeping short-horizon control separate from cross-match meta-reasoning, with auxiliary belief and world model objectives stabilizing training under PPO in a large-scale JAX simulator. The agent explicitly infers hidden game parameters and opponent style, and the reported gains cover match win rate, episode win rate, adaptation gain, and league rating against recurrent and feedforward baselines. Ranked play in shipped games has the same shape as this setting. The same opponent returns across matches, the hidden parameters stay hidden, and the exploit that worked in match one is often the reason match two is lost.

### 5. The Convention Gap: Towards Measuring Implicit Communication in Cooperative AI Evaluation

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.11489) / [GitHub](https://github.com/dockmfgit/hanabi-convention-gap) / [Zenodo archive](https://doi.org/10.5281/zenodo.21975884)
- **Vibotaku's Note:** Cooperative agents are usually evaluated against other agents, while human cooperation runs on conventions that the literal content of a message does not contain. The convention gap is the difference between the failure probability a literal-content posterior predicts and the failure rate actually observed, and Hanabi makes that posterior exactly computable because the deck is finite and hints are deterministic. Replaying roughly 101,000 play actions from human-human, AI-AI, and human-AI corpora gives +26.2 percentage points for human pairs, -0.7 for AI pairs, and +16.4 for mixed pairs, with the human gap concentrated on plays of cards that had received no hint (+46 pp). In mixed play the literal information available to humans was similar across the three AI partners (38% to 41% predicted failure) while human failure rates ranged from 14.4% to 34.4%, so the partner that produced the largest gap also produced the fewest human failures. Game scores carried different information, since they depend on each corpus's roster, while the gap separated human from AI play at the agent level. For any game with human players and AI teammates, this is an evaluation target that scores communication directly, and the code, pseudonymized per-play data, and a notebook regenerating every figure are public. The checked repository had 0 stars and its last push was 2026-09-11.

### 6. Do LLMs Trust the Accuser or the Accusation? Measuring Belief Shifts in Werewolf

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.12446) / [GitHub](https://github.com/rlglab/werewolf-accusation-benchmark) / [Benchmark page](https://rlg.iis.sinica.edu.tw/papers/werewolf-accusation-benchmark)
- **Vibotaku's Note:** Most social deduction evaluations report who won the game. This one annotates suspicion and accusation messages in LLM-played Werewolf games and measures how an observer's beliefs move after each message, across 40 open-weight configurations and 1,224 annotated messages. Larger models separate wolves from villagers better from game history, and accusations still move beliefs anyway: the accused becomes more suspicious and the accuser less suspicious, even when the accuser is wolf-aligned, with the effect strongest when the accuser is trusted. Larger models do resist accusations from accusers they already distrust. The measurable gap is between reading what a claim says and weighing who said it, and any game with deception or negotiation depends on that distinction holding up in the model you ship. The authors report that the benchmark and code are public; the checked repository had 1 star and its last push was 2026-09-02.

### 7. Environments as Scaffold: Enriching Feedback to Bootstrap Self-Evolving Agents in Long-Horizon Tasks

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.08404) / [Hugging Face](https://huggingface.co/papers/2609.08404) / [GitHub](https://github.com/HongbangYuan/EnvAsScaffold)
- **Vibotaku's Note:** Reward sparsity stalls long-horizon agent training more often than policy capacity does, and the usual remedy of warming up the agent with supervised fine-tuning runs into scarce data and narrow exploration. This work moves the intervention into the environment, building feedback-enriched environments that shift from action guidance to observation enrichment as an episode unfolds and as training progresses, so observation detail appears where standard feedback goes quiet. On SciWorld and BFCL, across several Qwen3 model scales and the GRPO, GSPO, and DAPO algorithms, enriched environments beat standard settings, reduce entropy volatility during training, push exploration into states the policy would otherwise skip, and internalize the guidance into policy weights instead of leaving it as an inference-time aid. The paper also identifies intra-group feedback consistency as a boundary for stable optimization. Hint delivery and observation detail are knobs game teams already control, and this offers a measured reason to spend design effort on them rather than treating the environment as fixed. The checked repository had 3 stars, 0 open issues, and its last push was 2026-09-09.

## References

- Programmable World Model arXiv: https://arxiv.org/abs/2609.10540
- Programmable World Model Hugging Face page: https://huggingface.co/papers/2609.10540
- Programmable World Model GitHub: https://github.com/AlayaLab/pwm
- Programmable World Model project page: https://alaya-lab.github.io/pwm/
- Valerant arXiv: https://arxiv.org/abs/2609.09418
- PlayTrain arXiv: https://arxiv.org/abs/2609.09059
- Hierarchical Belief Modeling arXiv: https://arxiv.org/abs/2609.12422
- The Convention Gap arXiv: https://arxiv.org/abs/2609.11489
- The Convention Gap GitHub: https://github.com/dockmfgit/hanabi-convention-gap
- The Convention Gap Zenodo archive: https://doi.org/10.5281/zenodo.21975884
- Do LLMs Trust the Accuser or the Accusation? arXiv: https://arxiv.org/abs/2609.12446
- Do LLMs Trust the Accuser or the Accusation? GitHub: https://github.com/rlglab/werewolf-accusation-benchmark
- Do LLMs Trust the Accuser or the Accusation? benchmark page: https://rlg.iis.sinica.edu.tw/papers/werewolf-accusation-benchmark
- Environments as Scaffold arXiv: https://arxiv.org/abs/2609.08404
- Environments as Scaffold Hugging Face page: https://huggingface.co/papers/2609.08404
- Environments as Scaffold GitHub: https://github.com/HongbangYuan/EnvAsScaffold
