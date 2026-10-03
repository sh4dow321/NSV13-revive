# Stage 2 report: weapon datum refactor `84a268eb9e` only

Commit `84a268eb9eece412b7210dc53a981504282471cc` (2025-12-01 19:21 +0100, "The unholy weapon datum refactor (#2820)"), parent `e09bf3db6de385f8b14d39b39232b641f4eb738e` (the Stage 1 boundary). 74 files, +3147/-2156: 8 added, 3 deleted, 63 modified. Safety tag (local): `aquila-stage1-complete` = `7b72f09d65`. No later upstream commit is included.

## Old to new architecture (from the diff, not the commit message)
| Concern | Old (Stage 1) | New (Stage 2) |
|---|---|---|
| Weapon data | `/datum/ship_weapon/*` (35 types) in `nsv13/code/datums/weapon_types.dm` (404 lines) + base in `weapons/ship_weapon.dm` (133 lines), both deleted | `/datum/overmap_ship_weapon/*` in `delta_overmap_ship_weapons/` (8 new files, 1,744 lines): `_overmap_ship_weapon.dm` 383, `new_weapon_types/weapon_datum_types.dm` 549, `autonomy.dm` 259, `firing_checks.dm` 216, `firing.dm` 123, `selection.dm` 112, `physical.dm` 72, `facing_checks.dm` 30 |
| Fire mode id | 18 `FIRE_MODE_*` defines + `MAX_POSSIBLE_FIREMODE`, `WEAPON_CLASS_LIGHT/HEAVY` | removed; each weapon machine sets `weapon_datum_type = /datum/overmap_ship_weapon/<x>`; new `OSW_AMMO_LIGHT/HEAVY/MISSILE/TORPEDO/FREE`, `OSW_FIGHTER_MAIN_WEAPON/SECONDARY_WEAPON`, `COMSIG_STOPPED_PILOTING` |
| Ship storage | `var/list/weapon_types[MAX_POSSIBLE_FIREMODE]`, `fire_mode`, `weapon_numkeys_map`, `torpedoes`, `missiles` | `overmap_weapon_datums` list on `/obj/structure/overmap`; `max_light_shots_left`, `max_shots_left`, `max_missiles`, `max_torpedoes`, resupply timers |
| Machine linking | `fire_mode = FIRE_MODE_MAC`, `set_position(OM)` | `weapon_datum_type = /datum/overmap_ship_weapon/mac`, `link_to_overmap_weapon_datum(OM)`; `apply_weapons()` creates the datums |
| AI | `overmap/ai.dm` (163 lines, deleted): `handle_ai_behaviour`, `ai_target`, `retreat`; `special_fire`, `fire_torpedo` | AI logic in `ai-skynet.dm`; `ai_fire` / `ai_elite_fire` iterate `overmap_weapon_datums` with `OSW_CONTROL_AI`, `get_ammo()`, `can_fire()`, `get_ai_range_penalty()` then `fire_weapon(target, firing_weapon = ...)` |
| Autonomous defence | `/datum/ams_mode`, `handle_autonomy`, `handle_pdc_intercept`, `handle_flak` | `autonomy.dm`: `autonomous_fire`, per-datum `autonomous_handling` (`flak`, `pdc_mount`); `/datum/ams_mode` kept |
| Fighters | `FIRE_MODE_FIGHTER_SLOT_ONE/TWO`, `fighter_primary/secondary` datums | `OSW_FIGHTER_*`, `/datum/overmap_ship_weapon/fighter/{primary,secondary}`, `hardpoint_fire(target, used_ship_weapon, ...)` |
| Selection / UI | `select_weapon`, `swap_firemode`, tgui keyed on fire modes | `selection.dm`; `cycle_firemode(M)` usable by gunner and pilot; new `special_weapon_action` keybind (F); `TacticalConsole.js`, `OrdnanceConsole.js` updated |

