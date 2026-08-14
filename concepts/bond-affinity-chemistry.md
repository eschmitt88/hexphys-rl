---
kind: concept
name: "bond-affinity-chemistry"
status: seedling
added: "2026-08-14"
sources: []
related_concepts: ["constraint-force-readout", "lattice-body-representation", "xpbd-compliant-constraints"]
related_experiments: []
tags: [chemistry, game-design]
---

# bond-affinity-chemistry

## Definition

The project's minimal chemistry: ~4-6 element types; a symmetric
type-pair matrix giving each potential bond its compliance α, damping β,
and breaking force; plus one scalar energy resource (actuation and
bond-breaking cost energy, absorbing "food" elements yields it). Optionally
one or two contact reactions (A+B→C). Deliberately no more than this.

## Why it matters here

The whole game layer — materials (rigid/springy/brittle/slick), foraging,
competition over matter, stealing body parts by out-pulling bonds — falls
out of one small parameter table read by the same XPBD solver, keeping the
computation graph fixed. Design guard: emergent chemistry is an entire
artificial-life research field (Lenia, particle life); scope creep here
kills the project. Own concept (no external source yet) — candidate
follow-up literature: artificial-life chemistries, if this layer grows.

## Connections

- Bond stress from [[constraint-force-readout]] drives breakage and cost.
- Parameterizes bonds in [[lattice-body-representation]].
