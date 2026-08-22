---
kind: concept
name: "oscillatory-closure-limits"
status: seedling
added: "2026-08-22"
sources: []
related_concepts: ["reaction-closure-damkohler", "adaptive-surrogate-fallback", "learned-closure-strategy"]
related_experiments: []
tags: [chemistry, oscillators, closure, kuramoto, limits]
---

# oscillatory-closure-limits

## Question

How does tile-lumping fare on oscillating reactions? Measured with a
Brusselator (a=1, Hopf threshold b=2) reaction-diffusion on the 2257-cell
hex composite vs 61 lumped tiles, frozen spatial heterogeneity in b,
diffusion D as the coupling dial (diag17, 2026-08-22).

## Measured: two regimes, and the criterion is SYNCHRONY, not wavelength

1. **Sync-dominated regime** (weak heterogeneity b±0.125, any D down to
   0.02): coherence rho = tile-mean amplitude / cell amplitude stays
   0.89-0.99; the lumped model is essentially exact (1.0-1.1x). Diffusive
   coupling entrains limit-cycle oscillators far more readily than
   naively expected — prediction of easy decoherence DENTED. A tile of
   synchronized oscillators IS one oscillator; lumping is legitimate.
2. **Decoherent regime** (heterogeneity spanning the Hopf threshold,
   b=2.5±0.8, weak coupling D<=0.05): coherence collapses to 0.21-0.25.
   The lumped tile then reports oscillation **5-5.5x too strong** at a
   frequency **~40% too low** (0.153 vs truth tile-mean 0.22-0.27 — the
   surviving signal in a dephased ensemble is dominated by the fast,
   large-amplitude cells, while the lumped model oscillates at the
   mean-parameter frequency). Truth also contains non-oscillating cells
   (b<2) and partial amplitude death — states the lumped tile cannot
   represent at all.

This is the Kuramoto transition wearing our hex costume: the validity
criterion is coupling strength vs frequency spread (+ whether
heterogeneity crosses a bifurcation), not merely wavelength vs tile size.

## Why the existing closure ladder cannot fix the decoherent regime

- A static multiplier (naive/moment/GNN eta) corrects a RATE; phase
  cancellation is not a rate error. The same coarse mean can be a
  dispersing or a converging phase ensemble with different futures — the
  tile mean is **non-Markovian**. This is an information limit of the
  state, not a capacity limit of the model (same class of finding as the
  MLP-vs-table ceiling).
- Fix = add state, not smarter maps: per-tile complex order parameter
  (amplitude R, phase theta) via phase-reduction theory (Kuramoto /
  Ermentrout) — each coarse tile literally becomes a small oscillator —
  or a learned recurrent latent (the retained-internal-modes rung, which
  transients made optional and oscillations make mandatory).
- ISAT/trust regions: recurrence is actually FRIENDLY (bounded attractor
  -> finite coverage), but audits must score statistical/spectral
  invariants (amplitude, frequency, wave speed), because pointwise state
  error grows without bound under any small frequency mismatch (phase
  drift), even for a closure whose climate is perfect.

## SPICE board + oscillations (2026-08-22, diag19 on the ported world)

Brusselator RD on the ported composite: fine truth (1525 cells in 61
chambers) vs a 61-node SPICE lane (one (u,v) oscillator per tile, coupled
via diffusivity-scaled port-hop conductances G_D = G_HOP*D/K).

| regime | chamber coherence | SPICE amp | SPICE freq | phase corr |
|---|---|---|---|---|
| mild random het (b±0.125, D=0.08) | 0.97 | 1.04x | 1.01x | 0.30 |
| wild random het (b±0.8, D=0.05) | 0.58 | 2.86x | 1.03x | -0.02 |
| wild random het (b±0.8, D=0.01) | 0.29 | 4.38x | 0.67x | 0.06 |
| extreme (b±1.2, D=0.003) | 0.25 | 4.47x | 0.48x | 0.03 |
| **ENGINEERED: uniform b per chamber, ±0.8 ACROSS chambers, D=0.01** | **0.95** | **0.97x** | **1.00x** | **0.94** |

Findings:
1. **Ports do not rescue wild heterogeneity** — prediction falsified
   (again). Like-for-like vs wide seams: 2.86x vs 2.6x (D=0.05), 4.4x vs
   5.0x (D=0.01) — identical within noise. Proof that this failure mode
   lives in within-tile state representation, which interface
   architecture cannot touch. Chambers internally decohere by the same
   Kuramoto criterion; internal coupling is the same D, and 5-cell
   chambers do not sync a Hopf-crossing spread at weak D.
2. **The engineered case is essentially exact.** Same spread, same weak
   coupling, but uniform material per chamber: coherence 0.95, amplitude
   0.97x, frequency 1.00x, phase correlation 0.94, and all 11 dead
   chambers (b<2) correctly dead in both worlds. A component made of one
   material is internally coherent BY CONSTRUCTION, so one oscillator per
   tile is the true model, not an approximation. The ports philosophy
   extends from space to dynamics: engineered = coherent = representable;
   wild = decoherent = needs sub-tile state.
3. **Phase drift is the residual even when climate is perfect**: mild
   random regime tracks amplitude/frequency but phase r=0.30 (b-bar is
   not exactly the collective frequency of a heterogeneous chamber ->
   slow drift). In the engineered case frequencies are exact and r=0.94.
   If deterministic phase matters, the fix is PLL-style re-sync from
   periodic device checks — machinery the ports page already has.

## Game note

Oscillating elements = clocks, pumps, signal lines; hex spiral waves are
visually spectacular. Coarse-scale support means giving each tile (R,
theta) state — tiles become pendulums on the map — with the Kuramoto
order parameter as the moment-closure analogue.
