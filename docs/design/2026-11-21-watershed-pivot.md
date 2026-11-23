---
kind: design
name: "watershed-pivot"
status: prototype built (docs/watershed.html)
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

## Decisions and the first prototype (2026-11-23)

The user decided three things:

- **Labor and food stay.** People already have an intuition for them.
- **Water and lava are export currency.** They can also merge into
  obsidian.
- **Lava is a resource that is occasionally dangerous,** not a threat
  system.

`docs/watershed.html` implements this on one island megatile of
67,951 hexes, running in a web worker at about 3.5 ms per step.

**Weather.** Storm cells drift on a seasonal wind and rain where they
pass. A light drizzle falls everywhere.

**Lava.** Lava is a second fluid that carries heat. It is viscous, and
cools by air and by water. When it solidifies it adds to the terrain:
- cooled mostly by air, it becomes **basalt** (resists erosion 3);
- quenched by water, it becomes **obsidian** (resists erosion 9);
- quenching boils water off as steam.

Vents leak a little lava all the time. They rumble for 10 to 17 days,
then erupt at 14 to 30 times the base rate. A cut deeper than 3.5 m into
hot ground opens a breach vent.

**Erosion.** Flowing water wears down built ground (earth fill, basalt,
obsidian) in proportion to the flow divided by its resistance. That
makes spillways necessary.

**Intent earthworks.** Four tools, each turned into terrain changes by
a solver:
- **channel:** routed by Dijkstra over low ground, with a bed that
  always runs downhill;
- **dam:** a flat crest set at the lower bank's height;
- **levee:** a bank 1.5 m high;
- **basin:** a flat-bottomed excavation.

Earthworks are a Workforce row that takes every settler left over below
it. Each digger moves 1.5 earth per day. Cuts produce earth and fills
consume it, or consume obsidian.

**Export gates replace collectors.** Playtest arithmetic showed that a
cistern in a natural lake exported 2,000 water in year 1 against a
charter of 250, with no engineering. Natural water is already collected
water.

The fix is two or three fixed gates on dry coast, at least 7 hexes from
any river. Each gate takes at most 8 water per day, and only from land
hexes; an earlier intake reached into shallow sea and leaked seawater.

Measured, before any engineering: about 0 water per day. With one
93-hex channel from a lake: 3.2 per day, about 1,000 per year. The
charter is 600 in year 1, 2,400 in year 2 and 5,000 in year 3. Meeting
the later years needs several sources, storage, lava and obsidian.

**Lava traps** stay as local collectors, at 2 per day. **Obsidian
quarries** cut obsidian rock.

**Export value** is water + 3 × lava + 10 × obsidian.

### Not yet in the prototype

- the level above (seasonal export cards);
- intent blueprints that transfer to another megatile;
- terraform-to-template;
- an explicit way to steer lava beyond levees and channels.

## Autonomous iteration with Fable reviews (2026-11-23/24)

The user asked for the level above and the level below to be built, and
for iteration against Fable's critical reviews until the user would be
excited by every part.

### What exists

**`docs/watershed.html`, island and tile (the play level).** Fixes and
additions on the island:
- export gates no longer leak seawater;
- dam siting uses a cross-section solver;
- channels are aqueducts with built banks, and their routes avoid dams
  and reservoirs;
- the dam plan floods the would-be reservoir and reports the storage the
  dam actually adds;
- a sluice gate block can be set into a channel intake;
- a flow lens shows river discharge, and a channel readout reports what
  is available, carried and delivered, with a diagnosis;
- the island charter is set from measured runoff;
- other additions: dry-season water price, hired crews, lava traps as
  freezing pits, Quench works, blueprints.

A systemic bug was fixed along the way: the main thread had been
discarding almost every terrain update from the simulation worker.

**`docs/realm.html`, the level above.** Three contiguous tiles from one
global terrain function, coupled one way across their shared edges.
- The realm is surveyed headlessly when created.
- The capital asks for a share of the wild dry-season flow, rising only
  in years you meet it, from 10% up to 100%. Every dry-season unit
  beyond the wild flow pays a surplus bonus.
- The map shows rivers on the survey thumbnails and seasonal flow charts
  at each border and at the capital.

**`docs/works.html`, the level below.** A device board for the quench
works. Its card is a measured test year with dredging, at three inflow
rates. The default design makes 13.2 obsidian per 100 lava at flow rate;
a design that routes water down the lava chute makes 19.7.

### Measured findings

**Physics bugs found by the scripted players**, all fixed:
- the seawater leak through export gates;
- channels routed through their own dam;
- contour channels spilling over low banks;
- dam siting walking downhill;
- reservoirs spilling around the dam at a lower saddle;
- natural lakes counted as storage the dam added;
- the sluice reading the banks instead of the water level;
- the terrain-sync bug above.

**Island deliveries, water per year, three seeds:**

| Strategy | Water per year |
|---|---|
| Nothing built | 4–125 |
| Channel from a river | 121–247 |
| Dam plus channel | 276–470 |

**Storage is physically honest but weak.** The best sites add 500–1,350
units of storage, but the catchments above them carry only about 1–5 per
day. Following Fable's round 2, storage is no longer load-bearing: it
earns the realm's surplus contract.

**Blueprint transfer.** A source island delivering 450 per year was
applied to five unseen islands. They delivered 130–651 per year against
baselines of 1–9, a median of 44% of the source.

**Works card.** Keyed by basalt alone, the card's prediction missed the
fine board by 110–234%. The measured test year with dredging misses by
29–44%, and that comparison is now a developer metric.

### What is not solved

**Realm 12's coast (since resolved).** About 28 water per day of
dry-season inflow crossed the border into a large eastern lake, and the
capital sat on the far west coast. Three changes fixed it:
- the capital is now placed on dry coast about 32 hexes from the mouth
  of the main river, found by flow accumulation seeded with the upstream
  inflow;
- canals are now graded, with a steady fall from intake to destination:
  cut where the ground is higher, an embanked aqueduct where it is lower;
- the readout's own advice was followed: a 1 m diversion weir below the
  intake.

A scripted weir plus graded canal then delivered 260 dry-season water
against a year-1 demand of 268. Before the changes it delivered 0.

**Terraform-to-template** has not been built.
