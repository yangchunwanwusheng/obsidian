---
type: paper-section
section: problem-formulation
paper: Guardian-Controller
status: draft
---

# Problem Formulation

We formalize error propagation in LLM multi-agent reasoning as a partially observable Markov decision process (POMDP) where the system must control communication and recovery actions based on imperfect observations of contamination.

## Multi-Agent Collaboration as a Temporal Process

Consider a multi-agent system with $K$ agents $\mathcal{A} = \{a_1, \ldots, a_K\}$ collaborating over $T$ rounds on a task with input query $q$. At each round $t$, each agent $a_i$ produces a message $m_t^{(i)}$ based on the query, its role, and message history. Messages are exchanged according to a communication protocol $\Pi$:

- **FREE**: unstructured natural language
- **SEMI-STRUCTURED**: slot-based format with claim/evidence/confidence fields
- **GRAPH-GROUNDED**: structured graph operations (inspired by G2CP)

## Contamination and Error Propagation

**Contamination** = factually incorrect or logically invalid content in an agent's message.

Three sources:
1. **S1 — Endogenous hallucination**: spontaneous incorrect reasoning
2. **S2 — Agent-targeted evidence corruption**: falsified retrieval evidence
3. **S3 — Communication-targeted tampering**: message altered during transmission

Once contamination enters the system, it propagates through the interaction graph—an agent receiving contaminated messages may incorporate the error, passing it to subsequent rounds.

## POMDP Formulation

Defined by $(\mathcal{X}, \mathcal{O}, \mathcal{A}, \mathcal{T}, \mathcal{R}, \gamma)$.

### Latent State $\mathcal{X}$

$$x_t = (\mathbf{c}^{\text{agent}}_t,\; \mathbf{c}^{\text{edge}}_t,\; h_t,\; \rho_t)$$

- $\mathbf{c}^{\text{agent}}_t \in [0,1]^K$: contamination level of each agent
- $\mathbf{c}^{\text{edge}}_t$: contamination along communication edges
- $h_t$: overall reasoning health
- $\rho_t$: recoverability

The state $x_t$ is **never directly observed**.

### Observation $\mathcal{O}$

$$o_t = \phi(G_t, M_t, E_t, B_t)$$

- $G_t$: temporal interaction graph (nodes=agent outputs, edges=communication with semantic/confidence attributes)
- $M_t$: message-level features (claim–evidence alignment, semantic consistency)
- $E_t$: evidence support and provenance
- $B_t$: budget constraints

The observation vector $o_t \in \mathbb{R}^{d_o}$ includes: graph structure risk features, semantic/evidence features, collaboration behavior features, and cost features.

### Action Space $\mathcal{A}$

$$a_t = (m_t, u_t)$$

Communication mode $m_t \in \{\text{FREE}, \text{SEMI-STRUCTURED}, \text{GRAPH-GROUNDED}\}$.

Recovery action $u_t \in \{\text{CONTINUE}, \text{ISOLATE}(i), \text{ROLLBACK}(\Delta), \text{REROUTE}, \text{RETRIEVE-AGAIN}, \text{RE-DEBATE}, \text{STOP-AND-SYNTHESIZE}\}$.

Constraints: one high-level action per round; ISOLATE targets top-1/2 suspicious agents; ROLLBACK limited to 1–2 rounds.

### Reward $\mathcal{R}$

$$r_t = \lambda_1 R_{\text{task},t} - \lambda_2 R_{\text{prop},t} - \lambda_3 R_{\text{cost},t} - \lambda_4 R_{\text{over},t}$$

- $R_{\text{task}}$: task correctness
- $R_{\text{prop}}$: error propagation penalty
- $R_{\text{cost}}$: token/latency cost
- $R_{\text{over}}$: over-intervention penalty

Objective: $J(\pi_\theta) = \mathbb{E}_{\pi_\theta}[\sum_{t=1}^{T} \gamma^t r_t]$

### Belief State

Since $x_t$ is not directly observed, the controller maintains:

$$b_t = p(x_t \mid o_{1:t}, a_{1:t-1})$$

In practice, we learn a compact belief $b_t \in \mathbb{R}^{d_b}$ via a recurrent encoder. The controller input is $z_t = [b_t; c_t]$ where $c_t$ encodes collaboration context.

## Why This Formulation Matters

Static communication protocols (G2CP) = fixing $m_t$ for all $t$. Detection-only (GUARDIAN) = observing $o_t$ but never acting with $u_t$. Heuristic recovery = fixed policy $\pi_{\text{heuristic}}$ with no learning. Our POMDP framing casts these as special cases of a unified control problem, enabling systematic study of what is gained by closing the loop and learning the policy.
