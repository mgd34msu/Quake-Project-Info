# Quake 2 TypeScript ports: Quake-2-TS and Quake-2-Rerelease-TS

Two related TypeScript ports of Quake II, both running on Bun. **Quake-2-TS** is a line-by-line port of the id Software v3.21 GPL source (1997-2001): engine, server, game module, client, software renderer and OpenGL renderer. **Quake-2-Rerelease-TS** starts from a copy of Quake-2-TS v1.1.0 and turns it into a re-release-style engine that hosts both the 2023 Quake II re-release game module (also ported to TypeScript) and the classic v3.21-era game modules (baseq2, ThreeWave CTF, LMCTF, The Reckoning, Ground Zero).

## Links

| Project | GitHub | Local path |
|---|---|---|
| Quake-2-TS | https://github.com/mgd34msu/Quake-2-TS | `~/Projects/quake-2-ts` |
| Quake-2-Rerelease-TS | https://github.com/mgd34msu/Quake-2-Rerelease-TS | `~/Projects/quake-2-re-ts` |
| Quake-2-TS-baseq2 (private) | https://github.com/mgd34msu/Quake-2-TS-baseq2 | not documented here |

Quake-2-TS-baseq2 ("files for q2ts") is a private repository that holds game data for the Quake 2 ports. Game data is commercial content. Nothing from it is described or included here.

## Status

- **Quake-2-TS: playable, finished at v1.2.1.** The README describes it as complete. The tree has not been touched since 2026-09-01. It is now the "faithful 3.21" reference that the re-release port inherits from.
- **Quake-2-Rerelease-TS: playable, in active development.** Latest tag is v1.1.1 (2026-09-06). The changelog has an "Unreleased" section with later work, and the last commit is 2026-09-08. A release candidate build exists in `dist/quake2-rerelease-ts-rc1-linux-x64`. The README says release builds are single-file executables for Linux, Windows and macOS; only the Linux x64 candidate directory is present in the local checkout.

Both need the original game data, which is not included in either repo.

## Timeline (from git history)

| | Quake-2-TS | Quake-2-Rerelease-TS |
|---|---|---|
| First commit | 2026-08-29 | 2026-08-29 |
| Last commit | 2026-09-01 | 2026-09-08 |
| Commits | 109 | 276 |
| Authors | one (Mike Davis) | one (Mike Davis) |
| Tags | v1.0.0 (2026-08-29), v1.1.0 (2026-08-29), v1.2.0 (2026-09-01), v1.2.1 (2026-09-01) | v1.0.0 and v1.0.1 (2026-09-01), v1.1.0 (2026-09-03), v1.1.1 (2026-09-06) |

Tag dates are the dates recorded on the git tags. The changelog headings differ slightly (v1.1.0 is dated 2026-09-02 in the re-release changelog).

### Quake-2-TS phases

1. **2026-08-29, scaffolding and engine core.** The first commit sets up strict tsconfig, a zero-`any` check, `PORTING.md` conventions and an orchestration store. Early commits port `q_shared`, `crc`/`md4` (with reference tests), `pmove` and `common`/`cvar`/`cmd`.
2. **Same day, game module and boot.** The game type core is ported along with stubs for all 46 game modules, then `g_monster`, `p_view`, `p_hud` and the server bridge. A commit titled "DEDICATED SERVER RUNS" lands when `Qcommon_Init`/`Qcommon_Frame` are wired in `main.ts`.
3. **Renderers and platform.** The software renderer (`r_image`, `r_draw`, and so on) is ported and headless frames and screenshots are produced. A 1920x1080 mode is added. Mission-pack mappings (Xatrix, Rogue) are added to `PORTING.md`.
4. **2026-09-01, polish releases.** v1.2.0 adds a flat video mode table up to 3840x2160, custom resolutions, render-resolution scaling, QBSP extended maps, and parity fixes. v1.2.1 is a video UX patch. A mid-stream commit adding re-release weapon-wheel handling was reverted to keep this tree identical to vanilla 3.21 behaviour.

The whole port took four calendar days of commits (2026-08-29 to 2026-09-01). The git history shows no earlier work in this repo.

### Quake-2-Rerelease-TS phases

