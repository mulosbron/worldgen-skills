---
name: fastnoiselite
description: Portable multi-language FastNoise Lite library reference and integration guide (C#, C++, C99, GLSL, HLSL, Rust, TypeScript/JS, Go, Zig, Odin, Java) for procedural noise, fractals, cellular Voronoi, and domain warping across game engines and shaders.
---

# FastNoise Lite Reference & Integration

Use this skill alongside `worldgen-noise` whenever the project uses the portable upstream **FastNoise Lite** library outside built-in GDScript (e.g., Unity C#, Godot C#, custom C++/C99 engines, GLSL/HLSL shaders, Rust/Bevy, TypeScript/Three.js, Go, Zig, Odin, or Java) or needs exact tuning across FastNoise Lite noise types, fractals, cellular metrics, and domain warping.

- **Single-file portability**: C#, C++98, C99, GLSL, HLSL, Go, Java, Odin, Zig, Haxe, Pascal, GML, Fortran, PowerShell.
- **Packages**: npm `fastnoise-lite` (JS/TS), crates.io `fastnoise-lite` (Rust, supports `no_std` via `libm` and `f64` feature flag).
- **Output range**: `GetNoise` / `get_noise_*` / `fnlGetNoise*` returns values bounded roughly in `-1.0 .. 1.0`. Remap to `0..1` via `(n + 1.0) * 0.5`.

---

## Core Parameters & Exact Defaults

| Parameter | Default | Notes |
|---|---|---|
| `Seed` | `1337` | Integer seed for all noise & domain warp calculations |
| `Frequency` | `0.01` | Base spatial frequency (`1.0 / wavelength_in_units`) |
| `NoiseType` | `OpenSimplex2` | `OpenSimplex2`, `OpenSimplex2S`, `Cellular`, `Perlin`, `ValueCubic`, `Value` |
| `RotationType3D` | `None` | `None`, `ImproveXYPlanes`, `ImproveXZPlanes` |
| `FractalType` | `None` | `None`, `FBm`, `Ridged`, `PingPong`, `DomainWarpProgressive`, `DomainWarpIndependent` |
| `FractalOctaves` | `3` | Number of layered octaves when fractal is enabled |
| `FractalLacunarity` | `2.0` | Frequency multiplier per octave |
| `FractalGain` | `0.5` | Amplitude multiplier per octave |
| `FractalWeightedStrength` | `0.0` | Weight attenuation by previous octave (`0..1` preserves `-1..1` bounds) |
| `FractalPingPongStrength` | `2.0` | Strength parameter when `FractalType` is `PingPong` |
| `CellularDistanceFunction` | `EuclideanSq` | `Euclidean`, `EuclideanSq`, `Manhattan`, `Hybrid` |
| `CellularReturnType` | `Distance` | `CellValue`, `Distance`, `Distance2`, `Distance2Add`, `Distance2Sub`, `Distance2Mul`, `Distance2Div` |
| `CellularJitter` | `1.0` | Max offset of feature point inside cell (`> 1.0` causes visible grid artifacts) |
| `DomainWarpType` | `OpenSimplex2` | `OpenSimplex2`, `OpenSimplex2Reduced`, `BasicGrid` |
| `DomainWarpAmp` | `1.0` | Maximum warp displacement distance in input coordinate space |

---

## Algorithm Selection Guide

### Noise Types
- **`OpenSimplex2` / `OpenSimplex2S`**: Best general-purpose terrain/climate noise; isotropic gradient behavior and smooth transitions without classic Perlin axis-aligned grid artifacts. `OpenSimplex2S` trades ~slightly more work for smoother 2nd-order derivatives.
- **`Perlin`**: Classic gradient noise; cheap and familiar, good for 2D/3D textures and ridges.
- **`Cellular`**: Voronoi / Worley distance fields. Combine with:
  - `CellularReturnType.CellValue` for flat constant-value Voronoi regions (great for discrete biome/tectonic plates).
  - `CellularReturnType.Distance2Sub` (`F2 - F1`) for sharp cell borders, crack maps, or canyon ridges.
  - `CellularDistanceFunction.EuclideanSq` (fastest) or `Euclidean` (true metric) or `Manhattan` / `Hybrid` (blocky/crystalline patterns).
  - Keep `CellularJitter <= 1.0` to avoid neighbor-search cutoff artifacts.
- **`Value` / `ValueCubic`**: Fast lattice interpolation (`ValueCubic` gives smooth bicubic/tricubic ramps).

### Fractal Types
- **`FBm`**: Standard terrain height, temperature, moisture, clouds.
- **`Ridged`**: `1 - |noise|` inverted peaks; use for mountain spines, rivers, and cave webs.
- **`PingPong`**: Terraced / marbled waves controlled by `FractalPingPongStrength`.
- **Important**: `DomainWarpProgressive` and `DomainWarpIndependent` **only** affect `DomainWarp(...)` calls, not `GetNoise(...)`.

### 3D Slicing Pitfall (`RotationType3D`)
When sampling a 2D plane inside 3D noise (e.g., animated `z = time`, or `y`-slices in voxel terrain), set `RotationType3D` to `ImproveXYPlanes` (for Z-slices) or `ImproveXZPlanes` (for Y-slices) to prevent diagonal/grid directional artifacts along the slice plane.

---

## Multi-Language Usage Snippets

### C# (Unity / Godot C# / .NET)
```csharp
var noise = new FastNoiseLite(seed);
noise.SetNoiseType(FastNoiseLite.NoiseType.OpenSimplex2);
noise.SetFractalType(FastNoiseLite.FractalType.FBm);
noise.SetFractalOctaves(4);
noise.SetFrequency(0.01f);

// Optional domain warp before GetNoise:
var warp = new FastNoiseLite(seed + 101);
warp.SetDomainWarpType(FastNoiseLite.DomainWarpType.OpenSimplex2);
warp.SetDomainWarpAmp(25.0f);

float x = worldX, y = worldZ;
warp.DomainWarp(ref x, ref y);
float n = noise.GetNoise(x, y); // -1..1
```

### C++98
```cpp
FastNoiseLite noise;
noise.SetSeed(seed);
noise.SetNoiseType(FastNoiseLite::NoiseType_OpenSimplex2);
noise.SetFractalType(FastNoiseLite::FractalType_FBm);
float value = noise.GetNoise((float)x, (float)y);
```

### GLSL / HLSL (GPU Terrain & Shader Noise)
```glsl
fnl_state noise = fnlCreateState();
noise.seed = 1337;
noise.noise_type = FNL_NOISE_OPENSIMPLEX2;
noise.fractal_type = FNL_FRACTAL_FBM;
noise.octaves = 4;
float n = fnlGetNoise2D(noise, worldPos.x, worldPos.y);
```

### Rust (`fastnoise-lite` crate)
```rust
use fastnoise_lite::*;

let mut noise = FastNoiseLite::with_seed(seed);
noise.set_noise_type(Some(NoiseType::OpenSimplex2));
noise.set_fractal_type(Some(FractalType::FBm));

// Optional domain warp:
// let (wx, wy) = noise.domain_warp_2d(x, y);
let n = noise.get_noise_2d(x, y);
let zero_to_one = (n + 1.0) * 0.5;
```

### TypeScript / JavaScript (`npm install fastnoise-lite`)
```typescript
import FastNoiseLite from "fastnoise-lite";

const noise = new FastNoiseLite(seed);
noise.SetNoiseType(FastNoiseLite.NoiseType.OpenSimplex2);
noise.SetDomainWarpType(FastNoiseLite.DomainWarpType.OpenSimplex2);
noise.SetDomainWarpAmp(1.5);

// Accepts any { x, y, z? } object (e.g. THREE.Vector2 / Vector3)
const coord = { x: worldX, y: worldY };
noise.DomainWarp(coord);
const n = noise.GetNoise(coord.x, coord.y);
```

---

## Performance & Precision Guidelines

- **Keep instances immutable per chunk pass**: Configure `FastNoiseLite` state once (at startup or seed change), never allocate per pixel/tile.
- **Double precision for huge worlds**: In C# (`using FNLfloat = System.Double;`), Rust (`features = ["f64"]`), C99 (`typedef double FNLfloat;` with `#define FNL_IMPL` in one source file), or C++ (pass `double` coordinates directly to templated `GetNoise(x, y)`), use double input precision when coordinates exceed ~1e6 units from origin.
- **Keep Warp & Noise separate**: Configure a dedicated `FastNoiseLite` instance for `DomainWarp` (`Frequency` + `DomainWarpAmp`) and another for `GetNoise`, rather than sharing one frequency across both passes.
