---
kind: concept
name: "generalist-pretrain-then-finetune"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/matthews2024kinetix.md"]
related_concepts: ["ued-learnability-curriculum"]
related_experiments: []
tags: [rl, transfer, findings]
---

# generalist-pretrain-then-finetune

## Definition

Training one policy across a vast procedurally generated task distribution,
then fine-tuning on a specific hard target, yields solves that tabula-rasa
RL cannot reach — Kinetix's pretrained generalist solves deceptive tasks
(Car-Ramp: must move away from the goal first) that from-scratch PPO never
solves at 1B steps.

## Why it matters here

Template hypothesis for the late curriculum: pretrain across many random
hex-body locomotion/construction tasks, then fine-tune into the hard
targets (foraging with chemistry, multi-agent competition). Also motivates
the Kinetix agent-side architecture — permutation-invariant transformer
over entity tokens with adjacency-masked attention — as the natural policy
class for variable-topology bodies. Kinetix caveat: zero-shot generalist
wins were the exception (2/74 levels), fine-tuning speedups the rule.

## Connections

- Builds on [[ued-learnability-curriculum]] for the pretraining
  distribution.
