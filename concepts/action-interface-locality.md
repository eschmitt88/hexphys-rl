---
kind: concept
name: "action-interface-locality"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/frans2022powderworld.md"]
related_concepts: ["runtime-morphology-vs-bilevel-codesign"]
related_experiments: []
tags: [rl, embodiment, design-axis]
---

# action-interface-locality

## Definition

A design axis for grid-world agents: global "puppet master" interfaces
(observe the whole grid, place any element at any (X,Y) — Powderworld's
choice) versus spatially local embodiment (cell-entity, field creature,
force field) where action reach is constrained by position.

## Why it matters here

Locality is where the interesting credit assignment lives — Powderworld's
teleporting placement is a much easier problem than our engineer-ant, so
its RL results are optimistic bounds for our setting, not baselines. The
three Fields-dashboard agent demos are three points on this axis
(cell-entity: strict locality; field creature: locality through dynamics;
force field: none). Recommended build order matches decreasing difficulty
value: cell-entity first, force field as the easy calibration baseline.

## Connections

- Interacts with [[task-complexity-rl-inflection]] — harder interfaces
  likely hit the variance inflection sooner.
