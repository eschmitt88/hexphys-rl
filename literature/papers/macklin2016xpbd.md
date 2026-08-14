---
kind: paper
title: "XPBD: Position-Based Simulation of Compliant Constrained Dynamics"
authors: ["Miles Macklin", "Matthias Müller", "Nuttapong Chentanez"]
institutions: ["NVIDIA"]
year: 2016
venue: "MiG '16 (ACM SIGGRAPH Conference on Motion in Games)"
peer_reviewed: true
url: "http://dx.doi.org/10.1145/2994258.2994272"
code_url: null
citations: null
source: "raw/papers/macklin2016xpbd.pdf"
added: "2026-08-14"
relevance: 5
credibility: 4
status: read
related_experiments: []
related_concepts: ["xpbd-compliant-constraints", "iteration-count-independent-stiffness", "solver-parallelism-vs-stability", "constraint-force-readout"]
tags: [xpbd, pbd, constraint-solving, compliant-dynamics, simulation]
---

# XPBD: Position-Based Simulation of Compliant Constrained Dynamics

## TL;DR

XPBD extends Position-Based Dynamics (PBD) with a per-constraint Lagrange
multiplier and compliance term (inverse stiffness) so that constraint
stiffness becomes independent of time step and iteration count — the same
asset behaves consistently whether you run 1 solver iteration or 1000, which
the original PBD algorithm cannot do.

## Claims

- Traditional PBD (Müller et al. 2007) has constraint stiffness coupled to
  time step size and iteration count: more iterations or smaller time steps
  make constraints arbitrarily stiffer, which is a poor authoring property
  and breaks scenes mixing soft and stiff materials.
- Adding a single scalar Lagrange multiplier per constraint plus a compliance
  parameter α (inverse stiffness, block-diagonal) gives constraints a direct
  correspondence to well-defined elastic and dissipation energy potentials,
  and makes convergence behavior (not final stiffness) the only thing
  iteration count affects.
- The algorithm is a near-trivial extension of standard PBD: Algorithm 1 adds
  only 3 lines (initialize multipliers, compute Δλ, update λ) to the standard
  PBD loop.
- XPBD is derived from an implicit Euler discretization of the constrained
  equations of motion and is validated against a reference nonlinear Newton
  solver, matching it closely.
- Per-constraint force/multiplier storage (one extra scalar per constraint)
  enables constraint force estimates usable for haptics or breakable joints —
  a feature PBD lacks.
- Damping can be added via a Rayleigh dissipation potential with an
  independent damping-stiffness parameter β (not an inverse/compliance-style
  parameter).

## Methods

- Core update (Gauss-Seidel form, Eq. 26):
  Δλ_j = [-C_j(x_i) - α̃_j λ_ij - γ_j ∇C_j·(x_i - x^n)] /
  [(1+γ_j)∇C_j M⁻¹∇C_j^T + α̃_j], where α̃ = α/Δt² is the time-step-scaled
  compliance and γ = α̃β̃/Δt combines compliance and damping.
- When α=0 (rigid constraint), the update reduces exactly to standard PBD's
  scaling factor s_j — XPBD is a strict generalization.
- Approximates the stiffness matrix K ≈ M (mass matrix), discarding geometric
  stiffness/constraint-Hessian terms (local error O(Δt²)); keeps the method
  as cheap as PBD (no Hessian evaluation) while only mildly affecting
  convergence rate, not correctness of the fixed point.
- Both Gauss-Seidel (sequential, CPU, 2D validation) and Jacobi (parallel,
  GPU, 3D results on a GTX 1070) iteration schemes are demonstrated — Jacobi
  is the relevant one for a vectorized/batched engine since constraints
  update independently within an iteration.
- Cantilever beam (FEM) validated with a compliant reformulation of a linear
  isotropic St Venant-Kirchhoff constitutive model via a compliance matrix
  built from Lamé parameters, correctly coupling strains (Poisson effect)
  rather than treating them independently as in prior strain-based PBD.

## Results

