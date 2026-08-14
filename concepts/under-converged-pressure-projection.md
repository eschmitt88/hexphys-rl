---
kind: concept
name: "under-converged-pressure-projection"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/stam1999stable.md"]
related_concepts: ["semi-lagrangian-advection", "unconditional-stability-via-dissipation", "iteration-count-independent-stiffness", "solver-parallelism-vs-stability"]
related_experiments: []
tags: [eulerian, solver, performance]
---

# under-converged-pressure-projection

## Definition

Running the iterative (Jacobi/red-black) Poisson solve that projects the
velocity field back to divergence-free for fewer iterations than
convergence requires, accepting a slightly compressible field in exchange
for speed.

## Why it matters here

The Eulerian track's literal analogue of the XPBD iteration dial: same
Jacobi-on-GPU structure, same error-becomes-something-graceful behavior
(springy, puffy fluid instead of soft bodies). Notably this regime is
*ours, not Stam's* — his implementations used exact spectral/direct solves
and he argued against few-iteration relaxation; games adopted it anyway
because slight compressibility is imperceptible at frame rate. Warm-start
from last frame's pressure for free convergence.

## Connections

- Fields dashboard Demo 02 exposes this dial with a live residual-
  divergence readout.
- Twin of [[iteration-count-independent-stiffness]] on the Lagrangian side.
