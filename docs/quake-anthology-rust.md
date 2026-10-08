# Quake Anthology Rust

Quake Anthology Rust is a native Rust engine meant to run Quake, QuakeWorld, Quake II and Quake III Arena (with Team Arena), their rereleases and mission packs, and mixed "custom game" setups (for example a Quake 1 map with Quake III movement and Quake II monsters) as one program built on shared engine primitives. The repository has two distinct eras. From 2026-09-29 to 2026-10-06 the TypeScript engine (Quake-Anthology-TS / quake-typescript) was ported to Rust directly. On 2026-10-07 `main` was restarted from an empty workspace and now takes behaviour from id Software's original sources, the feature list from quake-typescript, and proven algorithms and fixes from the C port. As of the latest commit the restarted engine is at milestone R0 (foundations): it opens an SDL window, measures frame times and has tested PAK/BSP/MDL header readers. Gameplay is not implemented yet.

## Links

| | |
|---|---|
| GitHub | https://github.com/mgd34msu/Quake-Anthology-Rust |
| Binary | `qa-rust` (`target/release/qa-rust`; qualified installs go to `qfiles/qa-rust`) |
| Retired tree | commit `eb9118cc` (2026-10-06) |
| Planning | Linear project "Quake-Rust" (P-THE-3), milestones R0-R13 (private workspace) |

## Status

**Early foundation, not playable.**

- Restarted `main` (2026-10-07): the R0 milestone. It has a Cargo workspace, the shared primitive types, an SDL window shell, format header readers, build, run, install and timing tools, a rule checker and CI. The README says plainly: "Gameplay is not implemented."
- Retired first tree (`eb9118cc`): reference only. It builds and reaches a rendered world in all three games, and some Quake 1 rules run. Most of it is unreachable or not visible in play (details below).

When this was written, local `main` was at `07162f30` and GitHub `origin/main` at `b1a97f25`, so six R0 commits were not yet pushed.

## Timeline

All dates and counts are from `git log` of the repository.

| | |
|---|---|
| First commit | 2026-09-29 (`e70cee07`, "pre-parallel baseline"). Earlier Rust history is not in this repository. |
| Last commit | 2026-10-07 (`07162f30`) |
| Commits on `main` | 1,007, all authored under Mike's git identity |
| First tree | 991 commits, 2026-09-29 to 2026-10-06 |
| Restart era | 16 commits, 2026-10-07 15:30-15:51 |

Commits per day in the first tree: 183 (09-29), 217 (09-30), 107 (10-01), 79 (10-02), 113 (10-03), 28 (10-04), 172 (10-05), 92 (10-06).

### Phases

1. **Direct TypeScript port (2026-09-29 to 2026-10-06).** The first README described the project as "the Rust port of Quake-Anthology-TS". Work ran in parallel `lane/*` branches merged into `main`. Ids such as `Q1-0258` referred to a 6,618-row feature checklist, and many commits were "WIP timer commit" checkpoints. The last days were spent on windowed Quake 1 episode runs (E2, E3 and E4 episode-run merges on 2026-10-06).
2. **Retirement (2026-10-07).** The first tree, ending at `eb9118cc`, was retired in favour of a fresh start.
3. **Restart (2026-10-07).** In order:
   - `c592b337` "THE-596 Restart main around shared Rust primitives" deleted 1,663,818 lines in 2,174 files.
   - `c2095019` added the new workspace and the SDL window candidate.
   - The remaining 14 R0 commits added the project rules document (owner rules, working protocol, C-port lessons), the private display run harness, the qualified installer, the pinned frame-timing harness, format readers checked against malformed and retail data, the GPL licence with id attribution, the pre-build rule checker and the test-share limit.

## Stack and size

- **Language:** Rust, edition 2024, Cargo workspace resolver 3. The licence field is `GPL-2.0-or-later`.
- **Libraries:** SDL2 for windowing (CI installs `libsdl2-dev`). The workspace crates have no other external dependencies.
- **Release profile:** `opt-level = 3`, `lto = "fat"`, `codegen-units = 1`, `panic = "abort"`, with full debug info. `tools/build.py` splits the debug info into `qa-rust.debug`.
- **Tooling:** Python 3 scripts in `tools/`.

Size of the restarted tree on 2026-10-07 at about 15:50, counted with `wc -l` (tokei and cloc are not installed). The tree is growing quickly, so these numbers are a snapshot:

| Part | Files | Lines |
|---|---:|---:|
| Rust (`crates/**/*.rs`) | 23 | 717 |
| Python tools (`tools/*.py`) | 6 | 644 |
| Notes (`docs/*.md`) | 3 | about 90 |
| Project rules document | 1 | about 150 |

