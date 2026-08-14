---
kind: concept
name: "constraint-force-readout"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/macklin2016xpbd.md"]
related_concepts: ["xpbd-compliant-constraints", "bond-affinity-chemistry"]
related_experiments: []
tags: [solver, xpbd, chemistry]
---

# constraint-force-readout

## Definition

XPBD accumulates a running Lagrange multiplier λ per constraint across
solver iterations, giving a physically meaningful constraint-force/energy
estimate at the cost of one extra scalar per constraint.

## Why it matters here

Free bond-stress signal: bond breakage thresholds (max force before a bond
snaps, per element-type pair), energy accounting for the chemistry layer,
and even agent proprioception ("how hard is my edge being pulled") can all
read λ directly instead of re-deriving stress from positions. The paper
notes this enables breakable joints — exactly our unglue-under-load
mechanic.

## Connections

- Feeds [[bond-affinity-chemistry]] (breakage + energy costs).
- Candidate observation channel for the agent's proprioception.
