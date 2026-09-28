---
title: "Vibotaku's AI in Games Weekly: 2026-09-28"
date: 2026-09-28
author: VibOtaku
tags: ai agents game-ai newsletter
lang: en
translation_key: ai-in-games-weekly-2026-09-28
---

**2026-09-21 - 2026-09-28**

## Highlights

- Game evaluation is being rebuilt to outlast model progress. GameHorizon ships 5,000 hours of annotated AAA gameplay with a three-level instruction pyramid and a stepwise online track, and Kaggle's Game Arena runs head-to-head competitions in Chess, Poker, and Werewolf so rankings cannot saturate.
- Rule fidelity is the missing layer in generated worlds. GameDirector keeps mechanics, state, and NPC tactics inside an agentic director and leaves pixels to the video model, reporting 99.6% mechanics fidelity against 21–34% for end-to-end baselines on the same games.
- World models are converging on physical priors and usable memory. Object permanence and solidity get a 150-generator Blender curriculum plus a 300-question exam, while WorldCrafter compresses multi-view history into viewpoint-shaped tokens for minute-scale exploration.
- Live-service game agents have real telemetry now. PUBG Ally collected roughly 39k sessions with real players, surveyed 141 countries, and documents the compression, guardrails, and memory redaction needed before an LLM teammate can ship.
- Memory and environment design are the bottlenecks for long-horizon agents. Read-time memory curation beats write-time distillation across three agent benchmarks, and new evaluations score collaboration and memory in persistent worlds instead of single-task success.

## Reading recommendations

| Paper | Recommendation Index | Highlight |
| --- | --- | --- |
| GameHorizon Suite: Multi-Horizon Data and Evaluation in Gameplay | SSS | Releases 5,000 hours of AAA gameplay with multi-horizon instructions and a reproducible offline plus stepwise online benchmark. |
| Training Object Permanence in World Models | SSS | Builds 150 Blender task generators for object permanence and solidity, releases a 1.5M-sample corpus and a 300-question exam, and trains a 16B model that tops its interface class among continuation models. |
| PUBG Ally: A Conversational Embodied Agent as an AI Teammate | SSS | Documents a shipped voice-enabled LLM teammate for a live battle royale, with 39k gameplay sessions of training data and player survey results across 141 countries. |
| WorldCrafter: Consistent Video World Model with Implicit 3D-aware Memory | SS | Stores long-horizon history as camera-queryable memory tokens so streamed exploration holds up across revisits and viewpoints. |
| GameDirector: Decoupling Gameplay Logic from Rendering for Player-Configurable Game World Models | SS | Moves rules and state transitions out of the video generator into a director agent, reaching 99.6% mechanics fidelity on three games. |
| Game Arena: Strategic LLM Evaluation in Competitive Environments | SS | Kaggle's platform plays frontier models against each other in Chess, Poker, and Werewolf, with 900k poker hands and Elo-style ratings. |
| HappyWorld-Bench | SS | Scores world models across video, spatial, and embodied tracks with human A/B Elo, and reports where generated worlds break under interaction. |
| AgentWorld: Benchmarking Long-Horizon Collaboration of Multi-agent LLMs | SS | Runs 100 annotated MMORPG tasks with 3–20 asymmetric agents and adds a causal metric for how much team effort actually mattered. |
| Just-in-Time Memory: Learning to Curate Task-Adaptive Memory for LLM Agents | A | Keeps raw trajectories and curates memory at query time, improving success over write-time memory systems on ALFWorld, WebShop, and τ²-bench. |
| Synthesizing Reactive Character Behaviors for Continuous Games via Programmatic Policy Search | A | Searches for editable programs that drive game characters, with a coding agent proposing structure and an enumerator filling the details. |
| Unity Insight: A Production Code--Asset Index for LLM Coding Agents in Unity Projects | A | Indexes the code-plus-asset graph of a Unity repository and cuts agent token use by 53% and wall-clock time by 52% on project-specific questions. |

## Detailed Notes

