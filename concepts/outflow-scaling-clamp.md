---
kind: concept
name: "outflow-scaling-clamp"
status: seedling
added: "2026-08-15"
sources: ["literature/papers/mei2007fast.md"]
related_concepts: ["virtual-pipe-shallow-water", "solver-parallelism-vs-stability"]
related_experiments: []
tags: [solver, stability, vmap-pattern]
---

# outflow-scaling-clamp

## Definition

A single-pass conservation guard for explicit flux schemes: scale all of a
cell's outgoing fluxes by K = min(1, available/(Σ outflux·Δt)) so the cell
can never go negative — replacing the iterative back-scaling loop that
would introduce cross-cell dependency chains.

## Why it matters here

The load-bearing trick that keeps the water update strictly local (one
reduction + clamp per cell) and therefore vmap-able — the same
"sacrifice a little correctness for a fixed parallel computation graph"
move as Jacobi-over-Gauss-Seidel in the projection solvers. Known cost,
flagged by Mei et al. themselves: clamping isn't propagated into
neighbors' balances, so strict conservation can leak under heavy clamping
— measure the drift in the JAX port. The same pattern guards the creep
mass flow and scent transport in the Arena demos.

## Connections

- The Eulerian cousin of [[solver-parallelism-vs-stability]]'s
  parallelism-over-sequential-accuracy trade.
