# Aquila Post-Migration QA Report

**Verdict: POST-MIGRATION QA INCOMPLETE. One open item (QA-14) is unresolved; no merge-caused regression confirmed so far.**
The audit stopped at the usage limit while investigating QA-14. A follow-up cloud session added static analysis of QA-14 (section 13). That session had no BYOND runtime because the download host is blocked, so QA-14 is still unconfirmed. No game code was changed.

Bug list: [AQUILA_POST_MIGRATION_BUGS.tsv](AQUILA_POST_MIGRATION_BUGS.tsv)

## 1. Repository state (Phase A)

| Check | Result |
|---|---|
| Branch | `update-do-najnowszego-nsv13` |
| HEAD | `df3bcf71330722463b1745ff42fd73032b1d6d45` (expected) |
| upstream/master (local and `git ls-remote`) | `69565b730fdf45c09861dfca6c669b67f8601a71`: fully caught up |
| merge-base HEAD upstream/master | `69565b730f` |
| Tracked or staged changes | none; only untracked audit docs |
| Bridge `d253381c64` | tree identical to `34fa11431f` (tree-neutral) |
| Pushed? | No. `origin/update-do-najnowszego-nsv13` is still `f847d3caca`; aquila/upstream push URLs are DISABLED |

## 2. Merge audit (Phase B)

