# LMCTF and LMCTF Buzzmod (Quake II Capture the Flag)

LMCTF (LM_CTF, "Loki's Minions CTF") is a 1999 Capture the Flag modification for Quake II. Mike maintains a fork of it, LMCTF Buzzmod, which adds persistent player statistics, rune and flag tossing, and (from July 2026) a set of native bots called SLIPGATE. A small web site and JSON API, q2lmstats, reads the statistics the game server records. Alongside these sits an archive of two Loki's Missions single-player packs (Escape for Quake II, The Xeno Project 2 for Half-Life) that come from the same group as LMCTF; they are not Mike's work.

This page covers four things on disk:

| Project | Local path | GitHub | What it is |
|---|---|---|---|
| LM_CTF 6.0 (upstream copy) | `~/Projects/lmctf60` | https://github.com/mgd34msu/lmctf60 | Mike's copy of QwazyWabbit's modernised LMCTF source; reference baseline |
| LMCTF Buzzmod | `~/Projects/lmctf6-stats` | https://github.com/mgd34msu/lmctf60-buzzmod | Mike's fork: stats, tossing, SLIPGATE bots |
| q2lmstats | `~/Projects/q2lmstats` | https://github.com/mgd34msu/q2lmstats | Node.js stats site and API over the Buzzmod database |
| Loki's Missions archive | `~/Projects/lm-singleplayer` | none (no git) | Escape and The Xeno Project 2 packs plus zips |

## Status

- LMCTF Buzzmod: playable and released. Tagged `release-1` to `release-6` (2026-07-28 to 2026-08-15) and `v1.0.0` (2026-09-02). Under active development; the bot subsystem (SLIPGATE) has an ongoing rewrite on branch `era4`, which is development work and not a release.
- q2lmstats: working, small. Version 2.0.0 of `package.json`; rewritten 2026-07-28.
- lmctf60: reference. It is the upstream-style baseline Mike's fork started from; its last commit is a merge of a pull request from QwazyWabbitWOS on 2026-07-28.
- lm-singleplayer: archive of third-party downloads, kept for reference. It was not run or tested for this write-up.

## What LMCTF is

Capture the Flag for Quake II with an unusually large set of server-side tools. From the upstream README and source:

- Two teams (red and blue) steal the other team's flag and return it to their own base.
- Runes (tech powerups: strength, haste, regeneration, resistance, per the stats columns) spawn on the map; the `runes` cvar sets how many (default 15).
- A grappling hook, a plasma rifle (`plasma.c`) and a turret (`g_turret.c`) are part of the weapon set; the repo also carries all the stock Quake II monster files, since the game module is built from the full id game source.
- A referee system: a player authenticates with `referee <password>` and then gets `startmatch`, `stopmatch`, `pausematch`, `lock`/`unlock` (teams), `setpassword`, `changemap`, `togglefastswitch`, and a `refmenu`.
- Match play: `match <mapname>`, a `countdown_time` cvar (default 15 s), `autolock`, a map list (`maplist_file`), a message of the day (`motd_file`), a skins list (`skin_file`), and voting (`g_vote.c`) and tournament (`g_tourney.c`) modules.
- Squad tools: `squadboard`, `squad <category>`, `squadstatus` for Offense / Defense / Middle.
- Server cvars worth knowing: `dmflags`, `ctfflags`, `maxclients`, `runes`, `fastswitch`, `disabled_weps`, `flag_init`, `mod_website`.

The full cvar and command tables are in `~/Projects/lmctf60/README.md`.

### Upstream history (from the lmctf60 repo)

