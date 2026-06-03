---
type: paper-section
section: related-work
paper: Guardian-Controller
status: draft
---

# Related Work

Our work sits at the intersection of three active research areas: multi-agent LLM reasoning, safety and hallucination mitigation in agent systems, and learning-based control for language model interactions.

## Multi-Agent LLM Reasoning

Multi-agent LLM systems improve reasoning by decomposing tasks across specialized agents that debate, verify, and synthesize. Frameworks like AutoGen and LangGraph provide infrastructure for orchestrating agent interactions, while debate-based approaches use multi-turn argumentation to refine answers. Recent work on decentralized coordination (AgentNet, NeurIPS 2025) and partner-aware collaboration (NeurIPS 2025) extends these paradigms to more complex agent topologies. These systems demonstrate that collaboration can outperform single-agent baselines, but they treat communication as an unconstrained natural language exchange, leaving them vulnerable to error propagation. Our work shows that *how* agents communicate—and when communication constraints should be applied—is a learnable control decision.

## Safety and Hallucination in Multi-Agent Systems

Hallucination in single-agent LLMs has been extensively studied, but multi-agent settings introduce distinct failure modes: hallucination amplification, where one agent's error spreads through the collaboration network, and targeted contamination, where corrupted evidence or tampered messages mislead downstream reasoning.

### G2CP (AAMAS 2026)

Replaces inter-agent free text with explicit graph operations (TRAVERSE, UPDATE, ASSERT) over a shared Neo4j knowledge graph. Achieves 90% task accuracy (vs. 67% for free-text), reduces hallucination rate from 23% to 2%, and cuts token consumption by 73%. **Limitation:** applies structured protocol statically—all messages are graph operations regardless of risk level. **Our use:** graph-grounded mode as a preventive action within a dynamic control framework.

### GUARDIAN (NeurIPS 2025)

Models multi-agent collaboration as a discrete-time temporal attributed graph with unsupervised encoder-decoder and Information Bottleneck-based graph abstraction. Strong detection AUROC across three safety scenarios with GPT-3.5/4/Llama 3. **Limitation:** relies on fixed-threshold heuristics for recovery; cannot decide *which* recovery action is most appropriate. **Our use:** temporal graph modeling as the observation mechanism.

### MARCH (ACL 2026)

Multi-agent RL for fact-checking with blind verification (Checker has no access to Solver's reasoning). 8B Checker reaches GPT-4 level performance via PPO with Zero-Tolerance Reward. **Limitation:** trains agents through expensive MARL (multi-GPU); no dynamic communication control. **Our contribution:** keep agents fixed, train only a lightweight (~10K parameter) controller.

### AgentBreeder (NeurIPS 2025)

Evolutionary multi-objective search over multi-agent scaffolds; 79.4% average safety uplift while maintaining capability. **Limitation:** offline scaffold optimization; no runtime control during interaction.

## Learning-Based Control for Language Models

RLHF and related methods train the LM itself—expensive and brittle. Recent work questions whether RL expands reasoning capacity beyond the base model distribution (NeurIPS 2025). Tool-use controllers (ReAct, Toolformer) and routing policies show lightweight control policies on frozen LLMs can be effective. Quantile-guided alignment (NeurIPS 2025) constrains tail risk rather than optimizing expected reward.

Our work extends this philosophy to multi-agent safety: learn a controller that manages the collaboration process, not the agents themselves.

## Positioning

To our knowledge, this is the first work to formulate error propagation in LLM multi-agent reasoning as a closed-loop risk control problem and to demonstrate that a learned high-level controller—operating on risk belief states rather than raw text—can dynamically balance prevention, detection, and recovery.
