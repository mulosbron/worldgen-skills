---
name: worldgen-scatter
description: >-
  Implement object, vegetation, resource and wildlife placement over generated
  terrain: weighted random tile variation (probabilities sum to 1), sparse
  object chance tables (< 0.03), rule-based surroundings constraints (trees
  only on surface, flowers only in grass, bats only in wide caves, lava by
  depth, ores by depth bands), YSort scene instancing with tile centering,
  population phase ordering (structures first, decorations second, ores
  last), and movement-context mob spawn zones tied to terrain type. Outputs
  worldgen/scatter-results.md. Use this skill whenever a project needs trees,
  plants, rocks, decorations, resources, item spawns, enemy spawn zones, or
  any objects placed on generated terrain.
---

# Scatter & Population

Read `worldgen/architecture.md` (features), `worldgen/noise-results.md` (vegetation noise) and `worldgen/terrain-results.md`/`worldgen/biomes-results.md` (the surface data you decorate). Scatter runs LAST in the generation order — decorate a finished world.

## Layer 1 — Tile variation (weighted random, fills 100% of a biome)
Per-biome tile tables with probabilities that **must sum to 1.0** (otherwise the tilemap gets gaps):
```gdscript
const BIOME_TILES := {
    "desert": {"sand": 0.95, "stone": 0.05},
    "plains": {"grass": 0.9, "flowers": 0.06, "dirt": 0.04},
}
func random_tile(biome: String) -> String:
    var roll := randf()   # seeded PRNG from pos_seed — must be deterministic per cell!
    var total := 0.0
    for tile in BIOME_TILES[biome]:
        total += BIOME_TILES[biome][tile]
        if roll < total: return tile
    return BIOME_TILES[biome].keys().back()
```
⚠️ Use the per-cell deterministic PRNG (seed = pos_seed(master, x, y, "tile")), not the global RNG, or chunked worlds change on revisit.

## Layer 2 — Object instancing (sparse scenes, does NOT fill everything)
- `OBJECT_TILES`: name → preloaded PackedScene (StaticBody2D tree/cactus/rock).
- `OBJECT_DATA`: biome → {object: chance}; chances need **not** sum to 1 — use small values (< 0.03) for natural sparseness; roll per tile, skip on miss.
- Placement: instance the scene, position = `map_to_world(cell) + Vector2(tile_size/2.0)` to center on the tile, add as child of the **YSort** node so the player walks behind/in front correctly.
- Performance: for chunked worlds instance objects per-chunk and free them with the chunk; never iterate the whole world.

## Layer 3 — Rule-based constraints (the "surroundings" system)
Object placement is determined by surroundings, evaluated as simple predicates per tile:
- trees/grass → surface tiles only; flowers → grass tiles only; mushrooms → cave floors; bunnies → where grass AND flowers coexist
- bats → caves wider than X; lava → below depth Y; ore type → depth band table (from caves skill)
- water mobs → large water bodies only, avoiding shallows; desert walkers → sand tiles only
Compose constraints as data: `{scene: "tree", requires: ["surface", "plains"], chance: 0.02}`. Complexity is tunable — start with 2 constraints per object, add more when the world feels sterile.

## Layer 4 — Population phase ordering (Minecraft rule)
When multiple systems place things, ORDER MATTERS (later placement must not overlap earlier):
1. **Structures first** (towns, dungeons — sites skill)
2. **Decorations second** (trees, plants, small features)
3. **Resources last** (ore veins, treasure)
Record the canonical order for this project so every system respects it.

## Layer 5 — Movement-context mobs (spawn zones, not behavior)
Define WHERE things can spawn as worldgen data (behavior is out of scope):
- zones tied to terrain: large-deep-water only / sand-only / dark-cave-only
- sensing-context notes: step-sensing predators need open sand; hearing-based trackers need enclosed caves — note these in the results so gameplay can balance
- density gradients from the caves skill (distance from entrance) and overworld rarity table

## Determinism & persistence
- Every roll derives from `pos_seed(master, x, y, salt)` — the same bush grows in the same place forever.
- With delta persistence, only player changes (chopped trees, picked flowers, mined ore) get stored as diffs and reapplied after generation.

## Output format
Write to `worldgen/scatter-results.md`: placement tables (per-biome tile weights, per-biome object chances with constraints), instancing code as written, population order, mob spawn-zone table, determinism notes, verification (spawn density per biome — count instances per 100×100 test area).

## Pitfalls
- Tile probabilities not summing to 1 → visible holes in the ground.
- Global RNG instead of per-cell hashed rolls → world mutates on reload.
- Forgetting the half-tile offset → objects float off the grid.
- Scatter before terrain is final → trees floating over later-carved caves. Population order exists for a reason.
