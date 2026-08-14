---
kind: concept
name: "lattice-body-representation"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/bhatia2021evolution.md"]
related_concepts: ["runtime-morphology-vs-bilevel-codesign", "dynamically-specified-scenes", "bond-affinity-chemistry"]
related_experiments: []
tags: [representation, soft-body, morphology]
---

# lattice-body-representation

## Definition

Representing a body as typed cells on a regular lattice (EvoGym: square
voxels typed empty/rigid/soft/actuator; hexphys-rl: hex cells = point
masses on a triangular lattice with typed, actuatable bonds), giving a
combinatorial but cheaply simulable design space from <100 elements.

## Why it matters here

EvoGym demonstrates the representation is expressive enough for complex,
natural-looking morphologies and a 30+-task benchmark. Our hex variant
differs in two ways that matter: bonds (not cells) carry the actuation and
material identity, and max degree 6 gives fixed-shape arrays for
vectorization. Their per-actuator rest-length target u ∈ [0.6, 1.6] is the
direct precedent for our "push" action's parameterization.

## Connections

- [[runtime-morphology-vs-bilevel-codesign]] — what our version adds.
- EvoGym's task taxonomy (Walker, Carrier, Climber...) seeds our curriculum.
