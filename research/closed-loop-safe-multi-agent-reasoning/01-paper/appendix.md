---
type: paper-section
section: appendix
paper: Guardian-Controller
status: draft
---

# Appendix

## A. Extended Related Work Comparison

| Method | Prevention | Detection | Recovery | Adaptive | Learning |
|--------|-----------|-----------|----------|----------|----------|
| G2CP | ✓ | — | — | — | — |
| GUARDIAN | — | ✓ | ✓† | — | — |
| MARCH | — | — | — | — | ✓‡ |
| **Ours** | ✓ | ✓ | ✓ | ✓ | ✓ |

†Fixed-threshold heuristics only. ‡Trains individual agents, not a control policy.

## B. Contamination Templates

### S1: Endogenous Hallucination
Solver output replaced with plausible but incorrect reasoning (confidence inflated 0.85–0.95). Templates: `[FILL from experiments]`

### S2: Agent-Targeted Evidence Contamination
Verifier receives fabricated evidence documents. Templates: `[FILL from experiments]`

### S3: Communication-Targeted Tampering
Messages altered during transmission (claim inversion, evidence swapping, confidence inflation). Templates: `[FILL from experiments]`

## C. Heuristic Policy Rules

1. If global_anomaly > 0.7: mode = GRAPH_GROUNDED; if propagation_risk > 0.5: ISOLATE else RE_DEBATE
2. Else if global_anomaly > 0.4: mode = SEMI_STRUCTURED; if evidence_deficit > 0.5: RETRIEVE_AGAIN else CONTINUE
3. Else: mode = FREE; action = CONTINUE

## D. Hyperparameters

| Parameter | Value |
|-----------|-------|
| Observation projection dim | 64 |
| GRU hidden dim | 64 |
| GRU layers | 1 |
| Belief output dim | 32 |
| Policy shared trunk dim | 128 |
| Learning rate | 1e-3 |
| Optimizer | AdamW |
| Training epochs | 200 |
| Batch size | 32 |
| AWR temperature β | 0.5 |
| Reward λ1 (task) | 1.0 |
| Reward λ2 (propagation) | 0.3 |
| Reward λ3 (cost) | 0.1 |
| Reward λ4 (over-intervention) | 0.2 |

## E. Compute Resources

- Base model: Qwen3-32B via SiliconFlow API
- Controller training: single CPU (Apple M2), <5 minutes
- Trajectory collection: ~`[FILL]` API calls, ~$5
- Full evaluation: ~`[FILL]` API calls, ~$`[FILL]`
- Total controller parameters: ~10K

## F. Prompt Templates

### Shared Role Descriptions
`[FILL from codebase: src/agents/solver.py, critic.py, verifier.py, synthesizer.py]`

### SEMI-STRUCTURED Mode Format
```json
{"claim": "...", "evidence": "...", "confidence": 0.0-1.0, "rationale": "..."}
```

### GRAPH-GROUNDED Mode Format
```json
{"operation": "TRAVERSE|ASSERT|QUERY", "subject": "...", "predicate": "...", "object": "...", "evidence": "..."}
```

## G. Additional Results

### Per-Task Frontier Plots
`[FILL from experiments]`

### Belief vs. Anomaly Score Visualization
`[FILL from experiments]`

### Per-Action Success Breakdown

| Action | S1 | S2 | S3 | Overall |
|--------|----|----|----|--------|
| ISOLATE | `[FILL]` | `[FILL]` | `[FILL]` | `[FILL]` |
| ROLLBACK | `[FILL]` | `[FILL]` | `[FILL]` | `[FILL]` |
| RETRIEVE-AGAIN | `[FILL]` | `[FILL]` | `[FILL]` | `[FILL]` |
| RE-DEBATE | `[FILL]` | `[FILL]` | `[FILL]` | `[FILL]` |

### Natural Failure Mapping
50 natural MA-FreeText failures manually annotated. Distribution: `[FILL]`% S1, `[FILL]`% S2, `[FILL]`% S3, `[FILL]`% Other. Inter-annotator agreement: κ = `[FILL]`. Qualitative examples: `[FILL]`.
