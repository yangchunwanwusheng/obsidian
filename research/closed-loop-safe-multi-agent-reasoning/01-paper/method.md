---
type: paper-section
section: method
paper: Guardian-Controller
status: draft
---

# Method

Guardian-Controller consists of four layers organized in a closed feedback loop: (1) a task collaboration layer where agents reason and communicate, (2) a communication control layer that dynamically selects the communication protocol, (3) a risk observation layer that monitors the interaction graph for contamination signals, and (4) a belief-guided control layer that maintains a contamination belief state and selects interventions.

> **Figure 1.** Framework overview — four horizontal layers with feedback loop arrows. Detailed spec in `paper/sections/method.tex`.

## 1. Task Collaboration Layer

We adopt a standard 4-agent scaffold used consistently across all methods:

| Agent | Role |
|-------|------|
| **Solver** | Generates initial solutions and reasoning chains |
| **Critic** | Identifies weaknesses, logical gaps, unsupported assumptions |
| **Verifier** | Fact-checks claims against evidence, reports consistency |
| **Synthesizer** | Integrates all contributions into a final answer |

Agents interact through a shared message history. This scaffold is deliberately simple—the contribution lies in *how the collaboration is managed*, not in novel agent role design.

## 2. Communication Control Layer

Three operating modes, selected round-by-round by the controller:

| Mode | Description | Cost | Safety |
|------|-------------|------|--------|
| **FREE** | Natural language, no structural constraints | Low | Low |
| **SEMI-STRUCTURED** | Slot-based: `{claim, evidence, confidence, rationale}` | Medium | Medium |
| **GRAPH-GROUNDED** | Graph operations: `{operation, subject, predicate, object, evidence}` | High | High |

Critically, the mode is **not fixed a priori**—it adapts to contamination risk.

## 3. Risk Observation Layer

Models collaboration as a **temporal attributed interaction graph** $G_t = (V_t, E_t)$ (GUARDIAN-style).

### Graph Construction
- **Nodes**: agent messages per round, with attributes (role, confidence, semantic embedding)
- **Edges**: cross-round (information flow) + within-round (typed: contradicts, verifies, supports)

### Risk Features (10-dim $o_t$)
1. Mean/max node anomaly scores
2. Mean edge contamination score
3. Subgraph inconsistency (contradiction edge ratio)
4. Propagation depth and breadth
5. Disagreement intensity (confidence std)
6. Evidence support deficit
7. Semantic contradiction score
8. Confidence drift across rounds

Features are computable **without ground-truth contamination labels**.

## 4. Belief-Guided Control Layer

### Belief Encoder (GRU, ~3K params)

$$\tilde{o}_t = \text{MLP}_\psi(o_t) \quad \rightarrow \quad h_t = \text{GRU}(\tilde{o}_t, h_{t-1}) \quad \rightarrow \quad b_t = W_b h_t + \mathbf{b}_b$$

- GRU chosen over Transformer due to short sequence length ($T \leq 5$) and training stability
- Belief $b_t \in \mathbb{R}^{32}$ distinguishes transient anomalies from persistent contamination

### Hierarchical Policy Network (~7K params)

Shared trunk MLP (128-dim) → three heads:
- **Mode Head**: $\pi_m(m_t \mid z_t)$ → {FREE, SEMI-STRUCTURED, GRAPH-GROUNDED}
- **Action Head**: $\pi_u(u_t \mid z_t, m_t)$ → 7 recovery actions
- **Target Head**: $\pi_g(g_t \mid z_t, u_t)$ → select among top-k suspicious agents

Total: **~10K parameters**, CPU trainable.

### Heuristic Policy Baseline

Fixed rules for ablation:
- If global anomaly > 0.7 → GRAPH-GROUNDED + ISOLATE/RE-DEBATE
- Else if > 0.4 → SEMI-STRUCTURED + CONTINUE/RETRIEVE-AGAIN
- Else → FREE + CONTINUE

## Training Procedure

### Trajectory Collection
- 500 trajectories (250 clean, 250 contaminated) from FEVER training set
- Heuristic policy + $\epsilon$-greedy exploration ($\epsilon=0.2$)
- Collect $(z_t, a_t, r_t, z_{t+1})$ tuples

### Two-Stage Offline RL
1. **Behavior Cloning (BC) Warm-start**: imitate heuristic teacher on successful states
2. **Advantage-Weighted Regression (AWR)**: re-weight BC by exponentiated advantage

$$\mathcal{L}_{\text{AWR}} = -\mathbb{E}_{(z,a)\sim\mathcal{D}}\left[ \exp(A(z,a)/\beta) \log \pi_\theta(a \mid z) \right]$$

### Why Offline RL
1. Online interaction with LLM agents is expensive and slow
2. Offline training is more stable and reproducible
3. Compact state space (risk features, not raw text) makes offline learning tractable

**Cost**: ~$5 API credits for data collection, <5 minutes CPU training.

### Algorithm

See formal pseudocode in `paper/sections/method.tex` (Algorithm 1).
