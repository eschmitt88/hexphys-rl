---
name: index
description: Entry-point index for this project's knowledge graph.
---

# Index

Orientation for the project knowledge graph. Updated by `/wrap`, `/ingest`,
and `/new-experiment`.

## Maps of Content

(promote a cluster of ≥5 related concepts into `mocs/<theme>.md`)

- **MoC candidate — solver design / error-for-speed** (now cross-track, 10
  concepts): [[xpbd-compliant-constraints]],
  [[iteration-count-independent-stiffness]], [[constraint-force-readout]],
  [[solver-parallelism-vs-stability]], [[penalty-contact-vs-projection]],
  [[dynamically-specified-scenes]], [[semi-lagrangian-advection]],
  [[under-converged-pressure-projection]],
  [[unconditional-stability-via-dissipation]] (natural spine),
  [[bgk-collision-stability-limit]]. **Ripe for `/promote-moc` now.**
- **MoC candidate — Eulerian gym design** (growing; now top-down per
  ADR 0001): [[lbm-collide-stream]],
  [[mass-conservative-ca-flow]], [[conservation-induced-localization]],
  [[density-ordered-gravity-swap]], [[sparse-pairwise-reaction-table]],
  [[action-interface-locality]], [[local-parameter-embedding-in-field]],
  [[axial-hex-storage]], [[virtual-pipe-shallow-water]],
  [[outflow-scaling-clamp]].

## Literature

Lagrangian track:

- [[macklin2016xpbd]] — XPBD (rel 5, cred 4): the solver method itself.
- [[matthews2024kinetix]] — Kinetix (rel 4, cred 5): vectorized-engine→RL
  blueprint, throughput anchor.
- [[bhatia2021evolution]] — EvoGym (rel 3, cred 5): lattice soft-robot
  prior art, task taxonomy.
- [[michaeltmatthews-jax2d]] — Jax2D repo (rel 4): fixed-shape/mask vmap
  pattern.

Eulerian track:

- [[stam1999stable]] — Stable Fluids (rel 5, cred 5): the projection
  solver; unconditional stability via dissipation.
- [[frans2022powderworld]] — Powderworld (rel 5, cred 4): falling-sand gym
  prior art; task-complexity/RL inflection warning.
- [[plantec2022flow]] — Flow-Lenia (rel 5, cred 4): mass conservation →
  localized field creatures.
- [[ataei2023xlb]] — XLB (rel 4, cred 4): LBM in JAX; throughput +
  stability data.
- [[mei2007fast]] — Pipe-model erosion (rel 5, cred 4): the top-down
  water layer's exact scheme (ADR 0001).
- [[redblobgames-hexagons]] — hex-grid reference (rel 4): axial storage,
  D₆ symmetry, the map module's spec.

## Active experiments

(list of `experiments/YYYY-MM-DD-<slug>/` folders currently in flight)

## Open questions

- Jacobi-XPBD error/divergence at 1-8 iterations on the hex lattice — first
  experiment.
- How to handle contact between disconnected blobs: universal short-range
  repulsion via lattice spatial hash, solved in the XPBD loop.
- Construction credit assignment: at what curriculum stage does glue/unglue
  enter the action space?
