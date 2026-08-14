---
kind: concept
name: "solver-parallelism-vs-stability"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/macklin2016xpbd.md", "literature/papers/matthews2024kinetix.md"]
related_concepts: ["xpbd-compliant-constraints", "dynamically-specified-scenes"]
related_experiments: []
tags: [solver, jax, performance]
---

# solver-parallelism-vs-stability

## Definition

The trade-off between Gauss-Seidel constraint sweeps (sequential — each
correction sees the previous one; faster convergence per iteration, but
unparallelizable) and Jacobi sweeps (all corrections computed from the same
start-of-iteration state — fully vectorizable, but slower-converging and
potentially divergent), with partial batching as the middle ground.

## Why it matters here

A batched JAX engine must use Jacobi-style updates across bonds, but the
XPBD paper validates mostly Gauss-Seidel, and Kinetix found empirically
that fully parallel constraint solving destabilizes — they settled on
partial batches of 16 as the stability/speed compromise. Expect to need
under-relaxation (scale Jacobi corrections by ~1/degree) and to measure
divergence directly. Max node degree 6 bounds the correction accumulation,
which should help.

## Connections

- Constrains the implementation of [[xpbd-compliant-constraints]] in JAX.
- Kinetix's batch-size-16 result is the empirical anchor to compare against.