1. **2026-08-29, seed and architecture.** First commit: "seed from Quake-2-TS v1.1.0; unified re-release architecture". `ARCHITECTURE.md` records the design decisions (dated 2026-08-29). The legacy game binding is extracted into `bindings/legacy.ts` and the 2023 KEX API surface is ported as types (`src/kexapi`).
2. **2026-08-30, KEX game and engine work.** The protocol codec seam is added (protocol 34 becomes the vanilla codec), the re-release game module is ported file by file (`p_weapon`, then CTF, Xatrix and Rogue systems, Rogue monsters), splitscreen and the shader pipeline for the GL renderer begin.
3. **2026-08-31 to 09-01, content and formats.** LMCTF hosting through the legacy binding, a hand-written baseline JPEG decoder, TGA and other image formats, lightgrid model lighting fixes, data-tree selection (original maps vs re-release maps per launch), omnidirectional shadow maps with entity casters. v1.0.0 and v1.0.1 tags.
4. **2026-09-02 to 09-06, play-test rounds.** v1.1.0 makes any content playable under any ruleset (see Features). Play-test fix rounds cover Call of the Machine, the classic module on re-release data, and LMCTF (for example, a fix so LMCTF creates a per-player stats record at connect, because scores stayed at zero). Networking fixes: host-name resolution, map checksum comparison.
5. **2026-09-07 to 09-08, bots.** v1.1.1 adds bots for the re-release module. Later commits (in the "Unreleased" changelog section) copy bot brain improvements back from the Quake 1 re-release port (`quake-1-re-ts`): swimming, flag-carrier behaviour, button pressing, corner cuts, distance-weighted targeting.

## Stack and size

Language is TypeScript (strict, `tsc --noEmit` as a gate), runtime is Bun, dev dependencies are `@types/bun` and `typescript`. The platform layer is SDL2 reached through `bun:ffi`. Line counts below are from `wc -l` on `.ts` files in the checkout.

| | Quake-2-TS | Quake-2-Rerelease-TS |
|---|---|---|
| `.ts` lines in `src`, `test`, `scripts` | 265,954 | 780,109 |
| `.ts` files in `src` | 371 | 860 |
| `.ts` files in `test` | 95 | 321 (316 named `*.test.ts`) |
| Package version | 1.2.0 (tags go to 1.2.1) | 1.1.1 |

Lines per `src` directory (from `wc -l`):

| Directory | Quake-2-TS | Quake-2-Rerelease-TS |
|---|---|---|
| `client` | 18,883 | 45,008 |
| `ctf` (ThreeWave CTF) | 26,524 | 79,699 |
| `game` (baseq2) | 40,512 | 78,182 |
| `kexapi` | n/a | 2,314 |
| `kexgame` (2023 game module) | n/a | 102,890 |
| `lmctf` | n/a | 90,079 |
| `null` | 53 | 53 |
| `platform` | 3,625 | 6,677 |
| `qcommon` | 7,275 | 29,811 |
| `ref_gl` | 9,554 | 17,823 |
| `ref_soft` | 12,562 | 14,490 |
| `rogue` | 63,012 | 80,070 |
| `server` | 5,402 | 17,492 |
| `shared` | 1,871 | 2,594 |
| `xatrix` | 48,557 | 77,382 |

The Rerelease-TS README says the ported 2023 game module is about 103k lines of C++; the 102,890-line `kexgame` directory is consistent with that. Counts are physical lines, not logical statements, and TypeScript line counts are not directly comparable to the C originals.

## Architecture

### Quake-2-TS

`PORTING.md` maps each C directory to a TypeScript directory. In outline:

- `src/qcommon`: common code, cvars, commands, message codec, filesystem, CRC/MD4, `pmove`, collision model.
- `src/server`: the server, including the game import/export bridge.
- `src/game`: the baseq2 game module (monsters, items, weapons, player code, 46 modules).
- `src/ctf`: ThreeWave CTF 1.52 game module.
- `src/xatrix` and `src/rogue`: The Reckoning and Ground Zero mission-pack game modules, ported by applying the C diff against baseq2 to the TypeScript baseq2 port and writing pack-only files fresh.
- `src/client`: the client, console, menus, HUD, input.
- `src/ref_soft` and `src/ref_gl`: the two renderers, switched at runtime with `vid_ref soft` or `vid_ref gl`.
- `src/platform`: SDL2 video, audio and input over `bun:ffi`.
- `src/null`: a null client for dedicated server mode. `src/shared`: shared types and math.
- `src/main.ts`: entry point; `scripts/check.sh` is the gate (strict compile plus a grep ban on `any`).

