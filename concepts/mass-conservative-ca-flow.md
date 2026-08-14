---
kind: concept
name: "mass-conservative-ca-flow"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/plantec2022flow.md"]
related_concepts: ["conservation-induced-localization", "semi-lagrangian-advection", "local-parameter-embedding-in-field"]
related_experiments: []
tags: [eulerian, cellular-automata, conservation]
---

# mass-conservative-ca-flow

## Definition

Flow-Lenia's core move: reinterpret a CA's growth field as an *affinity
map*, form a flow F = (1−α)∇U − α∇A_Σ (affinity gradient + anti-crowding
diffusion), and transport matter along it with reintegration tracking
(Moroz 2020) — each source cell's mass redistributed over a footprint that
integrates to exactly 1, so mass is conserved by construction.

## Why it matters here

The proper mechanism behind the field-creature archetype. Design choice to
make explicitly: exact conservation (reintegration tracking, intricate) vs
approximate (semi-Lagrangian advection + renormalization, cheap — what the
Fields dashboard demo does). Given the project's error-for-speed ethos,
measure the drift before paying for exactness. The flow decomposition also
hands the RL policy its actuation channel: bias ∇U.

## Connections

- Enables [[conservation-induced-localization]].
- The demo's cohesion-to-CoM term is *our addition* — Flow-Lenia holds
  creatures together implicitly via Lenia kernels.
