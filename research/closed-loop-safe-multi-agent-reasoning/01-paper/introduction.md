---
type: paper-section
section: introduction
paper: Guardian-Controller
status: draft
---

# Introduction

Large language model (LLM) multi-agent systems improve reasoning by distributing work across specialized agents that debate, verify, and synthesize. On complex tasks—fact verification, multi-hop question answering, mathematical reasoning—collaboration can outperform single-agent prompting. Yet collaboration introduces a vulnerability absent in single-agent settings: **incorrect intermediate conclusions propagate across agents, distort downstream reasoning, and cascade into system-level failure.**

This propagation dynamic is well-documented. A solver agent produces a plausible but incorrect reasoning chain; the critic misses the subtle error; the verifier lacks contradictory evidence; the synthesizer amplifies the error into a confident wrong answer. The core difficulty is not that individual agents hallucinate—single-agent hallucination is extensively studied—but that **errors spread through the interaction network, and the system lacks a mechanism for deciding when and how to stop them.**

Recent defenses attack pieces of this problem from three directions. **Communication constraints** reduce ambiguity by structuring how agents exchange information (G2CP, AAMAS 2026). **Anomaly detection** identifies suspicious patterns in the collaboration graph (GUARDIAN, NeurIPS 2025). **Self-verification** strengthens agents' ability to check their own outputs (MARCH, ACL 2026). These are valuable tools, but they address single stages of a larger control problem. A multi-agent system still cannot answer: *given the current collaboration state and budget, should I constrain communication, trigger a recovery action, or continue as is?*

We argue that this is a **closed-loop risk control** problem. The system never observes contamination directly; it must infer latent risk from interaction patterns, evidence consistency, and temporal disagreement signals—then choose whether to prevent, intervene, or trust the current trajectory. This perspective unifies existing defenses naturally: communication constraints become preventive actions, anomaly detection becomes the observation mechanism, and recovery operations become control actions. The missing piece—and our core contribution—is a **learned controller that closes the loop.**

We formalize error propagation as a partially observable Markov decision process (Section 3) and introduce Guardian-Controller, a framework with three integrated components: (1) **adaptive communication control** that dynamically selects among free-text, semi-structured, and graph-grounded modes based on estimated contamination risk; (2) **temporal graph-based risk observation** that models agent interactions as an attributed temporal graph and extracts anomaly features without requiring ground-truth contamination labels; (3) **belief-guided intervention policy** that maintains a latent contamination belief via a lightweight GRU encoder and selects recovery actions—isolation, rollback, re-retrieval, re-debate—at each collaboration round (Section 4).

The controller does not retrain the underlying LLM agents. It operates on compact risk features ($\mathbb{R}^{10}$ observation, $\mathbb{R}^{32}$ belief), uses a hierarchical policy network with $\sim$10K parameters, and is trained via offline reinforcement learning from contaminated trajectories. This makes the approach model-agnostic, computationally negligible (<5 minutes on CPU), and amenable to clean ablation.

We evaluate on three reasoning tasks under four contamination scenarios against seven baselines spanning static protocols, detection-only methods, and heuristic recovery (Section 5). Our results demonstrate:

1. Closed-loop adaptive control achieves a Pareto improvement on the joint accuracy–safety–cost frontier over all static and heuristic baselines;
2. The contamination-aware belief state improves intervention timing: higher recovery success with lower over-intervention than direct anomaly-to-action;
3. The learned policy outperforms hand-crafted heuristics with identical observation and action spaces—the gain comes from policy learning, not from better sensors or more actions;
4. Each architectural component (belief encoder, learned policy, graph-grounded mode, recovery actions) contributes independently to the overall improvement.

These findings point to a broader conclusion: safe multi-agent reasoning should be studied not only as a matter of better prompts, stronger detectors, or more accurate individual agents, but as a problem of **adaptive closed-loop control over collaborative inference**—learning when to trust the collaboration and when to intervene.
