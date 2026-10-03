# Phase 0.5: Pre-migration Aquila baseline repair

**Result: REPAIRED BASELINE PASS WITH WARNINGS.** Nothing is committed; the changes are in the working tree for your review.

Starting point: `f847d3caca` (tag `aquila-baseline-2025-09-09`, unchanged). No upstream content was merged, no BeeStation map content was imported, no push, no Phase 1.

Companion files: `AQUILA_MAP_BASELINE_REPAIRED.md` / `.tsv` (new authoritative map baseline, **not active until you approve**), `AQUILA_MAP_BASELINE.*` (original, untouched), `AQUILA_BASELINE_VALIDATION.md` (Phase 0 findings).

## 1. Files changed (11)

| File | Why |
|---|---|
| `_maps/map_files/Tycoon/Tycoon2.dmm` | 2 conflict hunks resolved (§2) |
| `_maps/map_files/Snake/snake_upper.dmm` | 1 conflict hunk resolved (§2) |
| `_maps/map_files/Serendipity/Serendipity2.dmm` | 12 conflict hunks resolved (§2) |
| `_maps/map_files/Serendipity/Serendipity1.dmm` | **Not on your list.** No markers, but the same #317 textual merge silently corrupted it: 2 tiles with undefined keys (`xS`, `Dq`) and 42 tiles that differ from the merge result. Same repair (§2.4). Easy to drop if you disagree. |
| `_maps/map_files/Atlas/atlas.dmm`, `Aquila_Atlas/Aquila_Atlas.dmm`, `Atlantis/Atlantis.dmm`, `Aquila_Snake/Aquila_Snake_Lower.dmm` | Deck-turret compatibility retype, one line each (§3.3) |
| `code/game/objects/effects/spawners/roomspawner.dm` | Nested map load (§3.2) |
| `code/game/machinery/stasis.dm` | One null guard (§3.4), **additional defect found by validation** |
| `config/game_options.txt` | `OVERFLOW_JOB` (§3.1) |

`git diff --stat`/`git status --short` are at the end of this file.

## 2. The broken maps

### 2.1 How the markers got there (Git evidence)

