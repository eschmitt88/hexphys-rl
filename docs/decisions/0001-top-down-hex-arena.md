# 0001 — Top-down hex arena replaces side-view worlds

- **Date:** 2026-08-15
- **Status:** accepted

## Context

The first two dashboards (Lab: Lagrangian XPBD; Fields: Eulerian solvers)
demonstrated solvers in *side view*, with gravity as the driving force
(falling sand, gravity-fed dam game, buoyant smoke). The project's game
direction is StarCraft-like, not platformer-like: top-down view, a large
hexagon map composed of a fine hex grid, multiple units/factions, layered
fields. The map must stay small enough that the grid itself is visible.

## Decision

1. **Perspective: top-down.** Gravity leaves the simulation plane. Drivers
   become head differences (water), reaction rules (CA), forcing (wind),
   and agent actions.
2. **Map: hexagon of radius N of fine hexes** (cell count 3N²+3N+1),
   stored as a dense (2N+1)² axial-coordinate array with a validity mask —
   the Red Blob-recommended layout. The mask waste (≤ ~50%) buys fixed
   array shapes, six-shift neighbor stencils, and vmap batching: the
   fixed-computation-graph property is preserved. N ≈ 16–24 for
   visible-grid play; larger only when the grid no longer needs to be
   read.
3. **Solver mapping under the pivot:**
   - Falling-sand CA → *spread CA*: isotropic hex neighbor rules + the
     sparse pairwise reaction table (creep, fire, growth). Hex improves
     isotropy over square.
   - Gravity water → *pipe-model shallow water* (Mei 2007): terrain height
     + water depth per cell, flux along the 6 edges from head differences.
   - Fluids → *D2Q7 hex LBM* (FHP's native lattice; streaming = 6 shifts,
     no interpolation) and/or projection with the 6-neighbor Laplacian.
4. **Layer stack per cell:** terrain height/type (slow), water depth,
   CA fields (creep, fire, fuel, resource), wind/current, scent/pheromone.
   Chemistry couples layers via the sparse reaction table.
5. **Agent archetypes re-skin, unchanged in kind:** cell-entity → unit;
   field creature → creep faction (conserved mass); force field →
   spells/abilities. The embodiment-locality axis is unchanged.
6. **Existing dashboards stay** as teaching archives (Lab, Fields); the
   new Arena page (docs/arena.html) is the current direction.

## Consequences

- The JAX gym targets one shared hex-map module (axial storage, mask,
  shifts, D₆ symmetry ops) consumed by every layer and solver.
- D₆ symmetry (rotations = cube-coordinate permutations) becomes free
  observation augmentation/canonicalization for policies.
- Side-view-specific mechanics (density-ordered *vertical* swaps, buoyant
  plumes) are dropped, not ported.
- EvoGym-style locomotion framing recedes; RTS-style tasks (harvest,
  dam/flood, territory spread, unit-vs-field combat) become the
  curriculum's task vocabulary.
