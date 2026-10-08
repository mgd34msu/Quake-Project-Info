# Quake 1 in TypeScript: Quake-1-TS and Quake-1-Rerelease-TS

Two related projects, both TypeScript on [Bun](https://bun.sh). **Quake-1-TS** is a file-by-file port of id Software's 1999 GPL Quake sources (WinQuake 1.09 and QuakeWorld 2.33) that aims to behave like the original. **Quake-1-Rerelease-TS** starts from a copy of Quake-1-TS v1.0.0 and turns it into one engine for the 2021 Quake re-release, adding the extended network protocols, re-release content support, bots and a unified NetQuake/QuakeWorld binary. Neither ships game data.

Contents: [Quake-1-TS](#quake-1-ts) | [Quake-1-Rerelease-TS](#quake-1-rerelease-ts) | [How the two relate](#how-the-two-projects-relate)

---

## Quake-1-TS

### Summary

A port of id's GPL Quake sources to strict TypeScript on Bun. It covers WinQuake 1.09 with both the software renderer and the OpenGL renderer, plus the QuakeWorld 2.33 client and server. Each `.c` file became one `.ts` module, under strict TypeScript with no `any` and no casts. Game logic still runs as `progs.dat` bytecode through a ported QuakeC virtual machine, so the base game, both mission packs and classic mods load from their original data. It builds three executables: `q1ts` (single player and NetQuake), `qwsv` (QuakeWorld server) and `qwcl` (QuakeWorld client).

### Links

- GitHub: https://github.com/mgd34msu/Quake-1-TS
- Local path: `~/Projects/quake-1-ts`
- Release: tag `v1.0.0`; `package.json` version 1.0.0

### Status

Playable, with the caveat that only Linux has been exercised. The README states it was verified against retail data with both renderers:

- Full single-player campaign: all 38 base maps, plus the Scourge of Armagon (`hipnotic`) and Dissolution of Eternity (`rogue`) mission packs.
- Save and load, demo playback and recording, `timedemo`.
- Console, menus, key bindings, `config.cfg` round-trip, video modes, and `vid_restart` switching between the renderers mid-game.
- Sound, and `music/NN.ogg` playback in place of CD tracks.
- NetQuake multiplayer over real UDP (listen and dedicated server).
- QuakeWorld: `qwsv` serving two real `qwcl` clients, covering frags, map changes, spectators, demos, `rcon` and bans.

Windows and macOS binaries cross-compile but are untested. The README gives a gate of `bun run check` (`tsc --noEmit` plus a grep that fails on any `any`) and `bun test` (the README states 1817 tests, 0 failures; I did not run them).

### Timeline

- First commit: 2026-09-04 ("Scaffold: package, tsconfig, check gate, PORTING.md contract, ledger").
- Last commit: 2026-09-06. Total: 40 commits, all within three days.
- CHANGELOG dates release 1.0.0 as 2026-09-05; the `v1.0.0` tag sits on the 2026-09-06 release commit ("Release 1.0.0: cross-platform runtime, public documentation, release tooling").

Phases visible in the git log:

1. 2026-09-04, foundation: scaffold and `PORTING.md` contract, then math, protocol and file-format headers, `common`, `cmd`, `cvar`, `crc`, `zone`, `wad`, the shared model loader, the networking layer (`net`, `net_main`, `net_loop`, `net_dgrm`, `net_vcr`), progs types, the QuakeC VM (`pr_edict`, `pr_exec`), server, `sv_phys`, `sv_move`, `sv_user`. A dedicated server boot was an early milestone.
2. 2026-09-04 to 09-05, client and renderers: client modules and sound stack, then the software renderer wave, then the OpenGL renderer wave.
3. 2026-09-05, QuakeWorld: common layer (protocol, netchan, MD4, pmove), then the server wave, then the client wave. Headless `qwsv` booted retail `qwprogs.dat`; then `qwcl` booted and all three binaries built.
4. 2026-09-05, real data and desktop: "retail paks load, all three binaries play on id's data", then end-to-end fixes (engine strings, UDP over libc, mid-game `vid_restart`, mouse capture, window resize), an aliasing audit, and QuakeWorld movement fidelity regression tests.
5. 2026-09-06, release: cross-platform runtime, public documentation, release tooling, CI that skips suites needing game data.

### Stack and size

Counted with `wc -l` over `*.ts` files (no tokei or cloc installed). Lines include comments and blanks.

| Directory | Files | Lines |
|---|---|---|
| `src/qw` (QuakeWorld client and server) | 47 | 30,379 |
| `src/ref_soft` (software renderer) | 36 | 14,053 |
| `src/client` | 23 | 14,193 |
| `src/common` | 22 | 12,440 |
| `src/ref_gl` (OpenGL renderer) | 17 | 10,188 |
| `src/platform` | 11 | 5,099 |
| `src/server` | 6 | 4,436 |
| `src/progs` (QuakeC VM) | 6 | 4,345 |
| **`src` total** | | **95,383** |
| `test` | 201 | 62,119 |

Plus `src/main.ts` and a few shell scripts in `scripts/` (`check.sh`, `release-build.sh`, `sandbox-verify.sh`). Runtime: Bun 1.3 or newer. SDL2 handles windowing, input and audio through `bun:ffi`; OpenGL (`libGL.so.1`) and optional libvorbisfile are also loaded through FFI. `qwsv` needs no SDL.

### Architecture

- `src/common`, `src/server`, `src/progs`: the shared engine core. Commands, cvars, file system, model loader, host loop, the server (`sv_main`, `sv_phys`, `sv_move`, `sv_user`, `world`), and the QuakeC VM (`pr_edict`, `pr_exec`, `pr_cmds`). `string_t` values are kept in an engine string table; `ED_Write` output is byte-identical to glibc `printf` and savegames round-trip byte-exact (per CHANGELOG).
- `src/client`: `cl_main`, `cl_parse`, `cl_input`, `cl_tent`, `cl_demo`, console, keys, menu, status bar, screen, view, particles, sound.
- `src/ref_soft` and `src/ref_gl`: the two renderers behind one renderer seam. The software renderer ports the `r_*` and `d_*` C files; the `.s` assembly paths are not ported (the `nonintel.c` C paths are used). The GL renderer ports the `gl_*` files over a table of GL functions bound from `libGL.so.1` via SDL's `GetProcAddress`.
- `src/qw`: the QuakeWorld tree, with its own `common`, `cmd`, `cvar`, `net_chan`, `md4`, `pmove`, `client/` and `server/` subdirectories, mirroring id's `QW/` layout.
- `src/platform`: the SDL, sockets (UDP) and system layer that replaces the DOS, VESA, SVGAlib and Windows-native files.
- Entry points: `q1ts/`, `qwsv/`, `qwcl/` at the repo root build the three executables with `bun build --compile`.
- `PORTING.md` is the contract: file-by-file mapping to the C, type discipline, the renderer seam, how the VM and string table work, and every documented deviation.

### Features

- Both renderers compiled in, chosen at run time. This is the one user-facing addition: a `vid_ref` cvar (`soft` or `gl`), since `GLQUAKE` is a compile-time define in the C. `-vid_ref` on the command line, the video menu, or `vid_ref gl` plus `vid_restart` all work.
- Mission packs need no engine code: they are QuakeC data plus the `-hipnotic` / `-rogue` handling WinQuake already has. `-game <dir>` loads other mods.
- NetQuake listen and dedicated servers (`-listen`, `-dedicated`), QuakeWorld server and client with downloads of maps, models, sounds and skins on connect.
- Real retail pak loading with case-insensitive path resolution (`Sys_ResolveCase`), so a mounted CD or unmodified install works as `-basedir`.
- Two C bugs from the original are kept on purpose in the net code (`VCR_Listen` is never installed; `VCR_GetMessage` bug), per CHANGELOG.

### How to run and build

```sh
bun install
bun src/main.ts -basedir /path/to/quake                      # software renderer
bun src/main.ts -basedir /path/to/quake -vid_ref gl          # OpenGL renderer
bun src/main.ts -basedir /path/to/quake -hipnotic            # mission pack
bun run start:qwsv -- -basedir /path/to/quake -port 27500 +map start
bun run start:qwcl -- -basedir /path/to/quake
bun run build            # ./q1ts   (also build:qwsv, build:qwcl, build:all)
bun run build:release    # cross-compile release set into dist/
```

Game data must be supplied by the user (`id1/pak0.pak`, and `pak1.pak` for registered data; `qw/qwprogs.dat` for QuakeWorld). Per-OS packages are listed in `docs/PLATFORMS.md` in that repo.

### Relationship to other projects

- Direct parent of Quake-1-Rerelease-TS (below), which was seeded from its v1.0.0.
- `PORTING.md` says its conventions were inherited from a sibling Quake 2 port (`quake-2-ts`, "not part of this repository") and that the QGL function table for the OpenGL renderer came from it (commit "U070 ... qgl table from q2ts"). It is provenance only; nothing in this repo depends on it.
- `PORTING.md` records that the original plan ruled the re-release out of scope for this repo, which is why the re-release work lives in a separate repo.

### Lessons and decisions

- One module per C file, strict types, zero `any`, enforced by a grep in `bun run check`. Struct sizes are asserted against the C `sizeof`.
- Run the real QuakeC bytecode rather than transliterating game logic; this is what lets mission packs and mods work unchanged.
- `Hunk_*` returns exactly the requested size; the C's 16-byte rounding applies only to mark counters, because TypedArray length is observable (CHANGELOG).
- Not ported, by ruling of 2026-09-03: assembly files, DOS/VESA/Sun/serial/IPX platform code, `QW/qwfwd`, the vendored SDKs, and the mission-pack tool sources.
- Hostnames resolve only to `localhost`, because Bun has no synchronous DNS and the C call sites are synchronous; connect by IP.
- Behaviour that looks wrong but matches the original is documented in the README's "Faithfulness notes" (for example, `quit` in-game opens the confirm menu; QuakeWorld's client port is the fixed 27001; GL and software renderers treat fullbright texels differently).

### Credits and licence

Quake, QuakeWorld and GLQuake engine source, Copyright (C) 1996-1997 id Software, Inc., released 21 December 1999 under GPL v2. The port is a derivative work under GPL-2.0-or-later (`LICENSE` in repo). Game data is not included and stays under its own terms. Quake is a registered trademark of id Software LLC; the project is not affiliated with id Software, ZeniMax or Bethesda.

---

## Quake-1-Rerelease-TS

### Summary

A TypeScript engine for the 2021 Quake re-release, on Bun. One engine plays the classic game, both mission packs and the re-release campaigns (Dimension of the Past, Dimension of the Machine, Dawn of the Machine, re-release CTF) under either a classic or re-release ruleset. It speaks NetQuake protocols 15, 666 and 999 and QuakeWorld protocol 28, plus its own wide QuakeWorld extension (protocol 29), and has software and OpenGL renderers. It builds to one binary, `q1rets`, where NetQuake versus QuakeWorld and client versus dedicated server are chosen per connection and per command line.

The 2021 re-release shipped QuakeC only (the KEX engine is closed), so the game code already runs as bytecode through the VM inherited from Quake-1-TS. The work is engine extension, specified from the re-release QuakeC, the retail re-release data, and the GPLv2 engines Ironwail, vkQuake and QuakeSpasm (per `ARCHITECTURE.md`).

### Links

- GitHub: https://github.com/mgd34msu/Quake-1-Rerelease-TS
- Local path: `~/Projects/quake-1-re-ts`
- Releases: tags `v1.0.0` and `v1.0.1`; `package.json` version 1.0.1

### Status

Playable, explicitly a work in progress. The README says the engine is played and tested daily and still has bugs, with the bot AI, menus, software renderer true-colour path and QuakeWorld side named as rough. Known gaps stated in the README:

- The `-qw` boot is a QuakeWorld client only; a QuakeWorld listen server is reached from the default boot with `sv_profile qw` then `map`. Merging the two boots is in flight.
- Menus are keyboard-only; mouse-driven menus have not landed.
- Connecting to a real 2021 KEX server is out of scope (KEX netcode is undocumented). Interop is tested between this engine's own binaries and by byte-vector tests against the three open engines.
- Windows and macOS builds are untested.
- Open items: a QuakeC union-progs compiler and a final byte-vector regate (phase 8).

### Timeline

- First commit: 2026-09-06 ("seed from Quake-1-TS v1.0.0 (86c6867)").
- Last commit: 2026-09-08. Total: 140 commits (113 on 09-06, 21 on 09-07, 6 on 09-08).
- `v1.0.0` and `v1.0.1` are both dated 2026-09-08 in the CHANGELOG. 1.0.1 fixed a view roll that stuck after respawns and hits, and moved to one shared `config.cfg` per user.

Phases from the log and `ARCHITECTURE.md` (design written 2026-09-06, with a phase plan in which each phase ships something playable):

1. Seed and architecture: copy of Quake-1-TS v1.0.0, then the design contract and rulings.
2. Libraries lifted from the Quake 2 re-release port (`quake-2-re-ts`): zip, PNG, JPG, TGA, TTF, kfont, localization (commit "U7"), an MD5 mesh/anim loader (U13).
3. Protocol and limits: codec seam for NetQuake 15, FitzQuake 666 and RMQ 999 (U3); 8192 models, textures of any size (U14).
4. Content and UI: menus driven from `mapdb.json`, episodes, rulesets, add-ons, autosave (U17); SDL game controllers and `.bnvib` haptics (U22).
5. Interpolation, rendering and bots: pose, movement and lightstyle interpolation in both renderers (U16, U39); fog as a software depth post-pass (U27); renderer seam additions (U44); game-agnostic bot brain with NAV2 pathing (U20, U37).
6. End-to-end test families `e2e` a through z (protocol pairings, every retail demo on both renderers, stability soak with the compiled binary).
7. Play fixes 2026-09-07/08: QuakeWorld connect from the default boot, UI scaling, and a long run of CTF bot work (flag carrying, lava avoidance). The CHANGELOG gives measured numbers, for example ctf8 lava deaths per seed going from 11/9/7 to 6/1/0.

### Stack and size

Counted with `wc -l` over `*.ts` files. Lines include comments and blanks. `ARCHITECTURE.md` says the repo is 99.9% TypeScript with a couple of shell scripts.

| Directory | Files | Lines |
|---|---|---|
| `src/qw` | 47 | 29,467 |
| `src/client` | 29 | 20,126 |
| `src/common` | 34 | 18,614 |
| `src/ref_soft` | 39 | 17,574 |
| `src/ref_gl` | 20 | 12,681 |
| `src/lib` (codecs, fonts, MD5, NAV2, etc.) | 27 | 12,294 |
| `src/platform` | 14 | 7,469 |
| `src/progs` | 19 | 7,029 |
| `src/server` | 7 | 5,215 |
| `src/bots` | 5 | 1,911 |
| **`src` total** | 242 | **132,961** |
| `test` | 326 | 123,145 |

Compared with the seed, `src` grew from 95,383 to 132,961 lines and `test` from 62,119 to 123,145 (a figure computed from the two counts above, not a diff). Runtime: Bun and SDL2 via `bun:ffi`, same as the seed.

### Architecture

Inherited from Quake-1-TS: `common`, `client`, `server`, `progs`, `qw`, `ref_soft`, `ref_gl`, `platform`. New or heavily extended:

- `src/lib`: lifted and new format code: zip (for `QuakeEX.kpf`), PNG, JPG, TGA, TTF and kfont, localization, MD5 meshes and animations, NAV2 navmesh reader.
- `src/bots`: the bot brain. The re-release `bot.qc` is two empty hooks because the bot AI lives in the engine; this directory implements it, using the re-release `characters.txt`, `weapons.txt`, `items.txt` and NAV2 navigation. The brain is written to be game-agnostic and shared with the Quake 2 re-release port's bot binding.
- Protocol codec seam in `src/common`, so protocol 15 stays byte-identical to the seed while 666, 999 and QuakeWorld 29 plug in alongside.
- One QuakeC VM with two host profiles (NetQuake and QuakeWorld). The re-release progs run with 18 name-bound `ex_*` builtins, `checkextension`, localized prints, `MOVETYPE_GIB`, `SOLID_CORPSE`, the extra opcodes, prompts and `setcolor`.
- `sv_ruleset classic|rerelease|auto`: a behaviour profile; `auto` detects the re-release from the loaded `progs.dat`.
- `ARCHITECTURE.md` (design contract and phase plan), `PORTING.md` (inherited conventions), `CHANGELOG.md`.

### Features

From the README:

- Protocols: NetQuake 15, 666 (FitzQuake) and 999 (RMQ); QuakeWorld 28; QuakeWorld 29, which is this engine's own wide extension with no other reference implementation, negotiated only when a client sends `*wide 1`.
- Wide limits: `max_edicts` up to 32000, 8192 models, 2048 sounds; BSP2 and 2PSB maps, `.lit` coloured light, BSPX directory, external `.ent` files and texture wads, textures of any size.
- Content roots: nested `rerelease/` detection, `QuakeEX.kpf` mounting, `-mg1 -mg3 -dopa -ctf`, and a runtime `game` command. Writes go to a per-user directory (`~/.local/share/q1rets` by default; `-homedir`, `-nohomedir`), so the retail install is only read.
- Unified client and server: `-dedicated` is NetQuake, `-dedicated -qw` is a QuakeWorld server, `-qw` alone is a QuakeWorld client. A connect address with a port means QuakeWorld, without means NetQuake (`cl_protocol` overrides).
- Local splitscreen for two to four players (`cl_splitscreen`), each seat its own loopback client.
- Multiplayer menu (start server with ruleset, bots, skill, protocol; join screen; CTF team selection).
- Bots: `addbot`, `bot_count`, `bot_skill`, NAV2 pathing, CTF behaviour.
- Fixed-step server clock under the re-release ruleset (`sv_tickrate`, default 72) with interpolated rendering (`r_lerpmodels`, `r_lerpmove`).
- MD5 skeletal replacement models (`r_enhancedmodels`) in both renderers; fog and skyboxes in both; coloured lightmaps in both; water/lava/slime/teleporter alpha; anisotropy (GL).
- Text: kfont/TTF rendering from `QuakeEX.kpf`, Unicode glyph table, separate console, status bar, crosshair and menu scale, `$key` localization, `language auto` from the system locale.
- KEX-format savegames and autosave; SDL game controllers with the re-release mappings and `.bnvib` haptics; menus driven by `mapdb.json`; a compat spawn table so re-release maps load under classic progs.
- Additions beyond the seed that are documented as such: `-clientport`, `cl_execonspawn`, `cfg_version` with `cfg_migrate`, `scr_menuscale`.

### How to run and build

```sh
bun install
bun src/main.ts -basedir /path/to/quake                          # NetQuake, software renderer
bun src/main.ts -basedir /path/to/quake -vid_ref gl              # OpenGL renderer
bun src/main.ts -basedir /path/to/quake -dedicated 8 +map e1m1   # NetQuake dedicated server
bun src/main.ts -basedir /path/to/quake -qw                      # QuakeWorld client
bun src/main.ts -basedir /path/to/quake -dedicated -qw +map start
bun run check && bun test                                        # gates; no game data needed
bun run build                  # ./q1rets
bun run build:release          # four release targets, each zipped
bash scripts/release-build.sh --aliases linux-x64   # also copy binary as q1ts/qwsv/qwcl
```

`q1ts`, `qwsv` and `qwcl` no longer exist as separate build targets (ruling R4 in `ARCHITECTURE.md`); the alias option only copies the one binary under the old names.

### Relationship to other projects

- Seeded from Quake-1-TS v1.0.0 (commit 86c6867) and transformed in place; the classic game is kept playable at each step.
- Borrows libraries and process from Quake-2-Re-release-TS (`quake-2-re-ts`), "the seed-and-transform process that built quake-2-re-ts". The bot brain is shared in design with that project.
- Uses Ironwail, vkQuake and QuakeSpasm (GPLv2) as references. FTEQW is excluded as a reference by the charter.

### Lessons and decisions

- The re-release is QuakeC-only, so the right shape is an engine that extends the existing VM, not a rewrite. Where the open engines disagree with the re-release QuakeC's stated intent, the QuakeC wins (for example `MOVETYPE_GIB`).
- Quality-of-life changes are allowed and recorded as additions, unlike the seed which was strict about fidelity.
- Protocol 15 stays byte-identical to the seed; `sv_protocol auto` never picks 15 unless asked, and refuses (rather than misplaces entities) a map it cannot address.
- Bot changes are measured. Per README, numbers for each change are in the commit messages.
- A single shared `config.cfg` per user replaced per-game-directory configs after bindings were lost on every game switch.
- A large share of the 140 commits are end-to-end test families driving the compiled binary, not only unit tests.

### Credits and licence

id Software and ZeniMax Media: Quake (1996), the 1999 GPL engine, and the 2021 re-release QuakeC ([quake-rerelease-qc](https://github.com/id-Software/quake-rerelease-qc)), GPLv2. Ironwail (Andrei Drexler), vkQuake (Novum and contributors) and QuakeSpasm (Ozkan Sezer and contributors) are GPLv2 reference engines. Licence: GPL v2 (`LICENSE`). Game data is not included. Not affiliated with id Software, ZeniMax or Bethesda.

---

## How the two projects relate

| | Quake-1-TS | Quake-1-Rerelease-TS |
|---|---|---|
| Goal | Faithful port of 1999 GPL source | Engine for the 2021 re-release, with additions |
| Binaries | `q1ts`, `qwsv`, `qwcl` | `q1rets` only |
| Protocols | NetQuake 15, QuakeWorld 28 | NetQuake 15/666/999, QuakeWorld 28/29 |
| Commits | 40 (2026-09-04 to 09-06) | 140 (2026-09-06 to 09-08) |
| `src` lines | 95,383 | 132,961 |
| Version | 1.0.0 | 1.0.1 |
