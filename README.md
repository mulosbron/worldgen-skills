# Procedural World Generation Skills

A collection of agent skills that turn your LLM coding assistant into a **procedural world generator architect**: it designs and implements complete world generation systems for 2D, 2.5D (isometric), and 3D games — terrain, biomes, caves, object scatter, towns/dungeons, autotiling, and map/presentation systems. Works natively with Claude Code, Codex, Opencode, Cursor and any other assistant that supports agent skills. No third-party tools required.

Techniques are abstracted from real production systems (voxel sandbox games, isometric open-world sandboxes, Godot/Unity tutorials, and classic roguelike heritage) into engine-agnostic guidance with concrete Godot 4 / GDScript implementations, plus portable pseudocode for any engine.

## How It Works

`CLAUDE.md` (for Claude Code) or `AGENTS.md` (for Opencode and other IDEs) orchestrates the entire world-building workflow automatically. The workflow runs in three steps:

1. **World Analysis & Specification** -- The `worldgen-analysis` skill locks the design: dimension (2D / 2.5D / 3D), engine, view (tiles vs. mesh), world model (fixed map vs. infinite chunked), seed & persistence strategy, and gameplay feature list. It writes its findings to `worldgen/architecture.md`.

2. **System Design & Implementation (parallel)** -- Up to 8 system skills run in parallel as subagents. Each skill designs its system: algorithms chosen via decision trees, concrete parameters with starting values, implementation code, and pitfalls. Results are written to `worldgen/*-results.md`. Steps whose output file already exists are skipped, and steps the analysis marked N/A are skipped automatically.

3. **Master Implementation Plan** -- The `worldgen-report` skill consolidates everything into `worldgen/final-report.md`: the full generation pipeline in execution order, code per system, a parameter cheat sheet, and a phased build plan.

## The Skill Set

| Skill | System |
|---|---|
| worldgen-analysis | World spec recon: dimension, engine, seed strategy, feature list, applicability matrix |
| worldgen-noise | PRNG/seeds, Perlin/Simplex/FastNoiseLite, octaves, fBm, value remapping, shaping math |
| worldgen-terrain | Heightmaps, 3D density scanning, overhang deformation, chunking, terrain shaping tricks |
| worldgen-biomes | Threshold cascades, Whittaker matrix, atlas lookup, infinite Voronoi, multinoise climate space |
| worldgen-caves | Noise-band carving, 3D cave intersections, worms, rivers, depth-based ore difficulty |
| worldgen-scatter | Weighted tiles, object/vegetation rules, population phase ordering, context mobs |
| worldgen-sites | Jittered grid sampling, Wave Function Collapse, room stitching, structure rings |
| worldgen-autotiling | Wang/Blob terrain sets, bitmasks, layered tilemaps, batched terrain connects |
| worldgen-presentation | World map, reveal/fog systems, water, depth-perception fixes, generation blending |
| worldgen-report | Consolidated pipeline, parameter cheat sheet, phased build plan |

## Installation

Create a project folder (or use your existing game project), then copy the `worldgen-files` contents into it and open that folder as your workspace in your AI coding assistant.

```bash
cp -r worldgen-files/.agents worldgen-files/AGENTS.md /path/to/your-game-project/
# or for Claude Code:
cp -r worldgen-files/.claude worldgen-files/CLAUDE.md /path/to/your-game-project/
```

> **Note:** If your project already contains a `CLAUDE.md` or `AGENTS.md` file, remove it before running — otherwise it will conflict with the orchestration file provided by this toolkit.

## Usage

Open your project in your AI coding assistant and ask:

> Build a procedural world generation system for my game

or

> Generate a world: 2D top-down Godot 4, infinite, with biomes, caves and towns

The entry point file (`CLAUDE.md` or `AGENTS.md`) orchestrates the full workflow automatically. It skips any steps whose output files already exist, so you can safely re-run it after adjusting decisions.

## Output

All output is written to a `worldgen/` folder in your project root:

| File | Description |
|---|---|
| `worldgen/architecture.md` | Locked world spec: dimension, engine, seed/persistence, feature list, applicability matrix |
| `worldgen/*-results.md` | Per-system design: chosen algorithms, parameters, code, pitfalls |
| `worldgen/final-report.md` | Master implementation plan ranked by pipeline order, with cheat sheet and phased roadmap |
