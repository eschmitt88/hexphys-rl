---
kind: paper
title: "Evolution Gym: A Large-Scale Benchmark for Evolving Soft Robots"
authors: ["Jagdeep Singh Bhatia", "Holly Jackson", "Yunsheng Tian", "Jie Xu", "Wojciech Matusik"]
institutions: ["MIT CSAIL"]
year: 2021
venue: "NeurIPS 2021"
peer_reviewed: true
url: "https://arxiv.org/abs/2201.09863"
code_url: "http://evogym.csail.mit.edu"
citations: null
source: "raw/papers/bhatia2021evolution.pdf"
added: "2026-08-14"
relevance: 3
credibility: 5
status: read
related_experiments: []
related_concepts: ["lattice-body-representation", "runtime-morphology-vs-bilevel-codesign", "penalty-contact-vs-projection", "morphology-search-regularity-bias"]
tags: [soft-robotics, co-design, benchmark, mass-spring, voxel-representation, ppo, reinforcement-learning]
---

# Evolution Gym: A Large-Scale Benchmark for Evolving Soft Robots

## TL;DR

Evolution Gym is a large-scale benchmark (30+ tasks) for jointly evolving the
*structure* and *control* of voxel-based soft robots, built on a fast 2D
mass-spring simulator with penalty-based contact; the authors also supply and
evaluate three baseline co-design algorithms (GA, Bayesian optimization,
CPPN-NEAT, each paired with PPO for the inner control loop), finding GA
generally strongest but all baselines fail on the hardest manipulation tasks.

## Claims

- Robot co-design (jointly optimizing body structure and controller) is
  under-explored relative to control-only RL, mainly because of (1) the
  bilevel optimization structure (an expensive inner control-optimization
  loop nested inside an outer design-optimization loop) and (2) the absence
  of a shared benchmark suite, which has forced prior co-design work into
  bespoke, non-comparable testbeds evaluated on only a few simple tasks.
- A voxel-based, multi-material representation (empty/rigid/soft/
  horizontal-actuator/vertical-actuator voxel types on a grid) is expressive
  enough to produce complex, natural-looking morphologies from fewer than 100
  voxels per robot, while remaining simple enough for fast simulation.
- Robots evolved from scratch frequently converge toward morphologies
  resembling natural creatures and can outperform hand-designed robots on
  easier tasks, but no tested algorithm solves the hardest benchmark tasks
  (Traverser, Catcher, Beam Slider) — motivating the benchmark as a research
  target.

## Methods

- **Representation**: robot body = material matrix M over a grid (voxel
  types: Empty, Rigid, Soft, Horizontal Actuator, Vertical Actuator) plus a
  connection list C of adjacent-voxel edges. Validity: connected, actuators
  must exist.
- **Physics engine**: 2D mass-spring system (Hooke's-law springs per edge, 5
  material-dependent spring constants), integrated with symplectic RK4 (not
  XPBD). Collision uses a bounding-box tree with penalty-based
  normal/tangential (frictional) contact forces proportional to penetration
  depth. C++ with Python bindings mirroring the Gym API. No GPU; benchmarked
  on 80-core Xeons, evaluations take hours to ~20 hours per algorithm-task
  pair.
- **Action space**: each actuator voxel receives a per-step deformation
  target u ∈ [0.6, 1.6] (fraction of rest length) — continuous, per-timestep
  actuation, not fixed open-loop patterns as in earlier voxel co-design work.
- **Observation**: (2N+3)-D robot state (relative corner positions to center
  of mass + COM velocity/orientation) plus a local terrain-elevation window
  and task-specific goal features.
- **Co-design framework** (bilevel): outer loop = design optimizer proposes
  structures (GA with elitism + voxel mutation; BO over categorical voxel
  space; CPPN-NEAT generating voxel types from spatial coordinates); inner
  loop = PPO trains a policy per proposed design, returning reward to score
  designs.
- **Benchmark suite**: 30+ tasks across locomotion (Walker, Bridge Walker,
  Up Stepper, Climber, Traverser) and manipulation (Carrier, Thrower, Beam
  Slider, Catcher, Lifter), tagged Easy/Medium/Hard.

## Results

- GA outperforms CPPN-NEAT and BO overall despite using only simple
  mutation/elitism — attributed to CPPN-NEAT's bias toward regular/simple
  structures (hurts on manipulation tasks needing irregular substructures)
  and BO struggling with the high-dimensional categorical design space under
  noisy RL-based reward evaluation.
- Evolved GA populations show increasing structural sophistication over
  generations (Carrier: reward 4.2 at Gen 1 → 8.4 at Gen 10 → 10.6 at Gen 20).
- Algorithm-evolved robots beat hand-designed (bio-inspired, PPO-controlled)
  robots on every tested task by at least one baseline, most dramatically on
  Climber.
- All baselines fail on the hardest tasks (e.g., Beam Slider): the best
  GA-found robot only partially completes the goal.

## Critique / open questions

- **Fixed-per-episode morphology vs runtime morphology manipulation**:
  EvoGym's bilevel loop treats structure as *outer-loop, between-episode*
  search — voxel layout is fixed for the lifetime of a control run; only
  actuation varies within an episode. This is the opposite regime from
  hexphys-rl, where the bond graph changes *during* an episode via
  glue/unglue as part of the policy. EvoGym's per-actuator rest-length target
  is directly analogous to our "push" primitive, but there is no analogue to
  glue/unglue. Transfer is at the level of task design, reward shaping, and
  representation — not the control loop.
- **Simulation approach vs XPBD**: mass-spring + explicit symplectic RK4 +
  penalty contact is an older, more numerically fragile family than XPBD.
  Penalty methods need small timesteps or stiff springs to avoid
  instability/tunneling; XPBD targets exactly the error-tolerance-for-speed
  trade we want. Their throughput (hours to ~20h per run on 80 CPU cores, no
  GPU) is a concrete baseline for what un-vectorized soft-body sim costs —
  and underscores why the vectorized-XPBD bet is meaningfully different.
- **What does transfer**: (1) the task-difficulty taxonomy and
  observation/action/reward interface pattern (local terrain window,
  per-actuator continuous action, task-specific goal features) is a good
  template for our locomotion/construction task suite; (2) naive design
  search with regularity priors (CPPN) systematically fails on tasks needing
  irregular structure — a cautionary prior for biasing construction toward
  "smooth" lattice patterns; (3) even simple GA beats hand-designed
  morphologies on several tasks — supports prioritizing learned morphology
  over hand-authored bodies in the construction curriculum.
- Single-agent only; no multi-agent self-play, competition, or
  resource/energy chemistry — those parts of hexphys-rl's scope have no
  antecedent here.

## Trust signals

- **Credibility:** 5 — NeurIPS 2021 (top-tier peer-reviewed venue), MIT
  CSAIL, full open-source C++/Python benchmark and code release
  (evogym.csail.mit.edu), DARPA-funded; strong reproducibility and provenance
  signals even without a confirmed citation count (API rate-limited, recorded
  as null rather than guessed).

## Follow-up

- **Relevance:** 3 — Useful prior art for the voxel/lattice representation,
  task taxonomy, and mass-spring-vs-XPBD contrast, and cite-worthy as the
  standard soft-robot co-design benchmark, but its core bilevel outer-loop
  structure-search paradigm does not transfer to hexphys-rl's runtime,
  in-episode morphology manipulation via glue/unglue.
- Mine their task list when designing our locomotion/construction curriculum;
  Walker/Carrier analogues are natural first tasks.
