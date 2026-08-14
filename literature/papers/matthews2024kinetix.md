---
kind: paper
title: "Kinetix: Investigating the Training of General Agents through Open-Ended Physics-Based Control Tasks"
authors: ["Michael Matthews", "Michael Beukman", "Chris Lu", "Jakob Foerster"]
institutions: ["FLAIR, University of Oxford"]
year: 2024
venue: "ICLR 2025"
peer_reviewed: true
url: "https://arxiv.org/abs/2410.23208"
code_url: "https://kinetix-env.github.io ; https://github.com/MichaelTMatthews/Jax2D"
citations: null
source: "raw/papers/matthews2024kinetix.pdf"
added: "2026-08-14"
relevance: 4
credibility: 5
status: read
related_experiments: []
related_concepts: ["dynamically-specified-scenes", "solver-parallelism-vs-stability", "ued-learnability-curriculum", "generalist-pretrain-then-finetune"]
tags: [jax, physics-engine, rigid-body, ued, vmap, transformer, generalist-agent]
---

# Kinetix: Investigating the Training of General Agents through Open-Ended Physics-Based Control Tasks

## TL;DR

Kinetix trains a single transformer policy over billions of procedurally
generated 2D rigid-body physics tasks (robotics locomotion/grasping through
classic control to arcade games) using a novel JAX-native impulse-based
engine (Jax2D), and shows the resulting generalist agent zero-shot solves
many hand-designed holdout levels and fine-tunes far faster than training
from scratch — including solving deceptive tasks tabula-rasa PPO cannot.

## Claims

- A large, open-ended, *dynamically specified* space of 2D physics tasks
  (built from only 4 primitive entity types: circles, convex polygons,
  joints, thrusters) is expressive enough to represent robotics
  grasping/locomotion, classic RL control (Cartpole, Acrobot, Lunar Lander),
  and arcade games (Pinball) within one unified obs/action/reward schema.
- Training a general agent on a large mixed-quality distribution of randomly
  generated levels (curated via UED/SFL for learnability) produces zero-shot
  transfer to unseen, hand-designed environments — LLM-style
  pretraining-on-diverse-data intuitions applied to online RL.
- Fine-tuning the pretrained generalist on a specific hard task is
  consistently more sample-efficient than tabula rasa RL, and in some cases
  (MuJoCo-Hopper-Hard, MuJoCo-Walker-Hard, the deceptive Car-Ramp) enables
  solving environments tabula-rasa PPO cannot solve at all, even at 1B steps.

## Methods

- **Jax2D**: deterministic, impulse-based 2D rigid-body engine written
  entirely in JAX (Box2D/ImpulseEngine heritage). Scene = circles + convex
  polygons + revolute/fixed joints + thrusters. Discrete Euler integration +
  instantaneous impulses + iterative constraint solving (configurable
  solver-step count trades accuracy for speed), warm-starting (reusing
  prior-timestep accumulated impulses), and *partial* batched
  parallelization of collision-constraint solving (default batch size 16) to
  balance vmap-parallel speed against solver stability.
- **Key design axis: dynamically vs statically specified scenes.** Unlike
  Brax (cannot vmap across different morphologies), Jax2D scenes run the
  *same computation graph* for every task — Half-Cheetah, Pinball, and
  Grasper execute identical instructions — enabling vmap across
  heterogeneous task types, the crucial enabler for on-device multi-task RL.
- RL spec: symbolic entity-based observation (position, velocity, inverse
  mass/inertia, friction, restitution, one-hot shape/role, joint/thruster
  attributes) processed by a transformer with self-attention over shapes +
  message-passing over joint/thruster connections; permutation-invariant, no
  recurrence (fully observable). Multi-discrete or continuous actions.
  Fixed reward: +1 green-blue shape collision (goal), −1 green-red, plus a
  dense potential-based auxiliary term.
- Level generator does rejection sampling (reachable goals, ≥1 actuator,
  discards no-op-solvable levels) but not unsolvable-level filtering; SFL
  (learnability p(1−p)) selects training levels — PLR/ACCEL gave no
  improvement over plain domain randomization (Appendix L). 74 hand-designed
  holdout levels in S/M/L size classes. Budget: 5B env interactions per
  method (PPO, multi-discrete), 5 seeds.

