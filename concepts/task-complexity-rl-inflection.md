---
kind: concept
name: "task-complexity-rl-inflection"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/frans2022powderworld.md"]
related_concepts: ["ued-learnability-curriculum", "generalist-pretrain-then-finetune"]
related_experiments: []
tags: [rl, curriculum, findings]
---

# task-complexity-rl-inflection

## Definition

Powderworld's key finding: richer procedurally-generated task
distributions monotonically improve supervised world-model generalization,
but improve RL generalization only up to a task-specific inflection
(~8 obstacle shapes in Sand-Pushing), past which reward-signal variance
swamps PPO and both train and test reward degrade.

## Why it matters here

The concrete risk when scaling either gym's PCG task distribution: naive
complexity ramps can hurt. It motivates learnability-aware curricula
([[ued-learnability-curriculum]] / SFL) over raw complexity scaling, and
suggests world-model pretraining tolerates distributional richness that
policy learning can't yet use. Fragility caveat: one PPO config, no
sweeps, only 1 of 3 tasks shows the inflection — treat as a risk flag, not
a law; re-measure in our setting.

## Connections

- The failure mode [[ued-learnability-curriculum]] exists to dodge.
