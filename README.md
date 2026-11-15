# Procedural World Generation Skills

A collection of agent skills that turn your AI coding assistant into a **procedural world generator architect**: it designs and implements complete world generation systems for 2D, 2.5D (isometric), and 3D games — terrain, biomes, caves, object scatter, towns/dungeons, autotiling, and map/presentation systems. Works natively with Antigravity, Claude Code, Codex, Opencode, Cursor, Windsurf, and any other assistant that supports `AGENTS.md` and agent skills.

Techniques are abstracted from real production systems (voxel sandbox games, isometric open-world sandboxes, Godot/Unity systems, and classic roguelike heritage) into engine-agnostic guidance with concrete Godot 4 / GDScript implementations plus portable pseudocode for any engine.

## Install

Copy this folder into your project (or keep it in your global `~/.agents/` directory) so coding agents discover `AGENTS.md`:

```bash
cp -r worldgen-skills/ my-project/.agents/
```

Agents that read `AGENTS.md` pick up the skill table and load individual skills on demand.

## How It Works

`AGENTS.md` orchestrates the entire world-building workflow automatically in three steps:

1. **World Analysis & Specification** — `skills/worldgen-analysis/SKILL.md` locks the design: dimension (2D / 2.5D / 3D), engine, view (tiles vs. mesh), world model (fixed map vs. infinite chunked), seed & persistence strategy, and gameplay feature list. It writes its findings to `worldgen/architecture.md`.
2. **System Design & Implementation (parallel)** — `skills/worldgen-noise/SKILL.md` runs first, then up to 7 remaining system skills run in parallel as subagents. Each skill writes its algorithms, starting parameters, code, and pitfalls to `worldgen/*-results.md`. Steps whose output file already exists or are marked `N/A` are skipped automatically.
3. **Master Implementation Plan** — `skills/worldgen-report/SKILL.md` consolidates everything into `worldgen/final-report.md`: the full generation pipeline in execution order, code per system, a parameter cheat sheet, and a phased build plan.

## Skills

| Skill | System |
|---|---|
| `skills/worldgen-analysis/` | World spec recon: dimension, engine, seed strategy, feature list, applicability matrix |
| `skills/worldgen-noise/` | PRNG/seeds, Perlin/Simplex/FastNoiseLite, octaves, fBm, value remapping, shaping math |
| `skills/fastnoiselite/` | Cross-language FastNoise Lite reference (C#, C++, C99, GLSL/HLSL, Rust, JS/TS), enums, defaults, cellular & domain warp |
| `skills/fastnoise2/` | C++17 SIMD node-graph noise (`FastNoise2` v1.1.1), FastSIMD dispatch, serialized Node Editor trees, C#/Unreal/Rust/Java bindings |
| `skills/worldgen-terrain/` | Heightmaps, 3D density scanning, overhang deformation, chunking, terrain shaping tricks |
| `skills/worldgen-biomes/` | Threshold cascades, Whittaker matrix, atlas lookup, infinite Voronoi, multinoise climate space |
| `skills/worldgen-caves/` | Noise-band carving, 3D cave intersections, worms, rivers, depth-based ore difficulty |
| `skills/worldgen-scatter/` | Weighted tiles, object/vegetation rules, population phase ordering, context mobs |
| `skills/worldgen-sites/` | Jittered grid sampling, Wave Function Collapse, room stitching, structure rings |
| `skills/worldgen-autotiling/` | Wang/Blob terrain sets, bitmasks, layered tilemaps, batched terrain connects |
| `skills/worldgen-presentation/` | World map, reveal/fog systems, water, depth-perception fixes, generation blending |
| `skills/worldgen-report/` | Consolidated pipeline, parameter cheat sheet, phased build plan |

## Usage

Open your project in your AI coding assistant and ask:

> Build a procedural world generation system for my game

or

> Generate a world: 2D top-down Godot 4, infinite, with biomes, caves and towns

`AGENTS.md` drives the full workflow automatically and skips any steps whose output files already exist, so you can safely re-run after adjusting decisions.

## Output

All output is written to a `worldgen/` folder in your project root:

| File | Description |
|---|---|
| `worldgen/architecture.md` | Locked world spec: dimension, engine, seed/persistence, feature list, applicability matrix |
| `worldgen/*-results.md` | Per-system design: chosen algorithms, parameters, code, pitfalls |
| `worldgen/final-report.md` | Master implementation plan ranked by pipeline order, with cheat sheet and phased roadmap |

## Layout

```text
worldgen-skills/
├── AGENTS.md
├── README.md
├── plugin.json
└── skills/
    ├── worldgen-analysis/SKILL.md
    ├── worldgen-noise/SKILL.md
    ├── fastnoiselite/SKILL.md
    ├── fastnoise2/SKILL.md
    ├── worldgen-terrain/SKILL.md
    ├── worldgen-biomes/SKILL.md
    ├── worldgen-caves/SKILL.md
    ├── worldgen-scatter/SKILL.md
    ├── worldgen-sites/SKILL.md
    ├── worldgen-autotiling/SKILL.md
    ├── worldgen-presentation/SKILL.md
    └── worldgen-report/SKILL.md
```
