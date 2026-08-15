# Project: hexphys-rl

Short orientation only. User-level `~/.claude/CLAUDE.md` holds the durable
principles; this file refines them for this project.

## What this project is about

Learn RL by building a fast, vectorized 2D physics game engine: a hex/triangular
lattice of point masses solved XPBD-style (error tolerance traded for speed),
where embodied agents manipulate their 6 edges (push = actuate bond rest length,
glue/unglue = create/destroy bonds) to build bodies, locomote, and compete.
Simple element-type "chemistry" (bond-affinity matrix + energy) drives foraging
and competition. Curriculum: engine → scripted agents → single-agent locomotion
RL → construction → multi-agent self-play.

Sibling track — **Eulerian gym**, now top-down and StarCraft-like (ADR 0001):
a hexagon map of fine hexes (axial array + validity mask), layered fields —
terrain, pipe-model shallow water, spread-CA chemistry (creep/fire/resources),
wind (D2Q7 hex LBM / projection). Agents embodied as units (cell-entities),
creep factions (conserved field creatures), or spells (force fields); discrete
local actions → the DQN/value-based half of the curriculum, image-like
observations. Same meta-lessons: fixed computation graph → vmap fleets;
deliberate under-convergence as the speed dial; D₆ hex symmetry as free
augmentation.

## Layout (see user CLAUDE.md for the full rationale)

- `raw/` — immutable source material. Read only.
- `literature/` — processed notes on papers, repos, posts.
- `concepts/` — atomic ideas. Promote to `mocs/` when ≥5 cluster.
- `experiments/YYYY-MM-DD-<slug>/` — self-contained runs.
- `docs/decisions/` — lightweight ADRs.
- `journal/` — daily session files (hook-written).
- `_meta/` — index, log, templates.

## Scoped rules

Detailed conventions live in `.claude/rules/` and are auto-loaded when you
touch matching paths:

@.claude/rules/experiments.md
@.claude/rules/notebooks.md
@.claude/rules/data.md

Framework rules load here (per-project, not globally — they only cost
context where they can apply):

@~/claude-system/claude/rules/evaluation.md
@~/claude-system/claude/rules/agency.md

## Budget & compute

Autonomous runs read `budget.yaml` at this project's root for hard
ceilings (wall time, tokens, disk) and model roles (ideator vs
implementer). Before proposing anything with non-trivial resource
demands — multi-hour training, large downloads, many seeds — read
`budget.yaml` and make sure the ask fits under the remaining headroom.
If it doesn't fit, say so in the proposal's `risks:` and either scope
down or explicitly flag the need to raise a ceiling.

@budget.yaml

## Project-specific facts

- Primary language: Python (JAX for the vectorized solver + RL loop);
  browser demos in plain JS in `docs/`.
- Environment: managed by `uv`; run `make env` to sync.
- Data: tracked by DVC. Large artifacts on SN850X via `~/projects/`.

## Housekeeping

- End sessions with `/wrap`. The SessionEnd hook backstops this.
- Use `/new-experiment <slug>` — don't hand-roll experiment folders.
- Run `/lint` weekly.
