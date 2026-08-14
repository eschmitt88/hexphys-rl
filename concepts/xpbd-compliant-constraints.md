---
kind: concept
name: "xpbd-compliant-constraints"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/macklin2016xpbd.md"]
related_concepts: ["iteration-count-independent-stiffness", "constraint-force-readout", "solver-parallelism-vs-stability", "penalty-contact-vs-projection"]
related_experiments: []
tags: [solver, xpbd]
---

# xpbd-compliant-constraints

## Definition

A position-based constraint solver augmented with a per-constraint Lagrange
multiplier and compliance (inverse stiffness) parameter α̃ = α/Δt², making
constraint stiffness converge to a well-defined value independent of time
step and solver iteration count.

## Why it matters here

This is the literal numerical method underlying the hex-lattice solver; it
determines how bond stiffness is parameterized and why iteration count can
be a pure speed knob. With α=0 it degenerates to vanilla PBD, so rigid and
soft bonds live in one formulation — the bond-affinity "chemistry" matrix is
just a table of α (and damping β) values per element-type pair.

## Connections

- Seeds the engine's core update loop (predict → project bonds → contact).
- [[iteration-count-independent-stiffness]] is the property that makes it
  the right choice for a deliberately under-converged game engine.
