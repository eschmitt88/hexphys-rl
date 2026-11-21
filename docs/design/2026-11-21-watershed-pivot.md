---
kind: design
name: "watershed-pivot"
status: proposal (under discussion)
added: "2026-11-21"
related: ["docs/frontier.html", "docs/design/2026-11-17-physics-ladder.md", "docs/decisions/0002-ported-elements.md", "docs/decisions/0003-physics-ladder-scope.md", "concepts/physics-adversary-blight.md"]
---

# Watershed pivot: water and lava as the point of the map

## The user's playtest critique of Frontier (2026-11-21)

1. **Designs don't climb the ladder.** A design built on one tile can't be
   pasted onto another, because the terrain underneath differs.
2. **No goal after gathering.** You collect resources, then nothing asks
   for them.
3. **Terraforming is click-heavy and low value.** It takes about 1000
   clicks to nudge water toward a farm, when a farm is one click anyway.
4. **The forge is pointless** (see the Fable consult): it is a supply with
   no demand.

## The user's ideas, as stated

- **Make routing and collecting water the goal.** Rain is a *moving
  field* that replenishes the water. Native terrain decides where water
  ends up; you reshape terrain to guide it and collect it.
- **Make big projects easy and clear.** Mass earthworks, not
  hex-by-hex clicking.
- **A second resource: lava or magma.** It rises at vents, or erupts if
  you dig too deep. At the level above you harvest both resources.
- **Many channels between areas,** not the single port per tile side
  from ADR 0002.
- **Copying a design to a new tile:**
  - *Terraform to template.* Pay for the diff that turns the new tile into
    the old one; this becomes affordable later in the game. Rain still
    behaves differently from tile to tile.
  - *Rules.* Learned from how you built, or defined by you: coded, or run
    through a language model or decision model. The user judged this a big
    diversion.
- **Progression:** the first megatiles are hand-engineered; later, a full
  terraform is cheaper than redoing the engineering.

## Synthesis (assistant, for discussion)

### 1. Intent tools fix the click problem and the copy problem together

Earthworks become declarative intents that a local solver turns into
terrain operations. Settlers then carry them out over days, paid for by
the volume of earth moved.

| Intent | What the solver does |
|---|---|
| Channel A to B | Finds a route; digs to a minimum gradient |
| Dam this valley | Follows a contour to the spill level; builds the wall along it |
| Basin or collector here | Excavates a catchment |
| Levee along this river | Raises banks along the reach |

A blueprint is then a list of intents, not raw heights: "dam the main
river at its narrowest gorge, then channel it to the collector." It can be
re-solved on any terrain, so it adapts instead of copying. That is the
user's "rules" option without needing an AI. A language model could later
write intents from plain language, but that is optional.

Terraform-to-template remains the exact-copy option, priced by the cut
and fill volume.

### 2. Lava replaces the forge

Lava is a second fluid: slow, viscous, hot. It solidifies into rock, and
by doing so it builds terrain. Where it meets water it makes steam (water
lost) and stone. The forge's anneal/quench idea moves onto the map:

- Lava cooled slowly becomes crystalline basalt: tough.
- Lava quenched in water becomes obsidian glass: hard, brittle,
  watertight.

Materials are made by routing lava and water to meet where you want a
wall. Ferrite and silica could become what lava carries, or be dropped.

### 3. Many channels between megatiles is compatible with the measured results

ADR 0002's ill-posedness came from **wide, open seams**. A dug channel is
naturally a narrow, single-cell interface. Each channel that crosses a
boundary can be its own port, so you can have as many as you like.
Open overland sheet flow across a boundary is the only hard case. Treat
that as a watershed divide, or as one more lumped port.

### 4. What goes up a level

A megatile's export card is its seasonal water and lava income at each
channel port. That is the SPICE characterization pattern, made
seasonal. The level above routes those flows between megatiles and
decides which megatiles to develop, adapt or terraform.

## Candidate next prototype

One megatile, about 70k cells (radius ~150), smaller and quicker than
Frontier. It would have:

- storm cells moving across it as the rain field;
- one or two lava vents, plus deep digs that can open new ones;
- intent tools (channel, contour dam, basin, levee), carried out by
  settler labor;
- collectors whose captured water and lava per season is the score;
- one simple contract or target to aim for.

Frontier's farms, lumber and the forge modal are stripped unless they
earn their place back.

## Open questions

- **Frontier's settlement economy** (farms, lumber, housing): keep a
  minimal labor and food layer, or strip it to water and lava?
- **What water and lava are for:** export currency that funds the level
  above, or consumed locally?
- **Lava as a hazard:** how dangerous should it be, given that the parked
  physics adversary (concepts/physics-adversary-blight.md) is another
  candidate threat?
