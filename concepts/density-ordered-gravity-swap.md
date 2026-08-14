---
kind: concept
name: "density-ordered-gravity-swap"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/frans2022powderworld.md"]
related_concepts: ["sparse-pairwise-reaction-table", "dynamically-specified-scenes"]
related_experiments: []
tags: [eulerian, falling-sand, implementation]
---

# density-ordered-gravity-swap

## Definition

Powderworld's vectorized falling-sand gravity: elements swap with a
lower-density neighbor below, resolved without write conflicts by
iterating the shift+mask update over discrete density levels instead of
processing all cells simultaneously. The whole engine is shift ops +
boolean masks + world·(1−mask) + new·mask blending — no per-cell loops.

## Why it matters here

The reference implementation pattern for the Eulerian gym's falling-sand
solver in JAX: identical computation graph every step, batch dimension
free, exactly the vmap-fleet shape the project requires. Their measured
compute split (gravity ~34%, velocity ~35%, reactions ~23%) also tells us
where optimization time will go.

## Connections

- Transport layer beneath [[sparse-pairwise-reaction-table]].
