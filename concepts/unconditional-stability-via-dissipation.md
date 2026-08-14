---
kind: concept
name: "unconditional-stability-via-dissipation"
status: growing
added: "2026-08-14"
sources: ["literature/papers/stam1999stable.md", "literature/papers/macklin2016xpbd.md"]
related_concepts: ["iteration-count-independent-stiffness", "under-converged-pressure-projection", "semi-lagrangian-advection", "bgk-collision-stability-limit"]
related_experiments: []
tags: [solver, design-principle, meta]
---

# unconditional-stability-via-dissipation

## Definition

A scheme-design pattern where guaranteed non-blowup at any step size or
iteration budget is purchased by allowing systematic, graceful energy/
detail loss — error becomes softness (XPBD), viscosity (semi-Lagrangian
advection), or compressibility (under-converged projection) — rather than
by bounding the timestep.

## Why it matters here

The unifying principle of the whole project, spanning both tracks:
"stability first, accuracy is the error budget" is what lets a game/RL
engine run fixed cheap per-step compute across thousands of parallel
worlds with adversarially flailing agents inside. Its boundary is also a
concept: not every grid method has it — see
[[bgk-collision-stability-limit]] for a scheme that genuinely explodes.

## Connections

- Candidate spine for the brewing solver-design MoC across both tracks.