Conventions from the README: zero `any`, no type casts (except `as const`), and bug-for-bug fidelity to the C where observable, with deviations documented in file headers and `PORTING.md`.

### Quake-2-Rerelease-TS

`ARCHITECTURE.md` is the contract for the design. The core idea: one engine, five game modules, two game-API bindings.

- The engine's internal model is the wide re-release shape (more entity and player state fields, `MAX_EDICTS` 8192, `MAX_MODELS` 8192, `MAX_SOUNDS` 2048, `MAX_IMAGES` 512) regardless of which module runs. Narrowing happens only at the wire.
- A **KEX binding** (2023 API) hosts the re-release game module in `src/kexgame`, with the API types in `src/kexapi`.
- A **legacy binding** (v3 API, in `bindings/legacy.ts`) hosts `baseq2`, `ctf`, `lmctf`, `xatrix` and `rogue`. These keep their original 1997 behaviour, quirks included, on purpose.
- Variable server tick: `sv_tick_rate` defaults to 40 for the KEX module; legacy modules run game logic at their native 10 Hz, with frame-division interpolation so the client still runs at full rate.
- A protocol codec layer handles legacy protocols 34/35/36 and the re-release protocol 1038.
- Savegames use one container format holding JSON blobs from either module family.
- Design decision: re-release-only content is never backported into the legacy modules' rules. The v1.1.0 changelog states that what the legacy trees gained is content (entity classes), not rule changes.
- Reference sources named in `ARCHITECTURE.md`: the ZeniMax re-release game source, `q2repro`, `q2proto` and the 1997 C source. A long-horizon reference library lives under `~/Projects/qsrc`.
- Other directories: `src/qcommon` (grown to include the shared bot brain, navigation, protocol codecs), `src/server` (including `bots/`), `src/client`, `src/ref_gl`, `src/ref_soft`, `src/platform`. Scripts: `check.sh`, `interop-matrix.sh`, `sandbox-verify.sh`.

## Features

### Quake-2-TS (v1.2.1)

