---
kind: concept
name: "morphology-search-regularity-bias"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/bhatia2021evolution.md"]
related_concepts: ["runtime-morphology-vs-bilevel-codesign"]
related_experiments: []
tags: [morphology, co-design, findings]
---

# morphology-search-regularity-bias

## Definition

Structure-generation methods with smoothness/regularity priors (CPPN-NEAT
generating voxel types from spatial coordinates) systematically underperform
unbiased mutation (plain GA) on tasks requiring irregular substructure,
such as manipulation — measured across EvoGym's benchmark.

## Why it matters here

A cautionary prior for hexphys-rl's construction stage: don't bias body
generation or construction rewards toward symmetric/regular lattice
patterns and assume it generalizes across tasks. Also relevant to random
task/body generation for curriculum (the generator should produce irregular
bodies too). EvoGym also found even simple GA beats hand-designed bodies on
several tasks — favor learned over hand-authored morphology once the
runtime regime works.

## Connections

- Conditions how [[runtime-morphology-vs-bilevel-codesign]] curriculum
  stages generate starting bodies.