### 1. GameHorizon Suite: Multi-Horizon Data and Evaluation in Gameplay

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.25001) / [Hugging Face](https://huggingface.co/papers/2609.25001) / [GitHub](https://github.com/TencentARC/GameHorizon) / [Project page](https://gamehorizon-suite.github.io/)
- **Vibotaku's Note:** Gameplay benchmarks tend to sample a handful of rollouts per game, which makes comparisons noisy and hides which part of a long objective failed. GameHorizon builds the infrastructure to avoid that. The dataset holds 5,000 hours of recordings from 21 AAA titles played by 100 expert players, with 4,571 videos at 60 fps, 411 million keyboard-mouse events, and roughly one new short-horizon instruction every 2.63 seconds. Instructions come in a three-level pyramid of 1–5 second operations, 1–2 minute goals, and 5–8 minute strategies, totalling 6.18 million annotations. The benchmark splits into an offline track of three primary tasks with 1,000 multiple-choice questions each plus ten diagnostic variants of 200, and a stepwise online track of 10 causal and 10 thematic tasks with 62 verifiable subtasks, each capped at 400 model calls so failures localize to a specific step. Across 44 question-answering models the mean accuracy is 64.7% over a 25% random baseline, stretching from 44.6% to 80.2%, and the difficulty gradient runs 57.3% on action prediction, 65.1% on goal decomposition, and 71.6% on cross-horizon consistency. Feeding medium- and long-horizon instructions to a vision-only model improves future-action planning by 7.2 points, which is the strongest argument yet for keeping instructions in the data rather than recovering actions with an inverse dynamics model. Checked 2026-09-28: 130 upvotes on Hugging Face; repository at 328 stars, 1 fork, 1 open issue, last push 2026-09-22.

### 2. Training Object Permanence in World Models

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.28654) / [Hugging Face](https://huggingface.co/papers/2609.28654) / [GitHub](https://github.com/hokindeng/object-permanence) / [Project page](https://www.object-permanence.world/)
- **Vibotaku's Note:** Generated worlds still break in ways players notice immediately: an object disappears behind an occluder and returns somewhere impossible, or passes through a wall without slowing down. WROP treats those two behaviors, object permanence and object solidity, as the foundation that everything else in a world model inherits, since a model that allows interpenetration cannot render a valid collision or support removal. The construction is deliberately cognitive-science shaped: 150 hand-designed Blender generators across six task families, with parameters like speed, lighting, and camera angle randomized while the task structure stays fixed, giving 10,000 samples per generator and a 1.5M-sample training corpus. Evaluation uses a fixed 300-question exam and a blind 20-rater pairwise study that placed the authors' 16B continuation model, PWM-WROP, first among continuation models at Elo 1679.5 and behind only a statistical tie between two commercial reference-to-video systems at 1723.6. Data, exam, model answers, weights, and the native-PyTorch training stack are released, so the recipe is reproducible rather than just reportable. Checked 2026-09-28: 209 upvotes on Hugging Face; repository at 271 stars, 7 forks, 0 open issues, last push 2026-09-26.

### 3. PUBG Ally: A Conversational Embodied Agent as an AI Teammate

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.29837) / [Hugging Face](https://huggingface.co/papers/2609.29837)
- **Vibotaku's Note:** This is the rare game-agent report that covers shipping rather than a demo build. Ally plays PUBG: BATTLEGROUNDS as a voice-enabled teammate, which forces a two-speed architecture: a language-model agent reads game state through a controlled interface, interprets player speech, keeps context, and issues high-level action choices, while a faster control layer handles movement, combat, and recovery. Speech has to stay synchronized with action, and every decision has to land inside a live match's latency budget. The team trained on nearly 39k sessions of real players alongside Ally, and found that offline evaluations and player preferences disagreed often enough that feedback and preference comparisons had to shape the evaluation criteria themselves. What makes the deployment section useful is its scope: model compression, context compaction, targeted safety training, runtime guardrails, and memory redaction all appear as prerequisites for serving a player-facing agent. Among surveyed players in 141 countries whose play was confirmed in game records, positive responses exceeded negative ones by 25.1 percentage points on whether they would recommend Ally. Checked 2026-09-28: 11 upvotes on Hugging Face; no public repository or project page listed.

### 4. WorldCrafter: Consistent Video World Model with Implicit 3D-aware Memory

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.24984) / [Hugging Face](https://huggingface.co/papers/2609.24984) / [GitHub](https://github.com/TencentARC/WorldCrafter) / [Project page](https://drexubery.github.io/WorldCrafter)
- **Vibotaku's Note:** A streamed world has a memory budget measured in tokens, and the way that budget gets spent decides whether a scene survives being revisited from another angle. WorldCrafter learns a camera-queryable implicit 3D-aware memory: the requested viewpoint shapes how multi-view evidence is compressed into a fixed set of view-specific tokens before denoising, with no explicit depth correspondence anywhere in the pipeline. The memory encoder and pose-conditioned readout train jointly with the video generator, and combining that memory with recent temporal context plus few-step distillation gives streaming exploration from a single image or text prompt. Gains show up in long-horizon consistency and camera-control accuracy during minute-scale exploration of static and dynamic scenes. For interactive worlds, the practical form of this claim is that a player can turn around and come back, and the room is still the room. Checked 2026-09-28: 156 upvotes on Hugging Face; repository at 408 stars, 11 forks, 2 open issues, last push 2026-09-28.

### 5. GameDirector: Decoupling Gameplay Logic from Rendering for Player-Configurable Game World Models

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.25652) / [Project page](https://jimntu.github.io/gamedirector/)
- **Vibotaku's Note:** Health deduction, skill activation, combat resolution, and termination conditions all depend on exact state transitions, which is where end-to-end game world models get into trouble: pixels are a poor place to enforce a rule. GameDirector splits the problem, letting an agentic director interpret visual observations, update game state, tactically control NPCs, and enforce gameplay rules, then express those decisions as text prompts that steer a video world model to render the result. Players configure characters, initial states, and rules much as a developer would. Across three commercial titles with 13 characters, the framework reports 99.6% mechanics fidelity against 21–34% for end-to-end baselines, and boss decision quality improves by 39.9% over the strongest baseline. The design also points at a plausible creator-facing product: a configurable world whose hard rules hold while the visual layer stays generative. Checked 2026-09-28: project page live at `jimntu.github.io/gamedirector`; no Hugging Face paper page or repository listed.

### 6. Game Arena: Strategic LLM Evaluation in Competitive Environments

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.31473) / [Hugging Face](https://huggingface.co/papers/2609.31473) / [Project page](https://www.kaggle.com/game-arena)
- **Vibotaku's Note:** Static benchmarks go stale as models saturate them, and a game ladder has the opposite property: opponents improve as the models do, so the ceiling moves. Kaggle's Game Arena opens with three environments chosen for their information structure, Chess for perfect information, heads-up no-limit Texas Hold'em for imperfect information and belief updating, and eight-player Werewolf for deception and social inference with natural-language communication. Rankings come from Elo-style Bradley-Terry fits for Chess, big blinds won per 100 hands for Poker, and game-theoretic evaluation for Werewolf. The pilot tournament ran ten models from five frontier labs and 900,000 poker hands, 180,000 per model, which is two orders of magnitude more than earlier poker benchmarks. o3 won or drew eight of nine pairings, and the strategy analysis found that Grok 4's high preflop steal frequency, 95.0% in the small blind, aligned with its +27.1 BB/100. For teams evaluating models for game-facing agents, the value is the methodology: enough matches for statistically meaningful separation, and an infrastructure that keeps adding games. Checked 2026-09-28: 4 upvotes on Hugging Face; project page live at kaggle.com/game-arena.

### 7. HappyWorld-Bench

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.24308) / [Hugging Face](https://huggingface.co/papers/2609.24308) / [GitHub](https://github.com/LivingFutureLab/HappyWorldBench)
- **Vibotaku's Note:** Visual quality is the easiest thing to measure in a generated world and the least informative about whether it can be played in. HappyWorld-Bench organises evaluation around six capabilities, from generative construction to unified world modeling, and instantiates them across three tracks: video world models with 1,138 prompts, spatial world models with 300 scenes, and embodied models with 254 test cases, using an A/B arena for human Elo ratings alongside automated metrics for behavioral correctness. The results are a catalogue of where generated worlds fail under interaction: video models lose consistency during extended rollouts and revisits, spatial systems reach at best 70.14% placement accuracy and 73.33% edit execution, and embodied candidates struggle to hold state across multi-step actions or respond correctly when action conditions and physical rules change. A studio picking a world model for an interactive feature needs exactly these numbers, because they describe what a player will see after the first impressive few seconds. Checked 2026-09-28: 51 upvotes on Hugging Face; repository at 9 stars, 0 forks, 0 open issues, last push 2026-09-21.

### 8. AgentWorld: Benchmarking Long-Horizon Collaboration of Multi-agent LLMs

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.31590) / [Hugging Face](https://huggingface.co/papers/2609.31590) / [Project page](https://agentworld.io/)
- **Vibotaku's Note:** Multi-agent evaluation usually either pits models against each other or averages individual scores and calls it teamwork. AgentWorld asks a different question inside an MMORPG sandbox: with 100 human-annotated tasks plus 100 augmented variants, more than 50 interaction rounds, and 3–20 agents holding asymmetric roles and abilities, how much of a team's effort actually contributed to the outcome? Each agent acts independently without access to others' internal state, so coordination has to happen through communication, joint planning, and resource sharing. The proposed metric, Causal Collaboration Effectiveness, traces causal dependencies between actions and measures the fraction of effort that mattered. The best of four frontier models reaches 52.0% task success, and the failure modes are familiar to anyone who has shipped companion or squad AI: communication breakdowns, role confusion, and the inability to hold a shared plan across rounds. Persistent worlds with several cooperating characters are the setting where these failures show up in players' games. Checked 2026-09-28: 6 upvotes on Hugging Face; project page live at agentworld.io; accepted at COLM 2026.

### 9. Just-in-Time Memory: Learning to Curate Task-Adaptive Memory for LLM Agents

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.27334) / [Hugging Face](https://huggingface.co/papers/2609.27334)
- **Vibotaku's Note:** Memory systems usually decide what is worth keeping at write time, distilling a finished trajectory into a reflection, workflow, or skill that later gets retrieved by similarity. That forces the choice before the future query exists, and it makes training awkward because a storage decision's value may only appear many tasks later. JitMem inverts the order: it keeps raw trajectories and curates at read time, when the current task is known, synthesizing a compact payload from retrieved traces. Because the payload is consumed on the same task, the curator trains on immediate task success. Across ALFWorld, WebShop, and τ²-bench it improves over the strongest baseline by 16.2, 16.3, and 3.9 absolute success-rate points, and an untrained curator is already competitive with learned write-time systems, which suggests the read-time framing carries much of the gain. For NPC or companion agents whose accumulated experience spans heterogeneous situations, deferring curation avoids permanently compressing away details that matter only in one context. Checked 2026-09-28: 47 upvotes on Hugging Face; no repository or project page listed.

### 10. Synthesizing Reactive Character Behaviors for Continuous Games via Programmatic Policy Search

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.24025)
- **Vibotaku's Note:** Character behavior in shipped games is still largely hand-authored through behavior trees, state machines, and scripts, while reinforcement learning tends to produce controllers that are expensive to train and awkward to edit. This work searches directly over a domain-specific language built around reactive geometric decisions, using constructs such as direction maximization to turn a continuous behavior space into enumerable program structures, plus a set of synthesis antipatterns that prune redundant forms without losing behavioral coverage. The search combines bottom-up symbolic enumeration with top-down guidance from a coding agent that proposes high-level policy structure and calls the enumerator to fill local slots, a method the authors call agentic sketching. On a benchmark of 14 continuous games, from classic control tasks to multi-agent football, pure enumeration often beats using a coding agent alone, and the combined method substantially outperforms both. The output is a readable program a designer can edit, which is what makes programmatic policy search a plausible authoring tool rather than a research artifact. To be presented at SIGGRAPH Asia 2026. Checked 2026-09-28: no Hugging Face paper page or repository listed.

