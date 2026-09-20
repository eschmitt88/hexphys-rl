# hexphys-rl

**Learning reinforcement learning by building a fast, vectorized 2D hex-lattice
physics engine where embodied agents build bodies, locomote, and compete.**

📂 **[Browse this repo →](https://eschmitt88.github.io/hexphys-rl/)** —
interactive, always-live view of experiments, concepts, literature, and maps of
content. Served via GitHub Pages from `docs/index.html`; reads the live file
tree, no build step.

## What this is

A learn-by-building RL project. The **Lagrangian track** is a hex/triangular
lattice of point masses solved XPBD-style, trading error tolerance for speed.
Agents are embodied as nodes that act on their six edges — actuate a bond's rest
length to push, create or destroy bonds to glue and unglue — and a simple
element-type chemistry (bond-affinity matrix plus energy) drives foraging and
competition. Curriculum: engine, scripted agents, single-agent locomotion RL,
construction, multi-agent self-play. The sibling **Eulerian gym** is a top-down,
StarCraft-like hex map with layered fields — terrain, shallow water, spreading
cellular-automaton chemistry, wind via a hex lattice-Boltzmann scheme — where
agents are units, creep factions, or spells, covering the value-based half of
the curriculum. Shared lessons throughout: fixed computation graphs vectorize
into fleets, deliberate under-convergence is the speed dial, and hex symmetry
is free augmentation.

## How it's organized

Plain Markdown + flat YAML frontmatter, cross-linked with `[[wikilinks]]`:

- `concepts/` / `mocs/` — atomic ideas; promoted to a map of content when ≥5 cluster.
- `literature/` — processed notes on papers, repos, posts (0–5 relevance scored).
- `experiments/YYYY-MM-DD-<slug>/` — self-contained runs (hypothesis → result, config, metrics, log).
- `raw/` — immutable source captures · `docs/decisions/` — ADRs · `_meta/` — index, log, templates.

## Local use

```sh
make env    # uv sync
make lint   # knowledge-graph / experiment health check
```

Built on the [claude-system](https://github.com/eschmitt88/claude-system)
research framework (upstream attribution — this project is its own repo).
See `CLAUDE.md` for the agent-facing orientation and `~/.claude/CLAUDE.md`
for the framework's durable principles.
