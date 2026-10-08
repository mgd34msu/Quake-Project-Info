# Quake-3-TS: Quake III Arena and Team Arena in TypeScript

Quake-3-TS is a TypeScript port of Quake III Arena and Team Arena, translated from the Quake III Arena 1.32b source release. The engine, game logic, bots, menus, HUD, audio mixing and cinematics are TypeScript running on Bun. Both games share one runtime, with per-product rules and presentation. There are two renderers: a CPU software rasterizer written in TypeScript and an OpenGL renderer. Both present through SDL2. Retail game data is not included; the user supplies their own Quake III installation.

## Links

- GitHub: https://github.com/mgd34msu/Quake-3-TS
- Local path: `~/Projects/quake-3-ts`
- Licence: GPL-2.0-or-later (see Credits)

## Status

Playable, with known gaps. The repo's own README says playable client and dedicated-server paths are integrated for both games, and that source parity and performance work continue. `docs/STATUS.md` records gameplay checks passing on Linux x64 for both games and both renderers, covering menus, switching between Quake III and Team Arena from the menu, bots, protocol-68 networking, demos and renderer restarts.

Honest limits, taken from the repo's own documents:

- CPU rendering is slow. Recorded timedemos at 640x480 measured about 9-10 FPS on the CPU renderer and 30-40 FPS on the OpenGL renderer, on one test machine (GL on an RTX 5060 Ti). The README says these are short replay measurements that exclude live server, bot and network work. OpenGL is the renderer to use for play.
- The repo states that completed plan gates and source examination "do not establish complete original-engine equivalence". `docs/PARITY_PUNCH_LIST.md` has all 23 main corrections checked off and 14 of 15 optional-source groups accepted. The remaining group (O15) still lacks four OmniTimer SDK blocks.
- Non-Linux platforms have less evidence. Release targets exist for Linux x64/ARM64, macOS Intel/Apple Silicon and Windows x64, but `docs/PLATFORMS.md` says a successful cross-build does not show an executable runs on its target, and that only Linux x64 has retail gameplay evidence for both games and both renderers.
- Physical joystick behaviour and external-mod compatibility are listed as qualified or open.
- `docs/STATUS.md` notes six broader browser-fixture test failures that reproduce unchanged in the preceding snapshot, and says no full-suite pass is claimed.
- `docs/RELEASE.md` describes the latest build as "a local runtime build, not a GitHub release". The README points to a GitHub Releases page; I did not check whether releases are actually published there.

## Timeline

From `git log` in the local clone (timestamps are -0500):

- First commit: 2026-09-09 17:00, "Initial Quake III Arena and Team Arena TypeScript port". It adds 1,084 files and 326,957 lines in one commit, so the earlier development history is not in this repo. The work was imported as a finished baseline.
- Last commit: 2026-09-10 23:44, "Correct native path aliases and release notice packaging".
- Commit count: 7, all within about 31 hours.

The seven commits:

1. 2026-09-09 `a1e09e1` Initial port.
2. 2026-09-10 `ce78de0` Preserve raw session leaders and verify catalog filename bytes.
3. 2026-09-10 `c608464` Port source options and integrate parity corrections (162 files changed, about 12.8k lines added).
4. 2026-09-10 `9e86b40` Port remaining diagnostic and standalone source algorithms (46 files, about 2.4k lines added). This is the commit the 01:57 UTC executable build was made from.
5. 2026-09-10 `4b91dce` Record recovered correction build and remaining source work (documentation).
6. 2026-09-10 `f4a1f0d` Add multiplatform runtime adapters and release packaging.
7. 2026-09-10 `8453c49` Correct native path aliases and release notice packaging.

Phases visible in the history and docs: the initial port, then a post-plan parity-correction round (the punch list), several CPU-renderer optimisation experiments with measured keep/reject decisions (recorded in `docs/STATUS.md` and `docs/RECOVERY.md`), then multiplatform packaging.

## Stack and size

- Language: strict TypeScript. The project rules in `PORT_PLAN.md` forbid `any`, type assertions, non-null assertions, ignored diagnostics, JavaScript source, native addons, embedded C and a WASM engine. `tools/type-policy.ts` (`bun run policy`) enforces this.
- Runtime and build: Bun 1.3.14 (verified version, `engines` is >=1.3.14). `bun run build` produces a standalone executable that embeds Bun (the 2026-09-11 build was 106,641,536 bytes). TypeScript 5.9.3 is the only other dev dependency besides `@types/bun`.
- External platform dependencies: SDL2 for window, input and audio output; the system OpenGL driver for `--renderer gl`; optional FreeType for font generation.

Line counts (tracked files only, counted with `wc -l`, so blank lines and comments are included):

