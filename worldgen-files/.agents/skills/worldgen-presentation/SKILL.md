---
name: worldgen-presentation
description: >-
  Implement the presentation and UX systems that make a generated world
  legible and alive: world map generation from terrain gradient data, biome
  and vegetation map overlays, reveal/fog-of-war systems (travel points,
  permanent marks, local radius), depth-perception fixes for isometric views,
  water systems at scale (ponds to oceans, rivers), generation blending when
  parameters change, pixel-art filtering, cave/entrance lighting, and the
  top-down map as a dev tuning tool. Outputs
  worldgen/presentation-results.md. Use this skill whenever a project needs a
  minimap, world map, fog of war, exploration reveal, map markers, depth
  clarity, or visual polish for generated worlds.
---

# Presentation & World UX

Read `worldgen/architecture.md` (features, view) and all built-system results. Generation data already exists — this skill makes it READABLE. Two user problems drive everything: **depth perception** (which tile is higher?) and **orientation** (where was that town again?).

## World map (the #1 legibility tool)
Build the map from data the generator already has — do NOT screenshot the world:
- **Terrain gradient coloring**: sample how height CHANGES across the world (the derivative/shading of the heightmap) and color by it → immediate elevation read, which fixes isometric confusion far better than darkening lower levels (try the darkening hack only as a fallback; it usually looks bad).
- Overlays: biome colors + vegetation density on top of elevation shading → one glance = full picture.
- Render at low resolution from the same noise/height functions (sample every N tiles) — cheap, infinite-world compatible.
- **Dev-tool side effect (bake this in from day one)**: the top-down map is the single best tool for TUNING the generator — evaluating a sea-level or frequency change from above beats judging from ground level. Expose a debug toggle.

## Reveal system (exploration preserved)
Handing players the full map kills exploration; total fog kills orientation. Reveal system:
- Reveal a radius around **activated travel points/pillars** (activating = small gameplay beat).
- Always reveal a small radius around the player.
- Discovered sites (town centers, cave entrances) get **permanent markers**.
- Map fill-in IS the exploration reward loop.

## Water at scale
- Keep small ponds/lakes; add **oceans** by lowering the sea-level frequency (bigger features) rather than just more water.
- **Rivers**: carve with the same noise-band technique as caves (keep only the band, fill with water) — cheap and consistent-looking.
- Depth variants (shallow/deep) from altitude bands; underwater vegetation/creatures as scatter zones.

## Depth perception (2.5D isometric)
- Elevation shading (map) + in-world edge highlights on higher tiles.
- **Interiors as world-spaces** (already decided in sites/caves) is itself the big depth fix: going inside/underground = loading a separate scene, never drawing under visible terrain.
- Soft drop shadows under elevated objects; avoid heavy outline hacks.

## Generation blending (version/param changes)
When generator parameters change mid-project (or old saves exist): blend new-generation chunks with surrounding old terrain at borders (lerp heights over a transition margin) so edits/old saves don't leave cliffs. Delta persistence keeps player edits; blending keeps the landscape.

## Polish checklist
- Pixel art: nearest filtering (autotiling skill), consistent palette for biome map.
- Lighting: cave entrances visible from afar (glow + rail/props); glow-mushroom-style ambience deep underground.
- Controller/menu support is OUT of scope here (gameplay, not worldgen) — note it for the backlog.

## Output format
Write to `worldgen/presentation-results.md`: map rendering approach (sampling rate, color ramp, overlays), reveal parameters (radii, marker types), water/river params, blending strategy, code as written, verification (map render at 2 zoom levels; description of reveal flow), tuning notes.

## Pitfalls
- Map generated from tile sprites instead of noise data → unreadable mush at any real world size.
- Reveal radius so large that exploration dies; so small that the map never fills.
- Darkening lower z-levels as the only depth cue → players just get lost AND it looks muddy.
- Skipping the dev-map: tuning terrain from ground level costs 10× the iterations.
