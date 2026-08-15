---
kind: concept
name: "axial-hex-storage"
status: seedling
added: "2026-08-15"
sources: ["literature/posts/redblobgames-hexagons.md"]
related_concepts: ["dynamically-specified-scenes", "virtual-pipe-shallow-water"]
related_experiments: []
tags: [hex-grid, implementation, jax]
---

# axial-hex-storage

## Definition

Store a hex grid in axial coordinates (q, r) with implicit s = −q−r: a
radius-N hexagon map (3N²+3N+1 cells) lives in a dense (2N+1)² square
array behind a validity mask, and the six neighbors are six fixed index
offsets — {±1, ±S, ±(S−1)} for row stride S — i.e. pure array shifts.

## Why it matters here

The bridge between the hexagon-map aesthetic and the
fixed-computation-graph requirement: every solver layer (water, CA, D2Q7,
scent) becomes shift+mask stencil ops on one shared array shape, vmap-able
across worlds. The masked corners (≤ ~50% waste) are the rent for fixed
shapes and O(1) indexing. Bonus: 60° rotations are cube-coordinate
permutations, so the D₆ symmetry group provides free observation
augmentation and canonicalization. Distance = cube max-norm; pixel↔hex via
the cube-round rule.

## Pitfall: row wraparound (bitten once, 2026-08-15)

Middle rows of the hexagon span the full array width, so a ±1 index shift
on an edge cell wraps to the *adjacent row's opposite end* — a valid cell,
so a mask check alone does not catch it. Fire leaked between the left and
right map edges in the Arena demos. Fixes: one guard column per side
(S = 2R+3) in flat-index code; in JAX, `jnp.roll` wraps *by design*, so
either keep the guard columns or mask the wrapped column after every roll.
Regression test worth porting: for every valid cell and all six shifts,
the destination must decode to exactly (q+dq, r+dr).

## Connections

- Extends [[dynamically-specified-scenes]] to the hex map.
- One shared map module in the JAX gym; ADR 0001 fixes this choice.
