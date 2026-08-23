---
kind: concept
name: "hierarchical-spice-scaling"
status: seedling
added: "2026-08-22"
sources: []
related_concepts: ["tile-homogenization", "adaptive-surrogate-fallback", "oscillatory-closure-limits"]
related_experiments: []
tags: [multiscale, hierarchy, scaling, ports, game-design]
---

# hierarchical-spice-scaling

## Board-size scaling (harness12)

Flow twin (own fluid, one probed conductance) at three board radii:

| board R | tiles | fine cells | settled err | speedup (measured) |
|---|---|---|---|---|
| 4 | 61 | 1525 | 3.8% | 27x |
| 6 | 127 | 3175 | 2.3% | 26x |
| 8 | 217 | 5425 | 1.7% | 25x |

**Error SHRINKS with domain size** — the residual lives at source/drain
boundary tiles, a vanishing fraction of bigger boards. Bigger game levels
are MORE accurate at coarse scale, not less. Speedup is flat (~26x; the
37x state ratio minus JS overhead) — cost per level scales exactly as
designed: fine ∝ cells, SPICE ∝ tiles.

## Level-3 recursion (harness13)

7 super-tiles x 7 tiles on the 61-board (aperture-7 at tile level),
recursive one-port-per-super-seam (extra crossings shut via port-mask
elements from the workshop library), super-element characterized by
15 EXACT linear solves on the 7-node tile-SPICE network — probing two
levels up costs microseconds.

Three levels lockstep, heads compared at super granularity:

- L1 tile-SPICE vs fine truth: ~15% (this masked, serially-bottlenecked
  config is harder for L1 than the plain board's 3.8%)
- **L2 super-SPICE vs fine truth: ~6%** — error ratio L2/L1 = **0.39**

**Error does NOT compound across levels — it contracted.** The super
element, probed end-to-end through its internal network, captures
aggregate transport better than the tile lane accumulates it node by
node. Mechanism not fully dissected (through-probe averaging + partial
cancellation of per-tile operating-condition biases); one config,
flow-only, wet regime — needs replication before leaning on it. Cost per
level: 1177 fine cells -> 61 nodes -> 7 nodes.

## Replication + rung 3 (2026-08-23, harness15/16)

**Replication (harness15)**: 8 randomized configs — random src/drn supers,
random designated-port choices per seam, 3 random valve obstacles each
(with L1-graph connectivity resampling). Per-super L2 elements probed with
each super's ACTUAL internal conductances. Result: **L2 < L1 in 8/8**,
ratios 0.50-0.81, median 0.60. The contraction is a property of the
architecture, not config luck.

**Rung 3 (harness16)**: chain of three 49-tile super-supers on a radius-12
board (147 active tiles, 322 solid), one-port rule applied at ALL levels
(one tile-crossing per super seam, ONE super-crossing per SS seam). Each
of 21 supers probed on its 7-tile network; the SS probed on its 7-super
network — every level characterized from the level below, microseconds
each. Four levels lockstep, 16k steps:

| level | state | error vs fine (SS granularity) |
|---|---|---|
| fine | 17,353 cells | truth |
| L1 | 469 nodes | 1.0% |
| L2 | 21 nodes | 0.3% |
| L3 | 3 nodes | 1.0% |

**5,784x state compression, error flat at ~1% through three recursion
rungs — no compounding.** The chain config is bottleneck-dominated and
thus lump-friendly (L1 already 1.0% vs ~15% on the dense R4 hierarchy),
consistent with the boundary-fraction scaling law. Caveats: flow-only,
wet/linear regime, uniform interiors, L3 probed as a 2-port along the
chain axis with the end-SS halves approximated by the middle SS's.

## Bugs eaten en route (both instructive)

1. Index-space collision: super assignment keyed by geometry index while
   pairs carry tile indices — caught by accounting (752 open cells where
   1177 were expected; 1 solid tile where 12 were placed). Lesson: print
   conservation/accounting BEFORE interpreting error numbers.
2. Silent patch failure: replacing 'var rate=0.9' missed 'var self=this,
   rate=0.9' — fine injected 1.8 while lanes injected 18, producing 600%+
   phantom errors that "grew like accumulation". Diagnosed by proving the
   fine system linear (g identical at rates 0.05-9, diag21), which made
   rate-dependent divergence impossible and forced the hunt to the
   harness itself. Lesson: patches need existence asserts (now added).

## Game consequence

The scaling story the game needs is measured end-to-end: levels can grow
37x in area per zoom while coarse cost grows 1x per tile, accuracy
IMPROVES with board size, and characterizing the next level up costs
microseconds because it probes the SPICE network, not the fine world.
