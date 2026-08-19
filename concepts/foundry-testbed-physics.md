---
kind: concept
name: "foundry-testbed-physics"
status: seedling
added: "2026-08-19"
sources: []
related_concepts: ["tile-homogenization", "adaptive-surrogate-fallback", "virtual-pipe-shallow-water", "lbm-collide-stream"]
related_experiments: []
tags: [physics, porous-media, scope]
---

# foundry-testbed-physics

## Definition

What the Foundry's fine simulation actually solves (diag14, 2026-08-19):

    dh/dt = div( K * sat(h) * grad h ),   sat(h) = min(1, h/h0),  K=0.12, h0=0.30

with donor-side (upwind) mobility. This is **degenerate nonlinear diffusion**
with two named regimes:

- **h >> h0**: sat=1, so `dh/dt = K lap h` — the linear heat/diffusion equation,
  infinite propagation speed.
- **h < h0**: sat = h/h0, so `dh/dt = (K/2h0) lap(h^2)` — the **porous medium
  equation** (m=2), a.k.a. the Boussinesq unconfined-aquifer equation. Compact
  support, finite propagation speed, sharp wetting fronts.

**Measured confirmation**: a dry blob spreads as R ~ t^0.22 (Barenblatt predicts
t^0.25 for m=2 in 2-D; the small deficit is finite-blob/finite-domain effect).
The same blob with mobility forced to 1 fills the entire domain between the
first two checkpoints — infinite propagation speed, exactly as linear diffusion
requires.

## What it is and is not

- **Single phase.** One mobile fluid. The sat() curve is a relative-permeability
  /mobility law — a two-phase *concept* with the second phase (air) assumed
  infinitely mobile at constant pressure, which is precisely the simplification
  that turns two-phase flow into **Richards' equation**. Donor-side evaluation
  is standard reservoir-simulator upstream weighting.
- **No viscosity in the Navier-Stokes sense.** There is no momentum equation, no
  velocity state variable, no inertia, no advection, no Reynolds number, no
  boundary layer, no vorticity. Viscosity enters only folded into K = k/mu
  (permeability over dynamic viscosity): changing mu just rescales K, which
  rescales time. It is a Darcy-type model, not a momentum-carrying fluid.
- **No gravity** (top-down), no free surface, no surface tension, no
  compressibility beyond linear storage, no chemistry, no temperature.

## Why this matters for the multiscale program

Upscaling Darcy flow is *the* canonical homogenization success story — it is
where effective-permeability/RVE theory comes from — so the surrogate machinery
is being tested on friendly ground. The 7.7%-at-6x result sits at the easy end
of the difficulty spectrum. Momentum-carrying flow (the Fields dashboard's
Stable Fluids, the Arena's D2Q7 LBM) is qualitatively harder to upscale:
inertia destroys scale separation and turns closure into turbulence modelling.

Sibling physics already built elsewhere in the project, for contrast:
- **Stable Fluids** (docs/fields.html): true incompressible Navier-Stokes —
  momentum, advection, pressure projection, real viscosity.
- **D2Q7 LBM** (docs/arena.html): momentum + viscosity via relaxation time tau,
  vortex shedding, a genuine stability floor.
- **Virtual-pipe shallow water** (docs/arena.html): gravity-driven hydrostatic
  head, terrain coupling.

The Foundry deliberately uses the simplest physics that still has a
nonlinearity, because the object under test is the homogenization machinery,
not the fluid.
