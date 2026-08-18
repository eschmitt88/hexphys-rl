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

## Built: rungs 1 and 2 (2026-08-18, harness6/harness7)

**Rung 2 — grid-convergence verification (the CFD discipline we lacked).**
Flowers of radius 1/2/3 all tile the plane, so the same problem re-tiles at
three coarse resolutions. A-priori flux error vs exact fluxes:

| tile radius | cells/tile | tiles | single-g | per-seam (donor,receiver) |
|---|---|---|---|---|
| 1 | 7 | 301 | 7.4% | 3.0% |
| 2 | 19 | 91 | 13.2% | 3.3% |
| 3 | 37 | 61 | 27.2% | 6.7% |

Observed order ~1.6 (single-g) and ~1.0 (per-seam). **Prediction falsified
again**: the lumped closure is NOT an O(1) correlation — it is a convergent
discretization. Error is controllable by two independent dials (tile size,
feature richness), which legitimises the whole multiscale scheme.

**Rung 1 — per-seam keying, and the a-priori/a-posteriori disconnect.**
Records are now per-element, per-seam tables keyed on (own head, neighbour
head); the donor/receiver asymmetry that directional k's tried to capture is
subsumed by the key, and both sides of an interface look up the same pair, so
conservation is structural. In-place refinement (blend on repeat visits to the
same operating point) keeps the store bounded — 749 records total, largest
seam table 154.

Result: a priori flux error improves 4x (27.2% -> 6.7%). **A posteriori error
does not move**: 15.5% before, 15.8% after; compute 6x, learning curve 98x
decay, all other properties preserved. This is a documented hazard in LES
subgrid modelling — better a-priori scores routinely fail to translate,
because in a live run the coarse state drifts and flux errors interact
instead of accumulating independently. Lesson for this project: **a-priori
tests rank model FORMS but cannot predict runtime accuracy; only the twin
can.**

Next lead (measure, don't guess): the residual bias is systematic — the
surrogate world holds ~18% less fluid than truth in every configuration
tested. That is a storage/boundary-coupling signature, not a flux-law one.

## Diagnosis of the residual bias (2026-08-18, diag10-12) — root cause found

The systematic ~15% error / ~12-18% fluid deficit was chased through four
hypotheses. Three died; the fourth is proven.

1. **Jensen gap** (surrogate evaluates sat(<h>), truth transports <sat(h)>;
   sat is concave so the surrogate should over-conduct): sign is right in 95%
   of interfaces but magnitude is only **0.4%** — most cells sit far above the
   saturation threshold, so the nonlinearity rarely bites. Swapping <sat(h)>
   for sat(<h>) in the flux model changed error 25.2% -> 25.2%. DEAD.
2. **Non-negativity clamps destroying mass**: instrumented both worlds —
   **exactly 0.0** mass destroyed over 3000 ticks. DEAD.
3. **Transient calibration** (dwell 22 ticks vs internal diffusion time ~204):
   clean-room test shows k converges fast under seam drive — 0.314 at 22 ticks
   vs 0.305 settled, only **3%** high. Not the cause, though the estimator was
   independently wrong (see below). DEAD as the dominant term.
4. **The star model cannot define a per-seam conductance at all in a live
   multi-port environment.** PROVEN. Measuring the *same tile* settled under
   different port conditions:

   | port condition | measured per-seam k |
   |---|---|
   | 2-port (others insulated) | 0.305, 0.305 |
   | 6-port, mild spread | 0.486, 0.149, 0.486, 0.149 |
   | 6-port, wide spread | 0.513, **-1.906**, **-0.533**, 0.454, 0.291, 0.154 |

   **Negative conductance is physically impossible** — it appears because flux
   entering one seam largely *leaves by another* rather than charging storage,
   and the star form books all of it as storage exchange. The quantity being
   calibrated is not a well-posed property of the tile. This is the same
   deficiency the open element's 65% star fit reported back in v2; in-situ
   calibration in a 6-port world converts it into a 17-23% conductance
   inflation, hence the drainage bias.

Also fixed en route (correct regardless): conductance was estimated by
**averaging the ratio F/dh**, whose right tail dominates when dh is small —
replaced with regression through the origin, k = S(F*dh)/S(dh^2). Statistically
correct; it did not move the runtime error, because the cause is model form.

**Next build, now well-posed**: records must hold the full port matrix (the
6x6 M = G - S used by the v2.2 full-G interface solver, which is already built,
conservative, and exact at steady state) instead of 6 scalars, looked up by
operating point. Cross-seam through-flux is then represented rather than
misattributed. This is the one remaining rung with direct evidence behind it.

## Connections

- Direct successor to [[tile-homogenization]]: same characterization
  machinery, now triggered on demand instead of once.
- Nonlinear castings (pipe-model water with drying, chemistry) are the
  first case where this earns its keep.
