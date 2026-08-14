---
kind: concept
name: "differentiable-physics-vs-rl-signal"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/ataei2023xlb.md"]
related_concepts: ["lbm-collide-stream"]
related_experiments: []
tags: [rl, differentiable-physics, open-question]
---

# differentiable-physics-vs-rl-signal

## Definition

The open question of whether end-to-end differentiable environment
dynamics (jax.grad through the solver, as XLB demonstrates through 100+
unrolled LBM steps) provides a training signal worth its cost, versus
standard RL that treats the environment as a black box needing only
forward rollouts.

## Why it matters here

Both hexphys engines will incidentally be differentiable (pure JAX). For
the planned curriculum (PPO / DQN) that buys nothing. It becomes relevant
only for model-based variants — differentiable-physics-as-critic, or
gradient-based task/level design (XLB's inverse-design demo is the
existence proof). Record now so the option isn't forgotten; don't build
for it.

## Connections

- A future experiment fork, not a current requirement.