- I recomputed every stage merge with `git merge-tree` and diffed the result against the real merge commits. The manual edits are limited to:
  - Stage 1: `_lists.dm` (took upstream's version, which contains Aquila's identical `list_clear_nulls`), `astrometrics.dm` (Polish text moved into upstream's `message` flow), and 12 maps restored to Aquila blobs.
  - Stage 2: none (a pure auto-merge).
  - Stage 3: `priority_announce.dm` and `combat_handling.dm` (Polish text plus upstream `silent`), Shrike metadata kept at the Aquila versions, and 4 maps restored.
  - Stage 4: Von Neumann map/config/template removal and 3 maps restored.
- All 235 upstream-changed files were classified:
  - 178 taken from upstream (Aquila never touched them).
  - 3 taken from upstream although Aquila had edited them; upstream already contains Aquila's content.
  - 26 merged: 22 are an exact line-level union of both sides, and the 4 manual merges are semantically correct.
  - 28 kept at the Aquila version, all because of the map policy.
- Every Aquila delta line (fa47→Aquila) is present in HEAD except the 3 intentional conflict rewrites. 0 Polish lines were lost.
- No upstream-deleted file was resurrected. All 18 upstream-added `.dm` files are present and included, except the 2 Von Neumann map files (intentional). No file lost its include.
- AQ markers (`AQ EDIT|AQUILA EDIT`, case-insensitive, outside `aquila/`): 458 in 201 files before the migration, 460 in 202 files at HEAD. The only change is +2 comments in `astrometrics.dm`.

## 3. Overlays (Phase D)

- All 7 full-copy overlays were rechecked: the upstream base procs are unchanged from fa47 to 69565 and since the Aquila copy (one whitespace-only difference). None calls `..()`, which is intentional, and no modern behavior is hidden. 65 same-type Aquila redefinitions in total (58 call the parent).
- No upstream-changed proc is masked by an Aquila overlay. The vampire mode and the Aquila point rewrite were tested against the new upstream code: AI-traitor pop limits and overmap pointing both work.

## 4. Maps (Phase E)

- 733/733 `.dmm` git blobs match `AQUILA_MAP_BASELINE_REPAIRED.tsv` / `34fa11431f`: 0 mismatches, 0 extra.
  - Note: the TSV's sha256 column for Serendipity1/2 and Tycoon2 was computed on LF content, while the working tree is CRLF (`core.autocrlf=true`). This is a documentation artifact only.
- Non-map `_maps` files (JSON, `.dm`, job_changes) are identical to Aquila.
- Von Neumann: the code and sprites are present; the map, rotation, `_basemap` include and vote entry are absent. `cargo_aiship` left no dangling references.

## 5. Config / job coupling (Phase F)

- No new problems: the results are identical to the baseline.
- Known issues remain: `Dyrektor Naukowr` and `Kurator`.
- New coupling from upstream works: `JOB_NAME_AI` = "SI Statku" is correctly restricted by the AI-traitor minimum pop (QA14).
- Pre-existing hardcoded English job titles are listed as QA-06.

## 6. Build and test matrix (Phases G and H), run in scratch exports

| Item | Result |
|---|---|
| Normal compile (`-DCBT`) | 0 errors, 0 warnings |
| CIBUILDING compile | 0 errors, 1 warning (expected `loop_checks`) |
| ALL_MAPS (CI template set incl. RandomRooms) | 19 errors in 4 Aquila RandomRooms maps, identical to the baseline (QA-05). With those excluded, 477 maps load with no other errors. The previous validation missed this because juke's include list skips RandomRooms. |
| dreamchecker 1.8 | 0 diagnostics (verified with an injected canary that it does detect errors) |
| tgui | install --immutable, build (webpack 5.105.0), tests 4/4, lint and tsc all pass |
| dmi.test / dmm_test | 1302 `.dmi` / 733 `.dmm` parsed (Pillow 12.3.0, on Python 3.13; 3.11 is not installed) |
| CI linters | check_grep (full, with a jq shim), filedirs, changelogs and json pass; map JSON paths resolve case-sensitively |
| DME | 4181 includes, 0 missing; `aquila/aquila.dm` is last; 304 Aquila includes resolve; 0 duplicates |
| Old weapon API | 0 references anywhere (`FIRE_MODE_*`, `MAX_POSSIBLE_FIREMODE`, `WEAPON_CLASS_*`, `/datum/ship_weapon`) |
| Macro conflicts between `aquila/` and core | 0 |

## 7. Runtime matrix (Phase I)

- HEAD: 25 runs over 8 maps × 3, plus 1 extra. The pre-migration baseline: 24 runs on the same maps.
- Unit tests: 40/40 in all 25 HEAD runs. Baseline: 2 runs failed `spawn_humans` because of random runtimes.
- Every HEAD runtime signature was either also present in the baseline or is a known intermittent with unchanged code:
  - Serendipity soporific "bad index" (QA-03)
  - `null.beacons_in_ship`
  - Atlantis spam
  - Listening Post ruin
  - the map-loading race
  - the monkey icon / light switch errors
  - the Cave Bat error
- Subsystem init order and init timing are identical to the baseline. Config and map error logs are identical.
- DreamDaemon exits with rc=139 (0xC0000005) at shutdown in every run, baseline included: environmental.

## 8. Targeted tests (Phase J), scratch-only `qa_tests.dm`, 16 tests on 7 maps

Passing everywhere:
- main-ship weapon datums, with all physical weapons linked (cargo launcher excluded by design)
- deck turrets and cores
- AI firing: 10 ship types, 40 projectiles
- PDC/AMS autonomy
- angle-based damage relay
- fighters
- railgun forge
- stormdrive
- overmap pointing on top of Aquila's point verb
- SolGov fleets, the VNC AI frigate and the Polish combat announcement
- security level cycle
- AI-traitor pop limit
- announcements
- astrometrics

Expected or harness failures:
- QA11 fails because of the pre-existing dead shuttle templates (QA-08).
- QA12 always records pre-existing teardown runtimes (QA-09, identical in the baseline).

Boarding results:
- Real-path boarding of all 6 Aquila-kept syndicate interiors loads 6/6 on Aquila_Snake, Aquila_Serendipity, Snake, Tycoon and Atlantis.
- **Open, QA-14:** on the default map (Aquila_Atlas, and once on Atlas), AI ships keep `interior_mode=0` in HEAD (0/6 loads, reproduced 3 times, including with a QA12-only build). The baseline with the identical test loads 6/6. The root cause is not yet found. It may be a real regression or a test-environment interaction, so it needs a live-round check before deploying.
- A new runtime class came in with upstream code (QA-01): stale weapon-datum references when a boarded AI ship is deleted.

## 9. Localization (Phase L)

- Polish lines by type outside `aquila/` (.dm/.txt/.json/.html): 1687/623/460/43, identical before and after.
- Inside `aquila/`: 366. Maps: 8.
- No translation became dead or stale because of the merge.
- The astrometrics conflict resolution now sends the Polish scan-start text to both say() and radio (gated by upstream's broadcast toggle, which defaults to TRUE).
- The roughly 219 new English upstream strings were left untouched (localization debt, not a regression).

## 10. Performance (Phase M)

- No unbounded loops were found in the imported weapon/AMS code: targets are capped.
- QA-01 also means datum references leak until a hard delete.

## 11. Deployment

- TGS DMAPI 7.3.0 → 7.4.0 (interop 5.11.0): confirm the production TGS server supports it.
- `CLIENT_MAX_VERSION 515` warns BYOND 516 clients.
- No DB schema change (DB 6.0).
- During the audit drive C: ran out of space (0 bytes free). The scratch runs were redone afterwards.
  - The previous migration session's scratch folder (`%LOCALAPPDATA%\Temp\claude\...\5708265b-...`, about 11 GB) is still on disk. Consider removing it yourself.

## 12. Fixes and commits

- None applied.
  - QA-01 and QA-02 are upstream defects (P3).
  - QA-03, QA-04 and QA-05 need map edits (policy: report only).
  - QA-14 is unresolved (see section 13).
- No commit, no push.

## 13. QA-14 follow-up (static analysis, cloud session)

There was no runtime reproduction: byond.com is blocked by the container's network policy, so DreamMaker and DreamDaemon could not be installed. Everything below comes from reading the code.

**What changed in the overmap init path between `34fa11431f` and HEAD.** The only relevant change in `nsv13/code/modules/overmap/overmap.dm` is that `apply_weapons()` moved from `LateInitialize()` into `Initialize()` (the Stage 2 weapon datum refactor), along with the new ammo max vars.
- None of the `apply_weapons()` overrides sleep.
- The `interior_mode` switch in `LateInitialize()` is byte-identical to the baseline.
- `boarding/interiors.dm` is unchanged.
- `SSatoms`, the map reader, `map_template.dm` and `custom_reader.dm` are unchanged.
- No new async map load was added anywhere.

So no merge-caused code change sets or skips `interior_mode`.

**The most likely mechanism (it exists in both baseline and HEAD).** `/obj/structure/overmap` returns `INITIALIZE_HINT_LATELOAD`, and `interior_mode` is only switched to `INTERIOR_EXCLUSIVE` inside `LateInitialize()`.
- `SSatoms.InitAtom()` calls `LateInitialize()` immediately only when `mapload` is false.
- `CreateAtoms()` runs `CHECK_TICK` while `SSatoms.initialized == INITIALIZATION_INNEW_MAPLOAD`. That happens during any template or interior load, for example `instance_overmap`, `initParsedTemplateBounds`, a boarding interior or a dynamic interior from `queue_for_interior_load`.
- Any `new /obj/structure/overmap/...` created during that sleep gets `mapload = TRUE` (see `code/game/atoms.dm:142-143`).
- Its `LateInitialize()` is then deferred into `SSatoms.late_loaders` until the sleeping `InitializeAtoms()` resumes.

During that window `interior_mode` is still `NO_INTERIOR`. That is exactly the QA-14 symptom: no runtime, and `ai_load_interior` refuses. Whether the window is hit depends on load timing (map size, what loads during the test, tick usage). That fits the map-dependent, HEAD-vs-baseline difference without any code difference. HEAD's heavier `Initialize()` (weapon datums) shifts that timing.

In a live round the effect would be a short delay (until the in-flight load finishes), not a permanent loss. **Verdict: probably timing or harness, not a merge regression. Still unconfirmed.**

**Related pre-existing fragility (not migration-caused, from the `34fa11431f` repair).** `/obj/effect/spawner/room` now loads its template from `LateInitialize()`. When that runs inside `SSatoms.InitializeAtoms()`'s late-loader loop (only for spawners created after `LoadStationRooms()`, e.g. gulag), the following happens:
1. The nested `template.load()` → `InitializeAtoms()` runs its own late-loader loop, which also processes the outer call's pending late loaders.
2. The nested call then runs `late_loaders.Cut()`.
3. The outer loop iterates its own copy and only checks `QDELETED`, so those atoms can get `LateInitialize()` a second time.

This is worth a later hardening pass. It is not a QA-14 cause, because room spawners exist on every map.

**How to confirm QA-14 before deploying (needs a BYOND host).**
- In the QA12-only test, before checking `interior_mode`, log `SSatoms.initialized` and `(ship in SSatoms.late_loaders)`. If it is `2` / `TRUE`, the mechanism above is confirmed.
- Alternatively, wait with `UNTIL(ship.interior_mode != NO_INTERIOR || world.time > start + 10 SECONDS)` and re-check: a pass means harness timing only.
- Live round on Aquila_Atlas: spawn a boardable syndicate AI ship (e.g. `/obj/structure/overmap/syndicate/ai/destroyer`) when no load is in progress and check `interior_mode == INTERIOR_EXCLUSIVE` and that boarding works.
