---
type: paper-section
section: results
paper: Guardian-Controller
status: draft
note: ALL quantitative values are [FILL] placeholders pending experiment execution.
---

# Results and Analysis

> ⚠️ **All numerical values are `[FILL]` placeholders.** Run `python src/experiment_runner.py` to populate.

## Main Results

**Table 1** — Main benchmark results (FEVER / HotpotQA / MATH). Accuracy + Token cost for 7 methods.

Expected patterns:
- Multi-agent > Single-Agent (collaboration benefit)
- G2CP-Static: high accuracy, high token cost
- Ours: highest accuracy, moderate token cost
- Key gap: Ours vs ClosedLoop-Heuristic (isolates learning gain)

**Table 2** — Safety & cost metrics on FEVER. Unsafe Pass Rate / Recovery Success / Over-Intervention / Graph-Grounded% / Detection F1 for 6 methods.

Expected patterns:
- Ours: lowest Unsafe Pass, highest Recovery, lowest Over-Intervention
- GUARDIAN-Heuristic: strong Detection F1, weaker Recovery
- G2CP-Static: very low Unsafe Pass (always constraining), high Over-Intervention in clean

## Safety–Cost Trade-off Analysis

**Figure 3** — Accuracy–Safety–Cost frontier scatter plot.

Expected: Ours (gold star) in upper-left region (high accuracy, low/moderate cost) with light color (low unsafe pass rate) across all three subplots. Pareto improvement over all baselines.

## Ablation Study

**Table 3** — Ablation results on FEVER. 5 ablated variants, each removing one component.

Expected degradation (ordered by effect size):
1. **--Learned policy**: largest safety degradation (Unsafe Pass ↑, Recovery ↓, Over-Int ↑)
2. **--Recovery actions**: Recovery drops to near-zero
3. **--Graph-grounded mode**: Propagation Rate ↑ in high-contamination scenarios
4. **--Belief encoder (MLP-only)**: Recovery ↓, Over-Int ↑ (no temporal memory)
5. **--Hierarchical heads (flat policy)**: small degradation

## Action Usage Analysis

**Figure 4** — Communication mode + recovery action distribution by scenario.

Expected: FREE >80% in clean episodes; RE-DEBATE dominates S1; ISOLATE dominates S2; GRAPH-GROUNDED escalation dominates S3. CONTINUE >90% in clean episodes.

## Per-Scenario Breakdown

**Table 4** — Recovery Success + Over-Intervention by scenario.

Expected: Ours advantage largest under S1 (subtle errors, hardest to detect) and S2 (targeted isolation). Smaller gap under S3 (obvious tampering signals).

## Case Study

**Figure 5** — Four-panel case study: (a) MA-FreeText failure, (b) risk signal timeline, (c) Guardian-Controller recovery, (d) final output comparison.

Illustrates: belief encoder's temporal memory distinguishes transient disagreement from accumulating contamination. Raw anomaly at round 1 is borderline; belief state correctly triggers intervention by round 2.

## Natural Failure Analysis

50 natural MA-FreeText failures manually annotated. Expected distribution: ~40% S1, ~30% S2, ~20% S3, ~10% other. Validates that contamination scenarios cover real failure modes.

## Statistical Significance

Paired bootstrap tests (10K resamples, p < 0.05). Primary comparisons (Ours vs. ClosedLoop-Heuristic, Ours vs. G2CP-Static, Ours vs. GUARDIAN-Heuristic) expected significant on combined accuracy–safety metric.
