---
kind: concept
name: "bgk-collision-stability-limit"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/ataei2023xlb.md"]
related_concepts: ["lbm-collide-stream", "unconditional-stability-via-dissipation"]
related_experiments: []
tags: [eulerian, lbm, stability]
---

# bgk-collision-stability-limit

## Definition

The BGK single-relaxation collision operator becomes numerically unstable
at low viscosity (relaxation time τ → 0.5); staying stable there requires
heavier collision models (KBC, MRT, cumulant) — XLB's own benchmarks
switch to KBC for exactly this reason.

## Why it matters here

The counterexample that sharpens the project's stability principle: unlike
XPBD and Stable Fluids, LBM-BGK *can* blow up, and untrained RL agents are
adversarial perturbation generators. Argues for starting the Eulerian
curriculum on Stable Fluids, or budgeting KBC complexity + conservative τ
margins if LBM leads. The Fields dashboard's LBM demo deliberately lets
you cross the floor and watch the auto-reset.

## Connections

- Boundary case of [[unconditional-stability-via-dissipation]].
