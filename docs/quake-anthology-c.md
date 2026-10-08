# Quake Anthology (C port)

Quake Anthology is a native C17 engine that runs Quake, QuakeWorld, Quake II and Quake III Arena, their mission packs, and the 2021 Quake and 2023 Quake II rereleases from one executable. It can also combine them: a Quake map played with Quake III movement, a Quake II character, Quake III weapons and Quake II rerelease monsters is a supported launch. The C code reimplements the feature set of the TypeScript project [Quake-Anthology-TS](https://github.com/mgd34msu/Quake-Anthology-TS). Where gameplay behaviour is in question, the original id Software sources decide what is correct. Mike playtested it and set the rules. Between 26 September and 7 October 2026 the repository reached 2,119 commits and about 890,000 lines of C. The engine builds and runs all four game families on both renderers. It is not finished: the README says it is "not ready to play yet", and owner-reported defects are still open.

## Links

| | |
| --- | --- |
| GitHub | https://github.com/mgd34msu/Quake-Anthology |
| Local clone | `~/Projects/quake-anthology` |
| Installed test build | `~/Projects/qfiles/qa-c` (single executable, about 142 MB) |
| Original id sources used as reference | `~/Projects/qsrc` (not part of this repo) |
| Issue tracking | Linear project "Quake-Anthology" (private) |

## Status

**A working engine with open defects.** All four game families reach gameplay on the CPU and GL renderers. Original saves load and save. Split-screen, multiplayer protocols, bots and the menus are implemented. Combined game launches also work. The repository README still calls it not ready to play.

Mike reported 37 defects in his playtests (MIKE-01 to MIKE-37). The installed-build sweep in `docs/playtests/2026-10-07-mike-sweep.md` (in progress on 7 October) records a mix of results:

- many issues pass bounded checks;
- some are partial, for example Q1 rerelease music output, some Q2 rerelease sounds and the Q1 status-bar backtile;
- two had failed outright at the time of the sweep: the Quake III menu "Play" path exiting before gameplay (MIKE-16), and repeated death/respawn checks, which were blocked by an add-bot crash after map travel (MIKE-35).

The CPU renderer has not met its frame-time targets, and the Q3 CPU path is the slowest. A public release is also waiting on the removal of the test-only input recording and replay code. That requirement is tracked as a release blocker.

## Timeline

Dates come from `git log` on `main`, which matches `origin/main`.

| | |
| --- | --- |
| First commit | 2026-09-26 16:13 CDT, "Document unified C engine scope, review, and implementation plan" |
| Last commit (at time of writing) | 2026-10-07 15:41 CDT |
| Commits | 2,119, all authored under Mike's Git identity |
| Busiest day | 2026-10-05, 424 commits |

Commits per day: Sep 26: 86, Sep 27: 64, Sep 29: 1, Sep 30: 284, Oct 1: 205, Oct 2: 207, Oct 3: 387, Oct 4: 131, Oct 5: 424, Oct 6: 244, Oct 7: 86.

Phases visible in the history:

1. **Design (26 Sep).** The first commits are documents only:
   - `docs/review.md` reviews the TypeScript engine. It measured the Q2 rerelease game code, run through CPU emulation in TypeScript, at a 48 ms median per tick against a 25 ms tick budget.
   - `docs/plan.md` is the C implementation plan.
   - `docs/dependency-graph.md` and `docs/dependencies.json` give the build order.

   The main design decision is that built-in gameplay is compiled C calling shared services directly. Loading a Q2 map must not start a CPU emulator.
2. **Baseline port (26 Sep to 30 Sep).** Subsystems were ported in numbered batches (B00 to B29 and later), each with acceptance JSON and evidence files under `docs/`. On 29 Sep a single commit, "Continue C baseline port and apply source review fixes", added 73,957 lines across 530 files.
3. **Feature completion (30 Sep to 4 Oct).** Native Q1/Q2/Q3 gameplay, mission-pack behaviour (Hipnotic, Rogue, Xatrix), menus, downloads, bots, demos and the shared save codec were completed in this phase.
4. **Owner playtests and unification (4 Oct onward).** Mike's playtest reports became MIKE-NN items. The work then moved to Linear issues (THE-NN): 41 distinct THE ids and 37 MIKE ids appear in commit subjects. Two other kinds of work also landed: save-format work (C-SAVE-NN, 10 ids) and frame-time work.

## Stack and size

- **Language:** C17. Python 3 is used only for build-time checks and generators.
- **Build:** CMake 3.20+. A single target, `quake-anthology`, is built from `src/main.c` plus the library sources.
- **Libraries:** SDL2 (2.0.18+), libcurl, zlib, libpng, libjpeg, giflib, ICU, FreeType, libffi, Vorbis, Ogg and Theora. The build downloads pinned native runtime dependencies on first configure.
- **Platform:** Linux x64. Windows-specific branches exist in places such as `src/compat/native/direct.c`.
- **Tracked files:** 3,790.

Line counts are raw `wc -l` over tracked files (blank lines and comments included). No `tokei` or `cloc` was available.

| Kind | Files | Lines |
| --- | ---: | ---: |
| C source (`.c`) | 1,730 | 798,355 |
| C headers (`.h`) | 1,323 | 90,571 |
| Python (`tools/`) | 9 | 3,011 |
| CMake | 6 | 653 |
| JSON (data, evidence) | 68 | 18,001 |
| Markdown (docs, playtests) | 243 | 35,753 |

`src/console/cvar_catalog_generated.c` (9,565 lines) is generated by `tools/generate_unified_cvars.py`. Its line count is included above.

## Architecture

`src/` has 26 top-level modules. The sizes below are raw lines of `.c`/`.h`.

| Module | Lines | What it does |
| --- | ---: | --- |
| `app/` | 305,654 | Application composition and frontend. `app/application/` (174k) resolves a launch into one session: providers, guest projections and bot projection. `app/frontend/` (129k) holds menus, save commands, seats and settings screens. `app/composition/` wires selections together. |
| `gameplay/` | 183,746 | Compiled gameplay. `q1/` (57k), `q2/` (87k) and `q3/` (21k) hold the per-game rules: monsters, items, weapons, triggers and mission-pack entities. `modes/` (13k) holds match rules. `builtin/` holds shared declarations. |
| `compat/` | 62,676 | Running original external game code. `qc/` and `qc_host/` run QuakeC `progs.dat`. `qvm/` runs Quake III QVM bytecode. `q3_host/` provides the Q3 game/cgame host interface. `native/` (38k) and `native_host/` load original native game modules: `native/guest/` contains an ELF loader, ABI and CPU code, plus checkpointing and instruction-region observation. |
| `network/` | 41,698 | Wire protocols for Q1 (NetQuake 15, FitzQuake 666, RMQ 999, QW 28/29), Q2 (34, R1Q2 35, Q2PRO 36, Q2REPRO 1038, KEX 2023), Q3 (68) and the project's own `unified-1`. Also the server browser, downloads, admin and runtime. |
| `console/` | 41,574 | The single console and cvar store, command buffer, dedicated-server console, per-seat console state and the generated cvar catalog. |
| `bots/` | 38,368 | The shared player-bot system, built on the Q3 bot foundation. |
| `render/` | 37,469 | Shared scene, material and model interpretation. `cpu/` (7.2k) is the software rasterizer and `gl/` (7.0k) the OpenGL backend. |
| `presentation/` | 28,709 | HUD, view weapon, effects and per-seat view construction. |
| `content/` | 13,472 | Finding installed games, mounting archives, product IDs and content identity. |
| `platform/` | 10,441 | SDL and OS boundaries, POSIX filesystem. |
| `audio/` | 8,398 | Sound bank, mixer, music policy and device output. |
| `core/`, `formats/`, `world/`, `movement/`, `persistence/`, `session/`, `navigation/`, `text/`, `ui/`, `input/`, `media/`, `player_services/`, `campaign/`, `settings/`, `tools/` | 3k to 8k each | Core utilities; BSP, MDL, MD2, MD3 and image formats; world collision and spatial index; per-game movement kernels; saves and recovery; session clock and scheduling; navigation; text and fonts; UI widgets; input bindings; RoQ and Theora video; profiles; campaign progression; settings storage; in-engine tools. |

`include/qa/` (444 files) holds the public headers. `tests/` has 10 C test programs: archive, BSP, core, guest, image, model, Q1 gameplay, recovery and VFS. `tools/` holds:

- `check_q2_monster_callbacks.py`, which runs during the build;
- `generate_unified_cvars.py`;
- `check_single_engine.sh`, a diff checker that rejects new per-game enum casts and family dispatch;
- `install_qualified_build.py`.

### The single-engine goal

The plan in `docs/plan.md` asks for **one implementation per shared capability**. That means one filesystem, resource cache, actor registry, spatial index, renderer, mixer, input system, connection lifecycle and save coordinator. Each game supplies only its own rules and format adapters. The point is the combined mode: map, movement, character, weapons and monsters can come from different games in one session.

The code did not start out that way. On 6 October a single-engine review found that the core libraries each existed once, but the application layer had grown copies. It counted:

- 8 HUD renderers and 6 scene builders;
- 5 local sound producers;
- about 15 usercmd converters;
- about 105 places that assumed the map's game also owned the player's movement, character and weapons.

The review listed 325 unification items. Since then the commits have merged these paths. Examples are THE-593 ("draw local Q2 HUD with the shared original layout interpreter") and THE-356/THE-365 (one shared cvar table with per-seat dialects). `tools/check_single_engine.sh` rejects regressions. The first project rule states it: "Keep one implementation per shared capability."

## Features (as implemented in the repo)

- **Games and campaigns.** Quake, Scourge of Armagon, Dissolution of Eternity, the Quake rerelease, Quake II, The Reckoning, Ground Zero, the Quake II rerelease, Quake III Arena and Team Arena. Each is selected by product ID (for example `q1-classic-id1` or `q2-rerelease-baseq2`).
- **Game discovery.** Installed games are found beside the executable, in surrounding directories and in Steam libraries. `--game-path` and `--save-game-path` add other locations, and the saved list goes to `install-locations.json`. No game data is included.
- **Combined sessions.** These options choose each part independently:
  - `--movement q1|qw|q2|q3`
  - `--character`
  - `--map-game`
  - `--mod PRODUCT/COMPONENT`, repeatable
- **Gameplay choice.** "Map gameplay" can be Anthology (built-in C gameplay), Original (the installed game's own module, also set with `--original`) or an authored game type (`--game-type`).
- **Mods.** The engine runs original QuakeC `progs.dat`, Q3 QVM bytecode and native game modules through `src/compat/`. Independent mod components are declared in a `gameplay-mods.json` in the game's installation.
- **Renderers.** `--renderer cpu` is a software rasterizer with worker threads. `--renderer gl` is OpenGL. Both draw all four game families.
- **Local split-screen.** 1 to 4 seats (`--seats`), each with its own viewport and input.
- **Multiplayer.** A dedicated server with a stdin console, and the client and server protocols listed under `network/` above.
- **Console and cvars.** There is one console and one cvar table. It is generated from a unified catalog of 1,260 canonical cvars with 237 aliases. Quake III names are canonical, and every name from Q1, QW, Q2, the Q2 rerelease and Q3 works in every game. `fov` and `cg_fov`, for example, are one setting. Typed text runs as a command or cvar without a leading slash in every game, and chat goes through `say`.
- **Saves.** Rules set by Mike on 5 October:
  - Saves hold game state only. They contain no retail asset bytes and refer to content by identity.
  - Original save formats load: WinQuake v5 text, rerelease v6, the Q2 3.20 save directories and the Q2 2023 rerelease JSON.
  - A fully vanilla session writes its game's own legacy format by default. Mixed or modded sessions use the project's shared format.
  - Autosave runs at level entry and on authored `target_autosave`, never on a timer.

  The 7 October sweep records classic v5 and rerelease v6 text save, load and resave passing in vanilla Q1 sessions (MIKE-22). Implementation is in `src/persistence/` and the per-game save code under `src/gameplay/*/save/`.
- **Demos, cinematics and media.** RoQ cinematics, Theora video and the demo journal used for crash recovery.

## Measured frame times

These are median frame times in milliseconds. Each case measured 600 presented frames after warm-up, with vsync off and no debugger. The machine has an RTX 3090 for GL. CPU cases used seven raster workers pinned to physical cores. The cases are stationary retail spawn points (Q1 `e1m1`, Q2 `base1`, Q3 `q3dm1`), not full-campaign runs. The targets set by Mike are GL 1080p under 2 ms, CPU 640×400 under 4 ms and CPU 320×200 under 2 ms.

Early run, 5 October 02:10 (Q2 rerelease only): GL 4.326, CPU 640×400 49.505, CPU 320×200 24.420.

`docs/playtests/2026-10-06-frame-budget.md`, installed build `ff437555`, median / p99:

| Game | GL 1920×1080 | CPU 640×400 | CPU 320×200 |
| --- | --- | --- | --- |
| Q1 classic | 1.237 / 1.903 | 4.347 / 6.498 | 2.435 / 3.563 |
| Q2 classic | 1.447 / 2.367 | 4.703 / 6.464 | 3.169 / 4.346 |
| Q2 rerelease | 1.679 / 3.216 | 7.918 / 11.347 | 5.089 / 8.420 |
| Q3 | 2.532 / 2.949 | 15.104 / 17.236 | 7.401 / 10.059 |

Latest recorded run (`c52e6ff6`, medians). All four GL medians are under 2 ms. The CPU renderer still misses its targets everywhere:

| Game | GL 1080p | CPU 640×400 | CPU 320×200 |
| --- | --- | --- | --- |
| Q1 | 0.877 | 4.150 | 2.154 |
| Q2 | 1.085 | 4.219 | 2.963 |
| Q2 rerelease | 1.336 | 6.702 | 4.441 |
| Q3 | 1.944 | 13.467 | 6.074 |

The playtest notes also record rejected optimisations. One was an SSE2 sampler change in `docs/playtests/2026-10-07-cpu-fog-spans.md`. It produced identical output across 13,800 test cases, but it was removed because it did not give a reliable retail speedup.

## How to build and run

From the repository README:

```sh
git clone https://github.com/mgd34msu/Quake-Anthology.git
cd Quake-Anthology
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --target quake-anthology --parallel
./build/quake-anthology                       # startup menu
./build/quake-anthology --list-content        # show discovered games
./build/quake-anthology --game q1-classic-id1 --map start
./build/quake-anthology --game q2-classic-baseq2 --map base1 --movement q3
./build/quake-anthology --game q3-baseq3 --map q3dm1 --renderer cpu
```

Requirements: a C17 compiler, CMake 3.20+, Python 3.8+, Git, pkg-config, and the development libraries listed above. The first build needs internet access. Game data comes from your own installations.

## Relationship to the other projects

- **Quake-Anthology-TS:** the source of the feature set and of the earlier design decisions. Examples are generational actor handles, keeping live positions separate from the last-linked collision bounds, and saves that store game state rather than renderer handles. `docs/review.md` reviews it at commit `5d6c6e31`.
- **Original id Software sources (`qsrc`):** decide gameplay behaviour where they and the TypeScript code disagree.
- **Quake-Anthology-Rust:** developed in parallel from the same TypeScript source. The C port's lessons file (`c-port-lessons-for-rust.md`) feeds its rules. Two examples: build the common primitives before any game code, and test combined mode from the first playable milestone.

## Credits and licence

- Original Quake, Quake II and Quake III Arena engine and game code: id Software, released under the GNU GPL. The repository `LICENSE` names "Original Quake game code: Copyright (C) 1996-2005 id Software, Inc."
- Protocol and format compatibility follows the published behaviour of community engines whose protocols it speaks: FitzQuake, RMQ, R1Q2, Q2PRO and Q2REPRO.
- Licence: GNU General Public License, version 2 or later.
- Game data is not included and must come from the user's own copies.
