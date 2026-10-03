# Stage 3 report: upstream through `ff23bf114d`

Boundary `ff23bf114d6c3675661add8d0e4904d6ac0002c8` ("Automatic changelog compile", 2026-06-06 00:49 UTC) is on upstream's first-parent line, position 62 of the 111 commits after Stage 2 (`84a268eb9e` is its ancestor). 62 upstream commits absorbed (31 non-changelog). The next upstream commit is `a3f98e8a10` "Some more random fixes (#2891)" (2026-06-27); Von Neumann (#2656, 2026-06-27) and everything later are **not** included. Safety tag (local): `aquila-stage2-complete` = `c4957efb82`.

Range `84a268eb9e..ff23bf114d`: 65 files (+16,648/-12,391; 5 A, 60 M, 0 D, 0 R): 40 `.dm`, 4 `.dmm`, 4+3 tgui (`HybridWeapons.js` + 3 new Railgun UIs), 3 `.dmi`, 5 json (incl. 4 package.json), config 1 (`game_options.txt`), DME +2 includes. The 4 `.dmm` (Aetherwhisp1, Shrike1, Shrike2, CentCom) are all restored to the baseline. Candidate after map handling: **59 files, +2,943/-325**, 39 `.dm`, 9 tgui (3 new), 3 `.dmi`, 1 config, 1 DME, 0 `.dmm`, 0 under `_maps/`.

