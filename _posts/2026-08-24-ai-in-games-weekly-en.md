---
title: "Vibotaku's AI in Games Weekly: 2026-08-24"
date: 2026-08-24
author: VibOtaku
tags: ai agents game-ai newsletter
lang: en
translation_key: ai-in-games-weekly-2026-08-24
---

**2026-08-17 - 2026-08-24**

## Highlights

- Training environments are becoming generated software: SPADE and AgentMercury both make world construction part of the learning loop rather than a fixed upstream asset.
- Long-running game agents need evaluations with consequences that accumulate. FM-Bench puts contract timing, budgets, rivals, and delayed payoffs in one persistent football-management world.
- Physics-grounded visual simulation is getting more usable. LaGSplat lets a user apply unseen forces to an object reconstructed from sparse monocular video and renders its response in real time.
- Competitive control benefits from a deliberate action interface. RoboStriker gives boxing policies a learned motion manifold before self-play begins.
- Community attention followed the environment-building thread: the checked Hugging Face pages showed 50 upvotes for SPADE and 16 for FM-Bench; both associated repositories were active this week.

## Reading recommendations

| Paper | Recommendation Index | Highlight |
| --- | --- | --- |
| SPADE: Self-Play in Adaptive Synthetic Executable Environments | SSS | A self-play loop where one LLM writes stateful, executable environments and another learns to solve them. |
| FM-Bench: A Benchmark for Long-Horizon Management with Competing Agents | SSS | A 20-year football-management benchmark with shared-world competition and 340 to 400 decision points. |
| LaGSplat: Inferring Physics-Governed Interactive Simulation from Monocular Video Using Latent Lagrangian Gaussian Splatting | SS | Learns an interactive, force-conditioned simulation from one or a few monocular videos. |
| RoboStriker: Latent-Space Strategic Games for Autonomous Humanoid Boxing | SS | Uses a learned boxing-motion latent space to make strategic self-play physically feasible. |
| AgentMercury: Your Agent Can Synthesize Verifiable Environments for Business Scenarios at scale | A | Builds persistent, executable worlds whose tasks emerge from entities, services, state, and invariants. |

## Detailed Notes

### 1. SPADE: Self-Play in Adaptive Synthetic Executable Environments

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2608.19197) / [Hugging Face](https://huggingface.co/papers/2608.19197) / [GitHub](https://github.com/spade-rl/spade) / [Project page](https://spade-rl.github.io)
- **Vibotaku's Note:** SPADE assigns one LLM the job of writing complete Gym-style environments, including state transitions, rewards, and verification code, while another learns inside them. The environment designer uses regret, measured with and without privileged hints, to keep producing feasible tasks near the learner's frontier. This is directly relevant to game-agent teams that have more design space than labeled trajectories. A live environment generator can turn rules, content data, and telemetry into a curriculum, provided the generated worlds have strong verifiers and a review path for bad mechanics. The authors report gains over fixed-environment training on held-out reasoning, tool-use, and game settings. The checked Hugging Face page had 50 upvotes; its repository had 58 stars, one open issue, and a push on August 22.

### 2. FM-Bench: A Benchmark for Long-Horizon Management with Competing Agents

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2608.18423) / [Hugging Face](https://huggingface.co/papers/2608.18423) / [GitHub](https://github.com/Analogy-AI/fm-bench)
- **Vibotaku's Note:** FM-Bench makes long-horizon behavior concrete. An agent manages a football club for 20 in-game years with 26 tools and roughly 340 to 400 decision stops, while transfers, contracts, facilities, youth investment, lineups, rival teams, and board pressure accumulate in a deterministic simulator. Its Arena track places agents in the same world, which turns market interaction and opponent modeling into part of the task. The authors find that high scorers renew contracts early, avoid slow-payoff investments near the end, and keep cash deployed. For game AI, this is a useful template: score the final state, keep the economy persistent, and make the opponents react. The checked Hugging Face page had 16 upvotes; the repository had 16 stars and a push on August 21.

### 3. LaGSplat: Inferring Physics-Governed Interactive Simulation from Monocular Video Using Latent Lagrangian Gaussian Splatting

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2608.16324) / [Project page and interactive demo](https://louenpottier.github.io/lagsplat.html)
- **Vibotaku's Note:** LaGSplat reconstructs an object from one or a few monocular videos, then lets a user push it with forces absent from the training footage. Its low-dimensional latent state is both the generalized coordinate of a learned dissipative Lagrangian and the condition for a Gaussian-splat decoder. That coupling gives an interaction tool a way to map an image-space force into the simulated dynamics. The paper deliberately restricts the dynamics to a few coordinates, so it trades broad physical coverage for bounded responses to unfamiliar forces. This is a promising authoring and data-generation direction for games: capture a real prop quickly, expose a compact control surface, then let artists or agents probe plausible reactions. The checked project page includes the interactive demo.

### 4. RoboStriker: Latent-Space Strategic Games for Autonomous Humanoid Boxing

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2608.16195)
- **Vibotaku's Note:** Raw motor self-play often spends too much of its budget discovering how to remain upright. RoboStriker first distills predefined boxing motions into a bounded latent manifold, then runs two-player self-play over that action space. The separation leaves the competitive policy to choose tactics while the motion decoder handles low-level execution. The authors report better win rates and striking efficiency than direct raw-action exploration, then validate policies on real humanoids. The same design applies to character control in games. A high-level combat planner needs an action vocabulary with stable contact, recovery, and locomotion built in; otherwise strategy learning gets buried under animation failures.

### 5. AgentMercury: Your Agent Can Synthesize Verifiable Environments for Business Scenarios at scale

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2608.20634) / [Hugging Face](https://huggingface.co/papers/2608.20634)
- **Vibotaku's Note:** AgentMercury starts with a persistent world rather than a task. It generates entities, services, tools, state, and executable cross-service invariants, then lets tasks emerge from that substrate. The authors created 4,783 environments across 14 industries and report that policies trained there improved both enterprise and out-of-domain evaluations; fine-tuning on construction traces also raised held-out environment-authoring success from 3.3% to 83.3%. Game teams can borrow the world-first idea for quest QA and agent training. Generate a town, inventory graph, economy, and rule checks first, then sample missions that must remain valid as the world changes. The checked Hugging Face page had 5 upvotes. No code repository was linked on the checked arXiv page.

## References

- SPADE arXiv: https://arxiv.org/abs/2608.19197
- SPADE Hugging Face page: https://huggingface.co/papers/2608.19197
- SPADE project page: https://spade-rl.github.io
- SPADE GitHub: https://github.com/spade-rl/spade
- FM-Bench arXiv: https://arxiv.org/abs/2608.18423
- FM-Bench Hugging Face page: https://huggingface.co/papers/2608.18423
- FM-Bench GitHub: https://github.com/Analogy-AI/fm-bench
- LaGSplat arXiv: https://arxiv.org/abs/2608.16324
- LaGSplat project page and interactive demo: https://louenpottier.github.io/lagsplat.html
- RoboStriker arXiv: https://arxiv.org/abs/2608.16195
- AgentMercury arXiv: https://arxiv.org/abs/2608.20634
- AgentMercury Hugging Face page: https://huggingface.co/papers/2608.20634
