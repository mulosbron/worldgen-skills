---
name: worldgen-autotiling
description: >-
  Set up and automate Godot 4 autotiling for generated tile worlds: Wang vs
  Blob terrain sets (Match Corners vs Match Corners and Sides vs Match Sides
  modes), bitmask painting layouts with templates, terrain set + terrain
  configuration, batched set_cells_terrain_connect vs per-cell set_cell,
  layered tilemaps (water under land) to fix coastline gaps, nearest texture
  filtering for pixel art, and marching-squares guidance for non-Godot
  engines. Outputs worldgen/autotiling-results.md. Use this skill whenever a
  tile-based 2D project needs automatic edge/corner tile selection — cliff
  edges, coastlines, path borders, blob connections — or when generated tiles
  show wrong seams and gaps.
---

# Autotiling (Wang & Blob)

Read `worldgen/architecture.md` (view profile) and `worldgen/terrain-results.md` (which cells get which terrain). Autotiling solves a presentation problem: a generated map says "land here, water there" — autotiling picks the correct edge/corner piece for every tile automatically.

## Godot 4 concepts
- **Terrain Set** (on the TileSet): a group of terrains with a matching mode.
- **Terrain** (inside a set): one material, e.g. "land", "water", "cliffs" — has a picker color used while painting.
- Matching modes: **Match Corners and Sides** (≈ blob, full 3×3 bitmask, 47-ish cases, most customizability), **Match Corners** (≈ wang 3×3 minimal), **Match Sides** (2×2). Godot-3 equivalents people cite: 3×3 / 3×3 minimal / 2×2 — not exactly 1:1, verify by painting.

## Wang vs Blob — which one
| | Wang (Match Corners) | Blob (Match Corners and Sides) |
|---|---|---|
| Tile count needed | fewer | many more |
| Best for | **top-down** worlds | **side-scrollers** (need ground/ceiling/wall variants) |
| Setup effort | quick | careful bitmask template |

## Setup recipe (TileSet editor)
1. TileSet → drag tileset image → auto-create tiles.
2. Terrain Sets → Add element → choose mode → add terrain(s) ("land", color picker).
3. Paint → Terrains → select terrain → **Ctrl+Shift drag** to assign all tiles to the terrain.
4. Paint each tile's **bitmask** using a template (do it identically every time; the standard layouts work ~99% of cases):
   - Wang: a plus/center pattern — center tile checks left/right/up/down neighbors; edge pieces check their two relevant neighbors; corners check both sides.
   - Blob: per-piece mental model — "3×3 grid filled except edges", "3×1 filled except ends", hollow square, T-shapes, then the L-pieces: paint a hashtag pattern first, then extend legs.
   - Mistakes: right-click to clear a bit.
5. **Pixel art**: Project Settings → default_texture_filter → **nearest** (linear makes tiles blurry).
6. One TileMap can host multiple TileSets; use **layers** to stack independent terrain sets (water layer 0 under land/cliff layer 1).

## Code — two call patterns
```gdscript
# A) Batched terrain connect — for autotiled material (land/cliffs): pick up ALL cells at once
var land_cells: Array[Vector2i] = []      # filled during generation loop
land_cells.append(coords)
# after the loop:
tilemap.set_cells_terrain_connect(land_cells, terrain_set_id, terrain_id)  # e.g. 0, 0

# B) Plain set_cell — for non-autotiled content (water variants, biome atlas tiles)
tilemap.set_cell(layer, Vector2i(x, y), source_id, atlas_coords)
```
Multi-biome worlds combine both: biome tiles via atlas lookup on layer 0 (biomes skill), cliff/edge autotiles on layer 1 — append a cell to `cliff_cells` whenever the biome tile is land (`atlas.x != water_column`), then one `set_cells_terrain_connect(cliff_cells, ...)` call paints all coastlines.

## Coastline gap fix
Generated worlds show gray/background slivers at land-water borders when land is placed WITHOUT under-tiles. Fix: draw water on layer 0 everywhere below sea level, autotiled land on layer 1 — layers exist precisely so terrains don't erase each other.

## Non-Godot engines
Same math, different names: marching-squares autotiles (Unity rule tiles), 47-blob bitmask / 16-tile wang sets (Tiled). Port the bitmask layouts 1:1; the batched-connect call becomes "run the bitmask resolver per cell after generation".

## Output format
Write to `worldgen/autotiling-results.md`: terrains defined (names, modes), bitmask template used (paste the painted layout), code as written, layer stack (which layer per terrain set), verification (screenshot/description of edges on coasts, cliffs, paths), known bad-tile notes (e.g. tiny diagonal gaps — acceptable or patched).

## Pitfalls
- Wrong mode for the art (blob tiles in a wang set) → missing corner cases.
- Painting bitmasks freehand without a template → one wrong bit = one permanently-wrong tile.
- Per-cell set_cells_terrain_connect calls in a loop → brutal slowdown; batch once after generation.
- Linear filtering on pixel art → blurry seams that look like bitmask errors.