## Compatibility check against the Stage 1 tree
Removed symbols: 22 defines (`FIRE_MODE_*`, `MAX_POSSIBLE_FIREMODE`, `WEAPON_CLASS_*`, `FIRE_INTERCEPTED`), 35 type paths (`/datum/ship_weapon/*` plus moved `/datum/ams_mode`, `/obj/machinery/computer/ams`, `/obj/structure/overmap`), and the weapon procs (`special_fire`, `fire_torpedo`, `valid_target`, `weapon_sound`, `swap_firemode`, `handle_*`).
Searched `aquila/`, `code/`, `nsv13/code/`, `config/`, `tgui/`, and all 733 maps in the merged tree:
- `aquila/`: 0 hits. `config/`: 0. `tgui/`: 0 (the two edited UIs are upstream's own change).
- `code/` and `nsv13/code/`: 0 hits for the removed defines or `/datum/ship_weapon`. Remaining matches of `fire_mode` are unrelated identifiers (`pneumaticCannon.dm`; fighter hardpoint arguments in the new code). `cycle_firemode` is still defined in `overmap/verbs.dm` and is called with the user.
- Maps: 0 references to removed paths or vars (matches for `ship_weapon` are `/obj/item/ship_weapon/ammunition/*` items). `ALL_MAPS` compiles and 8 maps boot with no weapon-related runtime.

**Aquila adaptations required: none.** The merge was conflict-free and the result for all 74 files has exactly the upstream commit's stat (+3147/-2156).

## Localization and gameplay preservation
- The 21 Polish lines in the touched Aquila files are all present verbatim: `overmap_mode.dm` 12/12, `damage.dm` 5/5, `ai-skynet.dm` 2/2, `overmap.dm` 2/2; `bitfields.dm`, `carbon.dm`, `nsv13.dme` have none. Polish lines outside `aquila/`: 2,816 before and after. `aquila/` untouched (0 files).
- `AQ EDIT` markers: 459 before and after.
- Aquila changes in the touched files (`overmap_mode.dm`, `damage.dm`, `ai-skynet.dm`, `overmap.dm` announcements; `carbon.dm` throw pacifism; `bitfields.dm`) are intact. No Aquila code references the removed weapon API, so no Aquila weapon behaviour existed to lose.
- Localization debt: see `AQUILA_STAGE2_LOCALIZATION_DEBT.md`. 110 literals on added lines; 70 are Stage 1 English text that was only moved (for example weapon select/failure alerts from the deleted `weapon_types.dm`); **40 are genuinely new** (`_overmap_ship_weapon.dm` 10, `firing_checks.dm` 10, `weapon_datum_types.dm` 4, `overmap.dm` 3, `_fighters.dm` 3, `firing.dm` 2, the two tgui consoles 4, others 4).

## DME and maps
DME: removed `weapon_types.dm` and `ship_weapon.dm`, added the 8 `delta_overmap_ship_weapons` files; no dangling includes; `aquila/aquila.dm` is still last; 304 Aquila includes resolve. (`ai.dm` was not in the DME.) `.dmm` changes: 0. 733 index blobs equal `AQUILA_MAP_BASELINE_REPAIRED.tsv`.

## Validation (scratch export of the merge index, BYOND 515.1633)
Plain compile 0/0; `ALL_MAPS` 0 errors (1 pre-existing `loop_checks` warning); dreamchecker 0 diagnostics; tgui build/test/lint/tsc pass; dmi.test 1,299 files; 733 maps parsed; `dmm_test` 733.

Runtime matrix (40 unit tests each, one run per map):
| Map | Result | Classification |
|---|---|---|
| default (`aquila_atlas`) | 40/40; first run 2 runtimes (Listening Post ruin pipe bug); re-runs: clean, then 1 harness race | PRE-EXISTING |
| serendipity | 40/40, 1 `bad index` | PRE-EXISTING |
| snake | 40/40, clean | same as Stage 1 |
| tycoon | 40/40, 1 `null -= Cave Bat` | PRE-EXISTING |
| aquila_serendipity | 40/40, 2 (`bad index`, `null.beacons_in_ship`) | PRE-EXISTING |
| aquila_snake | 40/40, clean | same |
| atlantis (not votable) | 39/40, 66 runtimes (the failed test is a random runtime caught by the harness, as in Phase 0.5) | PRE-EXISTING |
| atlas | 40/40, 1 `maploading while maploading` (unit-test vs overmap-load race) | PRE-EXISTING (seen in Phase 0.5) |

No weapon-related runtime, no null datum reference, no missing type path, no weapon UI error. **No new Stage 2 regression.** `0xC0000005` at shutdown: environmental.

Scratch-only weapon smoke test (not committed): the main ship has 5 weapon datums (`mac`, `torpedo_launcher`, `vls`, `gauss`, `pdc_mount`); a spawned AI destroyer and cruiser each linked 3 datums; 6 rounds of `ai_fire` between them plus `ai_elite_fire` ran with no runtime. It did not prove that damage was dealt (hit points unchanged), so this is evidence the code paths execute, not that combat is balanced.

## Before / after examples
1. Files: `weapon_types.dm` (404 lines), `ship_weapon.dm` (133), `ai.dm` (163) are gone; 8 files under `delta_overmap_ship_weapons/` (1,744 lines) are new.
2. Defines: `FIRE_MODE_ANTI_AIR 1` through `FIRE_MODE_HYBRID_RAIL 18`, `MAX_POSSIBLE_FIREMODE`, `WEAPON_CLASS_*` removed; `OSW_AMMO_*` and `OSW_FIGHTER_*` added in `nsv13/code/__DEFINES/overmap.dm`.
3. Machine linking (`mac.dm`): `fire_mode = FIRE_MODE_MAC` and `set_position(OM)` became `weapon_datum_type = /datum/overmap_ship_weapon/mac` and `link_to_overmap_weapon_datum(OM)`.
4. Ship state (`overmap.dm`): `weapon_types[MAX_POSSIBLE_FIREMODE]`, `fire_mode`, `torpedoes`, `missiles` became `max_light_shots_left`, `max_shots_left`, `max_missiles`, `max_torpedoes`, resupply timers; weapons live in `overmap_weapon_datums`.
5. AI firing (`ai-skynet.dm`): `ai_fire` now loops `for(var/datum/overmap_ship_weapon/ai_weapon in overmap_weapon_datums)`, filters by `weapon_control_flags & OSW_CONTROL_AI`, uses `get_ammo()` / `can_fire(target)` / `get_ai_range_penalty()`, then `fire_weapon(..., firing_weapon = ...)`.
6. Controls: `cycle_firemode` now accepts the pilot as well as the gunner and passes the user; new `special_weapon_action` keybind (`F`).

Diff totals: 74 files, +3,147/-2,156; 71 `.dm`; 2 tgui (`OrdnanceConsole.js`, `TacticalConsole.js`); 1 DME; 0 config; 0 assets; 0 `.dmm`.
