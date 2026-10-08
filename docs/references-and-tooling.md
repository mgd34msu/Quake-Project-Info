# References and Tooling

This page covers the material Mike's Quake projects lean on but did not write: third-party engines and libraries, the id Software source releases, and the shared test and data area. None of this is a finished product of its own. It is the ground truth the ports are checked against.

All counts and dates below were read from the local git clones and files on 2026-10-07. Anything not measured is marked as an estimate or stated as unknown.

## Contents

1. [Summary table](#summary-table)
2. [yquake2 fork](#yquake2-fork)
3. [q2repro (Paril)](#q2repro-paril)
4. [q2proto (res2k)](#q2proto-res2k)
5. [quake2-rerelease-dll (id Software)](#quake2-rerelease-dll-id-software)
6. [quake-2-c (id Software, Quake 2 3.19)](#quake-2-c-id-software-quake-2-319)
7. [qsrc: the original-source collection](#qsrc-the-original-source-collection)
8. [qfiles: the shared test and data area](#qfiles-the-shared-test-and-data-area)
10. [How these pieces relate](#how-these-pieces-relate)
11. [Credits and licences](#credits-and-licences)

## Summary table

| Item | Local path | Upstream | Role | Mike's changes |
|---|---|---|---|---|
| yquake2 fork | none found on this machine | https://github.com/mgd34msu/yquake2 (fork of yquake2/yquake2) | Fork of the Yamagi Quake II client | None pushed as of 2026-10-07 (see below) |
| q2repro | `~/Projects/q2repro` | https://github.com/Paril/q2repro | Re-release-compatible Quake 2 engine, reference for Q2 re-release behaviour | None (plain clone) |
| q2proto | `~/Projects/q2proto` | https://github.com/res2k/q2proto | Quake 2 network protocol library | None (plain clone) |
| quake2-rerelease-dll | `~/Projects/quake2-rerelease-dll` | https://github.com/id-software/quake2-rerelease-dll | id's 2023 re-release game code | None (plain clone) |
| quake-2-c | `~/Projects/quake-2-c` | https://github.com/id-software/quake-2 | id's original Quake 2 3.19 source | None (plain clone) |
| qsrc | `~/Projects/qsrc` | many (listed below) | Collection of original and community engine sources | None (read-only reference) |
| qfiles | `~/Projects/qfiles` | not a repo | Shared test and data area, built ports | n/a |

## yquake2 fork

- **GitHub:** https://github.com/mgd34msu/yquake2
- **Upstream:** https://github.com/yquake2/yquake2, the Yamagi Quake II client (a Quake 2 client and server aimed at modern systems and the original game data).
- **Fork created:** 2026-08-14 (GitHub API `createdAt`). The last push date equals the creation date, so the fork carries no commits of its own. Its tip at inspection was an upstream merge (pull request #1349, "Strlist API and major code simplification and cleanup" by BjossiAlfreds).
- **Local clone:** none found under `~/Projects` or within four directory levels of the filesystem root at the time of writing.
- **Licence:** GitHub reports the licence as "other"; the yquake2 project documents its own terms in its repository. Check upstream before redistributing.


## q2repro (Paril)

- **Upstream:** https://github.com/Paril/q2repro, a fork of Q2PRO (https://github.com/skullernet/q2pro).
- **Local path:** `~/Projects/q2repro`; a second copy sits in `~/Projects/qsrc/q2repro` (same remote).
- **History:** 4,612 commits, first commit 2007-08-14 (inherited from Q2PRO history), last commit 2026-08-28 ("Add missing `q_gameabi` to wrap_trace()").
- **Build system:** Meson (`meson.build`, `meson_options.txt`), see `BUILDING.md`. Contents: `src/`, `inc/`, `doc/`, `man/`, plus a `q2proto` subdirectory that holds the protocol library.
- **What it is:** by its own README, a drop-in replacement for the Kex Quake II re-release engine. Its stated goals are full playability of re-release content, compatibility with existing Quake 2 network protocols, netplay with the native re-release game DLL, no installation needed beyond the executable, and re-release-matching defaults. The README rates several of these as complete and others as work in progress; read it for the current state.
- **Features it lists (inherited from Q2PRO):** unified OpenGL renderer, persistent console history, separation of rendering, physics and packet rates, `.pkz` packfiles, JPEG/PNG textures, MD3 and MD5 models, Ogg Vorbis and Theora, OpenAL sound, demo seeking and server-side multiview demos, protocol extensions for larger maps.
- **Features it lists as new:** weapon wheel and carousel (`+wheel`, `+wheel2`), reverb (`al_reverb`), per-pixel lighting (`gl_per_pixel_lighting`, work in progress).
- **Licence:** GNU GPL version 2 (`LICENSE`).
- **Credit:** Paril and the q2repro contributors; Q2PRO by skuller (Andrey Nazarov) and contributors; id Software for the Quake 2 code underneath.

## q2proto (res2k)

- **Upstream:** https://github.com/res2k/q2proto
- **Local path:** `~/Projects/q2proto`
- **History:** one commit in the clone, dated 2026-04-19 (a shallow or squashed snapshot; the real upstream history is longer but is not present locally).
- **What it is:** "A Quake 2 protocol abstraction library." It reads and writes Quake 2 network messages for several protocol variants through one interface. It is meant to be included directly in another project's build, as a Git submodule or by copying the source files. Directories: `inc/`, `src/`, `tests/`, `tools/`.
- **How the projects use it:** q2repro carries its own copy as `q2repro/q2proto`. For the ports it is a reference for how the different Q2 wire variants (classic, extended, re-release 2023) map onto one decode/encode layer, which is the same shape of problem the ports' "one engine" rule addresses for their network layers.
- **Licence:** `LICENSE` in the clone is a GNU GPL version 2 text. Confirm against upstream if redistributing.
- **Credit:** res2k.

## quake2-rerelease-dll (id Software)

- **Upstream:** https://github.com/id-software/quake2-rerelease-dll
- **Local path:** `~/Projects/quake2-rerelease-dll`; a second copy sits in `~/Projects/qsrc/quake2-rerelease-dll`.
- **History:** 2 commits, 2023-08-07 to 2023-10-03 (last: "Update 1 changes").
- **Layout:** `rerelease/` (the new combined game code), `original/` (the original game code kept for reference), `fgd/` (combined entity definitions), plus a TrenchBroom game config. The qsrc copy contains 92 `.cpp` files (count of `.cpp` only, headers not counted).
- **What it is:** the game code for the 2023 Quake II re-release, for modders. It merges the separate original game modules (baseq2, ctf, rogue, xatrix) into one C++17 codebase and introduces a thin client-game ("cgame") module in the manner of Quake III Arena, plus network protocol version 2023. The README notes it has only been tested with Clang, VS2019 and VS2022, and that old mods may need rewriting for the changed structure.
- **How the projects use it:** this is the authority for Quake 2 re-release rules (the 40 Hz tick versus classic 10 Hz is one of the differences that came up in playtesting). The ports load or reimplement its behaviour for the "Q2 rerelease" game families.
- **Licence:** `LICENSE.txt`, GNU GPL version 2 for the code. Game data stays under its original copyright.
- **Credit:** id Software and Nightdive Studios (the README refers to Nightdive for the save directory and Kex engine).

## quake-2-c (id Software, Quake 2 3.19)

- **Upstream:** https://github.com/id-software/quake-2
- **Local path:** `~/Projects/quake-2-c`; a second copy sits in `~/Projects/qsrc/quake-2`.
- **History:** 1 commit, 2012-01-31 ("The original Quake 2 sources as originally released under the GPL license on December 21, 2001").
- **Layout:** `client/`, `server/`, `game/`, `ctf/`, `qcommon/`, `ref_gl/`, `ref_soft/`, platform directories (`win32/`, `linux/`, `unix/`, `solaris/`, `irix/`, `rhapsody/`, `null/`), change logs `3.15_Changes.txt` through `3.18_changes.txt`, and Visual C++ 6.0 project files.
- **What it is:** the complete Quake 2 source at version 3.19, per `readme.txt` buildable with Visual C++ 6.0, with Linux "buildable but not tested for the release". It includes the `ctf` game module, which is the base that Quake 2 CTF mods, including LMCTF, grew from.
- **How the projects use it:** the primary source ("qsrc") for classic Quake 2 behaviour: game logic, physics, the wire protocol, the software and GL renderers.
- **Licence:** GPL, as stated in `readme.txt` and `gnu.txt`. The Q2 data files remain copyrighted and may not be redistributed.
- **Credit:** id Software.

## qsrc: the original-source collection

`~/Projects/qsrc` holds the reference source trees. Per the standing rule in Mike's notes, the ports derive behaviour from "qsrc" (the original source), not from the TypeScript port's structure. Names only; no file contents are reproduced.

| Directory | Upstream remote | What it is | Last commit seen |
|---|---|---|---|
| `quake` | github.com/id-software/quake | id's Quake 1 source: WinQuake, QuakeWorld QuakeC (`qw-qc`), `progs106`, Mission Packs, `QW` | 2012-01-31 |
| `quake-2` | github.com/id-software/quake-2 | id's Quake 2 source (same as `quake-2-c`) | 2012-01-31 |
| `quake-2-tools` | github.com/id-software/quake-2-tools | Quake 2 tools: `bsp`, `qdata`, `qe4`, `texpaint`, `Unpack` | 2012-01-31 |
| `quake-tools` | github.com/id-software/quake-tools | Quake 1 tools: `qcc`, `QuakeEd`, `qutils` | 2012-01-31 |
| `quake-iii-arena` | github.com/id-software/quake-iii-arena | Quake III Arena source: `code`, `q3asm`, `q3map`, `q3radiant`, `lcc`, `ui` | 2012-01-31 |
| `gtkradiant` | github.com/id-software/gtkradiant | GtkRadiant level editor source | 2012-01-31 |
| `quake-rerelease-qc` | github.com/id-software/quake-rerelease-qc | QuakeC for the Quake 1 re-release: base, CTF, Hipnotic, Rogue, MG1, MG3 | 2026-08-31 |
| `quake2-rerelease-dll` | github.com/id-software/quake2-rerelease-dll | See above | 2023-10-03 |
| `quakespasm` | github.com/sezero/quakespasm | QuakeSpasm, the maintained Quake 1 engine based on FitzQuake | 2026-08-11 |
| `ironwail` | github.com/andrei-drexler/ironwail | Ironwail, a QuakeSpasm derivative focused on speed | 2026-08-31 |
| `vkquake` | github.com/Novum/vkQuake | vkQuake, a Vulkan Quake 1 engine | 2026-09-05 |
| `fteqw` | not read (no remote queried) | FTEQW, the FTE QuakeWorld engine (engine, `fteqtv`, documentation, build scripts) | not read |
| `q2repro` | github.com/Paril/q2repro | See above | 2026-08-28 |
| `lmctf60` | github.com/qwazywabbitwos/lmctf60 | LMCTF 6.0 Quake 2 CTF mod source, the upstream of the LMCTF work | 2026-01-24 |
| `quake-source.zip` | n/a | 932-file archive (about 18.7 MB uncompressed) of a `quake/` tree including a `.git` entry, `gnu.txt`, Mission Pack and `progs106` directories; appears to be a snapshot of id's Quake 1 source | n/a |

Notes:

- `~/Projects/lmctf60` is a separate clone with remote https://github.com/mgd34msu/lmctf60 (last commit 2026-07-28). That is Mike's own LMCTF work and is covered in its own document; the `qsrc/lmctf60` copy is the unmodified upstream used for comparison.
- The id Software releases dated 2012-01-31 are single-commit imports of source originally released under the GPL in 1999 (Quake 1), 2001 (Quake 2), and 2005 (Quake III Arena).
- The community engines (QuakeSpasm, Ironwail, vkQuake, FTEQW) are used as behaviour references for Quake 1 engine conventions such as demo handling, console commands and cvar names, not as code to copy. The unified cvar table lists Q1 and QW as separate sources.

## qfiles: the shared test and data area

`~/Projects/qfiles` is not a repository. It is the working area every project shares for running things against real game data. This section describes roles only; no file contents are reproduced here, and the folder also holds private configuration that is not part of this documentation and is not published.

| Entry | Role |
|---|---|
| `q1`, `q1rets`, `q2`, `q3a`, `quakelive` | Installed game data trees used for test runs (Quake 1 and its mission packs/CTF/re-release; Quake 2 base, CTF, LMCTF, Rogue, Xatrix and re-release; Quake III Arena including a `quake3-ts` directory; Quake Live). Sizes on 2026-10-07 by `du`: q1 about 2.2 GB, q2 about 6.9 GB, q3a about 1.9 GB, quakelive about 1.1 GB. |
| `quake-typescript`, `quake-typescript.build.json`, `QUAKE-TYPESCRIPT.md`, `quake-typescript.README.md` | The built single-file executable of the unified TypeScript project and its build record and user guide. The guide describes menu-driven single and multiplayer setup, a console on backtick, and flags such as `--game`, `--map`, `--movement`, `--character`, `--mode`, `--rules`, `--bot-skill`, `--renderer`. |
| `qa-c`, `qa-rust` | Built output of the C port (quake-anthology) and the Rust port (Quake-Anthology-Rust), the binaries the playtests run. |
| `native-runtime` | A prebuilt native runtime directory (`linux-x86_64`, about 16 MB). |
| `old-executable-archive` | Dated previous builds of the TypeScript executable (timestamps from 2026-09-12 onward, about 905 MB total) kept for regression comparison. |
| `main.ts.map`, `worker-entry.ts.map` | Source maps for the TypeScript build. |
| `test.sh` | A one-line launcher that runs the executable with a fixed game, map, movement, character, mode, rules and bot skill. |
| `q1-paks.zip` | Archive of Quake 1 data files. |

Game assets themselves are commercial content. They are not in this repository and may not be redistributed.

## How these pieces relate

- **q2repro, q2proto, the re-release DLL, and the yquake2 fork** are reference implementations for Quake 2 variants: how an engine hosts the game library, how multiple wire protocols map to one interface, and what the re-release game does differently from 3.19.
- **qfiles** is where the TypeScript executable, the C and Rust ports and the game data live side by side so any of them can be run for comparison.
- **LMCTF** work builds on `qsrc/lmctf60` (Quake 2 CTF mod source) and the `quake-2-c` `ctf` module.

## Credits and licences

| Component | Author(s) | Licence |
|---|---|---|
| Quake 1, Quake 2, Quake III Arena, tools, GtkRadiant source | id Software | GPL (released 1999, 2001, 2005 and later); game data remains copyrighted |
| Quake II re-release game code | id Software, Nightdive Studios | GPL version 2 (code); data copyrighted |
| Quake 1 re-release QuakeC | id Software, Nightdive Studios and mission-pack authors as credited in that repo | see `COPYING.txt` in the repo |
| q2repro | Paril and contributors; Q2PRO by skuller and contributors | GPL version 2 |
| q2proto | res2k | GPL version 2 text in the clone; confirm upstream |
| yquake2 | Yamagi and the yquake2 contributors | see upstream repository |
| QuakeSpasm | Ozkan Sezer, Stevenaaus and contributors (FitzQuake by John Fitzgibbons) | GPL version 2 |
| Ironwail | Andrei Drexler | GPL version 2 |
| vkQuake | Axel Gneiting (Novum) and contributors | GPL version 2 |
| FTEQW | Spike (Forethought Entertainment) and contributors | GPL (per its build files; no licence file was read for this page) |
| LMCTF 6.0 | qwazywabbitwos (upstream repo) and the original LMCTF authors | see upstream repository |

