---
kind: paper
title: "XLB: A differentiable massively parallel lattice Boltzmann library in Python"
authors: ["Mohammadmehdi Ataei", "Hesam Salehipour"]
institutions: ["Autodesk Research, Toronto"]
year: 2023
venue: "Computer Physics Communications (2024)"
peer_reviewed: true
url: "https://arxiv.org/abs/2311.16080"
code_url: "https://github.com/Autodesk/XLB"
citations: 42
source: "raw/papers/ataei2023xlb.pdf"
added: "2026-08-14"
relevance: 4
credibility: 4
status: read
related_experiments: []
related_concepts: ["lbm-collide-stream", "bgk-collision-stability-limit", "differentiable-physics-vs-rl-signal"]
tags: [lbm, jax, differentiable-physics, cfd, eulerian, gpu-scaling]
---

# XLB: A differentiable massively parallel lattice Boltzmann library in Python

## TL;DR

XLB is an open-source JAX lattice-Boltzmann library (Autodesk Research):
JIT-compiled, natively differentiable via jax.grad, scaling from one
desktop GPU to 512-GPU clusters — up to ~220,000 MLUPS on a 4096³ (67
billion cell) domain — behind a NumPy-like API.

## Claims

- A Python/JAX LBM library can match NumPy-style accessibility while
  retaining HPC-grade performance across CPU/TPU/multi-GPU/distributed
  systems, closing a gap left by C++ codes (Palabos, OpenLB).
- Native differentiability through the full step (collision + BC +
  streaming) enables physics-based ML (surrogate correction, inverse
  design) without hand-written adjoints.
- OO lattice/collision abstractions (D2Q9/D3Q19/D3Q27; BGK, MRT,
  cumulant, recursive-regularized, KBC) let users swap models without
  touching the core.

## Methods

- Arrays in (x,y,z,cardinality) layout; JIT-compatible OOP via a partial
  decorator; boundary conditions via a boolean mask array (built by
  streaming a halo-extended mask once) that auto-derives per-cell
  known/missing populations and normals.
- Streaming — the only non-local op — is jnp.roll in-device plus
  lax.ppermute under a manual shard_map for cross-device halo exchange
  (chosen over pmap to avoid an extra device axis). Sharding is
  x-axis-only; one JAX process per node via MPI.
- Mixed precision (independent compute/storage dtypes), distributed
  checkpointing, VTK/PyVista I/O, in-situ GPU visualization.

## Results

- **Accuracy:** 2D Taylor-Green (2nd-order convergence; f32 storage
  degrades it), 3D lid-driven cavity Re 1000–10000 vs
  spectral/experimental references, 2D cylinder Re=100 (Cd, Cl, Strouhal
  all within literature ranges), turbulent channel LES at Re_τ=180 vs DNS.
- **Throughput:** A100 80GB single-GPU up to ~2200 MLUPS (f32/f16), ~1500
  (f32/f32), ~320–450 (f64). Weak scaling >95% at 8 GPUs (one node);
  strong scaling ~90% at 8 GPUs; distributed weak scaling ~70% at 64 GPUs
  degrading to ~30% at 512 GPUs; peak 4096³ at 220,332 MLUPS.
- **Differentiability demos:** a ResNet corrector trained by backprop
  through 100+ unrolled LBM steps reduces coarse-grid wake-flow error; an
  MLP-parameterized initial condition optimized via jax.grad to make a
  density field spell "XLB" after 200 steps.

## Critique / open questions

- **One-big-grid vs many-small-grids:** XLB's distributed design targets a
  single enormous sharded domain. Our RL topology is the opposite —
  thousands of small independent grids vmapped on one GPU. What transfers:
  the array-layout convention, the lattice class hierarchy, the mask-array
  BC trick, and the *single-GPU* MLUPS figures as the throughput
  reference. What doesn't: all the cross-device machinery.
- **Differentiability vs RL:** jax.grad through collide+stream buys
  nothing for standard policy-gradient/value-based RL (black-box forward
  rollouts). It would matter for model-based or
  differentiable-physics-as-critic variants — an open design question,
  weighed against LBM's memory cost (q populations vs 2–3 fields for
  Stable Fluids).
- **Stability is the real fork:** the paper's own results switch to the
  KBC collision model where BGK is unstable (Re=10000 cavity, turbulent
  channel). BGK has a genuine low-viscosity stability floor — the opposite
  of Stable Fluids' unconditional stability. Untrained RL agents are
  adversarial perturbation generators; that argues for starting the
  Eulerian curriculum on Stable Fluids, or budgeting KBC/MRT complexity
  and conservative τ margins if LBM leads.
- **Mesoscopic vs macroscopic:** LBM's per-direction populations make rich
  BCs (bounce-back, momentum-exchange forces) and future multi-species
  chemistry fall out naturally; Stable Fluids is cheaper per step and
  unconditionally stable but has no mesoscopic structure to hang chemistry
  on. Note: D2Q9's isotropy comes from its square-lattice weights — a
  different mechanism from FHP's hexagonal-symmetry requirement; don't
  conflate the two hex threads.
- f32/f32 storage flagged by the authors as unreliable without a
  not-yet-implemented rescaling trick — relevant on modest GPUs.

## Trust signals

- **Credibility:** 4 — peer-reviewed (Computer Physics Communications
  2024), Autodesk Research with NVIDIA JAX-team acknowledgment, Apache-2.0
  code with reproducible benchmark scripts, 42 citations at ~18 months.

## Follow-up

- **Relevance:** 4 — anchors the LBM solver candidate and supplies
  directly reusable JAX patterns (lattice classes, mask-array BCs,
  differentiable step); its distributed emphasis is orthogonal to our
  many-small-worlds batching.
- Solver bake-off experiment: use their single-GPU MLUPS numbers as the
  sanity bar for our D2Q9 port at RL grid sizes.
