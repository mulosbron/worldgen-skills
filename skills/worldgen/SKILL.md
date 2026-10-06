---
name: worldgen
description: >-
  Build a procedural world generator from scratch for the game in this repo
  (2D top-down, side-scroller, 2.5D isometric, or 3D voxel/mesh, any engine).
  Use whenever the user asks for terrain, biomes, caves, towns, infinite
  worlds, or says "make a world generator" / "add biomes to my game".
---

# Procedural world generator

Goal: a seeded, deterministic generator the player can walk around in, written
in the project's own engine and language. Working code is the deliverable.
Write design notes only where a decision is not obvious from the code.

## Decide first

Inspect the project before asking anything. Then write a short
`worldgen/DECISIONS.md` covering:

- Rendering vs simulation dimension. Isometric 2.5D usually simulates a 3D grid.
- Fixed map or infinite chunks. If chunks: chunk size > view distance, and a chunk
  is a pure function of (seed, chunk coords), never of generation order.
- Master seed → one salted sub-seed per system. What, if anything, is persisted
  (prefer storing only player edits as diffs and regenerating the rest).
- The feature list the user actually asked for. Build only those.
- Play scale: how far will players really roam? Tune variety for that area, not
  for the whole map.

## Build order

Walkable terrain → biomes → carving (caves, rivers) → sites (towns, dungeons)
→ scatter (vegetation, ores, spawn zones) → top-down debug map.
Show the user a result after each stage before moving on.

Build the top-down debug map as early as terrain exists. It is the tuning
tool, not a feature.

Read `worldgen-pitfalls` before starting. Read `worldgen-sites` if the world
needs towns, dungeons, or anything with spacing or connectivity guarantees.

## Done when

- The same seed produces the same world twice, including after reload.
- The player can traverse the world and every requested feature is visible.
- The debug map shows no biome, water, or cave class dominating or missing.
- Development runs on a fixed seed; the release seed path is a one-line switch.
