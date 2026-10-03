# Aquila Baseline Validation (Phase 0)

**Result: BASELINE FAIL** (pre-existing defects in the unmodified Aquila HEAD; details below). Nothing was fixed.

- Repo state verified before starting: branch `update-do-najnowszego-nsv13`, HEAD `f847d3caca6e23d4ad0cfed965698c2f54412e74`, remotes as expected (`aquila` and `upstream` push disabled). Working tree clean apart from my two analysis files.
- Safety tag: `aquila-baseline-2025-09-09` → `f847d3caca` (annotated, local only, not pushed). The tag did not exist before. Git had no committer identity configured, so I supplied one only for that command via environment variables; no Git config was changed.
- Map baseline: [AQUILA_MAP_BASELINE.md](AQUILA_MAP_BASELINE.md) / [AQUILA_MAP_BASELINE.tsv](AQUILA_MAP_BASELINE.tsv) (733 maps).

## Method (so the repo stays untouched)

Builds write outputs (`*.dmb`, `*.rsc`, `_maps/templates.dm`, `data/`, `tgui/.yarn`, bootstrap cache) into the source tree. To keep the real repo unmodified I exported HEAD with `git archive f847d3caca` into a scratch directory and built there. The repository itself was never built in. The only edit anywhere was a **diagnostic, scratch-copy-only** comment-out of two `#include` lines in `_maps/_basemap.dm` (restored afterwards) to find out whether anything else breaks behind the first map error.

## Toolchain (versions taken from repo config, then verified)

| Component | Repo requirement (source) | What I used | Notes |
|---|---|---|---|
| BYOND | 515.1633 (`dependencies.sh`, `code/__byond_version_compat.dm`) | **515.1633** | Downloaded `http://www.byond.com/download/build/515/515.1633_byond.zip` (8,998,512 bytes), unzipped to `%USERPROFILE%\BYOND-515.1633\` (outside the repo). Build finds it via `DM_EXE`. |
| Node | 18.14.2 (`dependencies.sh`) | **v18.14.2** | Fetched by the repo's own `tools/bootstrap/node_.ps1` into the gitignored `tools/bootstrap/.cache` of the scratch copy. System Node: none. |
| Yarn | 3.4.1 (`tgui/package.json`) | **3.4.1** | Vendored in `tgui/.yarn/releases`. |
| Python | 3.11.2 (`dependencies.sh`) | **3.11.2** (embed, via repo bootstrap) | System Python 3.13.15 was **not** used. Packages from `tools/requirements.txt` (Pillow 10.3.0, PyYAML 6.0, pygit2 1.11.1, bidict 0.22.1, bs4 4.11.2). |
| SpacemanDMM / dreamchecker | suite-1.8 | **dreamchecker 1.8.0** (build 2023-10-29) | Downloaded `https://github.com/SpaceManiac/SpacemanDMM/releases/download/suite-1.8/dreamchecker.exe` (4,102,101 bytes) to `%USERPROFILE%\BYOND-515.1633\spaceman\`. |
| rust_g / auxmos | 3.1.3 / 2.5.2-b | repo-shipped `rust_g.dll`, `auxmos.dll` (both 32-bit x86 PE) | Loaded successfully during the DreamDaemon run (no library errors). I did not independently verify their embedded version strings. |
| webpack (via tgui) | `^5.94` | 5.96.1 | |
| `jq`, Rust toolchain | CI scripts / not needed for build | **not installed** | `check_grep.sh` needs `jq`; skipped. |

No repository version was changed or upgraded. Total downloads: BYOND zip, dreamchecker.exe, Node exe, Python embed + pip/requirements, and ~1000 npm packages for tgui (yarn cache is in the scratch copy).

## Results

All commands run from the scratch copy of HEAD with `DM_EXE=%USERPROFILE%\BYOND-515.1633\byond\bin\dm.exe`.

| # | Check | Command | Result |
|---|---|---|---|
| 1 | Normal DM compile | `tools\build\build.bat --ci dm` | **PASS**: `0 errors, 0 warnings`, 17.8 s |
| 2 | CI-define compile + ALL_MAPS | `tools\build\build.bat --ci dm -DCIBUILDING -DCITESTING -DALL_MAPS` | **FAIL**: `_maps\map_files\Tycoon\Tycoon2.dmm:46720:error: unexpected input: <<`; `Failed to load map file`; `1 error, 1 warning` |
| 2b | Same, diagnostic only: `Tycoon2.dmm` and `snake_upper.dmm` includes commented out in the scratch copy | same | **PASS**: `0 errors, 1 warning` (89 s). So these two are the only blockers for ALL_MAPS. `Serendipity2.dmm` is not in the `ALL_MAPS` include list at all, so the compiler never sees it. |
| 3 | dreamchecker | `dreamchecker -e nsv13.dme` | **PASS**: `Found 0 diagnostics` |
| 4 | tgui build | `build --ci tgui` | **PASS** (webpack 5.96.1 compiled successfully; yarn peer-dependency warnings only) |
| 5 | tgui tests / lint / tsc | `build --ci tgui-test`, `tgui-lint`, `tgui-tsc` | **PASS** (4/4 tests; eslint and tsc clean) |
| 6 | DreamDaemon boot + unit tests | `build --ci dm-test -DCITESTING` | **Boot OK, run not clean**: see below |
| 7 | `dmi.test` | `python -m dmi.test` | **PASS**: 1298 `.dmi` files parsed |
| 8 | `mapmerge2.dmm_test` | `python -m mapmerge2.dmm_test` | **FAIL**: `KeyError: '<'` on the conflict markers. A per-file run over all 733 maps shows only `Serendipity2.dmm` fails that parser. The other two marker maps are not caught by it, but the DM compiler rejects `Tycoon2`. |
| 9 | JSON verifier | `tools/json_verifier.py` on 114 `.json` files | **PASS** (0 bad) |
| 10 | `template_dm_generator.py` | | **PASS** |
| 11 | `check_filedirs.sh nsv13.dme` | | **PASS** |
| 12 | `check_changelogs.sh` | | Example file OK; the script then needs `yaml` in the system Python, which is not installed, so it did not complete. Environment limit, not a repo defect. |
| 13 | `check_grep.sh` | | **Not run to completion**: `jq` is missing (exit 127). The early "map issues"/"stacked cables" sections ran. |
| 14 | Static conflict-marker scan | `git grep -E '^(<<<<<<< |>>>>>>> )' HEAD` | **3 files** (below) |