- Harmonic oscillator (spring, α=0.001): PBD's oscillation period and damping
  depend heavily on iteration count and never match the analytic solution;
  XPBD matches the analytic solution closely regardless of iteration count.
- Hanging chain (20 particles, α=10⁻⁸, 100 frames): max relative error in
  constraint force at the fixed support vs. reference Newton solver was 6% at
  50 iterations, 2% at 100, 0.5% at 1000.
- Cantilever beam (triangular FEM, E=10⁵, ν=0.3, Δt=0.008, 50 frames): 20
  XPBD iterations visually indistinguishable from the reference Newton solver.
- Cloth (64×64 grid, ~24k distance constraints): PBD becomes progressively
  stiffer and more damped as iteration count increases (20/40/80/160 tested);
  XPBD's qualitative behavior is unchanged across the same range.
- Performance overhead of XPBD vs. PBD on the cloth example: <2% of total
  per-step simulation time (e.g. PBD 0.95 ms vs XPBD 0.97 ms at 20
  iterations; 5.61 ms vs 5.65 ms at 160).

## Critique / open questions

- Directly load-bearing for the "trade error tolerance for speed" design:
  under-converging (fewer iterations) still gives a *consistent*, just less
  accurate, physical behavior — stiffness doesn't drift with iteration count
  the way vanilla PBD's does. A fast, deliberately-low-iteration hex-lattice
  solver can dial iteration count as a pure speed/accuracy knob without
  re-tuning every bond's stiffness whenever the iteration budget changes.
- The K ≈ M approximation (dropping geometric stiffness and constraint
  Hessians) is precisely the kind of simplification a game-engine solver
  wants: cheaper per-iteration cost, parallelizable, no Hessian assembly —
  good fit for a JAX-vectorized Jacobi-style update.
- Caveat: convergence *rate* at genuinely tiny iteration budgets (1-3
  iterations, likely needed for a real-time RL env with hundreds of parallel
  lattices) is not characterized; the paper's low-iteration demos (cloth at
  20 iterations) still spend more than a heavily vectorized engine may want.
- The Gauss-Seidel vs. Jacobi distinction matters directly: Jacobi is what a
  batched JAX solver would use, but the paper gives less quantitative
  validation for the Jacobi variant — worth characterizing Jacobi convergence
  at very low iteration counts before relying on it.
- Damping parameter β is independent of compliance α — useful for tuning bond
  dissipation in the "push" (rest-length actuation) mechanic without
  conflating stiffness and energy loss.
- No public code released with the paper (NVIDIA internal implementations
  only); reimplementation needed from the equations, though the derivation is
  short and explicit enough that this is low risk.
- Contact constraints are simplified to zero compliance in the 3D results —
  the glue/unglue bond mechanic is closer to the compliant elastic
  constraints (springs/distance constraints) than to contact, so the
  harmonic-oscillator and chain validations are the more directly relevant
  reference cases.

## Trust signals

- **Credibility:** 4 — Peer-reviewed (MiG '16, ACM), from NVIDIA research
  (Macklin/Müller/Chentanez are established PBD/simulation authors),
  reproducible derivation with explicit equations and a reference-Newton
  validation baseline; docked one point for no released code and unconfirmed
  citation count (Semantic Scholar rate-limited; left null rather than
  guessed) — informally this is one of the most widely adopted real-time
  simulation methods (NVIDIA Flex/PhysX, Unity, Unreal Chaos), so actual
  influence is likely higher than the score alone suggests.

## Follow-up

- **Relevance:** 5 — This is the exact numerical method the project's spec
  names ("solved XPBD-style, error tolerance traded for speed"); the core
  result (iteration-count-independent stiffness, cheap Jacobi-friendly
  update, explicit low-iteration error curves) directly seeds the solver
  design and the tunable "under-converge on purpose" knob central to the
  engine's performance model.
- Characterize Jacobi-XPBD error at 1-8 iterations on the hex lattice as one
  of the first engine experiments.
- Use the accumulated λ per bond as the bond-stress signal for breakage
  thresholds and chemistry, rather than re-deriving stress from positions.