## Results

- **Throughput** (single L40S, engine-only best case): Jax2D 9,049K
  steps/sec vs Box2D 1,982K (≈4.5×). Inside a full RL loop
  (PureJaxRL-style vs SB3/Box2D): 824K vs 24K steps/sec (>30×) — though
  Jax2D only overtakes Box2D beyond ~1024 parallel envs (Box2D wins on raw
  few-env speed).
- Training the generalist for 1B timesteps on one L40S: ~7h (S), ~9h (M),
  ~14h (L); full experiments used 5B interactions.
- **Zero-shot**: S holdout solve rate rises to ~0.75; M peaks ~0.35; L stays
  low and non-monotonic — attributed to randomly generated large levels
  being disproportionately unsolvable, starving the curriculum of signal.
- Goal-conditioned locomotion probe: trained agent shows strong positive
  correlation between morphology position and goal position even on held-out
  random morphologies; random agent shows none.
- **Fine-tuning**: outperforms tabula rasa across the L holdout set;
  Hopper-Hard and Walker-Hard solved competently where tabula-rasa PPO
  fails; one exception (Thruster-Large-Obstacles) where fine-tuning is
  slower. On deceptive Car-Ramp, tabula rasa never reaches the target at 1B
  steps; the generalist solves it zero-shot ~5% of the time and reliably
  after light fine-tuning (authors note this is the exception, 2/74 levels).

## Critique / open questions

- Jax2D's core lesson for hexphys-rl is architectural, not physical: the
  payoff of dynamically specified scenes — identical computation graph, vmap
  across heterogeneous bodies — is exactly what a hex-lattice engine must
  preserve to train across many agent-built bodies at once. A fixed max
  node/edge count with active masking for glue/unglue (never dynamic graph
  resizing) is the natural analogue; Brax's static-shape-per-morphology
  limitation is the failure mode to avoid.
- Their engine is a materially different numerical approach from XPBD:
  velocity-impulse correction with Baumgarte-style positional bias, not
  position projection. But the batched partial-parallelization of collision
  constraints (batch size 16, tuned for stability/speed) is a useful
  empirical data point on how much solver sequentiality can be sacrificed to
  parallelism before divergence — directly relevant to how the hex-lattice
  Jacobi iterations should be structured.
- Kinetix is entirely rigid multi-body — no deformables, mass points, or
  bond/affinity chemistry. But the agent-side pattern (symbolic entity
  observation, permutation-invariant transformer over entities +
  message-passing over connections) is a strong architectural template for
  observing variable-topology hex bodies (entity/edge tokens, attention
  masked by bond adjacency), independent of the solver's soft/rigid choice.
- No ablation isolating transformer architecture vs UED curriculum vs sheer
  scale (5B steps) as the source of zero-shot generalization — check
  Appendices K/L/M before treating "SFL > PLR/ACCEL > DR" as settled for a
  different engine/task distribution.

## Trust signals

- **Credibility:** 5 — ICLR 2025 peer-reviewed, Oxford FLAIR (Foerster
  group, established JAX-RL lineage: PureJaxRL, JaxMARL), full code +
  trained models released (kinetix-env.github.io, Jax2D), detailed
  reproducible appendices (hyperparameters, ablations, speed benchmarks).
  Citation count unverified (Semantic Scholar rate-limited) — null, not
  guessed.

## Follow-up

- **Relevance:** 4 — Doesn't seed the lattice/soft-body/chemistry concepts
  directly, but is the closest available prior-art blueprint for the
  vectorized-JAX-engine-to-generalist-RL pipeline hexphys-rl intends to
  build; its dynamically-specified-scene/vmap-heterogeneity principle is
  directly load-bearing for the engine architecture.
- Their throughput numbers (824K steps/sec in-loop on one L40S) set the
  performance bar for our lattice engine.
- Revisit UED/SFL once random body/task generation exists — our random
  construction tasks will be mostly degenerate, the same regime SFL targets.
