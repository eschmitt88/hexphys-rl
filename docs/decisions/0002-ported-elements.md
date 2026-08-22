# 0002 — Ported elements: one open cell per side

- **Date:** 2026-08-22
- **Status:** accepted (Foundry v4, docs/ports.html)

## Context

Wide seams (7 shared edges per tile side) caused most of the surrogate
programme's pain: per-seam conductance proved ill-posed in multi-port
conditions (negative measured values), requiring chain calibration, MUSCL
face reconstruction, exact fine-side ports, and per-seam record keying.
The reaction closure failed up to 12x through within-tile covariance fed
by wide-seam interpenetration. User proposal: restrict every tile to one
open cell per side — six single-cell ports, structural rim otherwise.

## Decision

Adopt the port template as the standard element form. The tile grid
becomes part of the world's ontology (chambers-and-throats — a pore
network, as in petroleum engineering — not an approximation of open
continuum). Interfaces are single cells, so interface state is exact by
construction. Interiors remain fully continuum and paintable; ports can
be shut (valves) but the rim cannot be opened.

## Measured consequences (harness10)

1. Single-tile characterization equals chain calibration to **0.00%**
   (was 45% disagreement + ill-posedness with wide seams).
2. Network-flow twin: **5.3%** with one probed conductance and zero
   calibration campaigns (vs 7.6% wide-seam open world after three).
   Residual = bounded multi-port operating-condition sensitivity; two
   targeted probes (drain-hop helped 8.4->5.3; radial-source did not)
   — general fix is the v3 operating-point records, not more probes.
3. Reaction closure problem largely dissolves: chambers are forced
   mixers; naive closure 0.72-0.76x across all rates (was 1.25-2.51x);
   the ported zero-learning closure matches the wide-seam trained GNN
   (|log err| 0.27 = 0.27). Learned lanes now slightly over-suppress.

## Addendum (same day): the SPICE loop is implemented

docs/ports.html now carries the full component workflow: design a chamber
(ports can be shut = valves), **characterize** via 15 midpoint-referenced
2-port probes + LINEAR least-squares on resistances (1/T_ij = r_i + r_j;
Laplacian tiebreak resolves underdetermined port sets), cast to a library
slot, **place on the board** (both twin worlds re-stamp), and
**device-check** any placed component against a live 37-cell simulation
at its in-circuit operating point — the mixed-signal simulator pattern.

Measured (harness11): open element g uniform to 3 digits, star residual
4.1%; valve reads exactly zero on shut ports; a heterogeneous board (9
valves + 8 four-port manifolds among open tiles) runs end-to-end
(own-fluid flow + chemistry) at **3.5% flow error** on macromodels alone.
Honest wrinkle: per-port instantaneous fluxes deviate ~20% at in-circuit
operating points (star cross-port structure + multi-port sensitivity)
while aggregate state error stays low — misdistribution cancels. The
device-check readout states both numbers.

## Consequences

- Simplicity: the ports page needed ~200 changed lines over the reactor
  page; the wide-seam programme needed five fix campaigns.
- The GNN/moment closures remain valuable for continuum regions and for
  future physics (oscillations, Tier-2 clogging) but are no longer
  needed for ported flow+bilinear chemistry.
- Open-terrain (continuum) regions, if the game wants them, keep the v3
  adaptive machinery; ported machine regions run on the cheap exact-ish
  network. Two-tier world remains available.
