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

## Game note

Oscillating elements = clocks, pumps, signal lines; hex spiral waves are
visually spectacular. Coarse-scale support means giving each tile (R,
theta) state — tiles become pendulums on the map — with the Kuramoto
order parameter as the moment-closure analogue.
