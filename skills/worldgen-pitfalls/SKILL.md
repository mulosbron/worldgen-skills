---
name: worldgen-pitfalls
description: >-
  Mistakes that recur in procedural world generation and surface late (chunk
  seams, correlated noise, non-deterministic scatter, unreadable isometric
  caves). Read before writing any worldgen code; re-read when a generated
  world looks wrong and the cause is not obvious.
---

# Worldgen pitfalls

- Sampling noise with chunk-local coordinates → seams at chunk borders. Always sample by world coordinates.
- One noise instance serving two meanings (terrain + caves, altitude + moisture) → correlated features such as deserts always on hills. Salt every seed.
- Global RNG inside generation → chunks change on revisit. Hash (seed, position, salt) for every roll.
- Time-based seed during development → unreproducible bugs. Fixed seed until release.
- Assumed noise range (±1 vs 0..1 vs ±0.5) → thresholds silently never fire. Print min/max/mean once per noise and check against every threshold.
- Towns or dungeons placed by noise threshold → overlaps and no spacing guarantee. Use a jittered grid with dead zones (see `worldgen-sites`).
- Isometric: carving caves or interiors under visible terrain → unreadable. Load caves and interiors as separate spaces keyed by an id; doors and entrances are transitions.
- Scatter before terrain is final → trees over later-carved caves. Order: structures → decorations → ores.
- Weighted tile tables not summing to 1 → holes in the ground. Sparse object chances stay small (roughly < 0.03) and need not sum to 1.
- Side-scroller with only vertical noise offset → no cliffs or overhangs. Add a horizontal deformation pass.
- A single threshold for caves → blobs. Keep a band around the noise midpoint for winding tunnels; widen the band with depth.
- Biomes decided independently of terrain → mountains on beaches. Let biome modulate terrain height/scale or derive both from the same channels.
- Tile 2D: land placed without water underneath → background slivers on coastlines. Keep a full water layer below the land layer.
- Tuning the generator from ground level → ten times the iterations. Build the top-down map first.
- Hashing with a cryptographic function per cell → generation too slow. Use a cheap integer hash.
