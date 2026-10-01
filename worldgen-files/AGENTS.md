# Procedural World Generation — Build Workflow

Your goal is to design and implement a complete procedural world generation system for the game in the current directory (2D, 2.5D, or 3D — decided during analysis).

---

## Step 1: World Analysis & Specification

Before running, check if `worldgen/architecture.md` already exists. If it does, skip this step.

Run the worldgen-analysis skill directly (this one stays in-session since later steps depend on reading its output).

**Wait for this step to finish before proceeding.**

---

## Step 2: System Design & Implementation (Parallel)

Run only the skills the analysis marked as applicable. Skip any task where the output file already exists. Skip any task the analysis marked `N/A` (e.g., no towns → skip worldgen-sites' town parts are handled inside the skill; a pure 3D mesh game skips worldgen-autotiling entirely).

- Skip Noise Stack if `worldgen/noise-results.md` already exists.
- Skip Terrain if `worldgen/terrain-results.md` already exists.
- Skip Biomes if `worldgen/biomes-results.md` already exists.
- Skip Caves & Carving if `worldgen/caves-results.md` already exists (and analysis has no caves/rivers).
- Skip Scatter if `worldgen/scatter-results.md` already exists (and analysis has no objects/vegetation/mobs).
- Skip Sites if `worldgen/sites-results.md` already exists (and analysis has no towns/dungeons/structures).
- Skip Autotiling if `worldgen/autotiling-results.md` already exists (and analysis is not a tile-based 2D game).
- Skip Presentation if `worldgen/presentation-results.md` already exists.

Run the Noise skill first and wait for it (all other skills depend on the noise stack). Then start **one subagent per remaining check, all in parallel**, each with a dedicated task. Give each subagent the same instruction pattern:

> Read `worldgen/architecture.md` and `worldgen/noise-results.md` for context, then run the named worldgen skill. Implement the design decisions it prescribes for this project. Write all findings, parameters, and code to that skill's results file. Clean up any intermediate files for that skill when done.

| Skill | Results file |
|-------|--------------|
| worldgen-noise | `worldgen/noise-results.md` |
| worldgen-terrain | `worldgen/terrain-results.md` |
| worldgen-biomes | `worldgen/biomes-results.md` |
| worldgen-caves | `worldgen/caves-results.md` |
| worldgen-scatter | `worldgen/scatter-results.md` |
| worldgen-sites | `worldgen/sites-results.md` |
| worldgen-autotiling | `worldgen/autotiling-results.md` |
| worldgen-presentation | `worldgen/presentation-results.md` |

Wait for all subagents to finish before proceeding.

---

## Step 3: Master Implementation Plan

After all subagents from Step 2 finish, generate the final consolidated plan.

Skip this step if `worldgen/final-report.md` already exists.

Launch a single subagent:

> Read `worldgen/architecture.md` and all available `worldgen/*-results.md` files, then run the worldgen-report skill to generate `worldgen/final-report.md`: the full generation pipeline in execution order, per-system code, parameter cheat sheet, tuning checklist, and a phased build plan.
