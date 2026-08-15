---
kind: concept
name: "virtual-pipe-shallow-water"
status: seedling
added: "2026-08-15"
sources: ["literature/papers/mei2007fast.md"]
related_concepts: ["outflow-scaling-clamp", "axial-hex-storage", "unconditional-stability-via-dissipation"]
related_experiments: []
tags: [eulerian, shallow-water, solver, top-down]
---

# virtual-pipe-shallow-water

## Definition

An explicit height-field water model: each cell holds terrain height b and
water depth d; each cell-edge is a virtual pipe whose flux accelerates
with the hydrostatic head difference (b+d) − (b_n+d_n); water depth
updates from net flux divergence. Per-edge and per-cell local — no global
solve.

## Why it matters here

The top-down replacement for gravity-driven water after ADR 0001: same
gameplay (route, dam, flood — dig/dam is just editing b), physics that
actually matches the perspective. Neighbor-count agnostic, so the square
formulation ports to 6 hex pipes mechanically (Mei et al. say so
explicitly). Explicit scheme ⇒ CFL-limited Δt: pick one conservative
static Δt (or substep via lax.scan) rather than per-resolution tuning.

## Connections

- Stability and locality both hinge on [[outflow-scaling-clamp]].
- Erosion/sediment (Mei steps 3–4) are a specced future gameplay layer.
