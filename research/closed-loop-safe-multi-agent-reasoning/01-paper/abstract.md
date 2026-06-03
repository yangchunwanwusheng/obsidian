---
type: paper-section
section: abstract
paper: Guardian-Controller
status: draft
---

# Abstract

Large language model (LLM) multi-agent systems can improve complex reasoning through collaboration, but they introduce a structural vulnerability: incorrect intermediate reasoning can propagate, amplify, and cascade across agents. Existing defenses are fragmented across communication constraints, anomaly detection, and self-verification, leaving the system without a unified mechanism for adaptively deciding *when* to prevent risky exchanges, *when* to intervene, and *how* to recover once contamination has spread.

We frame error propagation in LLM multi-agent reasoning as a **partially observable closed-loop risk control problem**. Our framework, Guardian-Controller, combines adaptive communication constraints as preventive actions, temporal graph-based risk observation as the sensing mechanism, and a lightweight belief-guided high-level controller that selects recovery interventions—such as agent isolation, reasoning rollback, evidence re-retrieval, and structured re-debate—at each collaboration round. The controller does not retrain the underlying LLM agents; it learns a compact policy from round-level risk features, balancing correctness, safety, and interaction cost.

Across fact verification, multi-hop question answering, and mathematical reasoning benchmarks, we show that closed-loop risk control yields a stronger accuracy–safety–cost trade-off than static communication protocols, detection-only heuristics, and fixed recovery rules. Our ablation studies confirm that the contamination-aware belief state and learned intervention policy each contribute independently to this improvement.
