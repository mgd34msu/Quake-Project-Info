# Timeline

This page lists all of the Quake projects in date order. Dates come from `git log` in each repository, as recorded in the per-project docs in this folder. Tag dates are the dates on the git tags; where a changelog gives a different date, that is noted. Dates before 2020 come from READMEs and source headers, not from git.

## Project spans

| Project | First commit | Last commit | Commits | Doc |
|---|---|---|---:|---|
| lmctf60 (upstream LMCTF, Mike's copy) | 2020-03-09 | 2026-07-28 | 113 | [lmctf.md](lmctf.md) |
| LMCTF Buzzmod | 2020-03-09 (shared with upstream); Mike's first commit 2020-07-04 | 2026-09-03 | about 2,100 on `main` | [lmctf.md](lmctf.md) |
| q2lmstats | 2020 (README only) | 2026-07-28 | 8 | [lmctf.md](lmctf.md#q2lmstats) |
| yquake2 fork | fork created 2026-08-14 | none of its own | 0 | [references-and-tooling.md](references-and-tooling.md#yquake2-fork) |
| Quake-2-TS | 2026-08-29 | 2026-09-01 | 109 | [quake-2-ts.md](quake-2-ts.md) |
| Quake-2-Rerelease-TS | 2026-08-29 | 2026-09-08 | 276 | [quake-2-ts.md](quake-2-ts.md) |
| Quake-1-TS | 2026-09-04 | 2026-09-06 | 40 | [quake-1-ts.md](quake-1-ts.md) |
| Quake-1-Rerelease-TS | 2026-09-06 | 2026-09-08 | 140 | [quake-1-ts.md](quake-1-ts.md#quake-1-rerelease-ts) |
| Quake-3-TS | 2026-09-09 | 2026-09-10 | 7 | [quake-3-ts.md](quake-3-ts.md) |
| Quake-Anthology-TS | 2026-09-10 | 2026-09-26 | 805 | [quake-anthology-ts.md](quake-anthology-ts.md) |
| Quake-Anthology (C) | 2026-09-26 | 2026-10-07 | 2,119 | [quake-anthology-c.md](quake-anthology-c.md) |
| Quake-Anthology-Rust | 2026-09-29 | 2026-10-07 | 1,007 | [quake-anthology-rust.md](quake-anthology-rust.md) |

## Background (before Mike's work)

- **1996 to 1999**: id Software releases Quake (1996), Quake II (1997) and Quake III Arena (1999).
- **1999**: LM_CTF 5.2 Tournament Edition by Loki's Minions. "Loki's Missions - Escape", a Quake II single-player mission from the same group, is dated 1999-08-13.
- **1999-12-21**: the Quake, QuakeWorld and GLQuake source is released under GPL v2.
- **2001-12-21**: the Quake II source is released under the GPL.
- **2005**: the Quake III Arena source is released under the GPL.
- **2012-01-31**: id's source releases are imported to GitHub, one commit each. These are the copies kept in `qsrc`.
- **2023-08-07 to 2023-10-03**: id publishes `quake2-rerelease-dll`, the 2023 Quake II rerelease game code (2 commits).

## 2020: LMCTF and the first Buzzmod work

- **2020-03-09**: the first commit of the modernised LMCTF source ("State of batstone code with upgrade to VS2019"). QwazyWabbit made 101 of the 113 upstream commits.
- **2020-07-04**: the first Buzzmod commits, which log stats for runes, quad, armor and shield. Mike's `lmctf60` repository on GitHub was created the same day.
- **2020-07-16**: the `toss rune` command is added and stats are shown on the scoreboard.
- **2020-08 to 2020-09**: q2lmstats gets its first two commits (README only).
- **2020-09-12**: runes now drop when a player disconnects. Upstream tags `v6.2` the same day.
- After this the Buzzmod work sat on a backup drive. Three of its SQLite source files were never committed and were later lost.

## 2026-07 to 2026-08: LMCTF Buzzmod revived

- **2026-07-28**: the 2020 work is recovered from the backup drive and committed. The lost SQLite layer is rewritten from scratch. The same day the fork is merged with current upstream, stats crashes are fixed, and these are added: capture streaks and sweeps, `sv statsdb`, `cmd lifetime`, `cmd rank`, per-match history, and CI for Linux and Windows. The fork is renamed "LMCTF Buzzmod" and tagged `release-1` and `release-2`. q2lmstats is rewritten against the real database schema. This is also the last commit in Mike's `lmctf60` clone.
- **2026-07-30**: `release-3` previews bots built on an imported bot library.
- **2026-07-31**: `toss flag` is fixed. It had been a no-op because the flag's toss slot was never filled in the item table.
- **2026-07-30 to 2026-08-11**: native SLIPGATE bots replace the imported library, and RUNE route generation is added.
- **2026-08-11**: `release-4`. The "RELEASE PREP" commit makes the tuned bot cvars the built-in defaults.
- **2026-08-14**: the yquake2 fork is created on GitHub.
- **2026-08-15**: `release-5` and `release-6` (the HOOK milestone).
- August 2026 has 1,730 Buzzmod commits, the busiest month in that repository.

## 2026-08-29 to 2026-09-10: per-game TypeScript ports

- **2026-08-29**: the first commit of Quake-2-TS. On the same day it reaches a running dedicated server and is tagged `v1.0.0` and `v1.1.0`. Quake-2-Rerelease-TS is seeded from Quake-2-TS v1.1.0, and its `ARCHITECTURE.md` (one engine, KEX and legacy game bindings) is written.
- **2026-08-30**: Quake-2-Rerelease-TS adds the protocol codec layer and a file-by-file port of the 2023 rerelease game module, and starts splitscreen and the GL shader pipeline.
- **2026-08-31 to 2026-09-01**: Quake-2-Rerelease-TS adds LMCTF hosting, image formats and shadow maps. It is tagged `v1.0.0` and `v1.0.1` (2026-09-01).
- **2026-09-01**: Quake-2-TS `v1.2.0` (video modes up to 3840x2160, QBSP extended maps) and `v1.2.1`. This is its last commit.
- **2026-09-02**: LMCTF Buzzmod `v1.0.0`, the first release with SLIPGATE bots.
- **2026-09-03**: Quake-2-Rerelease-TS `v1.1.0` (the changelog dates it 2026-09-02), which makes any content playable under any ruleset. Buzzmod's last commit, which documents SLIPGATE's decision levels.
- **2026-09-04**: the first commit of Quake-1-TS ("Scaffold: package, tsconfig, check gate, PORTING.md contract, ledger"). The foundation work and the QuakeC VM are ported.
- **2026-09-05**: Quake-1-TS gets its client and both renderers, then QuakeWorld, and finally plays on retail data with all three binaries. Its CHANGELOG dates 1.0.0 to this day.
- **2026-09-06**: Quake-1-TS is tagged `v1.0.0` on its release commit, which is also its last commit. Quake-1-Rerelease-TS is seeded from it (commit `86c6867`) and makes 113 commits that day. Quake-2-Rerelease-TS is tagged `v1.1.1` (bots).
- **2026-09-07 to 2026-09-08**: CTF bot work in Quake-1-Rerelease-TS. The bot brain improvements are copied back into Quake-2-Rerelease-TS ("Unreleased" in its changelog).
- **2026-09-08**: Quake-1-Rerelease-TS `v1.0.0` and `v1.0.1`. This is the last commit in both rerelease ports.
- **2026-09-09**: the first commit of Quake-3-TS, which imports a finished port: 1,084 files and 326,957 lines in one commit.
- **2026-09-10**: Quake-3-TS gets parity corrections and multiplatform release packaging, and makes its last commit (23:44).

## 2026-09-10 to 2026-09-26: Quake-Anthology-TS

- **2026-09-10**: an empty initial commit. The source assessment, the project plan and the dependency graph come before any code.
- **2026-09-11**: bulk integration. One commit adds about 131,600 lines from the three source ports, and the x86/x64 interpreter for Quake II mod DLLs is added. This is 86 commits.
- **2026-09-12**: the busiest day, with 146 commits.
- **2026-09-12 to 2026-09-16**: 483 commits of integration and repair across subsystems.
- **2026-09-17 to 2026-09-24**: user documentation and fixes found in playtesting (116 commits).
- **2026-09-25**: 111 commits on the speed of the DLL interpreter.
- **2026-09-26**: `v1.0.0` is released as a Linux x64 executable. 22 of the 23 functional targets are accepted. The open one, T10, is about interpreter speed.

## 2026-09-26 to 2026-10-07: native ports

- **2026-09-26**: the first commit of the C port, Quake-Anthology (16:13 CDT). Like Quake-Anthology-TS, it starts with design documents: `docs/review.md`, `docs/plan.md` and the dependency graph. The review measured the emulated Quake II rerelease game code at a 48 ms median per tick, against a 25 ms budget. 86 commits.
- **2026-09-26 to 2026-09-30**: the C baseline is ported in numbered batches (B00 to B29 and later).
- **2026-09-29**: the first commit of the Rust port ("pre-parallel baseline"), developed in parallel `lane/*` branches. A single C commit adds 73,957 lines across 530 files.
- **2026-09-30 to 2026-10-04**: the C port completes native Q1, Q2 and Q3 gameplay, mission packs, menus, bots, demos and the shared save codec. Its busiest days before 10-05 were 09-30 (284 commits) and 10-03 (387).
- **2026-10-04 onward**: Mike's playtest reports (MIKE-01 to MIKE-37) drive the C port. Issues move to Linear (THE-NN).
- **2026-10-05**: the C port's busiest day, with 424 commits. Mike sets the rules for saves.
- **2026-10-06**: the plan to unify the C port's per-game code (325 items) and the unified cvar table (1,260 canonical cvars, 237 aliases). The input recording and replay code is listed as a release blocker. The first Rust tree ends at `eb9118cc`.
- **2026-10-07**: from 15:30 to 15:51 the Rust `main` is restarted: `c592b337` deletes 1,663,818 lines, and 15 more commits build the R0 foundation (workspace, primitives, SDL shell, format readers, rule checker, CI). The C port's playtest fixes continue, and its last commit at the time of writing is at 15:41 CDT.

## Notes on the dates

- Quake-3-TS and Quake-Anthology-Rust both begin with a baseline commit, so any earlier history is not in those repositories.
- Some GitHub repositories were created after their first local commit, because work was pushed later. Quake-Anthology was created on GitHub on 2026-09-28 and Quake-Anthology-Rust on 2026-10-02.
- Times quoted here are as recorded in the docs (Quake-3-TS and the C port record -0500 / CDT).
