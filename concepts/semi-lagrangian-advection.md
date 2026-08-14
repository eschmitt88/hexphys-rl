---
kind: concept
name: "semi-lagrangian-advection"
status: seedling
added: "2026-08-14"
sources: ["literature/papers/stam1999stable.md"]
related_concepts: ["unconditional-stability-via-dissipation", "under-converged-pressure-projection", "mass-conservative-ca-flow"]
related_experiments: []
tags: [eulerian, solver, stability]
---

# semi-lagrangian-advection

## Definition

Compute a grid point's new value by tracing its characteristic *backward*
through the velocity field by one step and interpolating the old field
there, instead of extrapolating forward with finite differences. New values
are interpolations of old values, so the scheme cannot amplify — stable at
any Δt.

## Why it matters here

The transport primitive of the Eulerian gym's Stable-Fluids solver, and the
reason an untrained RL agent can't blow the world up by flailing. Its cost
is built-in: repeated linear interpolation is a low-pass filter, so small
vortices die (numerical viscosity). Known fix if the world feels too calm:
vorticity confinement (Fedkiw 2001). Also note it is NOT exactly
conservative — the field-creature use renormalizes mass after advection, or
upgrades to reintegration tracking ([[mass-conservative-ca-flow]]).

## Connections

- One half of the Stam pipeline; the other is
  [[under-converged-pressure-projection]].
