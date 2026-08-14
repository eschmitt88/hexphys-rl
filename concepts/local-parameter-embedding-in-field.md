---
kind: concept
name: "local-parameter-embedding-in-field"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/plantec2022flow.md"]
related_concepts: ["mass-conservative-ca-flow"]
related_experiments: []
tags: [eulerian, multi-agent, alife]
---

# local-parameter-embedding-in-field

## Definition

Attaching update-rule parameters (a creature's "genome") as a field
co-advected with its mass, mixed at collisions by mass-weighted average or
softmax-by-mass competition — so distinct rule sets coexist and interact
in one shared grid.

## Why it matters here

The path to multi-agent field-creature competition without one-world-per-
policy: each blob carries its own parameters/policy conditioning through
the shared simulation. Flow-Lenia's large runs already show the emergent
regime (species spreading by contamination, two-parameter symbiosis,
food-driven division) that a competitive gym would inherit. Park this
until single-creature control works.

## Connections

- Multi-agent extension of [[mass-conservative-ca-flow]].
