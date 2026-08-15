---
source: https://www.redblobgames.com/grids/hexagons/
fetched: 2026-08-15
kind: post
title: "Hexagonal Grids (Red Blob Games)"
note: Extracted technical core via WebFetch; the original is a large interactive page.
---

# Hexagonal Grids — Amit Patel, Red Blob Games (technical extract)

## Coordinate systems

- Offset (rectangular maps; movement vectors position-dependent), Doubled
  (rectangular; vector-safe), Axial (store (q,r), s = -q-r implicit; vector
  ops work; right choice for non-rectangular maps), Cube (q+r+s=0; most
  elegant for algorithms, reuses 3D math).
- Recommendation: rectangular maps → Doubled/Offset; any other map shape →
  Axial/Cube.

## Axial/cube mechanics

- Constraint: q + r + s = 0.
- Six axial neighbor vectors: (+1,0), (+1,-1), (0,-1), (-1,0), (-1,+1), (0,+1).
- Distance (cube): max(|dq|,|dr|,|ds|) = (|dq|+|dr|+|ds|)/2.
- Range query radius N: -N ≤ q ≤ N; max(-N, -q-N) ≤ r ≤ min(N, -q+N).
- Line drawing: lerp in cube space + cube_round.

## Map storage

- Dense 2D array with padding (axial in a square array; hexagon-shaped map
  wastes ≤ ~50% but keeps fixed shape); per-row slicing (hexagon radius N:
  row r has 2N+1-|N-r| cells, no waste, ragged); hash for arbitrary shapes.
- Encapsulate storage behind a map class getter/setter.

## Orientation and pixel conversion (pointy-top, axial)

- x = size·√3·(q + r/2);  y = size·(3/2)·r.
- Pixel→hex: invert, then cube_round (re-derive the coordinate with the
  largest rounding error so q+r+s=0 holds).

## Symmetry

- Rotate 60° CW (cube vector): [q,r,s] → [-r,-s,-q]; CCW: [-s,-q,-r].
- Reflection: swap the two coordinates not on the axis.
