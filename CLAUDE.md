# CLAUDE.md

OpenStarbound: a C++17 fork of the Starbound 1.4.4 source (client, server, tools) with Lua scripting and JSON assets. Windows is the primary dev platform here. This is a personal working fork; upstream is OpenStarbound/OpenStarbound.

Planned changes live in [NOTES.md](NOTES.md). Check it before starting work and update it as items progress.

## Build (Windows)

Use an **x64** MSVC environment: open "x64 Native Tools Command Prompt for VS 2022" and type `powershell`. The plain Developer PowerShell can start as x86 and the link step then fails with `undefined symbol: mainCRTStartup`.

```powershell
cd <repo>\source
cmake --preset=windows-release-local     # configure; only needed when CMakeLists.txt or the source file list changes
cmake --build --preset=windows-release-local
```

- `windows-release-local` is defined in `source/CMakeUserPresets.json` (gitignored). It inherits `windows-release` and pins vcpkg to `C:/src/vcpkg` and the compilers to `C:/Program Files/LLVM/bin/clang(.exe|++.exe)`. If it is missing, recreate it; the stock preset needs `VCPKG_ROOT` set and `clang` on PATH.
- Compiler is **clang-cl via standalone LLVM** (winget `LLVM.LLVM`), with the MSVC headers/libs from VS 2022. Ninja generator, vcpkg triplet `x64-windows-mixed-md-clang` (in `triplets/`).
- Output goes to `dist/` (gitignored): `starbound.exe`, `starbound_server.exe`, tools, and test binaries. Build tree is `build/windows-release/`.
- Builds are incremental. Don't delete `build/` unless the CMake cache is corrupt (a stale cache after a toolchain change is the usual reason).
- `dist/` needs `sbinit.config` (copy from `scripts/windows/sbinit.config`, and point the mods entry at `..\mods\`) and `assets/packed.pak` from a Starbound install (gitignored). `assets/opensb/` holds OpenStarbound's own assets.
- Mods for play-testing go in `mods/` at the repo root (gitignored), so wiping `dist/` never loses them. `dist/` itself is gitignored.
- `reference/` (gitignored) holds material for reading only: `reference/vanilla/` (vanilla assets unpacked from `packed.pak` with `dist/asset_unpacker.exe`) and `reference/FrackinUniverse/`. It is deliberately outside every asset directory, so the game never loads it. Use it to see how things are done and to check compatibility.
- First configure is slow (vcpkg builds dependencies); after that it is cached.

## Tests

`core_tests.exe` and `game_tests.exe` are built into `dist/` (GoogleTest). Run them from `dist/`. There is no CI test step to mirror; `.github/workflows/build.yml` only builds.

## Layout (`source/`)

| Dir | Contents |
|---|---|
| `core/` | Containers, strings, JSON, IO, threading, networking primitives, Lua engine wrapper |
| `base/` | Assets, config, root, versioning |
| `game/` | Gameplay: world, entities, universe client/server, system world (celestial/ship travel), net packets, Lua bindings (`game/scripting/`) |
| `windowing/` | Widget and pane toolkit (`StarPane`, `StarPaneManager`, widgets) |
| `frontend/` | Concrete GUI panes and client-side UI (inventory, chat, options menus, etc.) |
| `rendering/`, `application/`, `platform/` | Renderer, SDL window/app loop, Steam/Discord integration |
| `client/`, `server/` | `main` for each executable |
| `test/` | Unit tests |
| `extern/` | Vendored third-party code |

Game data: `assets/opensb/` (OpenStarbound's additions and `.patch` files over vanilla assets), `assets/packed.pak` (vanilla, not in git; unpacked copy in `reference/vanilla/`). UI layouts are JSON under `assets/opensb/interface/`. Lua docs are in `doc/lua/`.

## Conventions

- Source files are `Star*.hpp/.cpp`; types and functions are in `namespace Star`, with `m_` prefix for members and `...Ptr` shared-pointer typedefs via `STAR_CLASS(...)`. Match the surrounding style.
- Formatting: `.clang-format` at the repo root (`scripts/format-source.sh` runs it). Format only code you touch; do not reformat whole files.
- Multiplayer is authoritative-server: changes touching game state usually need a net packet (`StarNetPackets`) or a netstate change on both `UniverseClient` and `UniverseServer`. Check whether a change is client-only (UI, settings) or needs the server before designing it.
- Existing compiler warnings (unused `remember` param, non-virtual destructor on `UniverseClient`) are known upstream issues. Ignore them unless asked.
- Keep game-data changes (JSON, Lua, patches) in `assets/opensb/` where possible rather than C++, for moddability and faster iteration (assets load from the directory, no rebuild needed).

## Working notes

- `dist/`, `build/`, `source/CMakeUserPresets.json`, and `.claude/settings.local.json` are local and gitignored.
- Do not commit or push unless asked. Don't touch `attic/` (legacy CI scripts).
