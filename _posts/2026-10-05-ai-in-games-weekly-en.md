---
title: "Vibotaku's AI in Games Weekly: 2026-10-05"
date: 2026-10-05
author: VibOtaku
tags: ai agents game-ai newsletter
lang: en
translation_key: ai-in-games-weekly-2026-10-05
---

**2026-09-28 - 2026-10-05**

## Highlights

- Coding agents are being pointed at whole games and whole scenes. RSIGame internalizes its own debugging history into the generator, and LEGO-Anything reconstructs a room from a single image as an executable Blender program rather than a mesh.
- World action models now have a generalization story. A controlled study locates the benefit of future prediction in the first denoising step, and Simple-WAM keeps it at latent-model cost.
- World models are being asked to track what nobody is looking at. World Observer adds decoupled panoramic observers so objects keep evolving off-camera and return in the right state.
- Interactive world models are getting the systems work to match. WorldAttention replaces sliding windows with paged KV memory for long sessions, and 4Director drives a video world model from rigid 3D geometry.
- Evaluation is moving from task success to whether a run was worth its budget: anomaly auditing inside Unreal Engine and Three.js scenes, generation checked against long-form design documents, and a 10+ hour, 1000+ tool-call agent run.

## Reading recommendations

| Paper | Recommendation Index | Highlight |
| --- | --- | --- |
| RSIGame: Autonomous Agentic Game Development with Recursive Self-improvement | SSS | Splits game development into local and global improvement loops, then trains the generator on its own debugging experience to cut generation tokens by 11x. |
| What Makes World Action Models Generalize? An Empirical Study of Test-Time Future Modeling | SSS | Shows the generalization gain of world action models comes from preparing a future representation, and delivers it with a one-pass Simple-WAM at latent-model cost. |
| LEGO-Anything: Coding Agents for 3D Scene Reconstruction | SSS | Turns single-image scene reconstruction into an Image-to-Code loop over Blender, with a simulator-grounded benchmark that scores artifacts, geometry, and appearance separately. |
| World Observer: Joint Actor-Observer Generation for Persistent World Modeling | SSS | Pairs the perspective actor with panoramic observers so out-of-view objects keep evolving and re-enter in an updated state. |
| AREX-2: Advancing Self-Improving Agents through Long-Horizon Reflective Tasks | SS | Trains reflection and long-horizon iteration on verifiable programming tasks, then transfers the behavior to deep research benchmarks. |
| Beyond Memory: Harnessing Long-Horizon Agents with Explicit Belief States | SS | Replaces history retention with maintained belief states that pair the current world estimate with unresolved task requirements, and detects agents that stop making progress. |
| WorldAuditBench: Interactive 3D World Auditing with Multimodal Agents | SS | Issues 213 anomaly-finding tasks across 13 Unreal Engine 5 and Three.js environments under a fixed exploration budget. |
| WorldAttention: An Efficient Attention Architecture for Interactive Video World Models | SS | Combines hybrid sparse attention with a hierarchical paged KV cache to hold long interactive sessions without sliding-window context loss. |
| A2Z GameSpec-Bench: How Faithfully Can Coding Agents Generate Games from Game Design Specifications? | SS | Converts 100 long-form game design documents into dependency-aware contracts and scores generation with scenario replay and adaptive playtesting. |
| 4Director: Controlling Video World Models with Rigid 3D Geometry | A | Conditions video generation on canonical meshes moved by rigid transforms, with a new metric for joint motion adherence and identity preservation. |
| Marathoner: Ultra-Long-Horizon Autonomous Intelligence | A | Builds a post-training pipeline for multi-hour execution, reporting runs of 10+ hours and 1000+ tool calls on frontier-difficulty tasks. |

## Detailed Notes

