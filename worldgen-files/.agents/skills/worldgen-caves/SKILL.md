---
name: worldgen-caves
description: >-
  Implement caves, tunnels, rivers and all carving systems: the 2D noise-band
  threshold technique (symmetric thresholds around 0.5 = winding cave paths),
  per-instance cave seeds from hashed entrance positions, separate
  world-space cave interiors for isometric games, threshold widening with
  depth, 3D two-noise intersection caves, cheese/spaghetti/noodle 3D-noise
  cave families, Perlin worm carving, cave elevation via second noise, ore
  distribution with distance-from-entrance difficulty gradients, and cave
  entrances/exits with lighting. Outputs worldgen/caves-results.md. Use this
  skill whenever a project needs caves, mines, tunnels, ravines, underground
  areas, rivers carving, or underground resource distribution.
---

# Caves & Carving

Read `worldgen/architecture.md` (dimension, underground features, persistence) and `worldgen/noise-results.md` (cave noise instances). Carving runs AFTER solid terrain exists.

## Technique 1 — Noise-band threshold (2D caves; also rivers)
The single most reusable carving trick. A raw noise-threshold (`noise > t`) gives blobs; a BAND gives winding paths:
```gdscript
# cave where noise value is INSIDE the band [t1, t2], symmetric around 0.5
var v := abs(cave_noise.get_noise_2d(x, y) * 2.0)   # 0..1
var d := abs(v - 0.5)
if d < band_half_width: place_cave()                  # e.g. band_half_width = 0.1
```
- Thresholds both pushed to the extremes → giant blobs; **symmetric around the middle** → connected winding paths that read as caves from above.
- Add **octaves** so edges aren't glassy-smooth.
- **Widen with depth**: increase `band_half_width` as y decreases (deeper = wider = more dangerous).
- **Elevation**: a second noise gives per-cell height offset inside the cave (rocky floors/ceilings).
- Same technique carves **rivers** in top-down worlds: keep the band, fill with water.
- Tune band width so caves never swallow huge portions of the map (test with a histogram: cave area ~5–15% is healthy).

## Technique 2 — Per-instance cave worlds (isometric / 2.5D)
Carving caves under visible isometric terrain reads as confusing. Instead:
1. Place a **cave entrance** on the surface (scatter skill or hand-authored spots).
2. Cave interior = its own **world-space/scene** keyed by an ID — the same mechanism used for building interiors.
3. Interior seed: `pos_seed(master, entrance_x, entrance_y, "cave")` → cave layout is unique per entrance AND fully reproducible. The interior is generated into the clean scene (void → fill with the band-technique shape + elevation noise).
4. Wire entrances like doors: walk in → load that cave's world-space; light the entrance so players can find it.

## Technique 3 — 3D caves (voxel worlds)
- **Two-noise intersection**: sample noise A along (x,z) and noise B along (y,z) (two axes); cave where the maps intersect (both above threshold) → tunnels + open chambers mix, very Minecraft.
- **Cave family by parameter** (all "threshold a 3D noise"): wide-open **cheese** caves (low frequency, high threshold), long **spaghetti** tunnels (two ridged noises intersected), thin **noodle** worms (tighter thresholds). Mix all three for depth variety.
- **Perlin worms**: a sphere walks a random-walk path from a seed, carving as it goes — classic ravines/tunnels. Combine with noise caves.

## Ore & reward distribution (risk-reward gradient)
- Rares belong underground, not on the surface: no challenge = boring acquisition.
- Place ores by rules, not pure rarity: **deeper = rarer** (depth bands per ore) AND **farther from the entrance = rarer**. Distance-as-difficulty: players must push past monsters to reach gold.
- Pair ores with light/vegetation dressing (glowing mushrooms etc.) so depth has atmosphere.
- Population order: carve caves → sprinkle ores (scatter skill handles placement) → decorations.

## Mobs & caves (hand-off note)
Record in the results file the *design hooks* other systems will use: darkness-adapted trackers that hunt by sound (player walking/mining makes noise), proximity bombers, ranged lobbers with a dodgeable AoE slam; spawn density scaling with distance from entrance. (Behavior implementation is out of scope here — but the spawn gradients ARE worldgen: put them in this file.)

## Entrances & exits
- Don't generate cave cells overlapping the entrance area; keep a safe room at the entrance.
- Mark entrances on the surface (rail, light, cracked rock) — findability is part of the design.
- If delta persistence is on, store only player edits inside caves (mined blocks) as diffs keyed by the cave's ID.

## Output format
Write to `worldgen/caves-results.md`: techniques chosen per feature, code as written, parameter table (band width, depth widening rate, thresholds per cave family, ore depth bands), verification (cave-area %, connectivity check), mob-spawn gradient table, tuning notes.

## Pitfalls
- One threshold instead of a band → blobs, not caves.
- Same noise instance for terrain and caves → caves mirror hills. Salt the cave seeds (noise skill).
- Forgetting entrance overlap masking → player spawns inside rock.
- 2.5D: trying to carve visible underground instead of world-space interiors → confusing mess.
