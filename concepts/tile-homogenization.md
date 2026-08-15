---
kind: concept
name: "tile-homogenization"
status: growing
added: "2026-08-15"
sources: []
related_concepts: ["axial-hex-storage", "virtual-pipe-shallow-water", "unconditional-stability-via-dissipation", "differentiable-physics-vs-rl-signal"]
related_experiments: []
tags: [multiscale, homogenization, game-design, foundry]
---

# tile-homogenization

## Definition

The Foundry mechanic: a designed fine-grid tile (walls/empty + flow) is
probed under an ensemble of boundary conditions and reduced to an
empirical element — capacity + per-side star conductances (+ the full 6×6
conductance matrix G) — which then runs at the coarse scale. This is
computational homogenization (RVE/FE², Darcy upscaling, Thévenin
reduction) as a game loop.

## Why it matters here

Measured facts from the prototype (docs/foundry.html, harness4):

- One linear fluid ⇒ G is exact, symmetric, rows sum to 0 (verified to
  measurement precision); nonlinearity residual 0.00%.
- **Star-model ceilings are real, not bugs**: an open tile star-fits only
  ~75% (diffusion favors adjacent sides over the far side — structure a
  star cannot express); a strongly cross-coupled channel ~59%.
- **Hex face geometry**: the tile's flat faces sit 30° rotated from the
  coarse neighbor directions, so every boundary cell's outward edges span
  two coarse directions — single-direction ports are impossible, corners
  couple three; anisotropy comes from interior routing (~1.8× measured
  for a wide-mouth channel).
- **Twin test methodology**: lockstep fine-composite (stitched tile
  graphs) vs coarse surrogate with live error + per-step timing is the
  honest validator — measured 11.7% head error at 49× speedup on a
  dam-with-channel-gap world; sealed dams stay sealed at both scales.

## The nesting fix: flower tiling (aperture 3R²+3R+1)

Radius-R hex flowers (3R²+3R+1 cells — always Löschian: a=R, b=R+1) tile
the plane exactly on a √(3R²+3R+1)-scaled hex superlattice rotated by a
fixed angle per level (~19.1° for R=1 — H3's aperture 7; ~25.3° for our
R=3, aperture 37). Proof sketch: nearest supercenters sit at hex distance
2R+1 ⇒ flowers disjoint; density counting ⇒ no gaps. Consequences for
Foundry v2: the composite fine world is a *true* lattice (no stitched
graphs); each of the tile's 6·(2R+1) outward edges pairs with exactly one
neighbor flower, so the six 7-edge seams are honest single-neighbor
interfaces — the "every side spans two coarse directions" problem
dissolves and directional ports become buildable; recursion is exact at
every level. Cost: the grid rotates ~25° per zoom level (compounding);
render each level in its own frame and mind element orientation.

## Connections

- Next fidelity rung: full-G coarse runtime via under-converged interface
  Jacobi sweeps — the project's standard dial, again.
- Foundry v2: seam-based characterization on the flower tiling (above).
- Level-3 recursion (characterize a coarse world as an element) is where
  error compounding becomes the experiment.
- Inverse design ("build a tile matching a target G") is a natural RL
  task with the twin error as reward.
