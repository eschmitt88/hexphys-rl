# 0003 — The physics ladder: one top-down game, physics spine, cards up

- **Date:** 2026-11-17
- **Status:** accepted
- **Design:** docs/design/2026-11-17-physics-ladder.md

## Context

The physics-ladder design note proposes a multi-scale game. Each level is
a full hex board with its own physics, and a sealed board ascends as a
*card*: a handful of parameters that become one cell's rules at the
level above. The six levels are BOND → MATTER → DEVICE → LAND → WORLD →
SYSTEM, with an optional STAR level.

The note left five open questions. The user answered four of them, and
the fifth (where the work lives) follows from the note's recommendation.

## Decision

1. **Top-down at every level.** This extends ADR 0001 to the whole
   ladder. Gravity never acts in the play plane. L3 DEVICE runs MPM
   without gravity, so its drivers are pressure and pumps, actuation,
   thermal expansion, and material pushed by agents. L6 SYSTEM is
   natively top-down because the orbital plane is the play plane.
2. **Physics spine first.** The ladder goes up to planet and orbits.
   A life spine (cell → organism → ecosystem) is deferred, and the
   intent is to merge it later on L2–L4.
3. **One game.** Every level must carry its own playable loop, and the
   levels form one campaign. Separate RL environments are a by-product
   of the game, not a goal of their own.
4. **Cards go up only, for now.** A sealed board is frozen with the
   boundary conditions it was sealed under. No downward forcing flows
   from a level to the one below: no climate rewriting land, no tides
   reaching devices. The dotted arrows in the design diagram are
   deferred.
5. **Home: hexphys-rl.** The work extends this repo rather than starting
   a new project, because L3, L4 and the bridge machinery (ports, SPICE,
   seal & ascend, adaptive fallback) already live here.

## Consequences

- **Each level is a stand-alone environment.** With up-only coupling,
  level *k* depends on the levels below only through the distribution of
  cards it receives. RL at level *k* can therefore train under domain
  randomization over plausible cards. Level *k* does not have to wait for
  level *k−1* to be solved, so levels can be built and trained in any
  order.
- **Revisiting a lower board is a redesign, not a re-simulation under new
  forcing.** The adaptive-surrogate validity check is needed only when a
  card is used outside the operating range it was characterized at, for
  example a device placed where the head exceeds its probe range.
- **Keep the merge with the life spine cheap.** Reserve an open
  "biological" block in the card schema at L2–L4: replication rate,
  metabolism (input → output), mutation. Life can then later appear as
  self-replicating devices and populations without changing the bridge
  format.
- **Top-down MPM is less studied than side-view MPM.** Validate the L3
  kernel early: confirm that granular and plastic behaviour without
  gravity still produces legible, interesting devices.
