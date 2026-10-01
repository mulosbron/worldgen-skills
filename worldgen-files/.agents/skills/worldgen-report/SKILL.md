---
name: worldgen-report
description: >-
  Consolidate all worldgen system results into worldgen/final-report.md — the
  master implementation plan: the full generation pipeline in execution order,
  per-system code inventory, a parameter cheat sheet table, a tuning checklist
  (isotropy/play-scale, fixed debug seeds, probability sums, chunk sizing),
  known risks, and a phased build roadmap (MVP → full world). Runs LAST after
  all other worldgen skills have written their results files. Use this skill
  when all worldgen/*-results.md files exist and the user asks for the final
  plan, summary, roadmap, or "put it all together".
---

# Master Implementation Plan

Read `worldgen/architecture.md` and EVERY available `worldgen/*-results.md`. Synthesize — do not copy-paste. The report is the single document a developer works from; every claim must trace to a results file.

## Report structure (use this exact template)

```markdown
# Procedural World Generation — Final Report

## Executive Summary
[3-6 sentences: game, dimension, world model, the pipeline in one line,
current state — what is designed vs. implemented.]

## Locked Decisions (from architecture.md)
[dimension/view, world model + chunk size, seed strategy, persistence,
implementation target]

## Generation Pipeline (execution order)
[The canonical order with WHY — e.g.:
 1. Seed derivation (noise skill) — everything depends on it
 2. Noise stack sampling
 3. Terrain core (heightmap/density scan)
 4. Water fill / sea level
 5. Biome assignment (reads altitude; may modulate terrain depth/scale)
 6. Rivers & carving (caves skill)
 7. Site placement check (jittered grid validity — BEFORE decorations)
 8. Population phase: structures → decorations → ores (scatter skill)
 9. Map data render (presentation skill)
 10. Player diffs reapplied (persistence)]
[Mark each step: designed / implemented / tested]

## System Inventory
| System | Results file | Key files created | Status |
[one row per skill that ran, with its output files and status]

## Parameter Cheat Sheet
| Parameter | Value | System | Effect when changed |
[every named constant across results files: sea level, band widths,
frequencies, multipliers, sector size, dead zone, chunk size, border
width, reveal radii... This table is the tuning dashboard.]

## Tuning Checklist
- [ ] Play-scale isotropy: world stays varied within the area players roam (~N tiles); beyond it repetition is accepted by design
- [ ] Fixed debug seeds active during development; release seed path documented
- [ ] Every per-cell roll uses pos_seed determinism (no global RNG in generation)
- [ ] All weighted tile tables sum to 1.0; object chances < 0.03
- [ ] Chunk size ≥ view distance; chunk output independent of generation order
- [ ] Noise ranges verified (±0.5 vs ±1) against every threshold
- [ ] Biome histogram: no biome < 1% or > 60%
- [ ] Cave area ~5–15% of underground; no blobs
- [ ] Coastline gaps fixed via layering
- [ ] Population order respected everywhere (structures → decorations → ores)

## Risks & Open Questions
[from the results files' pitfalls sections, ranked by likelihood of biting]

## Phased Build Plan
Phase 1 (MVP): seed utils + terrain + water — a walkable world.
Phase 2: biomes + scatter — a varied world.
Phase 3: caves/sites — a world worth exploring.
Phase 4: presentation + blending + tuning pass — a world that reads well.
[Each phase: concrete tasks referencing the system inventory, and the
demo/verification that proves it.]
```

## Rules
- Pipeline order conflicts between results files → architecture.md wins; note the conflict explicitly.
- Any parameter appearing in two systems (e.g. sea level in terrain AND biomes) must appear ONCE in the cheat sheet with one source of truth.
- Keep the report under ~400 lines; details live in the results files — link, don't duplicate.
- End with the Phased Build Plan, no artificial closing remarks.

## Output
Write `worldgen/final-report.md`. Then clean up: delete any intermediate files left by other skills (they were told to clean up; verify with a directory listing and remove strays).