### DreamDaemon / unit-test run (check 6)

- Compiled with `CIBUILDING`+`CITESTING` and started: world initialised, map `NSV Aquilas` (`Aquila_Atlas`, the default in `config/maps.txt`) loaded, a round was started, **40 unit tests ran, 40 PASS, 0 FAIL** (including `heretic_knowledge`, `heretic_rituals`, `gamemode_sanity`, `subsystem_init`, overmap/fighter tests), the round ended and the world rebooted.
- The process exited with `0xC0000005` (access violation) after the round-end reboot. **This is environmental, not an Aquila defect**: unmodified upstream at `fa47d5fe77` built the same way also exits with `3221225477` (`0xC0000005`) even though it wrote `clean_run.lk` ("Success!").
- Aquila did **not** write `clean_run.lk`, because the repo's clean-run rule treats any runtime error as failure. Upstream `fa47d5fe77` had 0 runtimes; **Aquila had 5**:

| # | Runtime | Source | Classification |
|---|---|---|---|
| 1 | `Cannot modify null.allow_bureaucratic_error` in `set_overflow_role("Midshipman")` | `code/controllers/subsystem/job.dm:61`, called from `Initialize` | **Code/config, pre-existing Aquila defect.** `JOB_NAME_ASSISTANT` was changed to `"Majtek"` (`code/__DEFINES/jobs.dm:137`) but `config/game_options.txt` still has `OVERFLOW_JOB Midshipman`, so `GetJob()` returns null. |
| 2, 3 | `We started maploading while we were already maploading` ×2 | First occurrence (23:06:31): `Gaming Room` (`sk_rdm_glg_07`) loaded by `/obj/effect/spawner/room/gulag` during atom init. Second (23:07:28): not traced. | **Probably the Aquila gulag/room spawner** (first one confirmed); not present in unmodified upstream. |
| 4, 5 | `Cannot read null.anchored` ×2 | `nsv13/.../deck_guns.dm:704` (`deck_turret/Initialize`): `locate(/obj/machinery/deck_turret/core) in SSmapping.get_turf_below(src)` returns null, for the "M4-15 'Hood'" turrets at (146,126,3) and (146,122,3) | **Map-related** (Aquila_Atlas has deck turrets with no core on the deck below, or the core is placed differently). The upstream code does not null-check. |

- Other log observations (not counted as failures): `Fail2topic failed to drop firewall rule` (Windows), SQL `Connect() failed` to `127.0.0.1:3306` (no database server; DB features not tested), `ASSIMILATION`/`DOUBLE_AGENTS` config validation failures and unknown `economy` setting (config), invalid research designs `furnace`, `furnace_console`, `sleepy` removed from nodes, donator loadout warnings, 3 interview-question config warnings. Several come from Aquila's modified `config/*`.

### Pre-existing defects summary (all present in unmodified `f847d3caca`)

1. **Merge-conflict markers committed in three maps** (`Serendipity2.dmm` ×12 hunks, `snake_upper.dmm` ×1, `Tycoon2.dmm` ×2), by #317. Serendipity and Snake are `votable` in `config/maps.txt`. The Aquila-forked variants (`Aquila_Serendipity`, `Aquila_Snake`, `Aquila_Atlas`) are clean and compile.
2. `OVERFLOW_JOB Midshipman` vs renamed job "Majtek" → runtime at init.
3. Gulag/room spawner re-entrant map load ×2.
4. `deck_turret` core lookup null on Aquila_Atlas ×2 (map-related).
5. Duplicate `ammo_sorter` design definitions (static finding from the first analysis).

### Environment-related (not repo defects)

`0xC0000005` on DreamDaemon exit; no `jq`; system Python lacks PyYAML; no MySQL server.

## Verdict

**BASELINE FAIL.** Compile, dreamchecker, tgui and the unit tests are good, but ALL_MAPS compile fails and the DM run is not "clean" because of pre-existing defects, all listed above. I stopped at the failure report and fixed nothing.

## What needs your decision before Phase 1

1. The three maps with conflict markers. Under the new policy they are "current Aquila maps", but the committed file contains both sides. The cleanest reading is that the `HEAD` side of each hunk (the pre-#317 Aquila content) is the authoritative Aquila map and the `d5afbe889c` side is upstream. Do you confirm? (Do you want them fixed as part of baseline repair, as a separate documented step before the bridge?)
2. Whether to fix the baseline runtimes (job rename vs `OVERFLOW_JOB`, gulag spawner, deck-turret cores) before migrating, so the migration comparison is clean, or just record them as known.

Scratch artefacts (logs) are in the session scratchpad: `build_dm.log`, `build_dm_plain.log`, `build_dmtest.log`, `build_allmaps_diag.log`, `build_tgui.log`, `dreamchecker.log`, plus the upstream control build `buildE_dmtest.log`.
