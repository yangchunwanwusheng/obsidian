---
type: paper-section
section: conclusion
paper: Guardian-Controller
status: draft
---

# Conclusion

We introduced a unified closed-loop framework for safe LLM multi-agent reasoning that treats error propagation as a partially observable risk control problem. By combining adaptive communication constraints, temporal graph-based risk observation, and belief-guided intervention policy, Guardian-Controller dynamically balances correctness, safety, and interaction cost throughout the collaboration process.

Our experiments demonstrate that this formulation yields consistent improvements over static communication protocols (G2CP-Static), detection-only heuristics (GUARDIAN-Heuristic), and fixed recovery rules (ClosedLoop-Heuristic) across multiple reasoning tasks and contamination scenarios. The ablation study confirms that each component—contamination-aware belief state, learned hierarchical policy, graph-grounded communication mode, and recovery actions—contributes independently to the overall performance.

**The key insight:** safe multi-agent reasoning is fundamentally a control problem, not merely a matter of better individual agents or stronger detection. By closing the loop between observation and action, and by learning when (and when not) to intervene, we achieve better outcomes than either rigid prevention or reactive recovery alone.

## Limitations and Future Work

- **Scale**: Current study limited to 4-agent, 3-round collaboration. Scaling to larger teams and longer horizons is important.
- **Features**: Controller operates on hand-crafted risk features; end-to-end learning from raw traces could improve performance.
- **Training**: Offline RL may leave improvements unexplored; online fine-tuning with lightweight budgets is a natural next step.
- **Scope**: Framework currently addresses factual correctness; extending to toxicity, bias, and privacy broadens applicability.

## Broader Impact

As LLM-based multi-agent systems are increasingly deployed for high-stakes applications—medical consultation, legal analysis, financial decision-making—the ability to dynamically control safety risks during collaboration becomes critical. Our framework provides a practical, model-agnostic approach that does not require retraining underlying LLMs, making it deployable with existing agent infrastructure. We hope this work encourages the community to view multi-agent safety through the lens of closed-loop control, where the question is not just "can we detect errors?" but "can we decide what to do about them?"
