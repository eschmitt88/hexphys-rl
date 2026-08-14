---
kind: concept
name: "dynamically-specified-scenes"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/matthews2024kinetix.md", "literature/repos/michaeltmatthews-jax2d.md"]
related_concepts: ["solver-parallelism-vs-stability", "lattice-body-representation", "ued-learnability-curriculum"]
related_experiments: []
tags: [jax, vmap, architecture]
---

# dynamically-specified-scenes

## Definition

A simulator design where every task instance runs the identical fixed
computation graph — same op sequence, fixed maximal array shapes with
active-entity masks — so heterogeneous scenes and morphologies parallelize
under a single vmap. Contrast: statically specified engines (Brax) compile
per-morphology and cannot batch across different bodies.

## Why it matters here

The single most load-bearing architectural idea from the Kinetix/Jax2D
line: it is what makes on-device multi-task, multi-morphology RL possible.
The hex lattice inherits it naturally — max degree 6 means bonds live in a
fixed N×6 masked table, and glue/unglue flips mask bits rather than
resizing anything. This must be preserved through every engine decision, or
vmap-across-worlds training dies. Jax2D's O(n²) pairwise collision is the
part *not* to copy — spatial hashing on the lattice should beat it at >100
elements.

## Connections

- Enables the fleets of parallel worlds that [[ued-learnability-curriculum]]
  and self-play training assume.
- Kinetix throughput anchor: 824K steps/sec inside a full RL loop on one
  L40S.
