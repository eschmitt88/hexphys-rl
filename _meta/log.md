# Project log

One line per mutation — ingests, new experiments, wrap entries. Written by
skills; read by `/lint`.
2026-08-14 20:10 fetch-paper XPBD (Macklin et al. 2016) → raw/papers/macklin2016xpbd.pdf
2026-08-14 20:10 fetch-paper arXiv 2201.09863 (EvoGym) → raw/papers/bhatia2021evolution.pdf
2026-08-14 20:10 fetch-paper arXiv 2410.23208 (Kinetix) → raw/papers/matthews2024kinetix.pdf
2026-08-14 20:11 fetch-paper github.com/MichaelTMatthews/Jax2D → raw/repos/michaeltmatthews-jax2d.md
2026-08-14 20:15 ingest raw/papers/macklin2016xpbd.pdf → literature/papers/macklin2016xpbd.md
2026-08-14 20:15 ingest raw/papers/bhatia2021evolution.pdf → literature/papers/bhatia2021evolution.md
2026-08-14 20:15 ingest raw/papers/matthews2024kinetix.pdf → literature/papers/matthews2024kinetix.md
2026-08-14 20:15 ingest raw/repos/michaeltmatthews-jax2d.md → literature/repos/michaeltmatthews-jax2d.md
2026-08-14 20:15 ingest seeded 11 concepts; solver/engine MoC candidate noted in index
2026-08-14 21:00 fetch-paper arXiv 2211.13051 (Powderworld) → raw/papers/frans2022powderworld.pdf
2026-08-14 21:00 fetch-paper arXiv 2212.07906 (Flow-Lenia) → raw/papers/plantec2022flow.pdf
2026-08-14 21:00 fetch-paper arXiv 2311.16080 (XLB) → raw/papers/ataei2023xlb.pdf
2026-08-14 21:00 fetch-paper dgp.toronto.edu ns.pdf (Stable Fluids) → raw/papers/stam1999stable.pdf
2026-08-14 21:12 ingest raw/papers/stam1999stable.pdf → literature/papers/stam1999stable.md
2026-08-14 21:12 ingest raw/papers/frans2022powderworld.pdf → literature/papers/frans2022powderworld.md
2026-08-14 21:12 ingest raw/papers/plantec2022flow.pdf → literature/papers/plantec2022flow.md
2026-08-14 21:12 ingest raw/papers/ataei2023xlb.pdf → literature/papers/ataei2023xlb.md
2026-08-14 21:12 ingest seeded 13 Eulerian-track concepts; solver MoC now cross-track and ripe
2026-08-15 02:25 fetch-paper inria-00402079 (Mei pipe-model erosion) → raw/papers/mei2007fast.pdf
2026-08-15 02:26 fetch-paper redblobgames.com/grids/hexagons → raw/web/redblobgames-hexagons.md
2026-08-15 02:27 ingest raw/web/redblobgames-hexagons.md → literature/posts/redblobgames-hexagons.md
2026-08-15 02:40 ingest raw/papers/mei2007fast.pdf → literature/papers/mei2007fast.md
2026-08-15 02:40 ingest seeded 3 concepts (axial-hex-storage, virtual-pipe-shallow-water, outflow-scaling-clamp)
2026-08-15 02:40 ADR 0001 top-down hex arena; CLAUDE.md mission updated
2026-08-15 03:05 fix: arena row-wraparound (guard columns); pitfall recorded in concepts/axial-hex-storage.md
2026-08-15 04:05 foundry: multiscale prototype (docs/foundry.html) + tile-homogenization concept
2026-08-15 04:20 concept: flower tiling (aperture 3R²+3R+1) recorded as the hex-nesting fix for Foundry v2
2026-08-15 05:10 foundry v2: aperture-37 flower tiling + seam characterization; twin 14.7% err @ 26x; tiling exactness asserted in harness
2026-08-15 05:50 foundry v2.1: two-port identification fix — twin error 50%→2.0% steady, channel directionality 2.3x→1783x
2026-08-15 07:10 foundry v2.2: user-found 32% steady leak → chain-calibrated conductances (all-open 7.6%, dam 1.2%); full-G runtime + mode toggle; fine view rotation-aligned
2026-08-18 concept: adaptive-surrogate-fallback (ISAT-style trust-region + on-demand refit) — vision capture, prior art, 8 design constraints
2026-08-18 foundry v3: adaptive runtime shipped — ISAT trust regions + audits + fine patches; 6x compute cut at 15.5% error; transient-dominance prediction falsified
2026-08-18 a-priori flux diagnosis: model form (not calibration) dominates; per-seam (donor,receiver) features 29.6%->9.5%; scale-separation criterion named
2026-08-18 foundry v3.1: per-seam keyed records + grid-convergence study; closure IS convergent (order ~1.6); a-priori 4x gain does NOT transfer a-posteriori
2026-08-18 diagnosis: residual is star-form error — per-seam k is ill-posed in a 6-port environment (negative conductances measured); Jensen/clamps/transient-dwell all falsified; ratio-estimator replaced with regression
2026-08-18 foundry v3.2: full port matrix (offline shape + online amplitude) — records 749->50, learning 260x, accuracy flat; spatial map localises error to steep-gradient region near the source
2026-08-18 foundry v3.3: MUSCL face reconstruction + exact fine-side port response — adaptive error 15.5% -> 7.7% at unchanged 6x compute
2026-08-19 concept: foundry-testbed-physics — fine sim identified as degenerate nonlinear diffusion (porous medium eq m=2 / Richards-like), single phase, no momentum; measured R~t^0.22 vs Barenblatt t^0.25
2026-08-20 reactor: Tier-1 chemistry (A+B->C) shipped at docs/reactor.html; reaction closure 1.17x->12.23x with Damkohler CONFIRMED; transport closure only 3.9%; CSTR hypothesis killed (1%)
2026-08-22 closure race shipped (reactor.html): naive/moment/GNN. GNN 320-param message-passer wins fast regime (2.51x->1.31x, 1.01x at k=0.24); moment wins slow; low-k GNN failure proven to be feedback drift (a-priori uniform 11-15%); DAgger-lite tried and reverted
2026-08-22 oscillatory closure limits measured (diag17): sync regime lumps fine (rho 0.9-1.0); heterogeneity across Hopf + weak coupling -> rho 0.21, lumped 5.5x too strong at 40%-low frequency; criterion is Kuramoto-style synchrony, not wavelength
2026-08-22 foundry v4 (ports) shipped: docs/ports.html + ADR 0002; P1 confirmed 0.00%, P2 failed-as-stated (5.3%, bounded), P3 failed-as-stated/succeeded-in-substance (naive 0.76x = wide-seam trained GNN)
2026-08-22 SPICE loop shipped on ports.html: design->characterize (15 linear probes)->cast->board-on-macromodels->in-circuit device check; heterogeneous board 3.5% flow err; per-port fluxes ~20% (cancel in aggregate)
