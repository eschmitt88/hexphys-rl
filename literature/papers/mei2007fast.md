---
kind: paper
title: "Fast Hydraulic Erosion Simulation and Visualization on GPU"
authors: ["Xing Mei", "Philippe Decaudin", "Bao-Gang Hu"]
institutions: ["CASIA-LIAMA/NLPR, Beijing", "INRIA-Evasion, Grenoble"]
year: 2007
venue: "Pacific Graphics 2007"
peer_reviewed: true
url: "https://inria.hal.science/inria-00402079v1"
code_url: null
citations: 119
source: "raw/papers/mei2007fast.pdf"
added: "2026-08-15"
relevance: 5
credibility: 4
status: read
related_experiments: []
related_concepts: ["virtual-pipe-shallow-water", "outflow-scaling-clamp", "semi-lagrangian-advection"]
tags: [shallow-water, erosion, gpu, height-field, pipe-model, terrain]
---

# Fast Hydraulic Erosion Simulation and Visualization on GPU

## TL;DR

An explicit, GPU-parallel "virtual pipe" shallow-water model drives a
five-step per-frame pipeline (rain → flow → erosion/deposition → sediment
advection → evaporation) on a height-field grid, running interactively on
256²–4096² grids on a GeForce 8800 GTX. The flux/height-update core (minus
erosion) is exactly the pipe-model regime hexphys-rl's hex water layer
adopts.

## Claims

- A modified virtual-pipe model (adapted from O'Brien & Hodgins 1995) can
  drive both shallow-water flow and hydraulic erosion with per-cell,
  spatially local update rules — embarrassingly data-parallel.
- The original pipe model's back-scaling step (redistributing flux when a
  cell would go negative) is GPU-hostile (dependent, non-local access);
  their single-pass clamp K removes the dependency while keeping the
  algorithm strictly local.
- Interactive rates on grids "usually difficult for previous techniques"
  (vs Kass & Miller implicit SWE; Beneš real-time erosion at ~5 fps on
  300²).
- 4-neighbor pipes suffice in practice, and the equations "can be easily
  extended" to more neighbors.

## Methods

Per-cell state: terrain height b, water depth d, suspended sediment s,
per-neighbor outflow flux f, velocity v — all 2D grid layers (GPU
textures).

Pipeline per iteration: (1) water increment (rain/river source);
(2) flow: flux acceleration by head difference —
f_i ← max(0, f_i + Δt·A·g·Δh_i/l) with Δh = (b+d) − (b_n+d_n) — then the
**conservation clamp** K = min(1, d·area / (Σf_out·Δt)), f_i ← K·f_i, then
height update from net flux divergence and velocity recovery from average
throughput; (3) erosion/deposition via transport capacity
C = Kc·sin(α)·|v| (erode if C > s, deposit if C < s); (4) sediment
advection via Stam's semi-Lagrangian backtrace; (5) evaporation
d ← d·(1 − Ke·Δt). Boundary: outward flux zeroed at edges.

Stability: explicit scheme with a CFL-like limit — Δt hand-retuned per
grid size (0.002 at 256² down to 0.000125 at 4096²). Flagged open issue:
the K-clamp isn't accounted for in neighbors' flow balance — strict local
conservation isn't guaranteed when clamping triggers.

GPU: 3 textures, 7 fragment-shader passes per cycle; boundary handling
branched in-shader.

## Results

On a GeForce 8800 GTX: 403 cycles/sec at 256², 186 at 512², 59 at 1024²,
16 at 2048², 4 at 4096² (sim time ~linear in cell count; visualization
dominates at large sizes). Demos: channel carving from a point source,
lake formation with sediment flattening, gully formation under rain vs a
fixed river source.

## Critique / open questions

- **What drops out without erosion:** steps 3–4 and the sediment layer
  vanish; what remains — source term, pipe flux + K-clamp + height update,
  evaporation, on just (b, d, f) — is a near-exact match for the planned
  water layer. Dig/dam gameplay is just editing b.
- **Square→hex generalization is mechanical:** every flux is a per-edge
  local quantity; replace 4 flux channels with 6, sum over 6 in the clamp
  and divergence. The paper's own "extends to 8 neighbors" note validates
  this. (The Arena demo already runs the 6-pipe variant.)
- **Tricks worth keeping:** the head-difference flux acceleration
  (vectorizable), the single-pass K-clamp (the load-bearing trick that
  makes the update vmap-able — one local reduction + clamp per cell), and
  semi-Lagrangian advection for any future field riding on the flow.
  Keep in mind the K-clamp's known conservation leak under heavy clamping.
- **RL batching mismatch:** one large grid per frame vs our many small
  grids — a JAX port adds a leading env axis and picks one conservative
  static Δt (or a lax.scan substep loop) instead of per-resolution hand
  tuning. No code ever released; equations are complete enough to
  reimplement directly.

## Trust signals

- **Credibility:** 4 — peer-reviewed top-tier venue (Pacific Graphics),
  INRIA/CASIA authorship, 119 citations and frequently cited as
  foundational in later GPU terrain-erosion work; docked one for no code
  release.

## Follow-up

- **Relevance:** 5 — the direct precedent for the pivoted water layer:
  the exact flux/head/K-clamp scheme the hex grid adopts, with the
  neighbor-count generalization explicitly de-risked by the authors.
- If erosion ever becomes gameplay (canals that deepen with use), steps
  3–4 are already specced here.
