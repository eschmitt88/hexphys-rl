---
kind: concept
name: "lbm-collide-stream"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/ataei2023xlb.md"]
related_concepts: ["bgk-collision-stability-limit", "differentiable-physics-vs-rl-signal", "dynamically-specified-scenes"]
related_experiments: []
tags: [eulerian, lbm, solver]
---

# lbm-collide-stream

## Definition

The lattice Boltzmann method evolves per-cell, per-direction velocity
populations f_i (D2Q9 in 2D) through two alternating purely local ops —
collide (relax toward local equilibrium; BGK/MRT/KBC operators) and stream
(shift populations to neighbors) — recovering Navier-Stokes in the
low-Mach limit without ever solving a pressure equation.

## Why it matters here

The third Eulerian solver candidate, and the most GPU-native: nothing
reaches across the grid (streaming is jnp.roll), unlike the projection
solve. Mesoscopic populations give rich boundary conditions
(bounce-back, momentum-exchange forces) and a natural hook for
multi-species chemistry. Costs: q floats/cell memory, and a real stability
floor ([[bgk-collision-stability-limit]]). XLB's patterns (lattice
classes, mask-array BCs, (x,y,cardinality) layout) are the JAX reference;
its single-GPU MLUPS numbers are the throughput bar. Historical hex note:
FHP lattice gas needed hexagonal symmetry for isotropy; D2Q9 achieves it
with square-lattice weights instead.

## Connections

- Bake-off vs Stable Fluids and falling-sand CA is Eulerian experiment 1.
