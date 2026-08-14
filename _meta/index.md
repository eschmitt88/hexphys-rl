---
name: index
description: Entry-point index for this project's knowledge graph.
---

# Index

Orientation for the project knowledge graph. Updated by `/wrap`, `/ingest`,
and `/new-experiment`.

## Maps of Content

(promote a cluster of ≥5 related concepts into `mocs/<theme>.md`)

- **MoC candidate — solver/engine design** (6 concepts):
  [[xpbd-compliant-constraints]], [[iteration-count-independent-stiffness]],
  [[constraint-force-readout]], [[solver-parallelism-vs-stability]],
  [[penalty-contact-vs-projection]], [[dynamically-specified-scenes]].
  Ripe for `/promote-moc` once the first solver experiment adds evidence.

## Literature

- [[macklin2016xpbd]] — XPBD (rel 5, cred 4): the solver method itself.
- [[matthews2024kinetix]] — Kinetix (rel 4, cred 5): vectorized-engine→RL
  blueprint, throughput anchor.
- [[bhatia2021evolution]] — EvoGym (rel 3, cred 5): lattice soft-robot
  prior art, task taxonomy.
- [[michaeltmatthews-jax2d]] — Jax2D repo (rel 4): fixed-shape/mask vmap
  pattern.

## Active experiments

(list of `experiments/YYYY-MM-DD-<slug>/` folders currently in flight)

## Open questions

- Jacobi-XPBD error/divergence at 1-8 iterations on the hex lattice — first
  experiment.
- How to handle contact between disconnected blobs: universal short-range
  repulsion via lattice spatial hash, solved in the XPBD loop.
- Construction credit assignment: at what curriculum stage does glue/unglue
  enter the action space?
