---
type: planning
topic: experiment-checklist
paper: Guardian-Controller
status: pending
---

# Experiment Execution Checklist

## Prerequisites

- [ ] SiliconFlow API key configured
- [ ] Qwen3-32B model available
- [ ] FEVER dev set downloaded
- [ ] HotpotQA distractor dev set downloaded
- [ ] MATH-500 subset prepared
- [ ] MMLU 4-subject subset prepared

## Phase 1: Smoke Test

```bash
cd C:/Users/oobbee/Desktop/mah_seasrch
python src/smoke_test.py
```

- [ ] All agents respond correctly
- [ ] Orchestrator runs 1 clean episode
- [ ] Orchestrator runs 1 contaminated episode
- [ ] Risk observer produces valid features
- [ ] Controller produces valid actions

## Phase 2: Trajectory Collection

```bash
python -c "from src.controller.trainer import collect_training_data; collect_training_data()"
```

- [ ] 500 trajectories collected (250 clean, 250 contaminated)
- [ ] Data saved to `outputs/trajectories/`
- [ ] Verify trajectory format matches `TrajectoryStep` dataclass

## Phase 3: Controller Training

```bash
python -c "from src.controller.trainer import OfflineRLTrainer; ..."
```

- [ ] BC warm-start converges (mode_acc > 0.8, action_acc > 0.7)
- [ ] AWR fine-tuning converges (awr_loss decreasing)
- [ ] Model saved to `outputs/models/`

## Phase 4: Main Experiments

```bash
python src/experiment_runner.py siliconflow 100
```

### Clean Scenario (5 baselines, n=100 per task)
- [ ] Single-Agent → `[FILL]`
- [ ] MA-FreeText → `[FILL]`
- [ ] MA-SemiStruct → `[FILL]`
- [ ] GUARDIAN-Heuristic → `[FILL]`
- [ ] ClosedLoop-Heuristic → `[FILL]`

### G2CP-Static (n=100 per task)
- [ ] G2CP-Static → `[FILL]`

### Ours (n=100 per task)
- [ ] Guardian-Controller (Ours) → `[FILL]`

### Contamination Scenarios (3 scenarios, n=100 each, FEVER)
- [ ] S1 (Endogenous) → `[FILL]`
- [ ] S2 (Agent-Targeted) → `[FILL]`
- [ ] S3 (Comm-Tampering) → `[FILL]`

### Ablation (5 variants, n=100, FEVER)
- [ ] w/o Belief (MLP only) → `[FILL]`
- [ ] w/o Learned policy (heuristic) → `[FILL]`
- [ ] w/o Graph-grounded mode → `[FILL]`
- [ ] w/o Recovery actions → `[FILL]`
- [ ] Flat policy (no hierarchical heads) → `[FILL]`

## Phase 5: Populate Paper

- [ ] Fill Table 1 (Main Results) with data
- [ ] Fill Table 2 (Safety Metrics) with data
- [ ] Fill Table 3 (Ablation) with data
- [ ] Fill Table 4 (Per-Scenario) with data
- [ ] Draw Figure 3 (Frontier) from data
- [ ] Draw Figure 4 (Action Usage) from data
- [ ] Select and annotate case study for Figure 5
- [ ] Complete natural failure analysis (50 trajectories)
- [ ] Compute statistical significance (bootstrap tests)
- [ ] Fill all appendix tables

## Phase 6: Pre-Submission

- [ ] All `[FILL]` replaced with actual values
- [ ] All 5 figures drawn and inserted
- [ ] Reproducibility Checklist completed
- [ ] Ethics Statement added
- [ ] Data/Code Availability statement added
- [ ] Final proofread and consistency check
- [ ] Compile LaTeX, verify no errors