- The upstream README says the code is "LM_CTF Version 5.2 Tournament Edition" (1999), bumped to 6.0 by whoever supplied it to QwazyWabbit, who did not know what changed between 5.2 and 6.0. The first commit (2020-03-09) reads "State of batstone code with upgrade to VS2019".
- QwazyWabbit (101 commits, plus 6 as QwazyWabbitWOS) modernised it for C11: removed 64-bit portability warnings, wrote a new `GNUmakefile`, made the Visual Studio project warning-free at /W4, later moved to VS2022 (2023-09), added Git revision info to the module (2023-09), removed old x86 assembly paths (2023-10), and added a `want_funky_gravity` cvar (2026-01).
- Other contributors in the log: Paril (pause-the-whole-simulation and menu fixes, 2020-09), Daniel Wolf (2 commits), Joe Reid, Marc Hassman, and Michael Davis (1 each).
- Local `lmctf60` clone: 113 commits, 2020-03-09 to 2026-07-28 (90 in 2020, 9 in 2022, 8 in 2023, 2 in 2025, 4 in 2026). Its working tree contains build output (`.o` files, a `.so`) that is not tracked.
- Size: about 53,000 lines of C and 8,900 lines of headers in tracked files.
- No LICENSE file is in the repo. The code derives from id Software's Quake II game source (GPL-2.0) and from the LMCTF authors; the repo itself does not state a licence.

## What Buzzmod adds

LMCTF Buzzmod shares the first 100 or so commits with upstream, then diverges. Mike's own work begins with three commits in July 2020 and resumes in July 2026. The repo has about 2,100 commits on `main`: Mike Davis 1,987, QwazyWabbit 101, QwazyWabbitWOS 6, and four others with one to two each (counts from `git log`; "Mike Davis" and "Michael Davis" are separate author strings). 1,988 of the commits are dated 2026, most of them in August (1,730) and September (175).

### Timeline

