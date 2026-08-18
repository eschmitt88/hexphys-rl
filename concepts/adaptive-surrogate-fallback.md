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

## Built and measured (v3, docs/foundry.html Station 04, harness6)

Nonlinearity: donor-side drying (conductance collapses below saturation
0.30) gives a 1.7x swing in tile conductance wet-vs-dry — one fitted k
provably cannot cover it. Runtime: trust-region lookup in 2-D operating
space (saturation, drive), ISAT grow/add, deterministic audits, fine
patches coupled cell-to-cell on the real composite lattice.

**Results**: novelty fallbacks 0.31 -> 0.00 per tick (68x decay); audit
traffic flat at 0.37/tick (permanent floor, by design); 83% of tiles on
surrogate; 378 of 2257 cells/tick (6x less physics) at 15.5% error vs the
truth running alongside; 600 records (capped), saturation coverage
0.00-2.07; forcing every tile bone-dry fires 45 fresh fallbacks.

**Four bugs the build surfaced, each a general lesson**:
1. *Imposed tiles poison calibration.* Source/drain fluxes are boundary
   conditions, not material response — a drain forced to zero implies
   infinite conductance. Never calibrate a sample while injecting into it.
2. *Novelty detection cannot see integration drift.* With input-space
   trust regions alone, error grew to 71% while the runtime stayed 97%
   confident: the surrogate's own trajectory keeps it in familiar input
   space while diverging from truth. Periodic audits (deterministic, so
   lockstep-safe) are not optional — they are the error-control mechanism.
3. *Per-tile flux computation is not conservative.* Each side of an
   interface using its own conductance loses mass at every seam (world
   held 1/4 of truth's fluid). Fix: fine-fine neighbours share the
   composite lattice directly, surrogate-surrogate exchange one symmetric
   harmonic flux, fine-surrogate uses the fine side's measured flux.
4. *Measure after the settling transient.* A freshly-flattened tile
   accepts fluid too eagerly; averaging the whole dwell window inflates k.

**Prediction falsified**: the residual is NOT transient-dominated (19.5%
near a pulse edge vs 20.1% in steady flow, 0.97x). Internal-state
compression is not the binding constraint; model form is. Next rung is
richer seam interfaces (mean+tilt), not retained internal modes.

## A-priori diagnosis of the residual (2026-08-18, diag8/diag9)

Question: is the ~15% adaptive error calibration, boundary conditions, or
model form? Answered with the LES-standard **a-priori test** — hand the
flux model the *exact* coarse state from truth and the best-fit
coefficient, then compare its predicted inter-tile fluxes with the exact
ones. 3432 interface samples across fill / steady / drain phases:

| flux model (exact coarse state, best-fit coefficient) | flux error |
|---|---|
| single global conductance (v2-style) | 29.6% |
| g(tile-mean saturation) — what v3 keys records on | 19.3% |
| g(DONOR saturation) — upwind, the actual physics | 15.7% |
| g(donor, receiver) — per-seam 2-D feature | **9.5%** |

**Verdict: model form, not calibration.** With a perfect coarse state and
the best possible coefficient, one scalar conductance is 29.6% wrong. The
cause is the nonlinearity: with saturation-dependent conductivity,
<f(h)> != f(<h>) — the classic closure problem, and literally the known-hard
problem of upscaling relative permeability in porous media.

**Why CFD does not suffer this**: two different regimes, which we
conflated. Where fields are smooth, CFD *discretizes* (cell averages +
consistent flux stencil, truncation error -> 0 as h -> 0, verified by
grid-convergence / MMS) rather than lumping. Where CFD *must* model
unresolved physics — RANS turbulence closures, wall functions, multiphase
relative permeability — it shows exactly our error magnitudes (10-30% on
separated flows is standard) and those closures are the acknowledged weak
point of the field. We are in the second business, so 15% is typical, not
anomalous.

**Scale separation is the missing criterion.** Homogenization is valid when
eps = l_micro/L_macro << 1 (Darcy: microns vs kilometres; RVE: grains vs
parts). A uniform medium homogenized into itself has *no* scale
separation — the one case where lumping buys nothing. This explains our
own data: dam+channel world 1.2% error (bottleneck-dominated, field nearly
piecewise-constant — what a lumped model represents well) vs all-open
world 15.5% (smooth continuum — the hard case). Same criterion as the
Biot number for lumped-capacitance validity in heat transfer.

**Evidence-backed ladder** (each rung measured, not guessed):
1. Key records on per-seam (donor, receiver) heads instead of a
   tile-global operating point: 29.6% -> 9.5% a priori. This is upwinding.
2. Carry a gradient so the donor's *local* head at the seam is known
   rather than its tile mean — attacks the remaining 9.5%.
3. MsFEM-style basis functions with oversampling for structured tiles.
4. **Run a grid-convergence study** (radius 2/3/6 tiles). CFD's
   credibility comes from verification that the scheme achieves its formal
   order; we have never run one, and a lumped closure would fail it by
   construction.

## Connections

- Direct successor to [[tile-homogenization]]: same characterization
  machinery, now triggered on demand instead of once.
- Nonlinear castings (pipe-model water with drying, chemistry) are the
  first case where this earns its keep.
