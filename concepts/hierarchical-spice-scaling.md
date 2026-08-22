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
