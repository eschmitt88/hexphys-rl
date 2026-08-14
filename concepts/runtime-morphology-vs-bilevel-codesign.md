---
kind: concept
name: "runtime-morphology-vs-bilevel-codesign"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/bhatia2021evolution.md"]
related_concepts: ["lattice-body-representation", "morphology-search-regularity-bias"]
related_experiments: []
tags: [morphology, co-design, rl]
---

# runtime-morphology-vs-bilevel-codesign

## Definition

Two regimes for optimizing body structure: bilevel co-design fixes
morphology per episode and searches structure in an outer loop around an
inner control-learning loop (EvoGym's GA/BO/CPPN + PPO); runtime morphology
makes structural change (glue/unglue) an in-episode *action* of the policy
itself.

## Why it matters here

This is hexphys-rl's central novelty relative to the co-design literature:
nearly all prior work assumes fixed-per-episode morphology, so its
algorithms (design-space search, structure mutation) do not transfer to our
control loop — but its task designs, reward interfaces, and failure
analyses do. Also defines the curriculum boundary: early stages can use
fixed bodies (bilevel regime, where EvoGym results apply), late stages make
construction part of the policy (runtime regime, largely unmapped
territory).

## Connections

- Breaks the assumption underlying [[morphology-search-regularity-bias]]
  findings — re-verify them in the runtime regime.
- Long-horizon credit assignment for construction is the known hard part.
