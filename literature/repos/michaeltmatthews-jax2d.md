---
kind: repo
name: "Jax2D"
url: https://github.com/MichaelTMatthews/Jax2D
commit:
source: "raw/repos/michaeltmatthews-jax2d.md"
added: "2026-08-14"
relevance: 4
status: scanned
related_experiments: []
related_concepts: ["dynamically-specified-scenes", "solver-parallelism-vs-stability"]
tags: [jax, physics-engine, 2d, rigid-body, rl-environments]
---

# Jax2D

## Purpose

A 2D rigid-body physics engine (impulse-based, Box2D-lineage) written
entirely in JAX, built as the backend for Kinetix. Its differentiator among
JAX physics engines (vs Brax/MJX): scenes are *dynamically specified* —
heterogeneous scenes parallelize under a single `vmap`, because every scene
is padded to the same fixed shapes (max counts of polygons/circles/joints/
thrusters) with active masks.

## Shape

- PyPI `jax2d`; MIT license. By Michael Matthews & Michael Beukman (FLAIR/Oxford).
- Core API: `StaticSimParams` (fixed shapes, compile-time), `SimParams`
  (runtime), `PhysicsEngine.step` (jit-compiled), scene builders
  (`add_rectangle_to_scene`, `add_circle_to_scene`,
  `add_revolute_joint_to_scene`, `add_polygon_to_scene`, fixation flags).
- Actions drive joint motors + thrusters — action vector length is
  `num_joints + num_thrusters`, fixed per StaticSimParams.
- Rendering via JaxGL (separate repo); Kinetix.js is a JS reimplementation
  with a browser editor.

## Useful bits

- **The fixed-shape + mask pattern is the load-bearing idea to steal**: our
  hex lattice's degree ≤ 6 gives an N×6 bond table with active masks — same
  trick, better fit (no O(n²) pair loop needed for bonded forces).
- Their honest caveat: Jax2D is O(n²) in entities (full pairwise collision),
  so it targets *lots of small diverse scenes*, not big scenes. Our lattice
  contact via spatial hashing on the grid should beat this for >100-element
  worlds — worth benchmarking against as a baseline.
- Static/dynamic parameter split (`StaticSimParams` vs `SimParams`) is a
  clean JAX architecture convention to copy.
- Kinetix.js precedent: a JS mirror of the engine for browser demos — same
  pattern as our `docs/` dashboard demos.
- Randy Gaul's 2D collision-detection notes (linked in README) recommended
  by the authors as the starting point for writing your own engine.

## Follow-up

- Read `engine.py` internals for how they mask inactive entities inside
  `vmap` without branching.
- Benchmark: Jax2D car scene steps/sec on our GPU vs our lattice prototype
  at comparable entity counts.