- `f847d3caca` (#317) is a single-parent commit. Its parent is `9a5e0b421d` (call it **P**, last Aquila commit before the import, 2024-04-23).
- I rebuilt the three files with `git merge-file P <base> d5afbe889c` (upstream "Hard mode" parent-state, call it **U**). With base `b6fc4ce967` or `d47b28bb5a` the output equals the committed file **byte for byte except for the marker label** (`>>>>>>> d5afbe889c` vs the full hash), with the same hunk counts (2 / 12 / 1). So:
  - **"HEAD" side of every hunk = P, the pre-#317 Aquila map.**
  - **"theirs" side = upstream U.**
  - Everything outside the hunks was auto-merged P⊕U and is *already* in the baseline.
- Aquila's own history for these files (`git log d47b28bb5a..P`): Serendipity2 was edited by `a242626be7 "poprawki mapowe do trawnika (#49)"` (55 tiles, smorgli: deliberate hand map fixes), `f6ba7befbf` (Joker, 3 tiles) and `78850026c7` (infiltrator docking, 6 tiles). Tycoon2 and Snake upper: Aquila's own delta is the `syndicate_*` docking ports (Tycoon2: 4 tiles) and the Snake rework.
- `Aquila_Serendipity2.dmm` is byte-identical to P's Serendipity2, so P *is* the Aquila map; the standard `Serendipity` path was meant to be P with upstream's later fixes merged in.

### 2.2 Why a hunk-by-hunk "pick a side" is wrong

DMM files use per-file key aliases. In all three files the same key name means different tiles on the two sides (e.g. Tycoon2 `naO` is a firealarm tile in P and a camera/ammo tile in U). Taking either side wholesale leaves tiles with **undefined keys** (I counted them: ours-everywhere leaves 2 / 1 / 0 tiles undefined, theirs-everywhere 2 / 3 / 1) and others with the wrong contents. The only reliable resolution is at the **tile level**, which is also how I verified the result.

### 2.3 Method (deterministic, auditable)

For each coordinate, with base **D**=`d47b28bb5a` (the upstream state Aquila forked from), P and U:
- P tile == D tile → take U's tile (Aquila never touched it; this is what #317 already did).
- U tile == D tile, or P == U → take P.
- Both changed it differently → **take P (Aquila)**. Policy: Aquila wins on any tile both sides edited.

The scaffold was the committed file (HEAD side of every hunk), fixed tile by tile with `tools/mapmerge2` (`DMM` parser/writer; I verified it round-trips P, U and D **byte-identically**, so no reformatting noise). Unused keys left over from the broken merge were pruned. Verification after writing, for all four files: 0 conflict markers; **0 tiles differ from the expected merge**; 0 undefined keys; 0 unused keys; every tile where P differed from D equals P (Aquila-origin tiles preserved: Tycoon2 4/4, Serendipity2 76/76, Snake upper 61,975/61,975, Serendipity1 62,269/62,269); all 733 maps parse; `ALL_MAPS` compiles.

### 2.4 Hunk-by-hunk resolutions

**Tycoon2.dmm** (both hunks are key-table insertions at the same spot):
1. Lines 46720-46744. P adds key `mZu` (docking port `syndicate_nw`, Aquila infiltrator port). U adds key `naa` (`sign/ship/pods/north`) next to it, and re-uses name `naO`. Resolution: **keep both**: `mZu` stays (Aquila), `naa` stays because the non-hunk grid already references it at (101,133), so dropping it would leave a dangling key. Result tiles: (101,133) = U's sign, 4 Aquila port tiles = P.
2. Lines 46759-46784. P adds key `nbp` (`syndicate_n` docking port). U adds key `nbG` (floor light decal at (149,104)). Resolution: **keep both**, same reasoning. Evidence: tile 3-way merge for the file gives 0 true conflicts (P changed 4 tiles, U 4,605).

**snake_upper.dmm** (one hunk):
1. Lines 7189-7206. P adds key `Pb` (docking port `syndicate_sw`, tile over `openspace/airless`). U changes the preceding key's area `/area/maintenance/department/engine` → `/area/engine/armour_pump` (upstream "Snake Area Mapping fix #2761"). Aquila never touched that key's area (P == base there), so **both are applied**: `Pb` kept, area change kept, consistent with the other 107 upstream area tiles that the baseline already contains. 0 true conflicts in the file.

**Serendipity2.dmm** (12 hunks; 11 tiles where both sides changed the same tile, all Aquila-wins):

| # | Lines | Nature | Resolution |
|---|---|---|---|
| 1 | 206-241 | P adds key `bf` (docking port `syndicatecutter_s`); U edits the next key (adds an atmospherics requests console) | both kept |
| 2 | 1091-1101 | P: cable `4-8` on tile (107,139); U: adds green pipes on the same tile | **conflict, P wins** |
| 3 | 4028-4058 | tile (108,138): P keeps the N2 tank console/manifold/APC; U replaces it with cyan pipes + cable `1-2` | **conflict, P wins** |
| 4 | 4592-4599 | tile (105,139): P filter without `piping_layer` (Aquila `#49`); U removes/moves the filter | **conflict, P wins** |
| 5 | 5040-5057 | tile (109,139): P keeps pump, drops cable; U drops the pump | **conflict, P wins** |
| 6 | 5063-5071 | cable `2-8` (P) vs cyan pipe (U); I did not map this one to a single tile | resolved by the tile-level rule (it is part of the engine/atmos corner); result verified against the expected merge |
| 7 | 7386-7393 | tile (146,150): P adds a poster; U adds a vent pump | **conflict, P wins** |
| 8 | 7944-7952 | tile (105,138): `piping_layer` removed by P vs changed to 4 by U | **conflict, P wins** |
| 9 | 8150-8170 | P adds key `KW` (escape-pod docking port); U adds key `KV` (vent pump used at (121,148)) | both kept |
| 10 | 10907-10933 | P adds `WF` (`syndicate_se` docking port) beside `WE`; U deletes `WE` | `WF` kept; `WE` is unused after resolution and pruned |
| 11 | 10938-10945 | tile (127,150): cable `4-8` (P) vs plain cable (U) | **conflict, P wins** |
| 12 | 39029-39037 | **grid** hunk, column x=107: P changes one row (`kx`→`Da`), U deletes a row. Taking either side shifted the whole column by one tile (the scaffold had a tile at y=0) | rebuilt by coordinate; no shift |

Tiles (119,132), (128,150) and (135,134) are true conflicts too but sit outside any marker hunk: the textual merge had resolved them silently and partly wrongly, and they are among the 18 tiles the tile-level pass corrected.

The 11 true tile conflicts (all P): (105,138) (105,139) (106,139) (107,139) (108,138) (109,139) (119,132) (127,150) (128,150) (135,134) (146,150). Eight of them are in the engine/atmos corner (x 105-109, y 138-139) where Aquila's `#49` hand-edits collide with upstream's pipe-layer redesign (`Dipity Mapping Fixes #2768`).

**Warning (cannot be proven headless):** that atmos corner now mixes Aquila's pipes on 8 tiles with upstream's redesigned pipes on neighbouring non-conflict tiles. The map loads and the round runs, but I cannot prove the Serendipity atmos network works. **It needs a manual in-game check** (or, as the alternative, reverting Serendipity entirely to the pure Aquila version `Aquila_Serendipity2`; decision for you).

**Serendipity1.dmm** (no markers): the textual merge left `xS` and `Dq` (tiles (98,137) and (116,155)) undefined and 42 tiles different from the merge result (27 of them true conflicts). Same method, P wins on conflicts. Runtime proof: before, loading `serendipity` printed `Undefined model key in DMM: xS`; after, it no longer does.

### 2.5 Content that came from upstream and is already in these maps

This was true at `f847d3caca`, and I did not change it: the baseline's standard Tycoon2 (4,605 tiles), Serendipity2 (~405), Serendipity1 (~364), Snake upper (108) contain upstream-origin tiles that #317's auto-merge brought in. They are required for consistency with the other decks. If you want *pure* pre-#317 Aquila content instead, that is a different repair (for Serendipity and Snake it equals the existing `Aquila_Serendipity` / `Aquila_Snake` forks; for Tycoon it would desynchronise deck 1 and deck 2). Say if you want that.

## 3. Runtime defects

### 3.1 `OVERFLOW_JOB Midshipman` (config, root cause confirmed)
`"Majtek"` **is** Aquila's intentional localisation of Midshipman: `code/__DEFINES/jobs.dm:137` `#define JOB_NAME_ASSISTANT "Majtek" //NSV13 - Midshipmen` (commit `29cbf8284c`, "tłumaczenia", 2024-01-23); `config/jobs.txt` and `config/ranks/royal_navy.txt` already say `Majtek`. Only `OVERFLOW_JOB` still said `Midshipman`, so `GetJob()` returned null in `set_overflow_role`. Fix: `OVERFLOW_JOB Majtek` (config only; the code default is `JOB_NAME_ASSISTANT`). Verified: the `null.allow_bureaucratic_error` runtime is gone.

### 3.2 Nested map load from the gulag room spawner (code)
`SSmapping.LoadStationRooms()` consumes `random_room_spawners` and sets it to `null` *before* `gulag.dmm` loads. The gulag's `/obj/effect/spawner/room/gulag` is therefore not registered; its `Initialize()` calls `template.load()` while `SSatoms.InitializeAtoms()` still holds its map-load state, hence `We started maploading while we were already maploading`. Fix (smallest): the spawner's `Initialize()` now returns `INITIALIZE_HINT_LATELOAD` after its template-availability check and the existing loading logic moved unchanged into `LateInitialize()`, which ends with `qdel(src)` (late loaders run after `clear_tracked_initalize()`). Regular rooms never reach `Initialize` (they are removed by `LoadStationRooms`), so their behaviour is unchanged. Marked `//AQ EDIT`. Verified: the nested runtime at gulag load is gone in every run.

### 3.3 Deck turrets on Atlas/`Aquila_Atlas` (map compatibility; root cause confirmed)
- `deck_guns.dm` `deck_turret/Initialize` does `locate(/obj/machinery/deck_turret/core) in <turf below>` and dereferences the result. That code came with the May-2025 snapshot. Upstream added it in `5437b72693` ("Batch of Fixes (1/?)", 2024-12-27), which **in the same commit retyped `/obj/machinery/deck_turret,` → `/obj/machinery/deck_turret/core,` in its maps** (Atlas, Eclipse1, Galactica2, Gladius2, Babylon2, Hammurabi2, Snake lower, Testship, Tycoon2, Vago2).
- #317 brought the code but the Aquila maps (authoritative, Jan-Apr 2024 base) still use the old type. Hence 2 null dereferences, and, worse, `rack_load` returns FALSE without a core, so those deck guns could not be loaded.
- Fix: the identical one-line change, in the four Aquila-side maps that still had the old key: `Atlas/atlas.dmm`, `Aquila_Atlas/Aquila_Atlas.dmm`, `Atlantis/Atlantis.dmm`, `Aquila_Snake/Aquila_Snake_Lower.dmm`. Each is the key definition `"XX" = ( /obj/machinery/deck_turret, /turf/..., /area/nsv/weapons...)`, a 2-tile object under the turrets. No layout change. **This is a compatibility edit under your exception rule**; it is already documented in the compatibility log below. `Aquila_Atlas`'s other twin `atlas.dmm` is the same map, so both changed identically (same blob `305f278a8e`).
- Verified: standard `atlas` and `aquila_snake` boot with 0 runtimes and a `clean_run`; `Aquila_Atlas` (default map) no longer logs the deck-turret runtimes.

### 3.4 Additional defect: `stasis.dm` Destroy (found by validation, one line)
`/obj/machinery/stasis/Destroy` did `if(op_computer.sbed == src)` with no null check (the original line was `if(op_computer && op_computer.sbed == src)`; Aquila's #233 "Łączenie Stasis Beds" dropped the guard). It runtimed (`Cannot read null.sbed`) whenever an unlinked stasis bed was deleted, e.g. when a dropship boarding level is killed. Restored the guard (`//AQ EDIT`). Verified gone in the matrix runs.

## 4. Map compatibility log (Phase 0.5)

| # | Map(s) | Location | Old | New | Reason | Upstream reference |
|---|---|---|---|---|---|---|
| C1 | Atlas, Aquila_Atlas, Atlantis, Aquila_Snake_Lower | key defs `CG`, `CG`, `fLz`, `OM` | `/obj/machinery/deck_turret` | `/obj/machinery/deck_turret/core` | code (already in baseline) requires the `core` type | `5437b72693` |

Blobs: see `AQUILA_MAP_BASELINE_REPAIRED.tsv` (8 repaired maps in total: 4 above + Tycoon2, snake_upper, Serendipity1, Serendipity2).

## 5. Validation (repaired tree, built from a scratch copy of HEAD + the 11 changed files; the real repo was never built in)

| Check | Result |
|---|---|
| Normal DM compile `build --ci dm` | **PASS** 0 errors, 0 warnings |
| `ALL_MAPS` compile (`-DCIBUILDING -DCITESTING -DALL_MAPS`) | **PASS** 0 errors, 1 warning (`loop_checks`, pre-existing); `Tycoon2` and `snake_upper` now compile |
| dreamchecker 1.8.0 | **PASS** 0 diagnostics |
| tgui build / test / lint / tsc | **PASS** all (4/4 tests) |
| `dmi.test` | PASS (1298 files) |
| Map parse (all 733 `.dmm` through `mapmerge2`) | **PASS** 0 failed (before: `Serendipity2` failed) |
| DreamDaemon start + 40 unit tests + round init | see matrix; 40/40 on 7 of 8 maps |

DreamDaemon exit code is `0xC0000005` after the round-end reboot on every run. It is identical with unmodified upstream on this machine, so I treat it as environmental.

Per-map runs (`data/next_map.json` set to each votable/available map; random gulag ruin; one run each, so intermittent ones can differ from run to run):

| Map | Tests | Runtimes | Notes |
|---|---|---|---|
| default (`aquila_atlas`) | 40/40 | 1 | `null.drop_location()` from a CentCom supply pod's mapload `take_contents` (vanilla `supplypod.dm`, unchanged vs upstream, intermittent) |
| `serendipity` (votable) | 40/40 | 1 | `bad index` (also in the untouched `Aquila_Serendipity`); the `Undefined model key` error is gone |
| `snake` (votable) | 40/40 | **0, clean_run** | was unloadable |
| `tycoon` | 40/40 | 2 | `null -= Cave Bat` (not traced) and a unit-test-vs-overmap-load race (below) |
| `aquila_serendipity` | 40/40 | 2 | `bad index`, `null.beacons_in_ship`: untouched map, pre-existing |
| `aquila_snake` | 40/40 | **0, clean_run** | |
| `atlas` | 40/40 | **0, clean_run** | deck turrets work |
| `atlantis` (not votable) | 39/40 | 64 | 65 runtimes in the **unmodified** baseline too (null `outfit` ×~55, etc.); the failing test (`spawn_humans`, a zombie `drop_loot`) is a random runtime caught by the harness. Pre-existing; Atlantis is not in rotation. |

Original 5 baseline runtimes: 4 gone (job, nested gulag load at spawn, 2× deck turret); the fifth (unit-test zone load overlapping a roundstart overmap interior load) is a **test-harness timing race**: the unit tests start 10 s after roundstart while the overmap mode is still loading Sabre/dropship interiors. It appears in some runs (tycoon above) and not in others.

## 6. Remaining known baseline problems (not fixed)

1. **Atlantis** (`atlantis.json`, not votable): ~65 runtimes at round start (null `outfit` for its jobs, missing `/area/tcommsat/server`, bad docking path, `Director does not have HEALTH_HUD`, etc.). Needs its own pass.
2. **Syndicate Listening Post gulag ruin** (`_maps/RandomRuins/GulagRuins/syndicate_listening.dmm`): vent pumps/scrubbers wired to wrong-type pipes, giving `list index out of bounds` in `set_pipenet` plus an `add_machinery_member` stack trace whenever that random ruin is rolled (reproduced on the original code with the ruin forced). A map-content fix; not touched.
3. **Test-harness race** described above (non-gameplay).
4. **Intermittent**: supply-pod `take_contents(null)`; `Cave Bat` null list (not traced); `bad index` on the two Serendipity variants; `null.beacons_in_ship` on `Aquila_Serendipity`.
5. `deck_turret/Initialize` still dereferences a null core on any map that lacks one (upstream code); no Aquila map in the matrix triggers it now.
6. **Serendipity atmos corner** needs a manual in-game check (§2.4).
7. Config noise from Aquila's `game_options.txt` (`ASSIMILATION`/`DOUBLE_AGENTS` validation, unknown `economy`), invalid research designs `furnace`, `furnace_console`, `sleepy`, donator-loadout warnings, SQL connection refused (no DB here).
8. Unchanged from Phase 0: duplicate `ammo_sorter` designs, `DEFECATION_*` values, orphan `recipes_pie.dm`, 18 unmerged Aquila branches, `check_grep.sh`/changelog scripts not runnable here (no `jq`, system Python without PyYAML).
9. Windows-only: DreamDaemon access violation at shutdown (identical in upstream).

## 7. What I did not do

No commit, no push, no merge/bridge/rebase/cherry-pick, no change to any file other than the 11 listed, no BeeStation content added. The Phase 0 baseline files are unchanged. The repaired map baseline is created but **not authoritative until you approve**.
