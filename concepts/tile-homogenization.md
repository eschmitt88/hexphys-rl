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

## v2 measurements (flower tiling, seam characterization)

Foundry v2 implements the aperture-37 fix. Harness results: the 42 outward
edges partition into 6 disjoint seams of 7 (verified); 61 flowers stamp
into ONE fine lattice with zero collisions — the tiling theorem checked
computationally every twin build. Seam-based channel reaches 2.3×
directionality (vs 1.8× with v1's overlapping sides); open-flower star fit
68% (seam adjacency structure differs from v1's 75%); twin error 14.7% at
26× measured speedup; sealed dams watertight at both scales. Residual
truth: seams meet at rim cells, so ports are exact per-edge, approximate
per-cell.

## The identification lesson (v2.1 — the big one)

The user called the twin "leaky" at 14.7% error; the diagnosis found the
surrogate ~3× too conductive, and at quasi-steady state the error was
actually 50%. Root cause: **G answers the wrong question for an RC
network.** The all-seams-clamped G experiments let flux take rim shortcuts
between adjacent clamped seams (2-3 cells), inflating G's diagonals ~3×
over genuine through-tile transport; a star fitted to G builds a coarse
network that transports ~3× too fast. Fix: identify the star from **15
two-port through-transport experiments** — clamp seam i high, seam j low,
other seams *insulated* (matching runtime conditions, where neighbors are
not clamped shorts) — the literal quantity the coarse series pairing uses.
Plus: half-edge (2K) referencing to seam midpoints so series pairing
reconstructs each seam edge exactly; monotone grid-search star fitting
(naive fixed-point iteration found degenerate alternating solutions); L2
regularization to break the 2-port harmonic degeneracy symmetrically; and
channel mouths built from *pure* seam cells (every outward edge in one
seam) — junction rim cells leak to adjacent seams.

Results after: channel directionality **1783×** (was 2.3×) with star fit
99.9% — true 2-ports exist now; twin error **2.0% quasi-steady / 7.3%
transient** (was 50% / 14.7%) at 26× speedup. General principle, the
closure problem in miniature: *identify the surrogate under the boundary
conditions it will actually run in.*

## v2.2: the user's leak, chain calibration, and the G surprise

User-reported 32% error on the default all-open world was real and steady
(fine held 7,671 fluid units vs surrogate 5,306 — the surrogate ~1.45×
too conductive). The dam benchmark had hidden it: bottleneck-dominated
worlds are easy; homogeneous continua are the hard case. Two findings:

1. **Even exact G over-mixes on composition.** Wiring the full 6×6 G into
   the runtime (interface heads solved by warm-started under-relaxed
   Jacobi; model F = (G−S)u + k(u−h_storage), exact at steady state,
   conserving because G and S have zero column sums) only improved
   all-open to ~18%: G itself was measured with uniform-head seams, and a
   single scalar head per seam acts as a perfect conductor along the seam
   line — free extra conductance under oblique flow.
2. **Chain calibration fixes it.** Per direction, simulate real 2- and
   3-tile composites of the element at full resolution and extract the
   per-hop conductance by length-differencing (1/g = 1/T₃ − 1/T₂ — end
   effects cancel exactly). Star runtime with chain-calibrated k:
   all-open 7.6%, dam 1.2% at 12k steps, 33× speedup; open-tile
   k = 0.303 (vs 0.44 from single-tile probes — the ~1.45× bias, found).

Closure principle, now learned three times: calibrate the surrogate under
the boundary conditions it actually runs in — and composite calibration
beats any single-tile identification. Next fidelity rung: multi-node seam
interfaces (mean + tilt mortar bases) to lift the uniform-seam assumption
in both runtimes. Fine/coarse twin views are now rotation-aligned
(fine view rotated −25.3°, sz = 27/√37) for point-for-point comparison.

## Connections

- Next fidelity rung: full-G coarse runtime via under-converged interface
  Jacobi sweeps — the project's standard dial, again.
- Foundry v2: seam-based characterization on the flower tiling (above).
- Level-3 recursion (characterize a coarse world as an element) is where
  error compounding becomes the experiment.
- Inverse design ("build a tile matching a target G") is a natural RL
  task with the twin error as reward.
