---
kind: concept
name: "adaptive-surrogate-fallback"
status: seedling
added: "2026-08-18"
sources: []
related_concepts: ["tile-homogenization", "unconditional-stability-via-dissipation", "differentiable-physics-vs-rl-signal"]
related_experiments: []
tags: [multiscale, surrogate, active-learning, uncertainty, game-design]
---

# adaptive-surrogate-fallback

## Definition

The Foundry's intended runtime: a tile is simulated in full detail until a
surrogate can be fitted; thereafter the surrogate is used *only while the
tile's operating point lies inside the model's validity region*. On
leaving it, the engine falls back to fine simulation for that tile, adds
the result as a calibration point, and refits — so machines get
progressively cheaper to simulate as they become well-understood.

## Prior art (near-exact matches)

- **ISAT** (Pope 1997, in situ adaptive tabulation, combustion): stores
  local linearization + *ellipsoid of accuracy* per record;
  retrieve/grow/add loop. This is the same algorithm, in production for
  ~30 years.
- **HMM** (E & Engquist) and **equation-free / gap-tooth** (Kevrekidis):
  run fine models only in on-demand patches.
- **On-the-fly MLIPs**: learn-on-the-fly (Csányi et al.), MTP
  extrapolation grade via D-optimality (Shapeev/Podryabinkin), FLARE
  GP-variance triggers (Vandermause et al.) — the modern uncertainty-
  triggered active-learning loop.
- **Adaptive QM/MM**, **AMR** (Berger & Oliger, refinement in space),
  climate **superparameterization** (embedded fine model per cell).
- Reduction side: **POD/DMD**, **Craig-Bampton**, deep material networks,
  neural operators.

## Design constraints (identified 2026-08-18)

1. **Uncertainty > surrogate.** Fitting is easy; knowing you left the
   region is what fails silently. Input space here is small (6 seam heads,
   12 with tilt modes) ⇒ GP variance or ISAT ellipsoids suffice; NNs only
   once chemistry/materials expand the space. Pair learned UQ with
   physics invariants (conservation, reciprocity/G symmetry, monotonicity)
   which distribution shift cannot fool.
2. **Internal state is the hard dimension.** Surrogate is a dynamical
   system (state, boundary) → (dstate, flux), not a static map. Current
   1-node storage is why transient error ~7%. Retained internal mode count
   is the real cost/accuracy dial (Craig-Bampton in time).
3. **Linear physics ⇒ region is infinite**, machinery never fires. Build
   the mechanism now against the verifiable linear case; it comes alive
   with drying/chemistry/multi-material.
4. **Determinism**: switching must be a pure function of game state, never
   wall-clock or available compute (lockstep multiplayer breaks otherwise).
5. **Hysteresis / dwell times** or a cell thrashing across the boundary
   costs more than either mode.
6. **Adversarial compute**: chaotic builds are a DoS vector in multiplayer
   ⇒ per-player compute budgets + graceful degradation, never stalls.
7. **Staleness**: version surrogate records by structure hash; any edit
   invalidates.
8. **Cross-level compounding**: calibrating level N against level N-1
   surrogates means "confident given the layer below was right" —
   periodically calibrate against fine truth or propagate error budgets.

## Game-design payoff

Compute becomes a first-class resource: predictable machines are cheap,
chaotic ones expensive, mastery = compression. A visible "understanding
map" (hot = simulated in detail / novel, cold = reduced to formula) is
both visualization and strategy layer. First-build cost has a diegetic
home: **commissioning** — a new machine's shakedown run *is* the fine
simulation, amortized in background ticks.

## Connections

- Direct successor to [[tile-homogenization]]: same characterization
  machinery, now triggered on demand instead of once.
- Nonlinear castings (pipe-model water with drying, chemistry) are the
  first case where this earns its keep.