### 1. RSIGame: Autonomous Agentic Game Development with Recursive Self-improvement

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.39045) / [Hugging Face](https://huggingface.co/papers/2609.39045) / [GitHub](https://github.com/WenyiWU0111/RSIGame) / [Project page](https://huggingface.co/spaces/RSIGame/rsigame-page)
- **Vibotaku's Note:** Agents that build games tend to iterate against a small set of test cases and overfit them, producing builds with unresolved bugs and missing behaviors. RSIGame splits the work into a local explore-diagnose-improve loop, where an evolving checklist accumulates testing and improvement guidance, and a global loop that tracks overall quality, preserves the best checkpoint, and detects saturation or regression across long development runs. Successful experience is then internalized into the generator through training rather than left in prompts or retrieved notes. Across 140 GameCraft-Bench tasks, two engines, and five generators it improves game quality at matched development budgets, and the internalized Qwen3.8-27B reaches 61.38 on Godot and 58.53 on Phaser while using 11x fewer generation tokens. The lesson for teams automating content or prototype work is that the durable gains came from learning the debugging history, not from resampling more candidates. Checked 2026-10-05: 84 upvotes on Hugging Face; repository at 120 stars, 13 forks, 1 open issue, last push 2026-10-01.

### 2. What Makes World Action Models Generalize? An Empirical Study of Test-Time Future Modeling

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.34981) / [Hugging Face](https://huggingface.co/papers/2609.34981) / [GitHub](https://github.com/LeapLabTHU/Simple-WAM) / [Project page](https://zrporz.github.io/Simple-WAM-Web/)
- **Vibotaku's Note:** World action models predict future frames alongside actions during training, and video denoising is expensive enough that many systems discard the future at inference. The study finds that latent variants match explicit ones on in-distribution tasks while losing the generalization that motivated joint modeling in the first place, measured across environmental perturbation, data efficiency, and task generalization under matched backbones, data, and budgets. The gap traces almost entirely to the first denoising step, so what matters is preparing a future representation and not the later rendering passes. Simple-WAM implements that insight as a single forward pass over fully noised video tokens, with the training noise schedule adapted to match, and lands at generalization on par with explicit models and efficiency comparable to latent ones. For anyone serving a playable world model on a GPU budget, this separates the part of future prediction that pays for itself. Checked 2026-10-05: 136 upvotes on Hugging Face; repository at 72 stars, 1 fork, 0 open issues, last push 2026-10-05.

### 3. LEGO-Anything: Coding Agents for 3D Scene Reconstruction

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.36380) / [Hugging Face](https://huggingface.co/papers/2609.36380) / [Project page](https://lego-anything.com/)
- **Vibotaku's Note:** A scene reconstructed as a mesh can be rendered but not interrogated, which limits what a downstream tool or agent can do with it. LEGO-Anything has a coding agent iteratively write and execute Blender code, inspect scenes and renderings, and revise, so the artifact is a program whose execution yields a scene that can be inspected, edited, and queried. LEGO-Bench scores 208 images from 104 indoor and outdoor scenes on artifact validity, visible-surface geometry, and rendered appearance as separate axes; the strongest agent reaches 53.4% indoor and 39.6% outdoor, and the gap between producing valid scene artifacts and faithfully recovering geometry is where the remaining work sits. The reported failure modes are the useful part for anyone wiring an agent into a DCC tool: weak scene initialization, regressive edits during iteration, and unreliable self-evaluation, which a training-free harness plugin partially repairs across all six evaluated models. LEGO-World then runs detections, instance masks, and relative depth as deterministic queries on the reconstructed scenes, landing well short of specialized vision models — a candid reading of how precise program-built scenes currently are. Checked 2026-10-05: 135 upvotes on Hugging Face; no public repository listed.

### 4. World Observer: Joint Actor-Observer Generation for Persistent World Modeling

- **Recommendation Index:** SSS (Must Read)
- **Links:** [arXiv](https://arxiv.org/abs/2610.02162) / [Hugging Face](https://huggingface.co/papers/2610.02162) / [GitHub](https://github.com/cvlab-kaist/world-observer) / [Project page](https://cvlab-kaist.github.io/world-observer/)
- **Vibotaku's Note:** Actor-centric video world models lose direct evidence the moment an object leaves the camera, and the state they invent on re-entry is often wrong. World Observer decouples observing from acting: alongside the perspective actor it generates one or more panoramic observers that watch selected world regions, so objects continue to evolve visually while off-view and return with updated state. Actor and observers are grounded by warping from a shared panoramic source, which gives explicit geometric correspondence, and an Observer Sink of high-resolution perspective references restores fine appearance on re-entry. Because observers are independent of the actor they can be placed freely, extended to multiple locations, and driven by control signals to steer out-of-view evolution, and the paper adds world-space metrics and a benchmark for exactly that behavior. Persistence of this kind is what players assume by default: the door you left open should still be open when you walk back. Checked 2026-10-05: 78 upvotes on Hugging Face; repository at 27 stars, 1 fork, 1 open issue, last push 2026-10-02.

### 5. AREX-2: Advancing Self-Improving Agents through Long-Horizon Reflective Tasks

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.38288) / [Hugging Face](https://huggingface.co/papers/2609.38288) / [GitHub](https://github.com/VectorSpaceLab/AREX-2)
- **Vibotaku's Note:** Self-improvement at test time rests on two habits: producing a solution better than the current one, and keeping that iteration effective over many rounds. AREX-2 synthesizes long-horizon improvement trajectories from machine learning and algorithmic programming, two domains with verifiable feedback and rewards for sustained iteration, and trains a Qwen3.8-27B agent on them. The agent scores 81.8 on MLE-bench Lite and 70.7 on Frontier-CS, and the same checkpoints transfer to deep research with 84.0 on BrowseComp, 92.2 on GAIA, and 93.8 on DeepSearchQA, continuing to improve as the round budget grows. The transfer is the claim worth testing on game work: reflection and long-horizon execution learned in well-supervised domains carry into tasks that offer no clean verifier, which supports investing in the reflective loop and not only in the environment. Checked 2026-10-05: 137 upvotes on Hugging Face; repository at 27 stars, 4 forks, 1 open issue, last push 2026-10-01.

### 6. Beyond Memory: Harnessing Long-Horizon Agents with Explicit Belief States

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2610.01415) / [Hugging Face](https://huggingface.co/papers/2610.01415) / [GitHub](https://github.com/luoyu100/PoS) / [Project page](https://luoyu100.github.io/projects/progression-of-states/project/)
- **Vibotaku's Note:** Organizing interaction history into memory does not guarantee a coherent picture of the current world, and long episodes are where that shows up. PoS builds and continually maintains explicit belief states as the decision context: each belief combines an estimate of the current world state with the task requirements still unresolved, making visible both where the agent is and what it still owes. The framework validates belief consistency and monitors progress to detect Belief Trapping, where an agent keeps acting without advancing, then selects recovery based on the trapping pattern and the type of unmet requirement. It achieves the best overall score on all four benchmarks across three LLM backbones, and context-scaling experiments show the approach holds up as context grows. Companion and squad AI fails in exactly this shape — units that stay busy while the objective stalls — so treating belief state rather than transcript as the decision context is a design worth borrowing. Checked 2026-10-05: 84 upvotes on Hugging Face; repository at 27 stars, 0 forks, 0 open issues, last push 2026-10-02.

### 7. WorldAuditBench: Interactive 3D World Auditing with Multimodal Agents

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.40325) / [Hugging Face](https://huggingface.co/papers/2609.40325) / [GitHub](https://github.com/UCSB-NLP-Chang/WorldAuditBench) / [Project page](https://ucsb-nlp-chang.github.io/WorldAuditBench/)
- **Vibotaku's Note:** Procedurally generated and agent-generated 3D content arrives with defects players find quickly: floating props, walls that can be walked through, objects inconsistent with their surroundings. WorldAuditBench makes finding them an agent task, with 213 anomaly tasks across 13 Unreal Engine 5 and Three.js environments in five anomaly families, evaluated under a fixed exploration budget through two paradigms: VLA-based exploration followed by VLM identification, and an end-to-end VLM agent where visual reasoning drives action selection. Success ranges from 6.6% to 42.3% against 83.4% for humans, which locates the difficulty in coupling navigation with visual reasoning rather than in either capability alone. The evaluation design is what transfers to a studio pipeline: a budget-bounded audit that scores the evidence an agent gathered, which is the shape of automated level QA. Checked 2026-10-05: 99 upvotes on Hugging Face; repository at 4 stars, 0 forks, 1 open issue, last push 2026-10-04.

### 8. WorldAttention: An Efficient Attention Architecture for Interactive Video World Models

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.34606) / [Hugging Face](https://huggingface.co/papers/2609.34606) / [GitHub](https://github.com/alibaba-damo-academy/WorldAttention) / [Project page](https://alibaba-damo-academy.github.io/WorldAttention/)
- **Vibotaku's Note:** Interactive video world models buy latency with sliding windows and pay in forgotten history, which is why long sessions drift. WorldAttention attacks the cost structure directly, combining Hybrid Sparse Attention (linear global attention plus head-adaptive sparse attention) with a hierarchical KV cache that organizes historical KV pairs into semantically indexed pages across memory tiers, allowing fine-grained retrieval and controlled GPU residency, with custom kernels to make the design pay off in practice. It reports subject-consistency of 0.9472 on VBench-Long and 0.9668 on InterVBench, ahead of prior methods. Treating memory as a paged, retrievable store instead of a fixed buffer is the same move showing up in agent memory research, and here it is what makes minute-scale interactive rollouts affordable. For an engine or streaming product, the interesting quantity is GPU residency over session length, and this paper is an argument that it can be bounded without discarding the past. Checked 2026-10-05: 49 upvotes on Hugging Face; repository at 14 stars, 1 fork, 1 open issue, last push 2026-09-29.

### 9. A2Z GameSpec-Bench: How Faithfully Can Coding Agents Generate Games from Game Design Specifications?

- **Recommendation Index:** SS (Strong Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.39564) / [Hugging Face](https://huggingface.co/papers/2609.39564) / [GitHub](https://github.com/krafton-ai/a2z-gamespec-bench) / [Project page](https://a2z-gamespec-bench.github.io/)
- **Vibotaku's Note:** Delegating whole application development to a coding agent is easy to demo and hard to grade, because a plausible build can quietly drop requirements that need to hold together. A2Z GameSpec-Bench takes 100 long-form game design documents and converts each into a dependency-aware contract of rules, constraints, and prerequisite relations, then measures faithfulness through source-code inspection plus agent-generated test policies that drive scenario-based replay and adaptive playtesting, with the contract fixed across agents and revision rounds. Evaluations show current agents failing to satisfy interdependent requirements across implementation and actual play, and requirement-specific feedback improving GDD fidelity by 10.9% relative to self-revision after two rounds. That last figure is the practical finding: the bottleneck for specification-following is targeted, requirement-linked feedback, and the benchmark is a way to measure whether a workflow is actually delivering it. Checked 2026-10-05: 14 upvotes on Hugging Face; repository at 13 stars, 0 forks, 1 open issue, last push 2026-10-01.

### 10. 4Director: Controlling Video World Models with Rigid 3D Geometry

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2610.02160) / [Hugging Face](https://huggingface.co/papers/2610.02160) / [GitHub](https://github.com/VVeiCao/4Director) / [Project page](https://stability-ai.github.io/4director/)
- **Vibotaku's Note:** Controlling objects inside a video world model through image-plane cues is ambiguous in depth and rotation, and 3D tracks or blobs lack complete geometry. 4Director conditions generation on an explicit 4D scene: each object is reconstructed once from the input image as a canonical mesh and moved by one prescribed rigid transformation per frame, which gives an interpretable control interface and stops unobserved geometry from being re-invented in every frame. The controlled scene is rendered as depth video and a Motion Adapter converts that geometric scaffold into video, synthesizing view-consistent appearance, illumination, and non-rigid dynamics. Training uses RealCOD-Rigid, 20,774 clips annotated with rigid 3D scenes by an automatic pipeline, and the paper introduces Identity-Gated IoU to score motion adherence and object identity preservation together. This sketches the controllable camera-and-object layer that a generated world needs before it behaves like a scene rather than a video. Checked 2026-10-05: 35 upvotes on Hugging Face; repository at 14 stars, 0 forks, 1 open issue, last push 2026-10-02.

### 11. Marathoner: Ultra-Long-Horizon Autonomous Intelligence

- **Recommendation Index:** A (Should Read)
- **Links:** [arXiv](https://arxiv.org/abs/2609.34378) / [Hugging Face](https://huggingface.co/papers/2609.34378)
- **Vibotaku's Note:** Marathoner targets executions that run for hours rather than minutes. Its post-training pipeline synthesizes task-level data from large GitHub release PRs containing 1000+ lines of new code, chains generated tasks into harder composites, rejection-samples fine-tuning trajectories from a strong teacher across diverse harnesses, and applies online reinforcement learning with a Later Stage Bonus Reward that pays specifically for meaningful maneuvers late in a run. On five ultra-long-horizon benchmarks it improves over its base model and passes a strong proprietary baseline, and the analysis reports consistent work over 10+ hours and 1000+ tool calls on difficult tasks. Long game sessions create the same reward-shaping problem: a teammate's contribution in hour three has to count as much as its first move, and late-stage effort is easy to under-reward through the usual trajectory-level scoring. Checked 2026-10-05: 42 upvotes on Hugging Face; no public repository or project page listed.

## References

- RSIGame arXiv: https://arxiv.org/abs/2609.39045
- RSIGame Hugging Face page: https://huggingface.co/papers/2609.39045
- RSIGame GitHub: https://github.com/WenyiWU0111/RSIGame
- RSIGame project page: https://huggingface.co/spaces/RSIGame/rsigame-page
- What Makes World Action Models Generalize? arXiv: https://arxiv.org/abs/2609.34981
- What Makes World Action Models Generalize? Hugging Face page: https://huggingface.co/papers/2609.34981
- What Makes World Action Models Generalize? GitHub: https://github.com/LeapLabTHU/Simple-WAM
- What Makes World Action Models Generalize? project page: https://zrporz.github.io/Simple-WAM-Web/
- LEGO-Anything arXiv: https://arxiv.org/abs/2609.36380
- LEGO-Anything Hugging Face page: https://huggingface.co/papers/2609.36380
- LEGO-Anything project page: https://lego-anything.com/
- World Observer arXiv: https://arxiv.org/abs/2610.02162
- World Observer Hugging Face page: https://huggingface.co/papers/2610.02162
- World Observer GitHub: https://github.com/cvlab-kaist/world-observer
- World Observer project page: https://cvlab-kaist.github.io/world-observer/
- AREX-2 arXiv: https://arxiv.org/abs/2609.38288
- AREX-2 Hugging Face page: https://huggingface.co/papers/2609.38288
- AREX-2 GitHub: https://github.com/VectorSpaceLab/AREX-2
- Beyond Memory: Harnessing Long-Horizon Agents with Explicit Belief States arXiv: https://arxiv.org/abs/2610.01415
- Beyond Memory: Harnessing Long-Horizon Agents with Explicit Belief States Hugging Face page: https://huggingface.co/papers/2610.01415
- Beyond Memory: Harnessing Long-Horizon Agents with Explicit Belief States GitHub: https://github.com/luoyu100/PoS
- Beyond Memory: Harnessing Long-Horizon Agents with Explicit Belief States project page: https://luoyu100.github.io/projects/progression-of-states/project/
- WorldAuditBench arXiv: https://arxiv.org/abs/2609.40325
- WorldAuditBench Hugging Face page: https://huggingface.co/papers/2609.40325
- WorldAuditBench GitHub: https://github.com/UCSB-NLP-Chang/WorldAuditBench
- WorldAuditBench project page: https://ucsb-nlp-chang.github.io/WorldAuditBench/
- WorldAttention arXiv: https://arxiv.org/abs/2609.34606
- WorldAttention Hugging Face page: https://huggingface.co/papers/2609.34606
- WorldAttention GitHub: https://github.com/alibaba-damo-academy/WorldAttention
- WorldAttention project page: https://alibaba-damo-academy.github.io/WorldAttention/
- A2Z GameSpec-Bench arXiv: https://arxiv.org/abs/2609.39564
- A2Z GameSpec-Bench Hugging Face page: https://huggingface.co/papers/2609.39564
- A2Z GameSpec-Bench GitHub: https://github.com/krafton-ai/a2z-gamespec-bench
- A2Z GameSpec-Bench project page: https://a2z-gamespec-bench.github.io/
- 4Director arXiv: https://arxiv.org/abs/2610.02160
- 4Director Hugging Face page: https://huggingface.co/papers/2610.02160
- 4Director GitHub: https://github.com/VVeiCao/4Director
- 4Director project page: https://stability-ai.github.io/4director/
- Marathoner arXiv: https://arxiv.org/abs/2609.34378
- Marathoner Hugging Face page: https://huggingface.co/papers/2609.34378
