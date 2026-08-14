---
kind: concept
name: "conservation-induced-localization"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/plantec2022flow.md"]
related_concepts: ["mass-conservative-ca-flow"]
related_experiments: []
tags: [eulerian, alife, findings]
---

# conservation-induced-localization

## Definition

Flow-Lenia's central empirical result: imposing hard mass conservation on
a pattern-forming field system makes spatially localized, bounded
"creature" patterns the *generic* outcome of random parameter sampling —
where vanilla Lenia needed curricula, diversity search, or gradients to
find them, and naive optimization only found exploding patterns.

## Why it matters here

Validates the field-creature bet at its root: individuality and
boundedness fall out of the conservation law itself, with no hand-
engineered containment. It also explains *why* behavior optimization
becomes tractable (ES works on Flow-Lenia, fails on Lenia) — a directly
relevant prior for hanging an RL policy on a blob. Caveat the paper
doesn't close: no ablation isolating conservation vs. the anti-crowding
diffusion term as the cause.

## Connections

- The reason [[mass-conservative-ca-flow]] is worth its complexity.
