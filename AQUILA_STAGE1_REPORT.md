# Stage 1 report: merge of upstream NSV13 through `e09bf3db6d`

Boundary `e09bf3db6de385f8b14d39b39232b641f4eb738e` (2025-11-29 00:19 UTC, "Automatic changelog compile"). 102 upstream commits after `fa47d5fe77` (position 102 of 214); the next upstream commit is `84a268eb9e` (weapon datum refactor, position 103), **not included**. Safety tag (local): `aquila-pre-upstream-migration` = `d253381c64`.

## Conflicts reached (4) and resolutions
| File | Resolution |
|---|---|
| `_maps/map_files/Serendipity/Serendipity1.dmm`, `Serendipity2.dmm` | authoritative: restored the baseline blob (`git checkout HEAD --`); no Bee content |
| `code/__HELPERS/_lists.dm` | upstream's VV port adds `unique_list_in_place` and its own, byte-identical `list_clear_nulls`; Aquila's only change in the file was that same proc. Result = upstream's file (inspection: 9 Aquila lines in the diff vs the base, all the identical proc) |
| `nsv13/code/modules/research/astrometrics.dm` | upstream now builds `message` and does `say(message)` + `radio.talk_into(message)` behind a broadcast toggle. Both scan-start branches use `message = "Rozpoczynanie skanowania: [scan_target]"` (Aquila's Polish text, `//AQ EDIT`). Complete-scan Polish radio line merged cleanly |

Maps silently changed by the merge and restored to the baseline: `Aetherwhisp1`, `Aetherwhisp2`, `Hammerhead`, `mining11`, and the 6 `_maps/templates/boarding/syndicate/*.dmm`. No new upstream map was added in this range. `config/maps.txt`, `_basemap.dm` were not touched by Stage 1.

## Preservation results (mechanical, 659 manifest text files)
- Polish lines outside `aquila/`: 2,816 before, 2,816 after (0 decrease). `aquila/`: untouched (0 files changed), 368. Maps: unchanged.
- `AQ EDIT` markers: 457 before, 459 after (+2 are the two I added in `astrometrics.dm`); 201 marker files intact.
- Aquila-added/changed lines missing from the result: only the 2 old `radio.talk_into(... "Rozpoczynanie skanowania ...")` lines in `astrometrics.dm`, superseded by the `message =` lines above (translation kept).
- Job/config coupling: `OVERFLOW_JOB Majtek` resolves. Known pre-existing failures unchanged and not touched: `Dyrektor Naukowr`, `Kurator`.
- DME: `// AQUILA` / `#include "aquila/aquila.dm"` is still the last include; 304 Aquila includes resolve; upstream added 5 includes (`debug_variable_appearance.dm`, `vv_ghost.dm`, `fluff.dm`, `teleportation.dm`, `dept_signs.dm`).
- 733 maps: index blobs equal `AQUILA_MAP_BASELINE_REPAIRED.tsv` (0 mismatches, 0 `.dmm` in the candidate diff).

## Silent-risk areas
`camera.dm` (upstream `movement_type = FLYING`, Aquila `return FALSE`: disjoint, both present), `human_defines.dm` (`can_buckle = FALSE` + Aquila `gender_ambiguous`: both present; `aquila/` only sets `can_buckle` on a crucifix), `security_levels.dm` (not touched until Stage 3). The 7 full-copy `aquila/` overlays, `laser.dm`, `security_levels.dm`: **their upstream originals did not change in Stage 1**. No Stage 1 file redefined by `aquila/` changed (the 5 scanner hits are macro noise).

## Localization debt
See `AQUILA_STAGE1_LOCALIZATION_DEBT.md` (79 new English literals in 19 files; none translated).

## Validation (scratch export of the merge index; BYOND 515.1633)
plain compile 0/0; `ALL_MAPS` 0 errors, 1 pre-existing `loop_checks` warning; dreamchecker 0 diagnostics; tgui build/test/lint/tsc pass; dmi.test 1299 files; map parser 733/0 failed; `mapmerge2.dmm_test` 733 parsed.
Runtime (`dm-test`, 40 unit tests each, one run per map):
| Map | Tests | Runtimes | Classification vs repaired baseline |
|---|---|---|---|
| default `aquila_atlas` | 40/40 | first run 4 (`supplypod take_contents(null)`) + 1 bad-map-message match; re-run: **0, clean_run** | PRE-EXISTING intermittent (same supplypod runtime seen in the Phase 0.5 baseline); the bad-map message did not reproduce |
| serendipity | 40/40 | 1 `bad index` | PRE-EXISTING (same in baseline) |
| snake | 40/40 | 0, clean | same as baseline |
| tycoon | 40/40 | 1 `null -= Cave Bat` | PRE-EXISTING |
| aquila_serendipity | 40/40 | 2 (`bad index`, `null.beacons_in_ship`) | PRE-EXISTING |
| aquila_snake | 40/40 | 0, clean | same |
| atlas | 40/40 | 0, clean | same |
| atlantis (not votable) | 40/40 | 62 | PRE-EXISTING (65 at original baseline, 64 at Phase 0.5) |
**New Stage 1 regressions: none found.** `0xC0000005` at shutdown: environmental.

## Physical code update examples (not maps)
1. **View-variables port (#2798, `5bdf419101`)**. Before: no `vv_ghost.dm` / `debug_variable_appearance.dm`. After: both new (104 and 228 lines), `code/modules/admin` +701/-119, new includes in the DME.
2. **Astrometrics broadcast mute (#2800, `5272c6ee0f`)**. Before: every scan start/complete did `say()` + `radio.talk_into()` unconditionally. After: `var/message`, a `broadcast` flag in `ui_data`, and a "Broadcast Scanning" toggle in `tgui/.../Astrometrics.js`; Polish text carried over.
3. **Xenobio server-freeze fix (#2812, `1cc813d9ff`)**. Before: `_structures.dm` cerulean crystals used `while(crystals < 3)` with random turf rejection (could stall). After: bounded `for(var/iter = 0; iter < 3; iter++)` over a candidate list with a fallback list.
4. **Stormdrive repair sprites (#2803, `92093ab899`) + ruin fix (#2835)**. Before: placeholder `icon_state = "broken"` marked `//TEMP`, core crate used `icons/obj/crates.dmi`. After: per-step states (`repair-1`, `reinforce-2`, `refit-3`), `nsv13/icons/obj/control_rod.dmi` `stormdrive_core`, new description; Aquila's Polish alerts and explosion signal intact.
5. **Soundtrack music for encounters (#2814, `f677c80202`)**. Before: `OM.play_music(pick(audio_cues))` in `ai-skynet.dm` and `ftl_jump.dm`. After: `play_soundtrack_music(pick(audio_cues), volume = 100)`; four new tracks under `sound/soundtrack/`.
6. **TGS DMAPI (#2818)**: `TGS_DMAPI_VERSION` 7.3.0 → 7.3.3 (`code/modules/tgs`, +30/-3).
7. **Buckle fix (#2804, `ccc074074a`)**: `buckling.dm` message now goes to `user` and excludes `user == src`. **KNPC failsafe (#2792, `1bc27b056c`)**: new `failsafe_timer` / `failsafe_trust` vars in `knpc.dm`. **Cat scream (#2830)**: new `scream_cat.ogg` + `get_scream_sound` branch.
Totals vs the pre-merge tree: 83 files, +1,793/-526; 62 `.dm`; 11 added (5 `.dm`, 1 `.dmi`, 5 `.ogg`); 0 deleted; 4 tgui files (`Astrometrics.js`, `Starmap.js`, `yarn.lock`, dev-server package.json); 3 html (changelog, admin css); 2 tools/ci scripts; 0 config; 1 DME; 0 `.dmm`.