| Area | `.ts` files | Lines |
| --- | ---: | ---: |
| `src/` (engine and game) | 530 | 144,415 |
| `tests/` | 569 | 166,540 |
| `tools/` | 36 | 5,581 |
| All tracked `.ts` | 1,135 | 316,536 |

Other tracked data: 20 Markdown files (3,133 lines), 3 JSON files (20,581 lines, mostly `docs/source-inventory.json` and lock/config files), 2 TSV files (1,439 lines). There are zero tracked `.js` files. The repo has 1,166 tracked files in total. The tests outnumber the source lines, and `tests/` contains 569 TypeScript files.

Per-directory source size inside `src/`:

| Directory | Files | Lines |
| --- | ---: | ---: |
| `render` | 77 | 24,145 |
| `ui` | 76 | 19,971 |
| `game` | 51 | 18,775 |
| `botlib` | 42 | 18,372 |
| `cgame` | 34 | 11,239 |
| `engine` | 44 | 10,972 |
| `assets` | 26 | 8,722 |
| `vm` | 54 | 4,693 |
| `script` | 7 | 4,467 |
| `platform` | 28 | 4,634 |
| `core` | 27 | 4,728 |
| `server` | 20 | 3,154 |
| `collision` | 10 | 2,664 |
| `shared` | 11 | 2,400 |
| `protocol` | 11 | 2,307 |
| `audio` | 6 | 1,826 |
| `cinematic` | 5 | 958 |

## Architecture

