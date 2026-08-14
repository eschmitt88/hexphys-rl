---
kind: concept
name: "sparse-pairwise-reaction-table"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/frans2022powderworld.md"]
related_concepts: ["density-ordered-gravity-swap", "bond-affinity-chemistry"]
related_experiments: []
tags: [eulerian, chemistry, game-design]
---

# sparse-pairwise-reaction-table

## Definition

Represent field chemistry as a small handcrafted table of pairwise
element→effect reactions (fire burns wood at rate x, acid dissolves at
20%/step, lava+water→stone), layered over a few *generic* transport rules
— rather than a dense N×N interaction tensor or a full reaction-diffusion
PDE.

## Why it matters here

Powderworld proves 14 elements + a sparse table generates a rich task
space; that's the Eulerian counterpart of the Lagrangian track's
[[bond-affinity-chemistry]] scope guard. Ladder: sparse table first (cheap,
readable), true reaction-diffusion (Gray-Scott) only where continuous
concentrations earn their keep (nutrient fields, plumes). Same "chemistry
is a small table, resist the ALife rabbit hole" discipline on both tracks.

## Connections

- Eulerian sibling of [[bond-affinity-chemistry]].