### 11. Unity Insight: A Production Code--Asset Index for LLM Coding Agents in Unity Projects

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.27585)
- **Vibotaku's Note:** In a game engine repository, the relationships that matter live outside the code: a gameplay change can span C# scripts, prefabs, scenes, and ScriptableObjects wired together by Unity GUIDs recorded in `.meta` files and YAML assets. Shell utilities and code-only indexes cannot answer the cross-file questions that follow, so an agent spends its context rediscovering structure that the project already knows. Unity Insight builds a persistent, agent-integrated index over the code-plus-asset graph and has been shipping inside Tuanjie Codely, the agent CLI of Tuanjie Engine, since 2026-07-28. In a paired experiment on 28 project-specific questions across two Unity games with the same model and harness, the index-backed agent used 53% fewer tokens and 52% less wall-clock time than a general-purpose exploration agent, with paired sign tests at p < 0.004. Retrieval tooling shaped to engine conventions is a large part of whether agent assistance survives contact with a production game codebase. Checked 2026-09-28: no Hugging Face paper page or public repository listed.

## References

- GameHorizon Suite arXiv: https://arxiv.org/abs/2609.25001
- GameHorizon Suite Hugging Face page: https://huggingface.co/papers/2609.25001
- GameHorizon Suite GitHub: https://github.com/TencentARC/GameHorizon
- GameHorizon Suite project page: https://gamehorizon-suite.github.io/
- Training Object Permanence in World Models arXiv: https://arxiv.org/abs/2609.28654
- Training Object Permanence in World Models Hugging Face page: https://huggingface.co/papers/2609.28654
- Training Object Permanence in World Models GitHub: https://github.com/hokindeng/object-permanence
- Training Object Permanence in World Models project page: https://www.object-permanence.world/
- PUBG Ally arXiv: https://arxiv.org/abs/2609.29837
- PUBG Ally Hugging Face page: https://huggingface.co/papers/2609.29837
- WorldCrafter arXiv: https://arxiv.org/abs/2609.24984
- WorldCrafter Hugging Face page: https://huggingface.co/papers/2609.24984
- WorldCrafter GitHub: https://github.com/TencentARC/WorldCrafter
- WorldCrafter project page: https://drexubery.github.io/WorldCrafter
- GameDirector arXiv: https://arxiv.org/abs/2609.25652
- GameDirector project page: https://jimntu.github.io/gamedirector/
- Game Arena arXiv: https://arxiv.org/abs/2609.31473
- Game Arena Hugging Face page: https://huggingface.co/papers/2609.31473
- Game Arena project page: https://www.kaggle.com/game-arena
- HappyWorld-Bench arXiv: https://arxiv.org/abs/2609.24308
- HappyWorld-Bench Hugging Face page: https://huggingface.co/papers/2609.24308
- HappyWorld-Bench GitHub: https://github.com/LivingFutureLab/HappyWorldBench
- AgentWorld arXiv: https://arxiv.org/abs/2609.31590
- AgentWorld Hugging Face page: https://huggingface.co/papers/2609.31590
- AgentWorld project page: https://agentworld.io/
- Just-in-Time Memory arXiv: https://arxiv.org/abs/2609.27334
- Just-in-Time Memory Hugging Face page: https://huggingface.co/papers/2609.27334
- Synthesizing Reactive Character Behaviors arXiv: https://arxiv.org/abs/2609.24025
- Unity Insight arXiv: https://arxiv.org/abs/2609.27585
