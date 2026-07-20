---
title: "Vibotaku's AI in Games Weekly: 2026-07-20"
date: 2026-07-20
author: VibOtaku
tags: ai agents game-ai newsletter
lang: en
translation_key: ai-in-games-weekly-2026-07-20
---

**2026-07-13 - 2026-07-20**

## Highlights

- Generative game engines are moving from video demos toward explicit state, player input, persistence, and runtime constraints.
- Fighting games are becoming a sharper testbed for interactive world models because two players expose binding, causality, and adversarial contact errors quickly.
- Multi-scene game generation now has a concrete failure target: transitions must execute in play, not just look plausible in isolated rooms.
- Agent engineering papers are treating the harness as editable infrastructure. The useful unit is behavior mapped back to source, especially when a change cuts across files.
- Hidden-information games remain a good stress test for agent audits because win rates alone hide why an agent trusted, accused, or poisoned another player.

## Reading recommendations

| Paper | Recommendation Index | Highlight |
| --- | --- | --- |
| From Pixels to States: Rethinking Interactive World Models as Game Engines | SSS | Frames interactive world models around the same action-state-observation loop that game engines already use. |
| WanToFight: Real-Time Generative Game Engine for Multi-Player Combat Interaction | SSS | Runs two-player KOF '97 style control at 30 FPS, with keyboard streams bound to separate fighters. |
| MAGIC: Transition-Aware Generation of Navigable Multi-Scene Game Worlds with Large Language Models | SS | Turns a prompt into a connected multi-scene game project and evaluates whether portals work during play. |
| Harness Handbook: Making Evolving Agent Harnesses Readable,Navigable, and Editable | SS | Builds a behavior-level map from agent harness code so humans and coding agents can localize changes. |
| Auditing Belief-Conditioned LLM Agents in Hidden-Information Social Deduction Games | A | Uses Werewolf to audit belief updates, information isolation, and belief-action mismatches across 1,080 games. |

## Detailed Notes

### 1. From Pixels to States: Rethinking Interactive World Models as Game Engines

- **Reading Priority:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.14076) / [Hugging Face](https://huggingface.co/papers/2607.14076) / [GitHub](https://github.com/AlayaLab/WildWorld)
- **Vibotaku's Note:** This is the paper I would hand to a game AI team before any new world-model prototype. It argues that an interactive model should be judged through player action control, game state dynamics, state-observation persistence, and realtime generation. The Black Myth: Wukong data engine adds over 90 hours of gameplay with frame-aligned actions, game states, observations, and annotations. That explicit state signal is the part to watch: it gives generative systems a way to remember consequences instead of drifting through pretty rollouts. The linked WildWorld repository had 411 stars when checked, with a README update on July 20.

### 2. WanToFight: Real-Time Generative Game Engine for Multi-Player Combat Interaction

- **Reading Priority:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.12592) / [Hugging Face](https://huggingface.co/papers/2607.12592) / [Project page](https://humanaigc.github.io/wantofight/)
- **Vibotaku's Note:** WanToFight is exciting because fighting games punish vague interaction. If the model loses track of which keyboard belongs to which character, or if a hit does not affect the opponent at the right moment, the illusion breaks immediately. The system uses a streaming autoregressive video diffusion model, a Player Association module, and local keyboard injection, then reports 30 FPS at 512x384 on a single RTX 5090. For game teams, this is a useful target shape for generative play: stable identity binding, low latency, and contact-heavy interaction in one loop.

### 3. MAGIC: Transition-Aware Generation of Navigable Multi-Scene Game Worlds with Large Language Models

- **Reading Priority:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.11594) / [GitHub](https://github.com/sereneee1201/MAGIC/)
- **Vibotaku's Note:** MAGIC attacks a very practical content problem: connecting rooms, portals, and scripts so a generated world can actually be traversed. The pipeline plans a transition-aware representation, checks portal reachability with flood fill, generates scenes and transition scripts, then combines them into a runnable project. The important detail is the evaluation agent that executes transitions in play. That pushes LLM scene generation toward shipped-content constraints: connectivity, navigation, and stateful movement between spaces. The repository was public when checked, with 0 stars, 1 fork, 6 commits, and a note that the 100-case transition benchmark is available upon request.

### 4. Harness Handbook: Making Evolving Agent Harnesses Readable,Navigable, and Editable

- **Reading Priority:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.13285) / [Hugging Face](https://huggingface.co/papers/2607.13285) / [Project page](https://ruhan-wang.github.io/Harness-Handbook/) / [GitHub](https://github.com/Ruhan-Wang/Harness_Handbook)
- **Vibotaku's Note:** This one matters for anyone building game-agent tooling rather than agent demos. Modern agents live inside harnesses that manage prompts, tools, state, permissions, retries, and execution. Harness Handbook turns those scattered behaviors into a navigable map tied back to source code, then uses Behavior-Guided Progressive Disclosure to help agents locate the right implementation sites. That is exactly the failure mode in studio tooling: a seemingly small behavior change touches prompts, tool schemas, state, telemetry, and tests. The paper was the #1 Hugging Face paper of the day on July 16 and had 202 upvotes when checked; the GitHub repo had 95 stars, 10 forks, and a July 18 README fix.

### 5. Auditing Belief-Conditioned LLM Agents in Hidden-Information Social Deduction Games

- **Reading Priority:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2607.10814)
- **Vibotaku's Note:** Werewolf is a good place to study agent reliability because the agent must act under private information, noisy speech, deception, and irreversible choices. This paper builds strict code-level information isolation, keeps an external belief state over hidden roles, and logs belief updates plus belief-action deviations. In 1,080 frozen games, the active-belief condition raises the good-side win rate from 0.205 to 0.390 in a 200-seed paired comparison, while direct action-belief consistency stays low at about 0.21. That mixed result is useful. It says belief can help, but the audit trail matters as much as the score because designers still need to know which belief actually drove a vote, accusation, or poison.

## References

- From Pixels to States: https://arxiv.org/abs/2607.14076
- From Pixels to States Hugging Face page: https://huggingface.co/papers/2607.14076
- WildWorld GitHub: https://github.com/AlayaLab/WildWorld
- WanToFight: https://arxiv.org/abs/2607.12592
- WanToFight Hugging Face page: https://huggingface.co/papers/2607.12592
- WanToFight project page: https://humanaigc.github.io/wantofight/
- MAGIC: https://arxiv.org/abs/2607.11594
- MAGIC GitHub: https://github.com/sereneee1201/MAGIC/
- Harness Handbook: https://arxiv.org/abs/2607.13285
- Harness Handbook Hugging Face page: https://huggingface.co/papers/2607.13285
- Harness Handbook project page: https://ruhan-wang.github.io/Harness-Handbook/
- Harness Handbook GitHub: https://github.com/Ruhan-Wang/Harness_Handbook
- Auditing Belief-Conditioned LLM Agents in Hidden-Information Social Deduction Games: https://arxiv.org/abs/2607.10814
