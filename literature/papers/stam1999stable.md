---
kind: paper
title: "Stable Fluids"
authors: ["Jos Stam"]
institutions: ["Alias|wavefront"]
year: 1999
venue: "SIGGRAPH 1999"
peer_reviewed: true
url: "https://www.dgp.toronto.edu/public_user/stam/reality/Research/pdf/ns.pdf"
code_url: null
citations: null
source: "raw/papers/stam1999stable.pdf"
added: "2026-08-14"
relevance: 5
credibility: 5
status: read
related_experiments: []
related_concepts: ["semi-lagrangian-advection", "under-converged-pressure-projection", "unconditional-stability-via-dissipation"]
tags: [eulerian, fluid-simulation, navier-stokes, semi-lagrangian, pressure-projection, numerical-viscosity]
---

# Stable Fluids

## TL;DR

Stam proposes the first unconditionally stable Navier-Stokes solver for
computer graphics: replace explicit finite-differencing (stable only for
small Δt) with an implicit pipeline — add forces, semi-Lagrangian ("method
of characteristics") advection, implicit diffusion, and a Helmholtz-Hodge
pressure projection — so simulations never blow up regardless of time-step
size, at the cost of excess numerical dissipation.

## Claims

- The full Navier-Stokes equations can be solved stably at arbitrarily
  large Δt for real-time/interactive use, trading physical accuracy for
  guaranteed stability — the opposite priority ordering from engineering
  CFD.
- Any vector field decomposes uniquely (Helmholtz-Hodge) into a
  divergence-free part plus a gradient field: w = u + ∇q. A projection
  operator P recovers the divergence-free component via a Poisson solve.
- Prior explicit Eulerian schemes (Foster & Metaxas) are conditionally
  stable: Δt must satisfy a CFL-like bound Δt < Δx/|u|, so fast flow or
  fine grids force tiny steps or blowup.

## Methods

One step w0 → w1 → w2 → w3 → w4; each stage may leave the divergence-free
space, the final projection restores ∇·u = 0:

1. **Add force** — explicit Euler add, forces constant over the step.
2. **Advect** (semi-Lagrangian / method of characteristics) — w2(x) =
   w1(p(x, -Δt)): trace the characteristic backward from each cell center
   through the previous velocity field, then linearly interpolate the old
   field there. The key stability trick: new values are *interpolations* of
   old values, never extrapolations, so the scheme cannot amplify — the max
   of the new field is bounded by the max of the old, for any Δt.
   (Higher-order interpolants were tried and reintroduce overshoot
   instability.)
3. **Diffuse** — implicitly: (I − νΔtΔ)w3 = w2, unconditionally stable in
   Δt regardless of viscosity.
4. **Project** — Poisson solve ∇²q = ∇·w3 (Neumann BC), then w4 = w3 − ∇q.

Two BC regimes: periodic (exact spectral FFT solution, O(N log N)) and
fixed walls (FISHPAK's POIS3D direct fast-Poisson solver; multigrid noted
as "theoretically optimal" but not used). Scalars (smoke/density) transport
through the same pipeline. Reference implementation ~500 lines of C;
pseudocode given in full.

## Results

Interactive 2D/3D solvers on an SGI Octane (R10K, 192 MB RAM) at 16³–100²
grids with mouse-driven force/density injection; volume-rendered smoke/fire
with texture-advected detail. No quantitative accuracy benchmarks — success
is demonstrated visually, consistent with the paper's stated priority
(visual plausibility over physical accuracy).

## Critique / open questions

- **Unconditional stability = error reshaped, not removed.** Stam is
  explicit that the method "suffers from too much numerical dissipation" —
  semi-Lagrangian backtracing with linear interpolation is a low-pass
  filter; every advect step smooths the field regardless of Δt. This maps
  precisely onto XPBD's under-iteration dial: XPBD trades convergence for
  softness; Stable Fluids trades resolution/interpolation order for
  numerical viscosity, and an under-converged projection for slight
  compressibility. Both are "stability first, accuracy is the error budget"
  designs.
- **The load-bearing gap between paper and plan:** Stam's own
  implementations do NOT use under-converged iterative projection — the
  periodic solver is spectrally exact and the wall solver is a direct fast
  solver; he even argues against few-iteration relaxation (while noting
  Foster & Metaxas got acceptable visuals with very few). A real-time
  game/RL engine picks the local, parallelizable, warm-startable iterative
  scheme (Jacobi/red-black) deliberately under-converged, because slightly
  non-divergence-free is visually and behaviorally indistinguishable at
  frame rate. Our engine adopts that regime — this paper proves the
  stability of the pipeline around it, not the regime itself.
- **Known inherited artifacts:** repeated interpolation kills small
  vortices (flow visibly "calms down"); the standard production fix is
  vorticity confinement (Fedkiw et al. 2001) — re-inject curl-aligned
  force. A likely future experiment if the base scheme feels too damped.
- **Implementable reference:** Stam's follow-up "Real-Time Fluid Dynamics
  for Games" (GDC 2003) strips this to one dense grid + Gauss-Seidel
  relaxation in ~a page of C — the more directly portable reference for a
  first implementation; worth fetching separately.

## Trust signals

- **Credibility:** 5 — SIGGRAPH 1999 (top peer-reviewed venue); the scheme
  became the de facto standard for real-time fluid graphics, adopted and
  extended by essentially every subsequent game/VFX solver; algorithm fully
  specified via pseudocode and independently reimplemented thousands of
  times. Citation count not established at ingest (rate-limited) — left
  null, though it is one of the most cited graphics papers.

## Follow-up

- **Relevance:** 5 — the literal seed algorithm for the Eulerian track:
  semi-Lagrangian advect + iterative pressure projection is exactly the
  planned solver, and "unconditional stability via excess dissipation" is
  the field-solver analogue of the under-converged-XPBD dial.
- Fetch "Real-Time Fluid Dynamics for Games" (GDC 2003) and Fedkiw et al.
  "Visual Simulation of Smoke" (2001, vorticity confinement) when the JAX
  port starts.
