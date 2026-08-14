---
kind: concept
name: "iteration-count-independent-stiffness"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/macklin2016xpbd.md"]
related_concepts: ["xpbd-compliant-constraints", "solver-parallelism-vs-stability"]
related_experiments: []
tags: [solver, xpbd, performance]
---

# iteration-count-independent-stiffness

## Definition

The property that a constraint solver's converged physical behavior
(stiffness, period, damping) does not change as a function of how many
solver iterations are run — only convergence accuracy does.

## Why it matters here

Directly justifies the project's "trade error tolerance for speed" premise:
the engine can under-converge per step (few Jacobi iterations) and the
error manifests as *softness*, not drift or explosion, without re-tuning
bond parameters whenever the iteration budget changes. Vanilla PBD lacks
this — its stiffness couples to iteration count — which is why XPBD, not
PBD, is the right base. Caveat from the paper: behavior at 1-3 iterations
is uncharacterized; measure it on the hex lattice early.

## Connections

- The tunable speed/accuracy dial in [[xpbd-compliant-constraints]].
- First engine experiment: error-vs-iterations curve at 1-8 Jacobi
  iterations.
