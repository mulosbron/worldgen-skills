---
name: worldgen-terrain
description: >-
  Implement the terrain core for any dimension: 2D top-down heightmaps with
  water thresholds, 2D side-scroller horizon + vertical noise offset + a
  second horizontal deformation pass for cliffs/overhangs, 3D voxel density
  column scanning (first noise3D(x,y,z) >= 0 = surface, 4x8x4 cell
  optimization, multi-fBm blending), 3D mesh heightfields (PlaneMesh
  displacement, central-difference normals, height-gradient shader), infinite
  player-centered chunking, and the terrain shaping toolbox (power shaping,
  ridged noise, domain warping, terracing, slope-based texturing). Outputs
  worldgen/terrain-results.md with working code. Use this skill whenever a
  project needs terrain generation, heightmaps, mountains, worlds, or ground
  — for any 2D, 2.5D or 3D game.
---

# Terrain Core

You implement the ground itself. Read `worldgen/architecture.md` (dimension, world model) and `worldgen/noise-results.md` (the stack you must consume). Implement ONLY the profile(s) that apply.

## Profile A — 2D top-down (TileMap)
Terrain = the altitude channel. Water where `alt < SEA_LEVEL` (sea level is a named parameter; typical 0.2–0.3 after remap to 0..1). Land otherwise. Per-cell:
```gdscript
var alt := altitude.get_noise_2d(x, y) * ALT_MULT
if alt < SEA_LEVEL: set_cell(water)  else: set_cell(land)  # biomes skill upgrades land tiles
```
Beaches come free later: a thin band just above sea level (e.g. `SEA_LEVEL..SEA_LEVEL+0.05`) → sand. Tune `ALT_MULT` / frequency until land:ocean ratio feels right (~50/50 is a good default; add a "Remove Too Much Ocean"-style pass if oceans dominate: ocean surrounded by ocean → 50% chance land).

## Profile B — 2D side-scroller (Terraria-style)
1. Define a flat **horizon** baseline.
2. **Vertical pass**: offset each column by 1D noise (pure random offsets = chaos; noise = smooth believable hills). Amplitude knob: less = rolling hills, more = rough mountains.
3. **Horizontal pass**: vertical-only offsets can never make overhangs or cliffs. Add a second pass that offsets points LEFT/RIGHT by another noise slice — terrain deforms, cliffs and overhangs appear.
4. **Cave carving happens in the caves skill**, not here — keep terrain solid.
5. More passes with different noise maps = rougher, more varied regions.

## Profile C — 3D voxel (Minecraft-style)
The height is NOT a 2D map — it's a **3D density scan**:
```gdscript
# per (x,z) column: first y (top→down) where density >= 0 is the surface
for y in range(WORLD_TOP, 0, -1):
    if fbm3d(x, y, z) >= 0.0: surface_y = y; break   # noise = density; <0 air, >=0 ground
```
- **Cell optimization** (essential): sample density on a coarse grid of 4×8×4 cells, trilinearly interpolate between cells — ~50× cheaper, visually identical.
- **Three fBm maps, not one**: sample two 3D fBm with different params (continental vs mountainous), blend by a third noise → biome-scale variety (plains vs ranges). If biomes exist, let the biome system modulate per-biome **depth** (average height) and **scale** (variation) so mountains never sit on beaches.
- Caves carve AFTER solid terrain (caves skill). Water fills below sea level y.

## Profile D — 3D mesh heightfield (Godot)
```gdscript
# MeshInstance3D + @tool script for live editor preview
var plane := PlaneMesh.new()
plane.subdivide_width = resolution; plane.subdivide_depth = resolution
var arrays := plane.get_mesh_arrays()
for i in vertex_array.size():
    var v: Vector3 = vertex_array[i]
    v.y = noise.get_noise_2d(v.x, v.z) * height          # displacement
    normal_array[i] = get_normal(v.x, v.z); tangent_array[i] = normal.cross(UP)
# central-difference normals: eps = size/resolution
#   normal = normalize( Vector3((h(x+eps)-h(x-eps))/(2*eps), 1, (h(x,z+eps)-h(x,z-eps))/(2*eps)) )
```
Shader (ShaderMaterial): `gradient_uv = model_vertex.y / (2.0*height) + 0.5` → sample a GradientTexture1D (deep water → sand → grass → rock → snow); add a seamless NoiseTexture2D normal map to break flat shading. Rebuild mesh when any noise param changes (`noise.changed → update_mesh`). For larger worlds: chunk the heightfield and apply the shaping toolbox per chunk.

## Terrain shaping toolbox (apply on top of any profile)
| Trick | Formula | Effect |
|---|---|---|
| Power shaping | `h = pow(n, 4.0)` | vast plains + dramatic mountains |
| Ridged | `h = abs(n) * -1.0` | sharp mountain RANGES instead of scattered peaks |
| Cellular | Voronoi distance noise | mountain masses / mesas |
| Domain warping | `h(x + w(x), z + w(z))` | rugged, flowing, natural coastlines |
| Terracing | `h = round(h / step) * step` | mesa plateaus |
| Slope texturing | `slope = (h(x2)-h(x1)) / dist` | steep→stone, flat→grass (no calculus needed) |

## Infinite chunking (if architecture says chunked)
- `generate_chunk(player_position)` on demand; convert world→tile/map coords once per chunk; loop the chunk with a `-size/2..+size/2` offset so chunks center on the player.
- Chunk **pure function of (seed, chunk_coords)** — never of generation order. Neighboring chunks must agree on shared borders (sample global noise by world coordinates, not chunk-local).
- Chunk size slightly larger than the view distance so the player never sees generation edges.

## Output format
Write to `worldgen/terrain-results.md`: Decisions (profile(s) used + why), the working terrain code (as written into the project), parameter table (sea level, amplitudes, frequencies, chunk size), verification (screenshots/descriptions of generated output), tuning notes.

## Pitfalls
- Sampling noise per-tile with chunk-local coords → visible chunk seams. Always sample by world coords.
- Forgetting the horizontal deformation pass in side-scrollers → boring wavy ceiling, no cliffs.
- 3D voxel terrain without cell interpolation → seconds-per-chunk generation times.
- Sea level set without checking the altitude distribution (print a histogram first).
