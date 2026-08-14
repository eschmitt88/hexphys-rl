---
kind: concept
name: "penalty-contact-vs-projection"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/bhatia2021evolution.md", "literature/papers/macklin2016xpbd.md"]
related_concepts: ["xpbd-compliant-constraints", "solver-parallelism-vs-stability"]
related_experiments: []
tags: [solver, contact]
---

# penalty-contact-vs-projection

## Definition

Penalty methods resolve contact with forces proportional to penetration
depth inside an explicit integrator (EvoGym: mass-spring + symplectic RK4)
— simple but fragile, needing small timesteps or stiff springs to avoid
instability and tunneling. Projection methods (XPBD) move positions to
satisfy non-penetration constraints iteratively and stay stable at large
timesteps.

## Why it matters here

Contact between disconnected bodies is the engine's main complexity beyond
lattice bonds. Plan: short-range repulsion/contact constraints between
nearby unbonded nodes found via spatial hashing, solved in the same XPBD
projection loop as bonds — one solver, not two force models. EvoGym's
CPU-bound, hours-per-run throughput is the cautionary baseline for the
penalty + explicit-integration path.

## Connections

- Same solver machinery as [[xpbd-compliant-constraints]].
- Spatial hashing on the near-lattice node distribution is the cheap
  neighbor search.