## Conflicts (4) and map handling
| File | Upstream | Aquila | Class | Resolution |
|---|---|---|---|---|
| `Aetherwhisp1.dmm`, `Shrike2.dmm` | Shrike 2.0 / Aetherwhisp layout edits | repaired baseline | map | baseline blob restored (also restored the silently merged `Shrike1.dmm`, `CentCom.dmm`) |
| `code/__HELPERS/priority_announce.dm` | `minor_announce(..., silent = FALSE)` + `!silent &&` sound check (#2849) | default title `"Uwaga:"` (Polish) | localization + code | `minor_announce(message, title = "Uwaga:", alert, from, html_encode = TRUE, silent = FALSE)`: Polish default kept, upstream's `silent` arg and sound logic kept |
| `nsv13/.../fleet_combat/combat_handling.dm` | passes `silent = TRUE` to stop stacked "entered combat" noise | Polish announcement text | localization + code | the Polish sentence with `silent = TRUE` appended; same meaning |
Not a Git conflict but a policy decision: `_maps/shrike.json` and `_maps/map_files/Shrike/job_changes.dm` (part of the Shrike 2.0 commit `5204ba5a54`) merged silently. They describe the new Bee layout ("1 railgun munitions forge", new strengths/weaknesses; detective job re-enabled, ore silo made buildable). Per the map policy I **kept Aquila's versions** (`git checkout HEAD --`). Shrike is not in the Aquila rotation (`config/maps.txt`: `map shrike` has no `votable`). `config/maps.txt` and `_maps/_basemap.dm` were not touched by Stage 3; no new map was added.

## Imported systems (not maps)
Railgun Forge (new `railgun_forge.dm`, 1,142 lines; `railgun_ammo.dm` +209; `hybrid_railgun.dm` +298; `projectiles_fx.dm` +35; new tgui `RailgunForge`, `RailgunFiller`, `RailgunCanisterCharger`, `HybridWeapons.js` +171; DME +1); dummy AI pilot mob (`dummy_pilot.dm`, #2885); angle-based `relay_damage` (#2825); AI-traitor minimum population (#2854, new config `AI_TRAITOR_MIN_PLAYERS 20`); `minor_announce` silent flag (#2849); TGS DMAPI 7.3.3 → 7.4.0 (#2868); ore silo "Bumped" throw deposit; singularity floor/openspace fixes (#2852); shuttle-summon exploit fix (#2855); fighter and escape-pod fixes (#2856); trader fixes (#2848); muni pointer / plasma caster board fixes (#2847); CentCom smuggling check on small craft (#2853); outfit backend (#2850); blast shower info (#2846); bartender shotgun fix (#2851, map-only, restored).

## Toolchain
BYOND 515.1633, Node 18.14.2, yarn 3.4.1, Python 3.11.2, rust_g, auxmos, SpacemanDMM: **unchanged** (`dependencies.sh`, `Dockerfile`, CI scripts untouched). Dependency pins changed upstream (reported before installing; installed into the **scratch** build copy only, with your approval): `tgui` webpack `^5.94.0` → `^5.104.1` (resolved 5.105.0), lodash `^4.17.21` → `^4.18.1`, axios `^1.12.0` → `^1.13.5`, js-yaml `^4.1.0` → `^4.1.1`, `tgui/yarn.lock` (+~500 lines); `tools/requirements.txt` Pillow 10.3.0 → 12.2.0. Deployment note: the TGS DMAPI jump to 7.4.0 pairs with TGS server-side support; I could not verify what TGS version the Aquila host runs.

## Preservation
- Polish lines outside `aquila/`: 2,816 before, 2,816 after (0 decrease). `aquila/` untouched (0 files). Maps unchanged.
- `AQ EDIT` markers: 459 before and after.
- Aquila lines absent from the result (all explained): old `minor_announce` signature line (superseded by the same line with `silent`); the old combat-entry Polish line (replaced by the identical Polish line with `silent = TRUE`); `relay_damage(relayed_type)` in `damage.dm` (superseded by upstream's `relay_damage(relayed_type, P.Angle)`); 15 generated lines of `html/changelog.html` (changelog rotation, not Aquila content).
- Files both sides touched in Stage 3: `priority_announce.dm`, `combat_handling.dm`, `security_levels.dm` (Aquila's commented `toggle_gq_lights` timers at lines 58 and 77 are intact beside upstream's Condition Z fix), `traitor.dm` (Aquila Polish untouched; upstream adds `ai_pop_config_check`; the restricted job is `JOB_NAME_AI`, localized "SI Statku", so it follows the define), `ai-skynet.dm` (dummy pilot; Aquila's 2 Polish alerts intact), `overmap.dm` (Aquila's 2 Polish lines intact; upstream adds `railgun_bluespace_recoil`), `damage.dm` (5 Polish announcements intact), `game_options.txt` (`OVERFLOW_JOB Majtek` intact; `AI_TRAITOR_MIN_PLAYERS 20` added), `nsv13.dme`.
- Full-copy overlays (7), `laser.dm`, `camera.dm`, `human_defines.dm`: **upstream did not change any of their originals in Stage 3** (only `security_levels.dm` of the silent-risk list changed, handled above). No overlay hides new behaviour.
- Weapon API residue: 0 references to `FIRE_MODE_*`, `MAX_POSSIBLE_FIREMODE`, `WEAPON_CLASS_*`, `/datum/ship_weapon`.
- DME: +`dummy_pilot.dm`, +`railgun_forge.dm`; no dangling or duplicate includes; `aquila/aquila.dm` still last; 304 Aquila includes resolve.
- Job/config coupling: `OVERFLOW_JOB Majtek` resolves; known pre-existing defects unchanged and untouched: `Dyrektor Naukowr`, `Kurator` (vs `Bibliotekarz`).
- Maps: 733 index blobs equal `AQUILA_MAP_BASELINE_REPAIRED.tsv`; 0 `.dmm` in the diff.
- Localization debt: `AQUILA_STAGE3_LOCALIZATION_DEBT.md/.tsv`: 100 literals on added lines; 15 existing text re-added/moved; **85 genuinely new** (railgun_forge.dm 43, RailgunForge.js 9, railgun_ammo.dm 8, RailgunFiller.js 7, hybrid_railgun.dm 6, RailgunCanisterCharger.js 3, others 9); nothing translated.

## Validation (BYOND 515.1633, scratch export of the final merge index)
Plain compile 0 errors / 0 warnings; `ALL_MAPS` 0 errors (1 pre-existing `loop_checks` warning); dreamchecker 0 diagnostics; tgui build (webpack 5.105.0), test, lint, tsc all pass; dmi.test 1,299 files (also with Pillow 12.2.0); 733 maps parsed, `dmm_test` 733.
Runtime matrix (40 unit tests each, one run per map), identical in character to Stage 2: default 40/40, 1 harness race; serendipity 1 `bad index`; snake clean; tycoon 2 (`Cave Bat`, harness race); aquila_serendipity 2; aquila_snake clean; atlantis 40/40, 65 runtimes (pre-existing); atlas clean. In the default-map runs the only "bad map" regex hit was `Failed to load antag reputation` (fresh data directory, environmental). **No new Stage 3 regression.** `0xC0000005` at shutdown: environmental.
Targeted smoke test (scratch-only, passed with 0 runtimes from the tested systems): AI ship pilot is a `/mob/living/dummy_pilot` and is deleted with its ship; `relay_damage` for 9 angles; `railgun_forge` created, `RefreshParts`, `process`, `ui_data`; AI-traitor restriction applies at low pop; `minor_announce` with and without `silent`.

## Physical before/after examples
1. **Dummy pilot (#2885)**: before `pilot = new /mob/living(get_turf(src))` plus manual `name`, `mouse_opacity`, `alpha`, `forceMove(src)`; after `new /mob/living/dummy_pilot(src)` with a `COMSIG_PARENT_QDELETING` handler (`dummy_pilot.dm`, `ai-skynet.dm`).
2. **Damage relay (#2825)**: before `relay_damage(proj_type)` picked `pick(GLOB.cardinals)`; after `relay_damage(proj_type, original_proj_angle)` derives the side from `(720 + angle - ship angle) % 360`, with `projectile_quadrant_impact(P)` (`damage.dm`, `armour_quadrant.dm`).
3. **AI traitor min pop (#2854)**: new `ai_pop_config_check()` in `traitor.dm`, new config entry `ai_traitor_min_players` (default 20) and `AI_TRAITOR_MIN_PLAYERS 20` in `game_options.txt`.
4. **Railgun Forge (#2837 code)**: new 1,142-line `railgun_forge.dm`, 3 new tgui panels, `HybridWeapons.js` +171, `railgun_ammo.dm` +209.
5. **Combat noise (#2849)**: `minor_announce(..., silent = FALSE)`, `if(!silent && (M.client.prefs.toggles & PREFTOGGLE_SOUND_ANNOUNCEMENTS))`; callers in `combat_handling.dm` pass `silent = TRUE`.
6. **TGS DMAPI (#2868)**: `TGS_DMAPI_VERSION` "7.3.3" → "7.4.0" (`code/__DEFINES/tgs.dm`, `code/modules/tgs`, 10 files).
7. **Shuttle-summon exploit (#2855)**: `security_levels.dm` zebra-downgrade branch changed from `else` to `else if(GLOB.security_level == SEC_LEVEL_BLUE)`.
8. **Ore silo (`machine_silo.dm`)**: new `Bumped()` that deposits thrown stacks into the silo's material container.
9. **Singularity (#2852)**: `reinf_floor.dm` / `openspace.dm`: the singularity-eats-floor proc was rewritten (upstream comment: "adjusted for proper behavior (singularity resistance, non-infinite draining)"), with an early return below `STAGE_FIVE`.
10. **Fighters/pods (#2856, #2853)**: `docking_act` brake engagement and `controlled_weapon_datum` guard; `GetAllContents` on small craft so smuggling checks see ship-level contents.