The largest Rust files are:

- `crates/platform/src/sdl.rs`: 157 lines
- `crates/app/src/main.rs`: 136 lines
- `crates/formats/src/headers.rs`: 110 lines
- `crates/core/src/primitives.rs`: 92 lines

Most capability crates are a one-line placeholder `lib.rs`.

The retired first tree, for comparison: 2,122 `.rs` files and 1,656,411 lines of Rust (counted with `git show` + `wc -l`). A 2026-10-07 review measured about 160k of those lines as unreachable from `main()`.

## Architecture (restarted tree)

The project rules define the layout:

- **`crates/core`** holds the engine's own values in `primitives.rs`: `Vec3`, `EntityId` (slot plus generation), `Body`, `Entity`, `PlayerState`, `UserCmd`, item and weapon ids, damage, sound and effect events, HUD state and `CvarHandle`. Core has no dependencies.
- **Capability crates** depend only on core: `world`, `movement`, `formats`, `content` (VFS), `render`, `audio`, `network`, `session`, `gameplay`, `compat` (module hosts for progs.dat, game .so/.dll and QVM), `navigation`, `bots`, `persistence`, `console`, `input`, `ui` and `platform`. There are no per-game console, cvar or HUD crates. Each game converts its data into the primitives at its file, network and module boundaries. Game-specific rules live inside gameplay modules.
- **`crates/app`** puts the capability crates together into the `qa-rust` binary. `main.rs` parses `--frames`, `--warmup`, `--frame-timings`, `--uncapped` and window size options, then runs the SDL frame loop. `proof.rs` adds scripted SDL input, but only when built with the `proof` feature. That build is a separate development candidate and is never shipped.
- **`crates/formats`** reads:
  - PAK directories: names are indexed once at load, member bytes stay borrowed, and the first copy of a duplicate name wins.
  - BSP29, IBSP38 and IBSP46 headers through one shared reader.
  - MDL header metadata.

  `tests/behaviour.rs` checks malformed bounds and record widths. An optional test against a retail `pak0.pak` (`QA_RETAIL_PAK=...`) matches e1m1's 1,810 planes, 7,358 vertices, 5,516 faces and 58 models, and player.mdl's 212 vertices, 408 triangles and 143 frames.
- **`crates/platform`** (`sdl.rs`) is the SDL window and event layer.

The tools in `tools/` are:

| Script | What it does |
|---|---|
| `build.py` | Release build, debug-symbol split, installation candidate; `--cpu native` for a machine-specific build |
| `check_rules.py`, `verify_rules.py`, `rules-allowlist.json` | Reject owner-rule regressions before every build (details after this table) |
| `private_run.py` | Runs a candidate on its own Xvfb plus Openbox display with a copied owner profile; X11 forced, Wayland unset, audio to the dummy or disk driver; stops only the process ids it started |
| `install_qualified_build.py` | Replaces `qfiles/qa-rust` only after a private run with a fresh profile copy reaches gameplay and quits normally |
| `frame_timings.py` | Pinned-core timing: 600 frames after warm-up, median and p99 per stage |

`check_rules.py` flags these patterns:

- `Rc<RefCell>` pools
- `NumericOps`, `fround` and `SaveJson` emulation
- SHA-256 and `DefaultHasher` fingerprints
- the names "donor", "mirror", "seam" and "shim"
- `panic!` and `unwrap` outside allow-listed load boundaries
- `HashMap<String, ...>` state
- `match`/`if` on `BspKind` or `GameFamily` outside per-game folders
- redefinitions of the primitive types
- `TEMP` diagnostics
- production files whose content is more than 40% tests

CI (`.github/workflows/check.yml`) runs on every push and pull request: the rule check, `cargo fmt --check`, `cargo clippy -D warnings` and `cargo test --workspace`.

## Features (what works now)

- `cargo build --release` produces `qa-rust`. It opens an SDL window, runs a bounded number of frames and exits.
- The private run harness (`docs/r0-evidence.md`):
  - runs a byte-identical candidate copy on its own Xvfb display;
  - copies all 34 of the owner's saved settings files into a private HOME and leaves the originals byte-identical;
  - records 14 real X server key auto-repeat events from a held key;
  - captures the window and confirms that no owned process is left running.
- Frame timing of the empty window shell (`docs/frame-times.md`, build `b1a97f25`, Xvfb, 600 frames). Median / p99 per frame:
  - 1920x1080: 13.40 / 14.91 ms
  - 640x400: 1.75 / 2.07 ms
  - 320x200: 0.52 / 0.61 ms

  The doc states these numbers only verify the timing tool and are not renderer results.
- PAK, BSP and MDL header readers with behaviour tests, as described above.

