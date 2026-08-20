---
kind: concept
name: "reaction-closure-damkohler"
status: seedling
added: "2026-08-20"
sources: []
related_concepts: ["tile-homogenization", "adaptive-surrogate-fallback", "foundry-testbed-physics", "learned-closure-strategy"]
related_experiments: []
tags: [chemistry, closure, multiscale, damkohler]
---

# reaction-closure-damkohler

## Definition

Tier-1 chemistry on the hex lattice: three species, one irreversible
bilinear reaction `A + B -> C`, species carried by the existing
porous-medium flow with donor-cell upwinding, mass-action kinetics.
The coarse surrogate lumps each tile to one number per species, so its
reaction term is `k<A><B>` while truth integrates `k<A*B>` — and the
difference is exactly the within-tile covariance:

    k*<A*B> = k*( <A><B> + cov(A,B) )

## Measured (docs/reactor.html, harness8/diag16, 2026-08-20)

- **Physics is sound**: closed-system A+C and B+C conserved to 0.0002%;
  stoichiometry exact; fluid drift 0.0000%.
- **Front sharpening is real and visual**: the A/B coexistence zone shrinks
  763 -> 79 cells as the rate rises. Reactants annihilate on contact, so at
  high rate they can only meet in a thin sheet.
- **Transport closure is NOT the problem**: with reaction off, the coarse
  model tracks species to **3.9%**. The stirred-tank assumption is also
  fine — fluid leaves a tile within **1%** of the tile-mean concentration
  (a CSTR-vs-plug-flow hypothesis, tested and killed).
- **Reaction closure fails exactly as Damkohler theory predicts**:
  lumped/true reaction rate = **1.17x -> 12.23x** across the rate range
  (worst individual tile 27x). k cancels in that ratio, so it measures
  the field's internal structure alone.
- **cov(A,B) is negative in 7/7 settings** — streams arrive through
  different seams and stay separated inside the tile, so the mean-field
  guess sees mixing that is not there.

## Caveat that shaped the presentation

A running twin shows only ~70% product error, roughly flat in k, which
initially looked like "no Damkohler dependence". Cause: at high rate the
coarse world over-consumes reactants until production becomes
**supply-limited**, so both worlds converge on the same total yield and
differ in *where* the product is made. The rate ratio is the honest
metric; total yield hides the failure.

## Consequences

- This is the predicted trigger for a learned closure: coarse state per
  tile goes 1 -> 4 numbers, so the ISAT table's key space goes 2-D -> 8-D
  (115 bins -> ~1e5). See [[learned-closure-strategy]].
- Carrying second moments now has a real justification: for a bilinear
  term the covariance IS the error, not a correction. (Contrast the
  concave-mobility Jensen gap, which measured 0.4% and was dropped.)
- Game consequence: a good mixer is a tile that destroys within-tile
  covariance — precisely the property that makes it hard to homogenize.
  The most valuable machines are the most expensive to approximate.

## Scope

Tier 1 only: one reaction, three species, one-way coupling (flow carries
chemistry; chemistry does not clog flow). Tier 2 (precipitation feedback
changing tile conductance, invalidating stored records) is deliberately
deferred.
