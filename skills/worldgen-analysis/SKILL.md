---
name: worldgen-analysis
description: >-
  Analyze a game project and lock the complete world generation specification:
  dimension (2D top-down / side-scroller / 2.5D isometric / 3D), engine and view
  (tilemap vs mesh), world model (fixed map vs infinite chunked), seed and
  persistence strategy, and the gameplay feature list (biomes, caves, towns,
  dungeons, ores, vegetation, mobs, minimap). Produces worldgen/architecture.md
  with an applicability matrix that decides which other worldgen skills run.
  Use this skill FIRST whenever the user asks to build, design, or improve
  procedural world generation, terrain generation, biome systems, or open
  worlds — even if they only say "make a world generator" or "add biomes to
  my game".
---

# World Analysis & Specification

You are the first phase of a world generation build. Your job is NOT to write generation code — it is to lock every upstream decision the other skills depend on, and write `worldgen/architecture.md`. A wrong decision here (e.g., fixed map vs. infinite) invalidates every downstream system, so resolve ambiguities explicitly.

## How to gather the spec

Inspect the project (engine, existing scenes, scripts, tilesets, shaders) and interview the user ONLY for what the project cannot answer. Prefer inferring from code over asking. Ask at most one batched round of questions.

## The decisions you must lock

### 1. Dimension & View
| Profile | Rendering | Simulation | Typical generators |
|---|---|---|---|
| 2D top-down | TileMap, orthographic | 2D grid (x,y) | noise heightmap + biomes |
| 2D side-scroller | TileMap, gravity | 2D column world (x, depth) | horizon + noise offsets, overhang pass |
| 2.5D isometric | 2D isometric sprites | **internal 3D grid** (x,y,z) | 3D data + 2D projection; interiors as separate world-spaces |
| 3D voxel | Block meshes/chunks | 3D grid (x,y,z) | 3D density scan |
| 3D mesh | 3D meshes/heightfield | continuous (x,z) → h | PlaneMesh displacement / chunked LOD |

Record: rendering dimension, simulation dimension, and whether going "underground/indoor" needs separate world-spaces (strongly recommended for isometric — carving caves under visible isometric terrain reads as confusing; loading interiors as their own scene keyed by an ID solves it).

### 2. World model
- **Fixed map**: generate once at startup (or on new-game). Simpler; fine up to ~1024×1024 tiles.
- **Infinite chunked**: generate around the player as they move. Requires: chunk = position-keyed pure function of the seed; chunk size slightly larger than screen/view distance so generation edges are never visible; generation must not depend on generation order.

### 3. Seed & persistence strategy (critical, decide now)
- One **master world seed** (64-bit). Every subsystem derives its own deterministic sub-seed: `sub_seed = hash(master_seed, system_name)` and per-instance: `hash(master_seed, position)` (e.g., per cave entrance, per sector).
- **Delta persistence**: never store generated content. Content is a pure function of (seed, position). Store ONLY player modifications (removed/placed blocks, opened chests) as diffs, and reapply them after generation. This makes near-infinite worlds free.
- **Fixed debug seeds**: during development every noise gets a hardcoded seed so failures reproduce; switch to `randi()`/time-based only for release. Bake this rule into the config.

### 4. Feature list (drives applicability matrix)
Mark each as YES/NO: biomes (which ones?), oceans/rivers, caves/underground, ores/resources, vegetation/objects, wildlife/hostile mobs, towns/NPC sites, dungeons/structures, minimap/world map, day-night or lighting.

### 5. Engine & implementation target
Default guidance in this toolkit is **Godot 4 + GDScript** (TileMapLayer/FastNoiseLite/terrain sets). If the project is Unity/Unreal/custom, keep every algorithm identical and translate the API calls; say so explicitly in the file.

## Applicability matrix (include this exact table)

```markdown
| Skill | Applicable | Why |
|---|---|---|
| worldgen-noise | YES/NO | every world needs the noise stack unless pure room-stitching |
| worldgen-terrain | YES/NO | terrain core |
| worldgen-biomes | YES/NO | only if >1 biome requested |
| worldgen-caves | YES/NO | only if caves/rivers/underground |
| worldgen-scatter | YES/NO | only if vegetation/objects/resources/mobs |
| worldgen-sites | YES/NO | only if towns/dungeons/structures |
| worldgen-autotiling | YES/NO | only if tile-based 2D view |
| worldgen-presentation | YES/NO | map/reveal/water polish — recommended if exploration matters |
```

## Output format

Write to `worldgen/architecture.md` using this exact template:

```markdown
# World Generation Architecture

## Game
- Name/genre, one-line pitch (if known)
- Engine + version, language
- Existing systems relevant to worldgen (player controller, camera, save system...)

## Locked Decisions
- Dimension & view: [profile from table]
- World model: [fixed map | infinite chunked] (chunk size if chunked: N×N)
- Master seed: [64-bit] — sub-seed derivation: hash(master, "system"/"x,y")
- Persistence: [none | delta persistence] — diff format: ...
- Implementation target: [engine specifics]

## Feature List
[YES/NO per feature, with requested specifics: biome names, ore list, mob ideas...]

## Design Notes
[Anything the user specified: art style, performance budget, target platform, scale of play (players will roam ~N tiles/blocks from spawn — design generator for the PLAY scale)]

## Applicability Matrix
[table above, filled in]
```

## Important reminders
- Design for the **play scale**, not the technical scale: worlds look repetitive far beyond the area players actually traverse; pick sizes where variety is dense.
- Record every threshold-ish user preference (how much water, how many towns) as a named parameter in the file — other skills must read parameters from here, not invent them.
- If the user's request is vague ("just make it look like Minecraft/Terraria/Isocore"), profile it: voxel 3D / side-scroller / isometric-2.5D respectively, and note the reference.