Not implemented yet: maps in play, rendering of game content, sound output (the shell opens no audio device), console, cvars, movement, gameplay, saves, mods, networking and bots.

## The plan: milestones R0-R13


| Milestone | Scope | Issues |
|---|---|---:|
| R0 Restart and foundations | Fresh tree, project rules, workspace and release profile, qualified installer, private run harness, timing harness, rule checker | 6 |
| R1 Primitives, VFS, formats | Structure-of-arrays entity tables, arenas, interned names, one trace interface over every BSP format; one VFS for .pak, .pk3 and directories; BSP 29/2/30/38/46 and model and image readers | 24 |
| R2 Console, cvars, input, frame loop | One console and one cvar table generated from the unified cvar list (Q3 names plus every alias, e.g. `fov` = `cg_fov`); no slash needed; chat only via `say`; frame cap honoured | 15 |
| R3 Renderers | GL with static VBOs, batching and a lightmap atlas; CPU renderer with per-surface span kernels, surface cache and worker pool | 21 |
| R4 Quake 1 e1m1 vertical slice | e1m1 playable end to end in the shipped binary, plus one mixed configuration | 24 |
| R5 Quake 1 complete | All episodes, rerelease content, Scourge of Armagon, Dissolution of Eternity, Dimension of the Past, MG1/MG3, QuakeWorld | 26 |
| R6 Quake II | Classic and rerelease campaigns, original save container, Xatrix, Rogue, CTF | 24 |
| R7 Quake III and Team Arena | q3dm1 with bots, shaders, menus, Team Arena | 13 |
| R8 Mods together | One module host: QuakeC VM, Q2 game .so/.dll on any OS, Q3 QVM; several mods loaded at once | 11 |
| R9 Combined mode | Map, monsters, weapons, movement, character and rules each chosen from any game; compact custom save format | 12 |
| R10 Audio, music, menus, UI | One mixer, music mapping, one HUD system, split-screen seats | 10 |
| R11 Network | NetQuake 15/666/999, QuakeWorld 28, Q2 34/35/36/2023 and Q3 68 protocols; demos | 12 |
| R12 Performance | GL 1080p under 2 ms, CPU 640x400 under 4 ms and 320x200 under 2 ms per frame, measured pinned (median and p99) | 10 |
| R13 Release | Strip input recording and replay; no SHA or fingerprint leftovers; licence, README, packaging | 8 |

## How to build and run

From the repo README, project rules and `docs/`:

```sh
cargo build --release                 # target/release/qa-rust (baseline CPU)
python3 tools/build.py                # candidate build with split debug info
cargo test --workspace                # format behaviour checks
QA_RETAIL_PAK=/path/to/pak0.pak cargo test -p qa-formats --test behaviour -- --include-ignored
python3 tools/private_run.py --binary target/candidate/qa-rust \
  --owner-profile "$PROFILE" --evidence "$EVIDENCE" -- --frames 180
python3 tools/frame_timings.py --binary target/candidate/qa-rust \
  --owner-profile "$PROFILE" --evidence "$EVIDENCE"
```

`RUSTFLAGS="-C target-cpu=native"` (or `build.py --cpu native`) makes a machine-specific build for timing. Record that choice with the measurements. Building needs a Rust toolchain that supports edition 2024 and the SDL2 development package. Game data is not included. Later milestones will need your own retail Quake files.

## Relationship to the other projects

- **qsrc** (`~/Projects/qsrc`, id Software's released sources plus the rerelease sources) is the authority for behaviour. Commits cite file and line references, for example `WinQuake/bspfile.h:59-95`.
- **quake-typescript / Quake-Anthology-TS** supplies the feature list only. The first tree had been a port of it, and the project rules now ban its structures in Rust: `Rc<RefCell>` entity pools, string-keyed ids, numeric emulation, JSON saves and async pumps.
- **The C port** (GitHub `Quake-Anthology`, `~/Projects/quake-anthology`) supplies proven algorithms, fixes and tooling. The installer, the cvar generator and the single-engine checker are being ported from it. Its per-game duplicated structure is not copied.
- **qfiles** (`~/Projects/qfiles`) holds the game data and the install target `qfiles/qa-rust`.

## Credits and licence

- **Original engine and game code:** id Software, Inc. (Quake, Quake II, Quake III Arena), released under the GNU GPL. The root `LICENSE` keeps the line "Original Quake game code: Copyright (C) 1996-2005 id Software, Inc." and distributes the project under GPL version 2 or later.
- **Feature reference:** Mike's quake-typescript project. **Implementation reference:** his C port.
- **Game data:** no game assets are in the repository. You need your own copies of the games.
