# NOTES

Running log of work sessions. `/wrap` appends a new dated section at the
end of each session with **Did / Findings / Next** subsections. The
SessionEnd hook backstops this if you forget.

<!-- entries go below this line, newest at bottom -->

## 2026-11-24 — Watershed: three levels, autonomous iteration with Fable

### Did
- Built `docs/watershed.html`: one island tile, water and lava simulated across the whole tile, intent earthworks (channel, dam, levee, basin), sluice gate, export gates or capital, a flow lens, channel and dam readouts, blueprints.
- Built `docs/realm.html` (the level above): three contiguous tiles from one global terrain, one-way edge coupling, a headless survey and year close, and a capital demand pegged to the measured wild dry-season flow. The map shows rivers and seasonal flow charts.
- Built `docs/works.html` (the level below): a quench device board whose card is a measured test year with dredging.
- Ran three rounds of Fable review (0, 1, 2) and adopted most of the advice. Scripted headless players measured every claim.

### Findings
- The scripted players caught eight physics bugs that playtesting by eye would have missed. All are fixed:
  - gates drank seawater;
  - a channel cut through its own dam;
  - contour channels spilled over their banks;
  - dam siting walked downhill;
  - reservoirs leaked round a lower saddle;
  - natural lakes counted as dam storage;
  - the sluice read the banks instead of the water level;
  - the main thread dropped worker terrain updates.
- After the fixes, island deliveries were: nothing built 4–125; channel only 121–247; dam plus channel 276–470 water/yr.
- Storage is physically honest but weak, because catchments carry about 1–5/day. It was demoted to the realm's surplus contract.
- Getting water to the destination is the real game. Graded canals plus a low weir delivered 260 dry-season water against a demand of 268 on the realm 12 coast; before the fixes it was 0.
- Blueprints re-solved on five unseen islands reached 130–651 water/yr from baselines of 1–9, a median of 44% of the source.
- A predictive works card failed (110–234% error) because where the basalt sits matters. A measured test year with dredging works: 13.2 vs 19.7 obsidian per 100 lava for two designs, with card and board 29–44% apart.

### Next
- Owner playtest of all three levels; the bots cannot judge fun.
- Terraform-to-template (not built).
- Better blueprint re-anchoring (44% → 60%+).
- Scale the realm beyond 3 tiles only once the 3-tile loop is proven.
