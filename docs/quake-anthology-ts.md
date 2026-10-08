# Quake Anthology (TypeScript)

Quake Anthology TS is one engine and one executable that plays Quake, QuakeWorld, Quake II and Quake III Arena, along with their expansions and the Quake and Quake II rereleases. It is written in strict TypeScript and runs on Bun. Rules, maps, movement, character model, weapons, monsters and equipment can each come from a different game. It can load original mod code without modification: QuakeC `progs.dat`, Quake III QVM bytecode, and Quake II game DLLs, which run in an x86/x64 interpreter written in TypeScript. Version 1.0.0 was released on 2026-09-26 as a Linux x64 executable. It was built by merging the earlier per-game TypeScript ports into one codebase, and it later served as the feature reference for the C and Rust ports.

## Links

- GitHub: https://github.com/mgd34msu/Quake-Anthology-TS
- Release: https://github.com/mgd34msu/Quake-Anthology-TS/releases/tag/v1.0.0
- Local path: `~/Projects/quake-typescript` (package name `quake-typescript`, and the executable is also named `quake-typescript`)

## Status

**Released (1.0.0) and playable, with one known performance problem.** The project defines 23 functional targets (T01–T23), and `docs/functional-targets/status.md` lists 22 of them as accepted. The open one is T10, gamecode and mod execution. It is open because of speed: Quake II game DLLs running in the x86 interpreter are slower than the original's frame budget. The release notes give a late-`base1` sample of 41.23 ms of native execution and 45.55 ms per application step, against a 25 ms native tick interval. In heavy scenes this shows up as low frame rates and stalls. The notes say this fix is planned for a patch release. Saving and loading are accepted. Autosaves happen only on level entry.

The project's own docs list what has not been verified. Full campaign playthroughs, every mod, every physical input device, every rendered language and every mixed native multiplayer combination are untested. Interoperability with retail multiplayer clients is also unverified. Ranking services are a stub; this is the only stub the project allows, and it was approved on 2026-09-22.

