---
name: worldgen-biomes
description: >-
  Design and implement the biome system: choose among threshold cascades,
  Whittaker temperature-x-moisture matrices, tileset-atlas-as-graph lookup,
  infinite jittered-grid Voronoi with domain warping and border blending,
  climate-space multinoise (temperature/humidity/continentalness/erosion/
  weirdness nearest-biome), stochastic cellular-automata layer stacks, and
  river generation via edge detection. Includes biome-variant patches,
  shorelines, ocean temperature/depth, and the biome-grid-chart design
  practice. Outputs worldgen/biomes-results.md. Use this skill whenever a
  project needs biomes, climate zones, regions, deserts/forests/snow, ocean
  depth zones, or rivers — for 2D, 2.5D or 3D worlds.
---

# Biome Systems

Read `worldgen/architecture.md` (biome list, dimension) and `worldgen/noise-results.md` (temperature/moisture/etc. instances). Pick the SMALLEST system that satisfies the design — biomes are a lookup problem, not a code problem.

## Decision tree
- **1–6 biomes, small fixed map** → Threshold cascade or **Whittaker matrix**.
- **Tile-based, want zero if-chains** → **Atlas-as-graph** (biome matrix image = lookup table).
- **Infinite world, organic borders** → **Jittered-grid Voronoi** + domain warp + blending.
- **Many biomes + terrain coupling + cave biomes** → **Multinoise climate space** (nearest-biome).
- **Retro/sandbox feel with distinct region shapes** → **CA layer stack** (advanced).

## Method 1 — Threshold cascade (simplest)
```gdscript
if alt < 0.2: ocean
elif alt < 0.25: beach
else:
    if moist < 0.4 and temp > 0.7: desert
    elif moist > 0.4 and temp > 0.6: jungle
    else: plains
```
Write ranges with a `between(v, lo, hi)` helper. Order matters: check water/edges first.

## Method 2 — Whittaker matrix (the chart practice)
Draw a **temperature × moisture grid and fill EVERY cell** with a biome (Whittaker diagram: cold+dry→tundra, cold+wet→taiga, hot+dry→desert, hot+wet→rainforest, plus swamp/plains/forest/shrubland/savanna). This guarantees no combination is unhandled and transitions feel natural (nearby cells = nearby biomes). Implement as a 2D lookup array indexed by the noise grid indices.

## Method 3 — Atlas-as-graph (Godot tiles)
Design the tileset atlas itself as the biome lookup: atlas X axis = moisture (increasing), Y axis = temperature (up = colder), one extra column for water variants. Then biome selection is one expression, no code tables:
```gdscript
var atlas := Vector2i(grid_index(moist), grid_index(temp))   # grid_index from noise skill
if alt < WATER_ALT: atlas = Vector2i(WATER_COL, grid_index(temp))
```
Expand the matrix with depth variants (shallow/deep water by altitude bands). The same image doubles as design documentation.

## Method 4 — Infinite jittered-grid Voronoi (the pro method)
Height-only biomes produce predictable rings — Voronoi regions look like real biome patches:
1. Divide the world into grid cells (e.g. 1 cell = 64 tiles). Per cell: `point = cell_origin + vec2(rand_from_hash(cell_x, cell_y))` — **jittered grid** gives uniform-but-random placement; checking only neighbor cells' points keeps it O(small).
2. Per world position: biome = nearest cell's biome. **Infinite by construction** — each cell's point is a pure function of (seed, cell coords).
3. **Domain warp**: offset the sample position by Perlin noise before the nearest-point search → organic curved borders instead of straight lines.
4. **Border blending**: track nearest AND second-nearest cells; `border_dist = d2 - d1`; if `border_dist < BIOME_BORDER_WIDTH` → blend the two biomes' colors/params. Kills harsh transitions.
5. **Climate coherence**: assign each cell a temperature + humidity from slow Perlin gradients; classify into quadrants (hot/cold × humid/dry) **plus a neutral middle band**; pick the cell's biome from its quadrant's table. Neutral buffer prevents deserts adjacent to tundra.

## Method 5 — Multinoise climate space (Minecraft-1.18-style, scales to huge)
Per position (per quarter-chunk in voxel worlds) sample **5 channels**: temperature, humidity, **continentalness** (coast distance), **erosion** (flat↔mountainous), **weirdness** (variants + peaks/valleys). Every biome is defined by an ideal 5-vector; assign the nearest-matching biome (nearest neighbor in climate space). Enables 3D biomes (cave biomes) by sampling the channels in 3D. Terrain height then derives from the SAME channels (depth from continentalness/erosion, peaks&valleys from weirdness) — biomes and terrain can never disagree.

## Method 6 — CA layer stack (advanced, authored feel)
Stochastic cellular automata layers on a low-res map, coarse→fine: Island (white noise, land:ocean 1:10) → Zoom ×N (each zoom doubles resolution AND adds imperfect-copy variation) → Add Island (land expands into shallow-water corners, sometimes erodes) → Remove Too Much Ocean → temperature assignment (e.g. warm:cold:freezing 4:1:1) + temperature smoothing layers → biome assignment tables (warm = 50% desert / 33% savanna / 17% plains) → variants (hilly versions, rare patches: 1-in-N conversions) → Mushroom-island-style rare biome (surrounded-by-water, 1/100) → Shore layer (beaches) → final Voronoi zoom to break straight edges. Rivers are a SEPARATE stack: edge detection along noise-patch seams → low-pass smoothing → carve into the main stack (skip in frozen biomes).

## Cross-cutting rules
- **Rare/patch biomes** (bamboo jungle, flower fields): convert 1-in-N cells INSIDE a parent biome, never standalone regions.
- **Ocean temperature/depth**: separate slow noise → warm/lukewarm/cold/frozen; depth from altitude bands.
- **Rivers** (simple alternative to the river stack): the same noise-band technique used for caves — threshold a noise and keep only the band, overlay on terrain (frozen in snow biomes).
- Version/blending: if you change generator params later, blend new chunks with old borders instead of regenerating (delta persistence handles edits).
- Print a biome histogram over a test area — any biome < 1% or > 60% needs parameter tuning.

## Output format
Write to `worldgen/biomes-results.md`: chosen method + why, the biome lookup data (matrix/table filled for THIS project's biome list), code as written, parameter table (cell size, border width, quadrant bounds), biome histogram verification, tuning notes.

## Pitfalls
- If-chain biome code with a gap in the ranges → unassigned tiles. Matrix/chart methods make gaps structurally impossible.
- Voronoi without domain warp → straight artificial borders; without neighbor-cell optimization → O(all points) per pixel.
- Biome decided independently of terrain → mountain beaches. Couple depth/scale (Method 5 does it inherently).
