---
kind: paper
title: "Flow-Lenia: Towards open-ended evolution in cellular automata through mass conservation and parameter localization"
authors: ["Erwan Plantec", "Gautier Hamon", "Mayalen Etcheverry", "Pierre-Yves Oudeyer", "Clément Moulin-Frier", "Bert Wang-Chak Chan"]
institutions: ["FLOWERS Team, Inria Bordeaux", "Poietis", "Google Research Brain Team, Tokyo"]
year: 2022
venue: "ALIFE 2023"
peer_reviewed: true
url: "https://arxiv.org/abs/2212.07906"
code_url: "https://tinyurl.com/mr2ncy3h"
citations: 24
source: "raw/papers/plantec2022flow.pdf"
added: "2026-08-14"
relevance: 5
credibility: 4
status: read
related_experiments: []
related_concepts: ["mass-conservative-ca-flow", "conservation-induced-localization", "local-parameter-embedding-in-field"]
tags: [cellular-automata, lenia, mass-conservation, alife, eulerian, flow-field]
---

# Flow-Lenia: Towards open-ended evolution in cellular automata through mass conservation and parameter localization

## TL;DR

Flow-Lenia turns Lenia's growth-based update into a mass-conservative
advection process — a flow field computed from the gradient of the growth/
affinity map transports matter via reintegration tracking — which makes
spatially localized creatures the *generic* outcome rather than a rare
needle in parameter space, and lets creatures carry their own local
update-rule parameters, enabling multi-species simulation and optimizable
behaviors (directed motion, obstacle navigation, chemotaxis).

## Claims

- A hard mass-conservation constraint on a Lenia-like CA (i) reliably
  confines emergent patterns to spatially localized creatures, (ii)
  enables multi-species simulations, (iii) can act as intrinsic
  evolutionary pressure via finite resource.
- Searching for spatially localized patterns — which in vanilla Lenia
  required curricula, diversity search, or gradients through the CA —
  becomes easy: random search already yields robust creatures.
- Update rules can be optimized with plain evolutionary strategies
  (OpenES) for user-specified fitness (directed motion, angular motion,
  obstacle navigation, chemotaxis); doing the same to vanilla Lenia is
  unstable and only finds exploding patterns.
- Parameters embedded as a field and advected with matter let
  heterogeneous "species" coexist in one grid (mixing by mass-weighted
  average or softmax competition) — impossible in vanilla Lenia.

## Methods

- **Core move:** reinterpret Lenia's growth field U as an *affinity map* —
  matter moves toward higher affinity. Flow field F = (1−α)∇U − α∇A_Σ, a
  convex mix of affinity gradient and a diffusion term (negative
  concentration gradient); α(p) = clip(A_Σ/θ_A, 0, 1)^n makes diffusion
  dominate near a critical-mass threshold, preventing runaway
  concentration.
- **Advection:** reintegration tracking (Moroz 2020) — each source cell's
  mass is redistributed over a square footprint centered at the
  flow-advected target; footprint integrates to exactly 1, so mass is
  conserved *exactly by construction*, not softly penalized. Clipping is
  gone; cell values become unbounded non-negative concentrations.
- **Parameter embedding:** a parameter map P locally modulates the rule
  (per-kernel weights h) and is co-advected with mass; collisions mix by
  weighted average or softmax-by-incoming-mass.
- **Optimization:** OpenES (evosax, pop 16) on fitness from
  center-of-mass trajectories. Optional food field converts to matter on
  contact (digestion) with background decay — soft evolutionary pressure.
- JAX implementation; ~255 µs/step on a Tesla T4 for 1 channel/10
  kernels/128×128.

## Results

- Random search yields spatially localized patterns as the typical
  outcome: gyrating creatures, snakes with attraction/repulsion dynamics,
  dividing/merging dots, multi-channel creatures whose morphology depends
  on total mass while retaining identity.
- Optimization found creatures for all 4 tasks; 2-channel configurations
  beat single-channel for directed motion (attraction-repulsion between
  channels).
- Control: identical optimization on vanilla Lenia was unstable — the
  conservation constraint is what makes behavior optimization tractable.
- Large-scale multi-species runs (1024², 144 parameter sets, 200k steps):
  species spreading by "contaminating" others, coherent creatures made of
  two parameter sets in a core-membrane arrangement (symbiosis-like),
  food-driven division events.
- Open-endedness explicitly not measured — mechanism-and-feasibility
  paper, not a demonstrated-open-evolution claim.

## Critique / open questions

- **Transfer to an RL-controlled field creature:** the paper only
  optimizes offline (ES over rollout fitness); it never closes an online
  control loop. The architectural transfer is the flow field as actuation
  surface: a policy can output a bias added to ∇U each step. Whether
  ES-discovered swimming strategies survive dynamic actuation is untested.
- **Actuation channels — one deliberate gap:** F decomposes into a
  directed term (∇U, with per-kernel weights as a natural low-dimensional
  handle) and a *concentration-avoiding* diffusion term. There is no
  cohesion-toward-centroid term — Lenia's kernels implicitly hold
  creatures together. Our demo/gym's explicit cohesion-to-CoM flow is an
  addition, not a carry-over, and should be labeled as such.
- **The key transferable insight is conservation-induced localization**,
  not the flow mechanics per se: mass conservation alone turns
  "spatially localized individual" from a search problem into the generic
  outcome. This validates the field-creature archetype's core bet.
- Reintegration tracking is intricate vs. naive semi-Lagrangian advection
  (which is NOT exactly conservative). Given the project's
  error-for-speed ethos, an approximately conservative cheaper scheme
  (e.g. SL + renormalization) may be the right engineering call — measure
  the drift before paying for exactness.
- Evaluation is qualitative; no ablation isolating conservation vs. the
  diffusion term α as the source of localization.

## Trust signals

- **Credibility:** 4 — peer-reviewed ALIFE 2023, reputable groups (Inria
  FLOWERS; co-author Chan is Lenia's creator, Google Research), public
  code, 24 citations; results largely qualitative and venue is
  niche-tier, capping below 5.

## Follow-up

- **Relevance:** 5 — directly seeds the field-creature archetype: mass-
  conservative advection along an affinity flow, with localization as a
  generic property, is the mechanism our demo approximates and the JAX gym
  would implement properly (reintegration tracking), plus flow-bias as the
  policy's actuation channel.
- If multiple competing field creatures ever ship, their
  parameter-embedding + softmax mixing is the mechanism to revisit.