`src/main.ts` is the single entry point and dispatches the subcommands `client`, `server`, `inspect` and `render`. The layout mirrors the original id Software modules, with their responsibilities mapped as follows (the module descriptions are from `PORT_PLAN.md` and `docs/ARCHITECTURE.md`; the original-to-target mapping is the plan's, not my inference):

- `src/core`, `src/engine`, `src/platform`: the qcommon equivalent. Common frame driver, console, cvars, event queue, system clock, Unix I/O and SDL adapters for windowing, input and audio.
- `src/assets`: virtual filesystem and file formats. PK3 (zip) reading with its own inflate, BSP, MD3, MD4, BMP/PCX/TGA/JPEG decoders, JPEG encoder (for screenshots), WAV, ADPCM-related loaders, animation, QVM bytecode files, and client download handling.
- `src/collision`, `src/shared`, `src/protocol`: BSP collision tracing, shared definitions and player movement, and the protocol-68 network format.
- `src/game`: the authoritative server-side game logic (the original `game` module), including the bot-facing game AI.
- `src/cgame`: client-side game: snapshots, prediction, entity effects, HUD. Publishes scenes that the renderer and audio consume.
- `src/ui`: menus for both products: the base-game menu set and the Team Arena menu and HUD widgets.
- `src/server`: connections, world links, game hosting and snapshot building, plus the server browser.
- `src/botlib`: the bot library: AAS file loading, area clustering, reachability, routing, movement, goals, weapons, chat and the script parser.
- `src/render`: both renderers. Material (shader) registration, BSP and patch handling, MD3 models, a CPU rasterizer with worker support, and the OpenGL backend through SDL2.
- `src/audio`: mixer, music streaming, ADPCM and wavelet decoding.
- `src/cinematic`: RoQ video and audio playback for the in-game cinematics.
- `src/script`, `src/vm`: the script lexer and preprocessor and a TypeScript QVM interpreter. `PORT_PLAN.md` describes the interpreter as optional, for mod compatibility after the direct TypeScript game modules work.

Design points recorded in `docs/ARCHITECTURE.md`: the engine composes owned instances rather than using global state; shared movement consumes explicit player state, commands and collision queries; cgame publishes scenes that rendering and audio consume; dedicated-server composition never initialises video.

Tools in `tools/`: `build.ts`, `release.ts`, `check.ts`, `type-policy.ts`, `inventory.ts`, `audit-assets.ts`.

## Features

What works, per the repo's README and status documents:

- Quake III Arena and Team Arena from one runtime, with menu-driven switching between them.
- Graphical client and dedicated server (`client` and `server` subcommands). The dedicated server uses retail data but does not initialise SDL video or audio.
- Bots, including the full botlib/AAS navigation path.
- Protocol-68 networking, server browser and demo recording/playback.
- Two renderers (CPU in TypeScript, OpenGL via SDL2), with video restart supported.
- Audio mixing, music, and cinematic playback.
- Release packaging for five targets: `linux-x64`, `linux-arm64`, `darwin-x64`, `darwin-arm64` and `windows-x64`. The five archives exist in the local `dist/releases` with a `checksums.txt`.
- Build-time override of the master and authorisation server endpoints (`--master-server`, `--auth-server`, `--auth-port`).
- Standalone libraries for IPX address routines and ranking codecs (257 ranking keys, 14 gameplay report bodies). The status doc says these are not an IPX transport or ranking service and are not new playable modes.
- A diagnostic `render` viewer (free camera and optional walking with BSP collision) and an `inspect` command for asset reports.

Verification: `bun run typecheck`, `bun run policy`, `bun test`, and `bun run check` (full run needs retail data, SDL2 and a working OpenGL driver). `bun run check:portable` skips retail coverage and the GL phase and does not certify a release.

## How to run and build

You need your own Quake III Arena retail data (and Team Arena data for that product). Point `--data` at the installation root that contains `baseq3/` (and `missionpack/`), not at `baseq3` itself. Use a separate writable `--home` directory.

Build from source:

```sh
bun install --frozen-lockfile
bun run build
```

This produces `dist/quake3-ts`. Do not install with `--production`; the build needs the dev dependencies.

Play with OpenGL:

```sh
./dist/quake3-ts client --data /path/to/Quake3 --home "$HOME/.local/share/quake3-ts" --product baseq3 --renderer gl -- +map q3dm1
```

For Team Arena use `--product missionpack` and `+map mpteam1`. For CPU rendering use `--renderer cpu --width 320 --height 240`. The `client` subcommand is required and engine `+commands` go after `--`. You can run from source with `bun run src/main.ts client ...` using the same arguments.

Dedicated server:

```sh
./dist/quake3-ts server --data /path/to/Quake3 --home "$HOME/.local/share/quake3-ts-server" --product baseq3 -- +set net_ip 127.0.0.1 +map q3dm1 +addbot Sarge 3
```

Install SDL2 first (for example `libsdl2-2.0-0 libgl1 libfreetype6` on Debian/Ubuntu, `sdl2-compat freetype2` on Arch, `brew install sdl2 freetype` on macOS). `docs/PLATFORMS.md` has the Windows steps and the unsigned-executable notes for macOS.

## Relationship to the other projects

This repo's own documents do not mention the other Quake projects. Going only by what is on disk, it sits next to the sibling TypeScript ports in `~/Projects` (`quake-1-ts`, `quake-2-ts`, the re-release ports and `quake-typescript`) and the Quake anthology work. Whether it shares code with them is not stated in this repo, and I did not verify it. Treat the relationship as "same author, same approach" until the README author fills this in.

## Notable lessons and decisions

- Strict type policy as a build gate. The "no `any`, no casts, no non-null assertions, no suppression comments" rule is machine-checked by `tools/type-policy.ts` and run during the build.
- Build-input integrity. The build copies its inputs, runs the type and policy checks on the copy, then confirms the live inputs have not changed before publishing the executable. It records source and binary hashes in `dist/build.json`.
- Documented deviations instead of silent ones. `docs/COMPATIBILITY.md` lists each place the port deliberately differs from undefined or unsafe behaviour in the C source (for example, a bounded shader `tcMod` copy buffer, a patch-stitching use-after-free handled by snapshotting LOD data, and a lightmap companion-image tail zero-filled where the original read past the allocation). `docs/decisions.tsv` and `docs/SOURCE_EXAMINATION.md` hold the source accounting.
- Evidence-based optimisation. `docs/STATUS.md` records each CPU renderer experiment with matched before/after medians, and says which were rejected and reverted (for example the mutable tess-cell experiment, the column-capture experiment and exact-input norm reuse). Some changes are kept with no isolated FPS claim.
- No retail assets in the repo or the executable; the port reads the user's installation directly.

## Credits

- Quake III Arena source code, Copyright 1999-2005 id Software, Inc. The project is an altered TypeScript translation and new implementation. It is not an official id Software release. The 1.32b source is the reference baseline.
- Independent JPEG Group release 6b: the JPEG decoder and encoder are altered translations of portions of it. The IJG README is in `licenses/IJG-README.txt`.
- The raw DEFLATE decoder (`src/assets/inflate.ts`) is an altered implementation of the zlib inflater bundled in `qcommon/unzip.c`, Copyright 1995-1998 Jean-loup Gailly and Mark Adler.
- The ADPCM code (`src/audio/adpcm.ts`) is based on the IMA Compatibility Project implementation, Copyright 1992 Stichting Mathematisch Centrum.
- `src/core/native-random.ts` is an altered translation of the GNU C Library default TYPE_3 random-number generator (see NOTICE.md).
- Other bundled notices: `licenses/Bun-LICENSE.md` (Bun runtime inside the executable) and `licenses/LGPL-2.1.txt`.
- Licence: the project code is GPL-2.0-or-later (`LICENSE`, `package.json`). Retail game assets remain separately licensed and must come from the user's own installation.