No commits have been made since the `v1.0.0` tag, which points at HEAD. The local working tree has uncommitted edits to three files (one source file, one test and one verification fixture). Active development has since moved to the C and Rust ports (see [Relationship to other projects](#relationship-to-other-projects)).

## Timeline

Figures come from `git log` on `main`.

- First commit: 2026-09-10 (empty initial commit). The last commit is the 1.0.0 release, 2026-09-26.
- 805 commits, all by Mike Davis, over 17 calendar days.

| Dates | Commits | What happened |
|---|---|---|
| 2026-09-10 – 09-11 (first two hours) | 5 (the initial commit plus the first 4 of 09-11) | The plan was written first: `docs/source-assessment.md`, `docs/project-plan.md` and a dependency graph, based on an inspection of the per-game ports and the original id source. Next came strict Bun tooling and the Linux platform layer, then the engine core and cross-game content loaders. |
| 2026-09-11 (rest of the day) | 82 | Bulk integration. The mixed-game runtime commit alone added about 131,600 lines from the donor ports. The x86/x64 guest loaders and interpreter were added the same day. Later commits that day connected Q1/Q2 content composition, expansions, multiplayer modes, bots, LMCTF state, grapple cores, and Q3 weapons running on Q1/Q2 actors. |
| 2026-09-12 – 09-16 | 483 | Peak activity: 146 commits on 09-12 alone. The work was integration and repair across subsystems: QVM async calls, writable user content overlay, materials and lighting, shadows, input devices, server browser, bot navigation on moving platforms, menus and saves, and many small renderer and allocation optimizations. |
| 2026-09-17 – 09-24 | 116 | User-facing documentation, plus fixes from playtesting: Quake coloured lightmap offsets, foreign monster teleports, armor rules. Also original QVM pickup and armor delivery, the expansion arsenal menus and rerelease-only point-of-interest entities. |
| 2026-09-25 | 111 | Performance work on the native DLL interpreter: cached instruction plans, fewer memory copies, reused call plans, faster string scans. Also the T12 (combat and equipment) completion. |
| 2026-09-26 | 8 | More interpreter optimizations (SIMD instruction blocks), then "Release 1.0.0 with level-entry autosaves", tagged `v1.0.0`. |

Commit timestamps show the work ran around the clock in parallel lanes. Repository docs refer to "waves", "lanes" and acceptance receipts under `.artifacts/` (local only, not committed).

## Stack and size

- Language: TypeScript 5.9.3 with very strict compiler settings (`strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noPropertyAccessFromIndexSignature`, unused-code checks).
- Runtime: Bun 1.3.14 or newer. `bun run build` compiles a standalone executable that includes the Bun runtime.
- Native libraries (system, loaded through Bun FFI): SDL2, OpenGL, FreeType, libvorbisfile, and libtheoradec for OGV video.
- Only two dev dependencies: `typescript` and `@types/bun`. There are no runtime npm dependencies.
- A custom checker, `tools/check-policy.ts`, uses the TypeScript compiler API to enforce project rules. It also rejects committed native binaries and non-TypeScript implementation files such as C, Rust, GLSL or WASM. The whole engine, including the x86 interpreter, is TypeScript.

Lines are counted with `wc -l` over git-tracked `.ts` files, so blank lines and comments are included:

| Area | Files | Lines |
|---|---|---|
| `src/` (engine) | 2,080 | 365,926 |
| `tests/` | 657 | 92,166 |
| `tools/` | — | 6,388 |
| `docs/` Markdown | 40 | 9,634 |

The repository tracks 2,888 files in total. Large verification data is stored as JSON under `verification/`.

Largest `src/` areas, in lines: `content` 122,854; `app` 67,533; `bots` 29,782; `compat` 29,590; `network` 23,093; `ui` 13,008; `render` 12,401; `guest` 10,735; `world` 9,875.

## Architecture

The top-level directories under `src/`:

| Directory | What it does |
|---|---|
| `app/bootstrap` | Application host (149 files): startup, the frame clock, campaign units and cinematics, demo recording and playback, Q3 arena menus and progression, config scripts, cvar archives, capture and effects. |
| `contracts` | Shared types that set ownership between subsystems. Examples: `ExecutableRecipe` separates map geometry, map entities, campaign gamecode, movement, character, weapons, enemies, combat, inventory and match rules. Also actor identity handles, source clocks (NetQuake, QuakeWorld, Q2 100 ms frames, Q2 rerelease frame prep, Q3 command subdivision), and numeric profiles for x87/SSE/binary32 arithmetic. Described in `docs/contracts.md`. |
| `content` | Archive readers (PAK, ZIP/PK3, loose files), per-product catalogs, mounts, the writable user overlay, and the per-game content trees `q1/`, `q2/` and `q3/`, including mission packs, Team Arena and rerelease material. `composition/` holds the cross-game supply tables (`q1-q2-supply.ts`, `q3-q1-supply.ts` and others) that place one game's items and weapons into another game's maps. Also monster rosters and the mod catalog and declarations. |
| `compat` | Hosts for original mod code. `qc/` is a QuakeC executor and builtins. `qvm/` runs Quake III bytecode. `q2/` holds classic and rerelease native game DLL adapters: entity and client layouts, combat, pickups, inventory, and "native primary" player and weapon profiles. `q3/native` is the Q3 native host. |
| `guest` | An x86 and x86-64 CPU interpreter in TypeScript. It has PE (Windows DLL) and ELF (Linux .so) loaders with relocations and imports, Windows and System V runtime shims, x87 and SSE floating point, and calling-convention adapters (cdecl, stdcall, fastcall, thiscall, Win64, System V). This is how unmodified Quake II mod DLLs run. `src/guest/abi/README.md` documents the ABI rules and their sources. |
| `world` | Actors, collision, gameplay rules, geometry, the scheduler, sessions and spatial queries. |
| `movement` | Player movement for each source game (Q1, QuakeWorld, Q2, Q3), selectable independently of the rules. |
| `formats` | BSP loaders for Q1, Q2 and Q3, Q1/Q2 and Q3 model formats, and image formats. |
| `render` | One backend contract with two implementations: a CPU software rasterizer (`render/cpu`) and OpenGL (`render/gl`), with shadow atlas, fog and gamma. An optional render worker exists but is off by default. |
| `materials` | Q3 shader evaluation (deforms, fog, dynamic lights, cinematic textures) used by both renderers. |
| `audio` | Mixer, music, streams, WAV/ADPCM decoding, reverb presets and environment geometry. |
| `network` | Native protocols for each game: NetQuake and QuakeWorld (`q1/`); Q2 classic, MVD/GTV and the KEX rerelease lobby and LAN transport (`q2/`, `q2/kex`); Q3 netchan, Huffman, snapshots, pure-server checks, rcon and downloads (`q3/`); IPX including a DOSBox IPX bridge (`common/`); and a separate mixed-game protocol (`unified/`, URL scheme `qts://`). |
| `bots` | Shared bot behaviour (Q3 and rerelease directors) and navigation. That includes a TypeScript AAS (Q3 area awareness) builder and reachability code, plus rerelease path following. |
| `console`, `settings`, `input` | Console with command and cvar discovery (`find`, `help`), per-seat key bindings, gamepad, haptics, MIDI and mouse handling. |
| `llm` | Optional LLM client: ChatGPT subscription sign-in, OpenAI API key, or any OpenAI-compatible endpoint, with SSE streaming. |
| `persistence` | Save formats: shared saves, original Quake v5/v6 import and export, and the save format of the older Quake-2-TS port. |
| `ui`, `text`, `media`, `camera`, `capture` | Menus and HUD, localization and fonts, cinematic and video playback, spline cameras, frame capture. |

Supporting directories: `tests/` (Bun tests organised by the same subsystems, plus `composition/`, `gameplay/` and `policy/`), `tools/` (build, policy check, source inventory, reference tooling, verification CLI) and `verification/` (feature ledger, reference cases and the product manifest).

## Features

These are features the repository documents as working in 1.0.0:

- **Games and products**: classic Quake (id1 and mission packs), the Quake rerelease, QuakeWorld (`qwprogs.dat`), classic Quake II and its mission packs, the Quake II rerelease, Quake III Arena and Team Arena. Classic and rerelease data stay in separate directories. Rerelease-only additions must not appear in an all-classic setup.
- **Custom game**: under **Play a game → Custom game**, rules (World), map source, movement, character source and model, weapons, monster replacements and equipment are chosen separately. The same choices are available on the command line (`--game`, `--map-game`, `--map`, `--movement q1|q2|q3|qw`, `--character`, `--model`, `--mode`). Documented examples include QuakeWorld movement with a Q3 Ranger model in Quake's `e1m1`, and the Q3 LRCTF mod running on Quake II's `base1`.
- **Mod composition**: components from different games can be enabled together (`--mod PRODUCT/COMPONENT`, repeatable). Mods still need a supported interface. For QVM and native modules, this is a JSON declaration that matches the exact module bytes and describes its item tables, player interface and combat entry points (`docs/mod-compatibility.md`). Being discovered on disk does not make a mod compatible. Threewave 1.7 has a built-in declaration. Early Q3 1.16n/1.17 server modules have their own profile (tested with InstaGib Plus).
- **Grappling hook and offhand equipment**: Hook can be Off, Weapon slot or Offhand. Hook style can be Threewave (Q1, Q2 or Q3), LMCTF (Q2) or LRCTF (Q3). Offhand grenades can be turned on for any game.
- **Multiplayer**: each game's own protocol, plus the mixed-game protocol (`--listen-unified` / `--connect-unified`, or `connect qts://host:port`). With the mixed-game protocol the host picks the composition, and clients predict their own movement against authoritative collision. Server browser, rcon, and Q2 HTTP and Q3 downloads are included.
- **Bots** for Q2 and Q3 worlds and mixed worlds, using shared navigation.
- **Saves**: shared save format, import and export of original Quake v5/v6 saves, and level-entry autosaves. Original formats reject mixed state they cannot represent.
- **Rendering**: CPU software renderer or OpenGL, selected with `--renderer`.
- **Demos**: recording and playback, including QuakeWorld MVD and Q2 GTV.
- **Console**: `find`, `help`, tab completion and history. `modcmd PRODUCT/COMPONENT <command>` sends a command to one enabled mod component.
- **LLM console (optional)**: `llm_ask "<question>"` answers questions. `llm_exec "<request>"` generates console commands, which are displayed and checked against the command table before being run by the normal command handler. `llm_cancel` stops a request. Configure it under **Options → LLM options**.

## How to run and build

From the repository README and `docs/getting-started.md`:

```sh
bun install --frozen-lockfile
bun run start --content-root /path/to/qfiles          # opens the menu
bun run start --content-root /path/to/qfiles --list-content
bun run build                                         # compiles the standalone executable
./quake-typescript --content-root /path/to/qfiles --menu --renderer gl
```

Game data is not included. The expected layout is `q1/id1`, `q1/rerelease/id1`, `q2/baseq2`, `q2/rerelease/baseq2`, `q3a/baseq3`, `q3a/missionpack` and `qw/` under one content root (`docs/content-layout.md`). Writable add-ons, saves and settings go under `~/.local/share/quake-typescript/`.

Development checks: `bun run check` runs the type check, the policy check, the test suite and the plan validators. The build compiles a snapshot of the source, checks that its inputs did not change, and writes `dist/runtime.json` with the executable location and source identity.

## Relationship to other projects

**Where the code came from.** Before this engine, there were separate TypeScript ports of each game, all made in late August and early September 2026:

| Port | Repo | Commits | Dates |
|---|---|---|---|
| Quake 2 | Quake-2-TS | 109 | 2026-08-29 – 09-01 |
| Quake 2 rerelease | Quake-2-Rerelease-TS | 276 | 2026-08-29 – 09-08 |
| Quake 1 | Quake-1-TS | 40 | 2026-09-04 – 09-06 |
| Quake 1 rerelease | Quake-1-Rerelease-TS | 140 | 2026-09-06 – 09-08 |
| Quake 3 | Quake-3-TS | 7 | 2026-09-09 – 09-10 |

`docs/source-assessment.md` (2026-09-10) records the starting point. Three repositories supplied code: Quake-1-Rerelease-TS, Quake-2-Rerelease-TS and Quake-3-TS. The rerelease ports already contained both classic and rerelease behaviour. Quake-2-TS was kept for comparison only, and its JSON save format can still be imported (`src/persistence/q2-typescript.ts`). Quake-1-TS is not used as a code source.

The assessment did not trust any of the earlier ports by default. Mike judged the Q1 and Q2 ports weaker and the Q3 port stronger. So Q3's strict tooling, platform layer and CPU/GL renderer interface became the base. Q1's QuakeC interpreter and Q2's campaign, audio, multiplayer and content-mount rules were carried over after review. Existing port behaviour that differed from the original id source was treated as a bug to fix, not as something to preserve. Examples include Q1 replacing destructible objects with fixed walls, the rune relay losing its sigil check, and Q2 classic saves dropping fields.

The Q2 rerelease port's `ARCHITECTURE.md` had already described a single engine for Q1, Q2 and Q3. This repository is where that design was built. It added the parts the old ports lacked: one actor and identity model, one scheduler that keeps each game's own clock, a generic x86 interpreter for native DLL mods (Q2 previously used a hard-coded list of known modules), and a mixed-game network protocol.

**What came after.** After 1.0.0, development moved to native rewrites that treat this engine as the feature reference:

- **Quake-Anthology** (C, `~/Projects/quake-anthology`, https://github.com/mgd34msu/Quake-Anthology): first commit 2026-09-26, the day of this release. Its README says it is not yet ready to play.
- **Quake-Anthology-Rust** (https://github.com/mgd34msu/Quake-Anthology-Rust): first commit 2026-09-29. Its README states: "Behaviour comes from qsrc, features from quake-typescript, and proven algorithms and fixes from the C port."

**LMCTF Buzzmod.** The Q2 LMCTF hook style and LMCTF match state in this engine are tied to Mike's LMCTF work for Quake II (about 44 source files mention `lmctf`). That project has its own document.

## Notable decisions and lessons

- **Plan before code.** The first three commits are planning documents: the source assessment, the project plan, a dependency graph and a plan validator (`docs/validate-plan.ts`). Implementation started after them. Status is tracked against 23 named functional targets, not a feature checklist.
- **Faithful to the original source.** A classic preset must behave like the original game. Behaviour is checked against the original id source, the rerelease QuakeC and DLL source and q2repro, all kept locally in `~/Projects/qsrc`. Older port docs and test results did not count as evidence.
- **Separate identities.** A single "game" string could not describe a mixed session. Gameplay implementation, behaviour profile, content, map source, asset precedence and wire protocol are stored separately, and saves and launch records keep all of them.
- **Run original mod code instead of rewriting it.** Q2 mod DLLs run unmodified in an x86/x64 interpreter written in TypeScript, so no per-mod port is needed. The cost is speed: this interpreter is the remaining 1.0.0 performance problem, and it is the main reason for the later native C and Rust ports.
- **Declarations instead of guessing.** Mods describe their private data layouts in JSON tied to exact module bytes. The engine does not try to infer those layouts automatically.
- **Strict typing everywhere.** Strict compiler settings, plus a policy checker that rejects committed native code and binaries.
- **Honest status.** The status pages separate "accepted in source" from "verified by running it", and they list what has not been verified.

## Credits and licence

- Engine code: GNU GPL version 2 or later (`LICENSE`, `package.json`). Source headers keep their original attribution.
- It is based on id Software's GPL source releases of Quake, QuakeWorld, Quake II and Quake III Arena. It also follows the published Quake rerelease QuakeC, the Quake II rerelease game DLL source and q2repro, through the earlier TypeScript ports and direct comparison.
- Mods mentioned (Threewave, LMCTF, LRCTF, InstaGib Plus, mission packs) belong to their original authors. The engine loads them from the user's own installation and does not include them.
- Game data is not included or relicensed. Players must supply their own copies of the games.
