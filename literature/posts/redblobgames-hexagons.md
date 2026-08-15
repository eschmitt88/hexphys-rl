---
kind: post
title: "Hexagonal Grids (Red Blob Games)"
author: Amit Patel
url: https://www.redblobgames.com/grids/hexagons/
source: "raw/web/redblobgames-hexagons.md"
added: "2026-08-15"
relevance: 4
related_experiments: []
related_concepts: ["axial-hex-storage"]
tags: [hex-grid, coordinates, implementation, reference]
---

# Hexagonal Grids (Red Blob Games)

## TL;DR

The definitive practical reference for hex-grid math: use axial/cube
coordinates (q+r+s=0) for anything non-rectangular, store the hexagon map
as a fixed square axial array with a validity mask, and do all geometry
(distance, lines, ranges, rotation) in cube space.

## Key points

- Axial neighbor set is six fixed offsets — (+1,0),(+1,-1),(0,-1),(-1,0),
  (-1,+1),(0,+1) — i.e. six array shifts: exactly the shape a JAX/vmap
  stencil engine wants. This is what keeps the hex pivot inside the
  fixed-computation-graph regime.
- Hexagon-of-radius-N map in a dense (2N+1)² axial array wastes ≤ ~50% of
  cells to the mask; the waste buys fixed shapes (vmap) and O(1) indexing.
  Cell count 3N²+3N+1.
- Distance = cube max-norm; range/ring queries are two nested loops with
  clamped bounds; 60° rotations are coordinate permutations — cheap data
  augmentation for policies (6-fold symmetry).
- Pointy-top pixel transform x = s√3(q+r/2), y = s(3/2)r; pixel→hex needs
  cube_round (fix the coordinate with the largest rounding error).
- Rotation-by-permutation + reflection give the full dihedral symmetry
  group D₆ — observation augmentation and canonicalization both come free.

## Follow-up

- Encapsulate axial storage + mask + shifts in one map module in the JAX
  gym; every layer (terrain, water, creep, wind) shares it.
- Use cube-space rotations for ×6 (or ×12 with reflections) training-data
  augmentation in the DQN baseline.
