---
kind: concept
name: "ued-learnability-curriculum"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/matthews2024kinetix.md"]
related_concepts: ["dynamically-specified-scenes", "generalist-pretrain-then-finetune"]
related_experiments: []
tags: [rl, curriculum, ued]
---

# ued-learnability-curriculum

## Definition

Unsupervised environment design that selects training levels by
learnability p(1−p) — success probability times failure probability —
concentrating training on the frontier where the agent sometimes succeeds,
and discarding both unsolvable (p=0) and mastered (p=1) levels. SFL
(Rutherford et al. 2024) is the current standard; Kinetix found it beats
PLR/ACCEL, which did no better than domain randomization.

## Why it matters here

Once random generation of bodies/tasks exists, most sampled levels will be
degenerate (Kinetix's L-size levels were disproportionately unsolvable and
starved the curriculum — their stated main limitation). SFL is the
candidate mechanism for keeping construction/locomotion training signal
alive at scale. Not needed for the early hand-designed curriculum stages.

## Connections

- Depends on cheap parallel evaluation from [[dynamically-specified-scenes]].
- Kinetix Appendix L is the comparison to revisit before adopting.
