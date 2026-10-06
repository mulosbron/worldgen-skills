---
name: worldgen-sites
description: >-
  Place towns, villages, ruins, dungeons, and quest-critical structures in a
  procedural world with hard guarantees noise cannot give: minimum spacing,
  one per region, valid terrain, houses connected to paths. Use whenever a
  generated world needs structures or spaced points of interest.
---

# Sites and structures

Organic systems want randomness. Sites want guarantees: "exactly one town per
region, never closer than N to the next, every house reaches a path". Raw
noise cannot promise any of that. Use structured randomness instead.

## Where: jittered grid with dead zones

Slice the world into sectors (one grid per site type, sized per type).
Sector seed = hash(master, sector coords, site type). From it pick one
jittered position inside the sector, excluding a border margin. The margin
guarantees minimum spacing between any two sites with no global coordination,
and the result is a pure function of the seed, so it works for infinite worlds.

Then validate the position against terrain and biome data (not in water, not
on a cliff). A site that fails validation simply does not spawn for this seed.
That is correct behaviour; log it during development.

## What: layout generation

- Rule-following layouts (towns, ruins): constraint collapse over a cell grid,
  seeded per cell, always collapsing the most constrained cell first. Keep the
  ruleset small at first (4–6 cell types).
- Authored-feel dungeons: stitch rooms from a prebuilt room library onto random
  sides of existing rooms until a target count, then place boss and treasure
  rooms far from the start.
- Game-critical structures (final dungeon, fast-travel anchors): compute
  positions geometrically first (concentric rings, roughly equal angles), then
  generate terrain around them.

## Interiors

Any site with an inside loads it as a separate space keyed by site id. This
keeps the outdoor view clean and is the same mechanism caves use.

## Persistence

Site data is a pure function of (master seed, sector). Store only player edits
inside a site (opened chests, broken walls) keyed by site id and cell.