| Date | What happened |
|---|---|
| 2020-03-09 | Shared starting point with upstream. |
| 2020-07-04 | First Buzzmod commits: stat logging for runes, quad, armor and shield. |
| 2020-07-16 | `toss rune` command added ("works ok-ish"); stats shown on the scoreboard. |
| 2020-09-12 | Runes are dropped when a player disconnects (this commit is in both histories). Upstream tag `v6.2` is on this day. |
| 2026-07-28 | The 2020 work, which had only existed on a backup drive (`g:\lmctf60`), is recovered and committed. The commit message states that `ctf_sqlite_player.c`, `ctf_sqlite_unidb.c` and `ctf_file_io.c` were never committed and could not be recovered, so the SQLite layer was rewritten. Same day: merged with current upstream, stats crashes fixed, capture streaks and sweeps added, `sv statsdb`, `cmd lifetime`, `cmd rank`, per-match history, CI for Linux and Windows, rebranded "LMCTF Buzzmod", `release-1` and `release-2`. |
| 2026-07-30 | Bot support: first with an imported bot library (`release-3`, shipped as a preview). |
| 2026-07-30 to 2026-08-11 | Native SLIPGATE bots replace the bot library; route generation (RUNE) added (`release-4`, 2026-08-11). |
| 2026-08-11 | "RELEASE PREP": about 30 tuned bot cvars become the code defaults; `sessiondb` and `statsdb` default on. |
| 2026-08-15 | `release-5` and `release-6` (HOOK milestone). |
| 2026-09-02 | `v1.0.0`, the first release with SLIPGATE bots. |
| 2026-09-03 | Last commit at time of writing (documentation of SLIPGATE's three decision levels). |

### Features, one by one

**1. Rune tossing (2020).** `toss rune` (also `toss artifact`, `toss tech`) in `Cmd_ItemToss_f` in `g_cmds.c` throws the carried rune using the rune's own toss handler. The 2020 commit also added a projectile-style `fire_rune` and a `P_ProjectSource` helper in the same file. Upstream already had rune toss helpers in `g_runes.c` (`tossrune`); the console command is the Buzzmod addition.

**2. Flag tossing (2026).** The `toss flag` command existed but did nothing: the flag item's toss slot was never filled in the item table, so it only printed "Item is not tossable." A fix on 2026-07-31 routes `toss flag` through `ctf_playerdropflag`, the same handler used when a carrier drops the flag normally (timers, position, team messages), and prints "You have no flag." if the player has none. The same commit fixes a server crash when an observer used `toss ammo`. Later bot logic (`sg_handoff`, off by default as of 2026-08-05) uses the toss command path so a pressured, low-health flag carrier can pass the flag to a nearby teammate.

**3. Expanded statistics.** With `ctf_statsdb` on, the game records per player, across maps and restarts: frags, deaths, suicides, captures, flag pickups, returns, carrier kills, offense and defense kills, assists, kill streaks, killing sprees, capture streaks, sweeps (a win in which the enemy never captured), shots, hits, damage given and received, railgun shots/hits/kills, ping, item pickups (quad, shield, armor, mega health), per-rune pickups, and time played. Column names are in `ctf_sqlite_unidb.c` and `ctf_sqlite_player.c`.
- `ctf_statsdb 0` off, `1` one file per player, `2` one shared database file `players.db` in the mod directory.
- Player commands: `stats`, `lifetime`, `rank`, `statsall`; with the shared database also `season`, `records`, `activity`, `momentum`, `card`, `vs`.
- A match summary prints at the end of each game (final score, top capper, defender, killer, accuracy leader).
- Admin console: `sv statsdb status | flush | top | player | export | backup | rename | prune | reset`. `rename` merges records when a player changes name (players are identified by name).
- Other server cvars: `ctf_switch_penalty` (clears the score of a player who joins the larger winning team), `ctf_hitsound` and `ctf_killsound` (confirmation sounds).
- Starting equipment: the `spawn_loadout` cvar, grammar `thing[:count]`, with named `@loadout_*` builds.
- Scoreboards are described as declarative layouts (`ui_boards.c`, `ui_layout.c`, `docs/LAYOUT.md`). This was converted in August 2026 with a rule that output stay byte-for-byte equivalent; two old display bugs it exposed (MVP rows drawing the rune line twice; observer sections repeating earlier lines) were then fixed. Scoreboard artwork and sounds ship in `lmctf6-buzzmod.pak`, which clients also need.

**4. Persistent per-match history.** Release 2 (2026-07-28) added the `matches` and `match_players` tables in the unified database, one row per match and one per player in it. These feed the web site's match pages.

**5. SLIPGATE bots.** Native bots in the game module (about 19,000 lines of C across 112 files in `slipgate/`, and more in the root `sg_*` and test files). They fill teams (`sv_botfill`, `sv sg add/remove/kick/list`), use the grappling hook as main movement, and decide at three levels (team goal, per-bot strategy, per-frame tactics). Their routes are built by the module itself ("RUNE": it carves each map into cells and proves every crossing with the game's own movement physics, on a second thread, writing `maps/<map>.rune`). Route hints (`maps/<map>.bites`, grapple anchor points) are shipped for 47 maps. Design details: `SLIPGATE.md`, `docs/RUNE.md`. Release 3 originally used an imported bot library; that was replaced (the CHANGELOG says all the v1.0.0 bot code is new). "Rune" in the SLIPGATE sense (route data) is a different thing from the powerup runes above.

**6. Packaging and CI.** A GitHub workflow builds Linux (gcc and clang) and Windows (x64 and x86) and publishes a release on every `v*` tag. Release files: `gamex86_64.so`, `gamex86_64.dll`, `gamex86.dll`, `lmctf6-buzzmod.pak`, `maps/*.bites`, `server-sample.cfg`, `SHA256SUMS`. Windows server runs are noted in `ROADMAP.md` as not yet done on a real server (modules cross-build only).

### Stack and size (Buzzmod, tracked files, approximate)

- C: about 86,600 lines in `.c` files and 13,600 in `.h`, excluding vendored SQLite (`sqlite3.c`, about 138,000 lines).
- Tests and tools: C unit tests (`tests/`, many `sg_*_test.c` plus makefiles), shell scripts (about 2,200 lines), Python tools (`tools/demobites.py`, `dm2trace.py`, `logbites.py`; about 400 lines), and large JSON data files (about 72,000 lines).
- Documentation: about 1,100 lines of Markdown in the top level and `docs/` (README, CHANGELOG, ROADMAP, SLIPGATE, RUNE, LAYOUT, release notes), plus `docs/internal/` design notes (ARCHITECTURE, DEPENDENCY-GRAPH, handoff notes).
- Runtime: Quake II game module (shared library) loaded by the engine. Yamagi Quake II is the tested engine.
- Branches: `main`, `era4`, `slipgate`, plus several working branches.

## How stats flow from the game server to the API

1. The game module (Buzzmod) counts events during play in per-client structures (`p_stats.c`, `p_stats.h`; kills, shots, flag events, item and rune pickups).
2. At intervals, on disconnect and at map end, it writes them to SQLite using the vendored SQLite (`sqlite3.c`) through `ctf_sqlite_core.c` (open with write-ahead logging, transactions, cached statements) and `ctf_sqlite_unidb.c` (the unified backend). Files: `<gamedir>/players.db` (path from `ctf_file_io.c`).
3. Tables (unified backend): `userdata` (name, first seen, last played, play time), `game_stats`, `ctf_stats`, `character_data`, `matches`, `match_players`, and `sg_session_events` for bot sessions. A player's rows are tied together by `char_idx`.
4. `sv statsdb backup <file>` makes a copy that is safe to take while players are connected.
5. q2lmstats opens `players.db` read-only (`DB_PATH` environment variable), so it cannot change game data. It serves HTML pages and JSON from the same queries.

## q2lmstats

- Node.js (`engines` says 20 or newer, README says 22 or newer), Express 4, ES modules. About 1,670 lines of JavaScript (`src/`, `test/`), tests with `node --test`.
- Modules: `server.js`, `app.js`, `config.js`, `db.js` (read-only SQLite), `schema.js`, `queries.js`, `routes.js` (API), `pages.js` and `view.js` (HTML pages).
- History: 8 commits. Two in 2020-08/09 (README only); on 2026-07-28 the project was rewritten against the schema the game actually writes and expanded to the full schema with a stats site.
- Pages: `/`, `/leaderboards` (any of 42 stats), `/players`, `/players/:id`, `/matches`, `/matches/:id`.
- API: `/api/players`, `/api/players/:id`, `/api/players/:id/matches`, `/api/players/:id/maps`, `/api/players/by-name/:name`, `/api/leaderboard/:stat`, `/api/ratio/:num/:den`, `/api/matches`, `/api/matches/:id`, `/api/maps`, `/api/columns`, `/api/health`.
- Derived stats (`accuracy`, `rail_accuracy`, `kd`, `damage_ratio`, `caps_per_hour`) are computed, with a minimum sample size so a small sample cannot top a board.
- Optional settings: `PORT` (3000), `HOST`, `CORS_ORIGIN`, `DEFAULT_LIMIT`, `MAX_LIMIT`, `RATE_WINDOW_MS`, `RATE_MAX`. Do not point it at a database on NFS.

Run:

```
cd q2lmstats
npm install
DB_PATH=/path/to/lmctf/players.db npm start     # http://localhost:3000
npm test
```

## How to build and run a server

**Build Buzzmod (Linux):**

```
cd lmctf6-stats
make -f GNUmakefile all      # produces gamex86_64-lmctf-<rev>.so
make -f GNUmakefile check    # unit tests
```

Rename the output to `gamex86_64.so`. On Windows, open `gravity.sln` in Visual Studio 2022 and build Release for x64 or Win32. SQLite is vendored; nothing else is needed. (The same `gravity.sln`/`GNUmakefile` pair exists in `lmctf60`, built the same way.)

**Install and run:**

1. Use a Quake II server (Yamagi Quake II is the tested engine) with the retail `baseq2` data and the LMCTF map files for the maps you run. Map and retail data are not part of the repo.
2. Download the release package from https://github.com/mgd34msu/lmctf60-buzzmod/releases and unpack into the `lmctf` directory: the game module for your platform plus `lmctf6-buzzmod.pak` (clients need the pak too).
3. Copy `server-sample.cfg` and edit it. Key settings: `ctf_statsdb 2`, `capturelimit`, `timelimit`, `maplist_file`, `sv_botfill`.
4. Start the engine with the `lmctf` game directory (for example `+set game lmctf`), then optionally run `q2lmstats` against `lmctf/players.db`.
5. With bots, the first load of each map builds its routes on a second thread and the bots wait until it finishes; hint files for 47 maps ship in `maps/`.

Nothing in this write-up was run or built; the commands come from the repo READMEs.

## lm-singleplayer (third-party Loki's Missions packs)

Contents of `~/Projects/lm-singleplayer` (no git; the zips are dated 2026-08-30 on disk):

- `escape/` and `escape.zip`: "Loki's Missions - Escape", dated 1999-08-13. A single-player mission for Quake II, six maps, a plasma rifle with spread and bounce modes taken from LMCTF (default key R), and a new plasma soldier monster. Launch with `quake2 +set game escape`. Credits in its readme list Mr. White, Der Kommissar and Vampire (level design), Vampire, Skwid and Batstone (programming), and others. Contains `gamex86.dll`, `pak0.pak`, `maps.lst`, a launcher. Note that Batstone is also the name in the first LM_CTF commit message ("batstone code"), which suggests shared code between this mission and LMCTF; the repo does not state this directly.
- `lmtxp2/` and `lmtxp2.zip`: "The Xeno Project 2", released 1999-11-10, a Half-Life single-player episode (14 BSPs, 2 custom skies, custom sounds and voices). It is a Half-Life mod (`hl.exe -game lm_txp2`), not Quake II. It needs the retail Half-Life.
- These are old freeware downloads from the Loki's Missions site; Mike did not write them. The folder holds game binaries and assets, so they are not copied into this repo.

## Relationship to other projects

LMCTF is a Quake II mod, so it depends on the Quake II engine and game API and not on the TypeScript, C or Rust ports. In particular it is its own C codebase and does not share code with `quake-anthology` or Quake-Anthology-Rust. Whether Mike wants Buzzmod's stats or bots in those ports is not stated in these repos.

## Lessons and decisions

- Work lost and rebuilt: three 2020 SQLite source files were never committed and were lost; the 2026 layer was written from scratch to match the surviving header and the compiled objects.
- Preserve behaviour first: the scoreboard conversion required byte-identical output before fixing the old bugs it revealed.
- Bots: the first approach (an imported bot library) was shipped as a preview, then replaced by native code that builds its own routes from the map and uses the game's physics to prove them. Strafe-jumping was tried and dropped because the grapple already did the job.
- Defaults: on 2026-08-11 the tuned bot configuration was made the built-in default, so a fresh server with no config runs the tested setup.
- Measured numbers in the CHANGELOG (for example "32 ropes per bot-minute on lmctf09") come from five-minute games with nine or ten bots; ROADMAP notes bots convert about one steal in five into a capture.

## Credits and licence

- Loki's Minions CTF: LM_Hati, LM_Surt, LM_JORM and the LM team (LMCTF 5.2, 1999).
- QwazyWabbit: the modern C11 port and build system (https://github.com/QwazyWabbitWOS/lmctf60).
- Mark Davies: StdLog logging (`stdlog.c`).
- Paril, and the other upstream contributors listed above.
- SQLite: public domain.
- id Software: the Quake II game source underneath.
- Licence: none of lmctf60, lmctf6-stats or q2lmstats contains a LICENSE file; `q2lmstats/package.json` says ISC. Check before redistributing.