- All C subsystems represented: engine core, server, game module, ThreeWave CTF 1.52, full client, software renderer, OpenGL renderer, with SDL2 as the single platform layer.
- Xatrix and Rogue game modules are in `src/` (the README's headline text does not list them; the source tree and `PORTING.md` do).
- Dedicated server mode (`+set dedicated 1`).
- Video modes 0-10 in the README; v1.2.0 expanded this to a mode table from 320x240 up to 3840x2160, custom width and height, render-resolution scaling, and a "scale to fullscreen" toggle.
- QBSP extended-format maps load in collision and both renderers (v1.2.0).
- Test suite reported in the changelog as 1385 of 1385 passing, order-independent across three forward runs and one reversed run (v1.2.0). I did not re-run it.
- Fixes found during porting, named in the changelog: a hang in `FS_NextPath` caused by a pointer-identity porting bug, signed-to-unsigned face/marksurface reads, and a software-renderer surface-extents clamp.

### Quake-2-Rerelease-TS (v1.1.1 plus unreleased)

From the README and changelog:

- The complete 2023 re-release game module, ported and coverage-audited, running at 40 Hz over the re-release protocol.
- Five classic modules as first-class citizens: baseq2, ThreeWave CTF, LMCTF, The Reckoning, Ground Zero.
- Any content under any ruleset: every shipped map, including the re-release's Call of the Machine campaign, loads under the classic and expansion modules with every entity present, and the 1997 maps play under the re-release module. A "New Game" screen picks content, ruleset and data tree (original 1997 data or re-release data).
- Two to four player local splitscreen.
- GL renderer: shader pipeline, per-pixel lighting, full shadow mapping (models cast and receive), fog, world text, glow maps, MD5 skeletal models, 1024px lightmap atlases for decoupled lightmaps. Software renderer retained for 1997-faithful output.
- Resolution-independent texture replacement: higher-resolution files in a game directory replace originals.
- Format coverage listed in the v1.0.0 changelog: QBSP extended maps, MD5 models, materials, kfont and TTF/OTF fonts, sprite/wal/pcx/tga/jpg/png/bmp/gif images, haptics files, environmental reverb.
- SDL gamepad support with haptics, MVD/GTV multi-view demos and relay, HTTP downloads, demo record and playback.
- Bots (v1.1.1): a game-agnostic bot brain shared with `quake-1-re-ts`, bound over the KEX entities, using the re-release's NAV3 navmesh. Commands `addbot`, `kickbot`, `bot_skill`, `bot_count`. Unreleased work adds team objectives, swimming, button pressing and flag-carrier logic.
- Localization tables (six languages) loaded as UTF-8.
- The v1.0.0 changelog describes LMCTF as one of the five modules; `src/lmctf` is 90,079 lines. See "LMCTF" below.

I have not run either game. Feature statements above come from the READMEs, changelogs and source tree.

## How to run and build

Both require a Quake II install (a `pak0.pak` and so on) and Bun. Quake-2-TS also needs libSDL2.

Quake-2-TS (from its README):

```sh
bun install
bun src/main.ts +set basedir /path/to/gamedir                       # software renderer
bun src/main.ts +set basedir /path/to/gamedir +set vid_ref gl +set gl_mode 10
bun build --compile src/main.ts --outfile q2ts                      # standalone binary
```

`+set vid_fullscreen 0` gives windowed mode. `bun run check` runs the type and zero-`any` gate; `bun test` runs the tests.

Quake-2-Rerelease-TS (from its README):

```sh
bun install
bun run src/main.ts +set basedir /path/to/quake2
```

`basedir` can be a classic 1997 tree, the 2023 re-release tree, or a classic tree with the re-release nested inside it; both are detected. `bun run check` and `bun test` work the same way.

## Relationship to other projects

- Quake-2-Rerelease-TS is seeded from Quake-2-TS v1.1.0 and shares its legacy game code. Fixes such as LMCTF item lists were made in the re-release repo.
- The shared bot brain in Quake-2-Rerelease-TS is kept in step with the Quake 1 re-release TypeScript port (`quake-1-re-ts`): commits say "fed back from quake-1-re-ts".
- The `qsrc` directory holds original source trees used as references, including `lmctf60`.
- The unified TypeScript project, the C port and the Rust port are documented in their own files in this folder; I did not inspect them for this document, so no claim is made here about how Quake 2 code is reused there.

## LMCTF

The Rerelease-TS repo contains LMCTF as a hosted game module (`src/lmctf`). The file header of `src/lmctf/g_main.ts` says it ports from the `lmctf60` source tree and notes that the diff of `g_main.c` against the ThreeWave CTF version is 785 of 858 lines. The v1.1.0 changelog records a fix where the module "shipped a partial item list" so LMCTF maps had no weapon, ammo or armor pickups; the full item list, weapon set and deathmatch item rules were then ported. This document does not cover the separate LMCTF Buzzmod mod project; see its own documentation.

## Notable lessons and decisions

- **Fidelity over cleanliness.** Quake-2-TS reproduces C behaviour including bugs, and a re-release-style weapon wheel change was reverted so the tree stayed identical to 3.21.
- **Strict typing as a bug finder.** The zero-`any`, no-cast rule surfaced real bugs (signed-to-unsigned reads, the pointer-identity hang).
- **Two bindings, not a proxy.** Quake-2-Rerelease-TS rejects routing legacy games through the 2023 API; both bindings sit on one engine core.
- **Configstring widening is per session.** Content that exceeds vanilla's 256-model limit switches to the wide layout mid-load; content that fits stays byte-identical to vanilla, so protocol-34 demos and vanilla clients are unaffected.
- **Bots carry across ports.** The bot brain is game-agnostic and developed across the Quake 1 and Quake 2 ports.
- All of this work was done by a single author in about ten days of commits (2026-08-29 to 2026-09-08), with a heavily structured process visible in the history (task identifiers like U001 and RS2a, ported file by file with tests).

## Credits and licence

- **id Software**: Quake II (1997) and the v3.21 GPL source release.
- **ZeniMax Media**: the 2023 re-release game module source (`quake2-rerelease-dll`, GPLv2); the `kexgame` port derives from it.
- **Q2PRO** (skullernet and contributors), **q2repro** (Paril and contributors, including Paril-KEX and Sam-KEX work) and **q2proto** (res2k, Frank Richter): engine lineage and reference specifications for the re-release engine behaviour and wire protocol.
- **ThreeWave CTF** and **LMCTF**: third-party mods whose game code the CTF and LMCTF modules port. I did not find author names in the files I read; check the source headers before crediting individuals.
- Licence: GNU GPL v2 for both repos (`LICENSE`). Quake II is a registered trademark of id Software. Game assets are not included and remain under their original terms.
