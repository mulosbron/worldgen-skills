# Procedural World Generation — Build Workflow

Your goal is to design and implement a complete procedural world generation system for the game in the current directory (2D, 2.5D, or 3D — decided during analysis).

## Skills

| Skill | Read when | Output |
|---|---|---|
| `skills/worldgen-analysis/SKILL.md` | First step on any new or existing worldgen project: locks dimension, engine, world model, seed strategy, and applicability matrix. | `worldgen/architecture.md` |
| `skills/worldgen-noise/SKILL.md` | Designing the PRNG/seed derivation, Perlin/Simplex/FastNoiseLite stack, fBm, and position-keyed hashing. | `worldgen/noise-results.md` |
| `skills/fastnoiselite/SKILL.md` | Multi-language FastNoise Lite reference (C#, C++, C99, GLSL/HLSL, Rust, JS/TS): exact enums, defaults, cellular, and domain warp. | _(reference alongside `worldgen-noise`)_ |
| `skills/fastnoise2/SKILL.md` | C++17 SIMD node-graph noise (`FastNoise2` v1.1.1): FastSIMD backends, Node Editor serialized trees, fractals, domain warp, and bindings. | _(reference alongside `worldgen-noise`)_ |
| `skills/worldgen-terrain/SKILL.md` | Heightmaps, 2D side-scroller deformation, 3D voxel density, mesh heightfields, and player-centered chunking. | `worldgen/terrain-results.md` |
| `skills/worldgen-biomes/SKILL.md` | Threshold cascades, Whittaker matrix, Voronoi domains, multinoise climate space, and rivers. | `worldgen/biomes-results.md` |
| `skills/worldgen-caves/SKILL.md` | Noise-band carving, 3D cave intersections, Perlin worms, and depth-based ore difficulty. | `worldgen/caves-results.md` |
| `skills/worldgen-scatter/SKILL.md` | Weighted tiles, object/vegetation rules, population phase ordering, and context mob spawns. | `worldgen/scatter-results.md` |
| `skills/worldgen-sites/SKILL.md` | Jittered grid sampling, Wave Function Collapse, dungeon room stitching, and concentric structure rings. | `worldgen/sites-results.md` |
| `skills/worldgen-autotiling/SKILL.md` | Tile-based 2D games (Godot 4 Wang/Blob terrain sets, bitmasks, layered tilemaps, marching squares). | `worldgen/autotiling-results.md` |
| `skills/worldgen-presentation/SKILL.md` | World map, reveal/fog-of-war, water systems, isometric depth-perception fixes, and dev tuning overlays. | `worldgen/presentation-results.md` |
| `skills/worldgen-report/SKILL.md` | Consolidating all results into the final pipeline, parameter cheat sheet, and phased build plan. | `worldgen/final-report.md` |

---

## Step 1: World Analysis & Specification

Before running, check if `worldgen/architecture.md` already exists. If it does, skip this step.

Read and run `skills/worldgen-analysis/SKILL.md` directly (this stays in-session since later steps depend on reading its output).

**Wait for this step to finish before proceeding.**

---

## Step 2: System Design & Implementation (Parallel)

Run only the skills the analysis marked as applicable. Skip any task where the output file already exists. Skip any task the analysis marked `N/A` (e.g., no towns → town parts of `worldgen-sites` are skipped; a pure 3D mesh game skips `skills/worldgen-autotiling/SKILL.md` entirely).

- Skip Noise Stack if `worldgen/noise-results.md` already exists.
- Skip Terrain if `worldgen/terrain-results.md` already exists.
- Skip Biomes if `worldgen/biomes-results.md` already exists.
- Skip Caves & Carving if `worldgen/caves-results.md` already exists (or analysis has no caves/rivers).
- Skip Scatter if `worldgen/scatter-results.md` already exists (or analysis has no objects/vegetation/mobs).
- Skip Sites if `worldgen/sites-results.md` already exists (or analysis has no towns/dungeons/structures).
- Skip Autotiling if `worldgen/autotiling-results.md` already exists (or analysis is not a tile-based 2D game).
- Skip Presentation if `worldgen/presentation-results.md` already exists.

Run `skills/worldgen-noise/SKILL.md` first and wait for it (all other skills depend on the noise stack). Then start **one subagent per remaining applicable skill, all in parallel**, each with a dedicated task. Give each subagent the same instruction pattern:

> Read `worldgen/architecture.md` and `worldgen/noise-results.md` for context, then read and run `skills/<skill-dir>/SKILL.md`. Implement the design decisions it prescribes for this project. Write all findings, parameters, and code to that skill's results file. Clean up any intermediate files for that skill when done.

Wait for all subagents to finish before proceeding.

---

## Step 3: Master Implementation Plan

After all subagents from Step 2 finish, generate the final consolidated plan.

Skip this step if `worldgen/final-report.md` already exists.

Launch a single subagent:

> Read `worldgen/architecture.md` and all available `worldgen/*-results.md` files, then read and run `skills/worldgen-report/SKILL.md` to generate `worldgen/final-report.md`: the full generation pipeline in execution order, per-system code, parameter cheat sheet, tuning checklist, and a phased build plan.

---

## Boundaries

- Do not skip `worldgen-analysis` on a greenfield project unless `worldgen/architecture.md` already exists.
- Keep generation deterministic: derive all subsystem seeds from the master seed + position/sector hash rather than unseeded global RNG.
- Do not overwrite existing game code outside `worldgen/` during planning unless the user asks for implementation in the codebase.
