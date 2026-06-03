---
type: paper-section
section: experimental-setup
paper: Guardian-Controller
status: draft
---

# Experimental Setup

We evaluate Guardian-Controller on three reasoning tasks under four controlled contamination scenarios, comparing against seven baselines. Principles: (1) fixed agent scaffold; (2) matched base model, retrieval, budget; (3) controlled contamination injection; (4) joint accuracy–safety–cost reporting.

## Tasks and Datasets

| Task | Dataset | Metric | Role |
|------|---------|--------|------|
| Fact Verification | FEVER (dev, 6,666 claims) | Label accuracy | Primary |
| Multi-hop QA | HotpotQA (distractor, 7,405 dev) | EM / F1 | Primary |
| Math Reasoning | MATH-500 | Answer correctness | Primary |
| General Knowledge | MMLU (4-subject subset, 600 q) | Accuracy | Validation |

Pilot: 100 instances per task. All methods share identical input.

## Contamination Scenarios

| Scenario | Method | Tests |
|----------|--------|-------|
| **S1: Endogenous Hallucination** | Solver output replaced with plausible incorrect reasoning, confidence inflated 0.85–0.95 | Detection of spontaneous errors |
| **S2: Agent-Targeted Evidence** | Verifier receives fabricated evidence | Source localization + isolation |
| **S3: Communication Tampering** | Messages altered during transmission | Protocol constraint effectiveness |
| **S4: Long-Horizon Drift** (appendix) | 5 rounds, subtle contamination at round 1 | Recovery over longer horizons |

Contamination applied with p=0.5 per episode for S1–S3.

## Agent Scaffold & Base Model

- 4-agent scaffold: Solver, Critic, Verifier, Synthesizer (identical across all methods)
- Base model: Qwen3-32B via SiliconFlow API
- Max rounds: 3 (all methods)
- Token budget: 10,000 per episode
- Retrieval access: identical across methods

## Baselines (7 methods)

| # | Baseline | Description | Role |
|---|----------|-------------|------|
| 1 | **Single-Agent** | Single LLM with CoT | Lower bound |
| 2 | **MA-FreeText** | Unconstrained NL communication | Standard multi-agent paradigm |
| 3 | **MA-SemiStruct** | Slot-based message formatting | Tests if simple structure suffices |
| 4 | **G2CP-Static** | Always-on graph-grounded | Pure prevention |
| 5 | **GUARDIAN-Heuristic** | Detection + fixed-threshold rules | Detection + heuristics |
| 6 | **ClosedLoop-Heuristic** | Full framework, hand-crafted policy | **Critical baseline**: isolates learning value |
| 7 | **Ours** | Full method with learned controller | — |

## Metrics

### Task Metrics
- Accuracy (FEVER label / HotpotQA EM / MATH correctness / MMLU accuracy)

### Safety Metrics
- Detection AUROC/F1
- Propagation Rate
- Recovery Success Rate
- Unsafe Pass Rate
- Over-Intervention Rate

### Cost Metrics
- Token consumption
- Latency (wall-clock seconds)
- Intervention count
- Graph-grounded mode usage ratio

**Primary evaluation**: accuracy–safety–cost frontier (Figure 3).

## Implementation Details

- Belief encoder: GRU, 64-dim hidden, 32-dim belief, PyTorch
- Policy network: 128-dim shared trunk, hierarchical heads
- Training: 500 trajectories, 200 epochs, AdamW (lr=1e-3)
- Reward weights: $\lambda_1=1.0, \lambda_2=0.3, \lambda_3=0.1, \lambda_4=0.2$
- Controller: CPU-only; LLM agents: API
