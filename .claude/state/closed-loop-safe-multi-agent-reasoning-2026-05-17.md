# Closed-loop Safe Multi-Agent Reasoning state

Date: 2026-05-17
Vault: `D:\note`

## Change made

Created and expanded the research design note:

- `research/01-topics/closed-loop-safe-multi-agent-reasoning.md`

Also updated:

- `wiki/index.md`
- `wiki/log.md`

## Content added

The research note now includes a first structured draft of:

1. problem definition and paper narrative
2. four-layer closed-loop system architecture
3. state / observation / belief / action / reward / learning formulation
4. minimum experiment package and evaluation protocol
5. reviewer attack map and claim-evidence sheet

## Key positioning decisions

- Main line: risk-aware closed-loop controller
- Enhancement: latent contamination belief under partial observability
- Avoid: end-to-end LLM retraining, heavy benchmark engineering, naive module composition
- Scenario: general QA / reasoning multi-agent systems
- Resource profile: medium-cost, method-first, light evaluation extensions

## Formula hygiene

The note uses conservative and internally consistent mathematical notation for:

- hidden contamination state `x_t`
- observation `o_t = \phi(G_t, M_t, E_t, B_t)`
- belief state `b_t = p(x_t | o_{1:t}, a_{1:t-1})`
- controller state `z_t = [b_t; c_t]`
- action `a_t = (m_t, u_t)`
- reward and trajectory objective

## Verification completed

- Confirmed note was created in the research topics area
- Confirmed wiki index was updated with a new direction entry
- Confirmed wiki log was appended with a matching research record
- Confirmed reviewer-facing risks are now explicitly documented in the note

## Recommended next additions

1. minimal belief encoder / policy network design
2. figure and table plan
3. abstract and introduction narrative draft

## Additional update

The note was further expanded with a reviewer-credible baseline ladder and fairness rationale:

- lower-bound baseline: Single-Agent
- standard baseline: MA-FreeText
- strong baselines: MA-SemiStruct, GUARDIAN-Heuristic
- strongest published-style baseline: G2CP-Static
- budget-matched baseline: ClosedLoop-Heuristic
- optional upper-bound proxy: Oracle Controller

It now also explicitly documents:

- same data access
- same tool access
- matched budget
- matched evaluation protocol

and defines which comparison pair answers which scientific question.

## Further update

The note now also includes a minimum implementable controller design:

- GRU-based belief encoder over compact risk observations
- hierarchical policy heads for communication mode and recovery action
- optional target-selection head restricted to top-k suspicious entities
- offline-first training strategy with heuristic warm start

This keeps the paper positioned around high-level closed-loop control rather than end-to-end LLM training.

## Latest update

The note was further expanded with a paper-facing figure/table plan:

- 5 main-text figures
- 3 main-text tables
- multiple appendix artifacts
- explicit artifact-to-claim bindings
- a minimum publishable artifact bundle prioritization

This helps constrain later experiments toward paper-ready evidence rather than ad hoc result collection.

## New writing-layer update

The note now also includes reviewer-facing prose planning for:

- title style candidates
- abstract structure and a first abstract draft
- introduction opening move
- four-paragraph introduction skeleton
- contribution phrasing rules
- gap framing rules that avoid "just combining prior modules"

This makes the note usable not only for design and experiments, but also as a direct staging document for paper drafting.
