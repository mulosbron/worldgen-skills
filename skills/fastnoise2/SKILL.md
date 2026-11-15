---
name: fastnoise2
description: Modular node-based SIMD noise generation library in modern C++17 (`Auburn/FastNoise2` v1.1.1) with FastSIMD runtime dispatch, visual Node Editor serialized node trees, and bindings for C#, Unreal Engine, Rust, and Java. Use when building high-throughput 2D/3D/4D procedural terrain, biomes, domain warping, or fractal noise graphs.
version: "1.1.1"
license: MIT
metadata:
  homepage: "https://github.com/Auburn/FastNoise2"
  repository: "https://github.com/Auburn/FastNoise2"
---

# FastNoise2 (`Auburn/FastNoise2`)

Use this skill when designing or integrating high-performance SIMD procedural noise graphs in C++17 (or C#, Unreal, Rust, Java bindings) using **FastNoise2** (`v1.1.1`, MIT).

## FastNoise2 vs FastNoise Lite

- Use **`fastnoiselite`** for lightweight, single-file cross-language noise needs with minimal build complexity.
- Use **FastNoise2** when you need fused SIMD node-graph evaluation (no intermediate heap allocations between noise passes), visual Node Editor authoring, or runtime serialized node trees (`FastNoise::NewFromEncodedNodeTree`).

## Core Capabilities

- **Coherent Noise**: Perlin, Simplex, SuperSimplex (`OpenSimplex2S`), Value, Cellular Value, Cellular Distance, Cellular Lookup
- **Fractals & Domain Warping**: `FractalFBm`, `FractalRidged`, Gradient/Simplex/SuperSimplex domain warp, Fractal Progressive / Independent
- **Operators & Modifiers**: Add, Subtract, Multiply, Divide, (Smooth)Min/(Smooth)Max, Fade, Remap, Terrace, Domain Scale/Offset/Rotate
- **Dimensions & Thread Safety**: 2D, 3D, 4D noise generation plus 2D tiling; thread-safe read-only node tree evaluation across worker threads
- **FastSIMD Backends**: Runtime dispatch across Scalar, SSE2, SSE4.1, AVX2, AVX512, NEON, and WASM SIMD

## C++17 Usage Patterns

### Build-from-Code Node Graph
```cpp
#include "FastNoise/FastNoise.h"

auto simplex = FastNoise::New<FastNoise::Simplex>();
auto fractal = FastNoise::New<FastNoise::FractalFBm>();
fractal->SetSource(simplex);

std::vector<float> noise(256 * 256);
fractal->GenUniformGrid2D(noise.data(), 0, 0, 256, 256, 1, 1, 1337);
```

### Deserializing a Node Editor Tree
Author graphs visually in the desktop or WASM Node Editor (`https://auburn.github.io/FastNoise2/`) and copy the encoded string:
```cpp
#include "FastNoise/FastNoise.h"

FastNoise::SmartNode<> generator = FastNoise::NewFromEncodedNodeTree("DQkGDA==");
if (generator)
{
    std::vector<float> noise(256 * 256);
    generator->GenUniformGrid2D(noise.data(), 0, 0, 256, 256, 1, 1, 1337);
}
```

## Build & Compiler Notes

- CMake project `FastNoise2` (`CMAKE_CXX_STANDARD 17`); options: `FASTNOISE2_TOOLS`, `FASTNOISE2_TESTS`, `FASTNOISE2_UTILITY`, `FASTNOISE2_STRICT_FP`.
- **Windows**: Prefer **ClangCL** over MSVC due to known MSVC SIMD compiler codegen bugs.
- **Official Community Bindings**: C# (`FastNoise2Bindings`), Unreal Engine CMake (`UE4_FastNoise2`), Unreal Blueprint (`UnrealFastNoise2`), Rust (`fastnoise2-rs`), Java (`FastNoise2Bindings-Java`).
