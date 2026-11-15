---
name: worldgen-noise
description: >-
  Design and implement the noise foundation layer for procedural world
  generation: PRNG/seed derivation, Perlin/OpenSimplex/FastNoiseLite setup,
  octaves and fractal Brownian motion (fBm), frequency tuning per parameter,
  value remapping tricks (2*abs, atlas-grid mapping, power/ridged shaping),
  and a shared hash function for position-keyed determinism. Outputs
  worldgen/noise-results.md with the project's complete noise stack design and
  reusable helper code. Use this skill whenever a project needs any noise-based
  generation — terrain, biomes, caves, scatter — and always run it BEFORE the
  other worldgen skills since they all consume the noise stack.
---

# Noise Foundations

You are designing the noise stack: the single most reused piece of the world generator. Terrain, biomes, caves, and scatter all sample your noise instances. Get the abstractions right once so every other system just calls them.

## Concepts you must get right

### Seeds and determinism
- Computers only do **pseudo-random**: a formula + a **seed** produces the same sequence every time. Same seed = same world; this is a feature, not a limitation.
- Derive sub-seeds per subsystem from the master seed so systems never correlate: `noise.seed = hash(master_seed, "temperature")`. Never reuse one noise instance for two meanings.
- **Per-position seeds**: for instance-keyed content (a specific cave, a specific sector), derive `hash(master_seed, x, y)`. This makes any point in the world independently reproducible without storing it.

### Shared hash function (Godot)
```gdscript
static func pos_seed(master: int, x: int, y: int, salt: String = "") -> int:
    var h := HashingContext.new()
    h.start(HashingContext.HASHING_SHA256)
    h.update(var_to_bytes(master))
    h.update(var_to_bytes(Vector2i(x, y)))
    h.update(salt.to_utf8_buffer())
    return hash(h.finish())
```

### Noise types and when to use them
- **Perlin / Simplex / OpenSimplex / FastNoiseLite**: smooth, continuous, seedable. Default choice for terrain, climate, everything organic. FastNoiseLite is built into Godot 4 and has fractal presets — prefer it.
- **White noise** (independent random per cell): for scattered variation (rare-biome patches, island seeds in cellular-automata stacks) — NOT for terrain, it looks like TV static.
- **Cellular/Voronoi noise**: sharp point-based features — mountains, mesa regions, crystal caves.
- **Perlin ≠ cave noise**: caves need a noise whose characteristic differs; use a separate instance with different fractal settings (or ridged), never the same instance as terrain.

### Octaves / fBm
Stack octaves: each octave doubles the frequency, halves the amplitude. More octaves = more fine detail. In FastNoiseLite: `fractal_octaves = 3..5`, `fractal_gain = 0.5`, `fractal_lacunarity = 2.0`. fBm is THE terrain look — a single octave is always too smooth/boring.

### Frequency tuning (the #1 tuning mistake)
- `frequency` = feature size. LOW frequency (e.g. 0.005) = continents/regions; HIGH = local bumps. One noise per meaning, each with its own frequency.
- Multiplier trick: noise ∈ ±0.5 (FastNoiseLite default) → multiply by a per-parameter scale to spread it over the working range: `moist*10`, `temp*10`, `altitude*150` (bigger multiplier = wider spread of that parameter's values).

## Remapping toolbox (implement all as helpers)
```gdscript
# map ±0.5 noise → 0..1 (tutorial classic)
func n01(x: float, y: float, n: FastNoiseLite) -> float:
    return 2.0 * abs(n.get_noise_2d(x, y))

# map ±0.5 → integer grid 0..N for atlas/matrix lookup
#   value*10 → ±5 ; +10 → 0..20 ; /step → round → index
func grid_index(v: float, step: float = 5.0, n: int = 4) -> int:
    return clampi(roundi((v * 10.0 + 10.0) / step), 0, n)

# shaping: pow(p) flattens lows & exaggerates highs → plains + mountains
# ridged: abs(n) * -1 → sharp mountain ranges
# domain warp: feed noise into the sample position → rugged, flowing terrain
func warped(n: FastNoiseLite, x: float, y: float, warp_amt: float = 30.0) -> float:
    var wx := n.get_noise_2d(x + 5.2, y + 1.3) * warp_amt
    var wy := n.get_noise_2d(x - 3.1, y + 7.7) * warp_amt
    return n.get_noise_2d(x + wx, y + wy)
```

## What to produce for THIS project
1. Read `worldgen/architecture.md` (dimension, features, seed strategy).
2. Define the **noise stack table**: for every consumer (terrain altitude, temperature, moisture, caves, rivers, vegetation, biome-variants...) → instance name, noise type, fractal settings, frequency, multiplier, derived salt.
3. Provide the helper script (hash, remaps, warp) as a real file in the project (`worldgen/noise_utils.gd` or engine equivalent) — other skills import it.
4. Set **fixed debug seeds** for every instance with a comment showing the release-time replacement.
5. Sanity-check: sample each noise over the world size and print min/max/mean — if a parameter's spread doesn't cover its lookup grid (e.g. biome matrix rows), fix the multiplier now, not after biomes break.

## Output format

Write to `worldgen/noise-results.md`:

```markdown
# Noise Stack Results
## Decisions
[table: consumer | type | octaves/gain | frequency | multiplier | seed/salt]
## Helper Code
[the actual noise_utils file you wrote, inline]
## Verification
[min/max/mean per instance, whether spreads fit the consumers' needs]
## Tuning Notes
[which knob changes what — e.g. "altitude frequency 0.005→0.002 = bigger continents"]
```

## Pitfalls
- Two consumers sharing one noise instance → correlated patterns (deserts always on hills). Salt every seed.
- Forgetting the ±0.5 vs ±1 range of your noise library → thresholds like `alt < 0.2` silently never trigger. Verify ranges once, in the helpers.
- Time-based seeds during development → unreproducible bugs. Fixed seeds until ship.
