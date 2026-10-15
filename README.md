<div align="center">

# DasVis

**A music visualizer for Windows that follows the song: build-ups, drops, cover colours.**
Direct3D 11, runs on old PCs. From [WreckBox](https://github.com/moloyb301-eng/wreckbox-releases/tree/windows-native).

![Platform: Windows](https://img.shields.io/badge/platform-Windows%2010%20%7C%2011-0078D4)
![C++20](https://img.shields.io/badge/C%2B%2B-20-00599C)
![Direct3D 11](https://img.shields.io/badge/Direct3D-11%20(feature%20level%2010__0)-5C2D91)
![CMake 3.21+](https://img.shields.io/badge/CMake-3.21%2B-064F8C)

[What it does](#what-it-does) ·
[Build](#build) ·
[Use](#use-as-a-library) ·
[Layout](#layout)

</div>

---

DasVis is the visualizer in WreckBox, as a library you can build and test on its own.

It analyses each song once and knows its beats, bars, build-ups and drops. Build-ups drain the colour and close in, the
bar before a drop goes quiet, and the drop hits with a hero scene. The kick, snare, hats, bass and lead each move their
own part of the picture, and the colours come from the album cover.

> [!NOTE]
> It's made to run on old PCs as well as new ones: Direct3D 11 at feature level 10_0 (graphics cards from about 2008),
> pixel shaders only, a quality tier measured on each PC, and about +20 MB of memory at the lowest tier.

## What it does

- **13 scenes:** Feedback Classic, Neon Tunnel, Kaleido Cover, Ink Ocean (a fluid simulation), Ridge Lines, Coral
  (reaction-diffusion), Galaxy, Synthwave, Hyperspace, Aurora, Mandelbox Flight, Liquid Chrome and Storm. A 14th,
  *MilkDrop*, shows a projectM preset that the app hands in.
- **A director** picks a scene, its layers and its transition for each part of the song. It uses only scenes the PC can
  run smoothly, learns your likes and skips per genre, and never repeats one of the last 4 scenes.
- **The song map** (`songmap.*`): beats, bars, parts, drops and per-instrument hits, worked out once per song and
  cached.
- **Effects:** HDR with bloom and tone mapping, a spectrum ring around the turning cover, beat sigils, a title card,
  synced lyrics in kinetic type, and drop events (shockwave, white-out, cover shatter).
- **A flash limiter** holds big flashes to 3 a second (1 with *reduce flashing*).
- **Extras:**
  - video export: H.264 + AAC MP4, up to 4K60
  - Spout output
  - WLED LED strip colours (`led_frame`)
  - a phone deck's camera, drop and shockwave controls
- **Your own scenes:** a `.hlsl` file with one `scene()` function, compiled at run time. `library_source()` gives you
  `dasvis.hlsli`, which lists everything a scene can use.

## Requirements

| Need | Notes |
|---|---|
| Windows 10 or 11 | x64 |
| Visual Studio 2022 | C++ desktop workload (MSVC, C++20) |
| CMake 3.21 or newer | |
| [vcpkg](https://github.com/microsoft/vcpkg) | installs `nlohmann-json` and `drlibs` from `vcpkg.json` |
| Windows SDK | its `fxc` compiles the shaders into the library at build time |

No graphics card is needed to run the tests: they use WARP (Direct3D on the CPU).

## Build

```powershell
cmake -S . -B build -G "Visual Studio 17 2022" -A x64 -DCMAKE_TOOLCHAIN_FILE=<vcpkg>/scripts/buildsystems/vcpkg.cmake
cmake --build build --config Release
```

Replace `<vcpkg>` with the folder you cloned vcpkg into.

## Test

```powershell
ctest --test-dir build -C Release
```

The tests (`tests/dasvis_tests.cpp`) cover every part, and the GPU parts run on WARP, so they need no graphics card.

## Use as a library

```cpp
#include "dasvis/dasvis.h"

wb::dasvis::Config cfg;                  // tier, auto quality, flash limits, your scenes folder…
wb::dasvis::DasVis vis(device, cfg);     // an ID3D11Device* (create it with D3D11_CREATE_DEVICE_BGRA_SUPPORT)
wb::dasvis::Features features;

// Each frame: the newest 2,048 mono samples heard, then render into a B8G8R8A8 render-target texture (a back buffer).
features.update(samples, dt, known_bpm);
wb::dasvis::Inputs in;
in.f = &features, in.dt = dt, in.song = song_id, in.song_time = seconds;
in.map = song_map ? &*song_map : nullptr; // std::optional<SongMap> from analyse_file(path, bpm), on a worker thread
vis.render(in, back_buffer);
```

`dasvis.h` documents the rest: covers, the title card, lyrics, scene control, tiers and costs. For a real app's use of
it (the window, the live wallpaper, other screens, the phone deck), see `src/ui/view_visualizer.cpp` in WreckBox.

## Layout

| Path | What |
|---|---|
| `src/dasvis/` | the library: audio features, song map, director, palette, renderer, video export, Spout, lyrics |
| `shaders/dasvis/` | `dasvis.hlsli` (the scene library), the scenes, simulations and post passes |
| `src/engine/` | audio decoding (dr_libs, Media Foundation) and the FFT, from WreckBox's engine |
| `tests/dasvis_tests.cpp` | every part tested, the GPU parts on WARP |
| `docs/DASVIS-PLAN.md` | the design, phase by phase, with measurements |
| `cmake/dasvis.cmake` | compiles the shaders with `fxc` (Shader Model 4) into headers at build time |

## Credits

DasVis comes from [WreckBox](https://github.com/moloyb301-eng/wreckbox-releases/tree/windows-native). Audio decoding
uses [dr_libs](https://github.com/mackron/dr_libs) and Windows Media Foundation; JSON uses
[nlohmann/json](https://github.com/nlohmann/json).
