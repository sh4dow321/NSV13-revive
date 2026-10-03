# Aquila Preservation Manifest (Stage 1-4 checklist)

Generated in Phase 1.5 (analysis only; HEAD = bridge `d253381c64`; base `fa47d5fe77`; target `upstream/master` `69565b730f`). Per-file detail for **every** Aquila-modified file outside `aquila/` (892) is in [AQUILA_PRESERVATION_MANIFEST.tsv](AQUILA_PRESERVATION_MANIFEST.tsv) (columns: kind, added/removed lines, Polish lines, `AQ EDIT` markers, upstream-changed, both-sides, textual conflict, upstream stages, risk, modularization class, sample Aquila commits). This file lists the significant entries. Kinds and classes are heuristic (content + commit titles), not a line-by-line review.

Legend. **Modularized**: YES = lives in `aquila/` as an added type or a composing overlay; PARTIAL = overlay plus in-place hooks; NO = in-place edit of an upstream-owned file. **Stage**: 1 = upstream up to just before the weapon-datum refactor (`84a268eb9e`), 2 = the refactor itself, 3 = up to `ff23bf114d`, 4 = the rest to `upstream/master`; `-` = upstream never touches it. **Class** A/B/C/D/E as defined in the audit section 4.

## Global invariants to verify after every stage

- `aquila/` untouched (579 files) and `aquila/aquila.dm` still the **last** include in `nsv13.dme`; all 304 includes resolve.
- 457 `AQ EDIT` markers in 201 files outside `aquila/` still present (per-file counts in the TSV).
- Aquila-added/changed lines vs `fa47d5fe77` present in every manifest file (multiset check). Polish line totals not lower than baseline: 2,816 outside + 368 inside + 8 in maps.
- 733 `.dmm` blobs equal `AQUILA_MAP_BASELINE_REPAIRED.tsv`; no `vonneumann` in `config/maps.txt`, `_basemap.dm` or the DME.
- Config/data coupling: every `config/jobs.txt` / `ranks/*` / `OVERFLOW_JOB` string resolves to a `JOB_NAME_*` define value (known failures today: `Dyrektor Naukowr`, `Kurator`).
- Compile (plain + `ALL_MAPS`), dreamchecker and unit tests per the Phase 0.5 matrix.

## A. `aquila/` (modular layer): 306 `.dm`, 202 `.ogg`, 71 `.dmi`

| Group (`aquila/code/...`) | Category | Files | Lines | Polish lines | Procs | Type-level var assignments (all) | Mirrored upstream path | Modularized | Risk / stage | Requirement |
|---|---|---|---|---|---|---|---|---|---|---|
| `modules/antagonists` | GAMEPLAY (infiltrator, demons, diseases, morph, role prefs) | 29 | 2390 | 12 | 174 | 400 | 3 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `game/objects` | GAMEPLAY + signs/contraband LOCALIZATION | 48 | 1648 | 184 | 75 | 636 | 38 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/research` | GAMEPLAY (nanites, designs, techweb) | 23 | 1052 | 9 | 55 | 582 | 20 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `game/gamemodes` | GAMEPLAY (vampire, infiltration) | 13 | 1020 | 2 | 80 | 212 | 2 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/mob` | GAMEPLAY (species, mobs, speech) | 64 | 859 | 5 | 87 | 114 | 61 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/clothing` | GAMEPLAY + LOCALIZATION | 9 | 547 | 31 | 16 | 261 | 8 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `game/machinery` | GAMEPLAY (computers, fabricators, jukebox, SMES) | 11 | 442 | 18 | 39 | 41 | 9 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `__DEFINES` | INFRASTRUCTURE | 18 | 418 | 3 | 2 | 0 | 17 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `` | GAMEPLAY/OTHER | 1 | 304 | 0 | 0 | 0 | 0 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `datums/mutations` | GAMEPLAY | 4 | 297 | 1 | 20 | 140 | 4 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `datums/diseases` | GAMEPLAY | 2 | 282 | 0 | 17 | 50 | 1 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `datums` | GAMEPLAY/OTHER | 8 | 182 | 42 | 6 | 46 | 8 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/requests` | GAMEPLAY/OTHER | 1 | 143 | 6 | 6 | 0 | 1 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/events` | GAMEPLAY/OTHER | 4 | 140 | 3 | 10 | 26 | 0 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `controllers/subsystem` | CONFIGURATION/GAMEPLAY | 2 | 122 | 1 | 9 | 0 | 1 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/admin` | GAMEPLAY/OTHER | 3 | 116 | 1 | 5 | 0 | 2 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `datums/martial` | GAMEPLAY/OTHER | 1 | 115 | 0 | 9 | 4 | 0 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `datum/wires` | GAMEPLAY/OTHER | 1 | 113 | 3 | 5 | 2 | 0 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/cargo` | GAMEPLAY/OTHER | 2 | 101 | 3 | 2 | 35 | 2 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/uplink` | GAMEPLAY/OTHER | 2 | 97 | 1 | 2 | 54 | 2 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/point` | GAMEPLAY/OTHER | 1 | 89 | 0 | 4 | 5 | 0 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/reagents` | GAMEPLAY/OTHER | 4 | 86 | 0 | 5 | 28 | 3 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/vending` | GAMEPLAY/OTHER | 4 | 86 | 3 | 0 | 79 | 4 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/power` | GAMEPLAY/OTHER | 4 | 81 | 0 | 8 | 13 | 3 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/spells` | GAMEPLAY/OTHER | 3 | 76 | 9 | 3 | 15 | 2 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/surgery` | GAMEPLAY/OTHER | 3 | 66 | 6 | 5 | 7 | 2 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `datums/keybinding` | GAMEPLAY/OTHER | 3 | 64 | 0 | 6 | 3 | 1 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/food_and_drinks` | GAMEPLAY/OTHER | 4 | 61 | 11 | 0 | 50 | 4 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `datums/weather` | GAMEPLAY/OTHER | 1 | 59 | 3 | 5 | 16 | 0 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| `modules/mining` | GAMEPLAY/OTHER | 2 | 58 | 0 | 8 | 8 | 1 | YES | LOW, stage - | PRESERVE UNCHANGED; keep include last |
| (remaining 18 groups) | mixed | 31 | 396 | 11 | | | | YES | LOW | PRESERVE UNCHANGED |

**Assets**: `aquila/sound` (202 `.ogg`: 128 voice, 29 machines, 17 misc, 10 effects, 9 ambience, ...) and `aquila/icons` (71 `.dmi`) are referenced by path from `aquila/` and from in-place hooks. Upstream never touches either directory. PRESERVE UNCHANGED.

### A.1 Mirrored-path overlays whose upstream original changed E to master (6)

| Aquila file | Upstream change | Interaction | Stage | Risk |
|---|---|---|---|---|
| `aquila/code/__DEFINES/inventory.dm` | comment/define text for `WEIGHT_CLASS_HUGE/GIGANTIC` | Aquila copy only adds `civilian_storage_allowed` lists; no clash | 1 | LOW |
| `aquila/code/__DEFINES/is_helpers.dm` | adds `iscolortext` | Aquila adds `is_infiltrator` etc.; disjoint | 1 | LOW |
| `aquila/code/controllers/configuration/entries/general.dm` | adds `client_max_version` | Aquila adds `rp_filter_enabled`; disjoint | 4 | LOW |
| `aquila/code/modules/mob/living/carbon/carbon.dm` | `throw_icon?.` null-safety | Aquila redefines `update_sight`; different proc | 2 | LOW |
| `aquila/code/modules/mob/living/carbon/human/emote.dm` | cat scream sound in `get_scream_sound` | Aquila redefines `fart` and `cry/get_sound`; different procs | 1 | LOW |
| `aquila/code/modules/projectiles/guns/energy/laser.dm` | rewrites `desc` of two lasers | **Aquila `desc` override (English, old text) hides upstream's new description** | 4 | LOW (cosmetic, silent) |

### A.2 Full-copy overlays (no `..()`): pin an old upstream body and hide later upstream changes

- `/obj/item/powersink/process` (26 lines): the upstream file did not change E to master. Requirement: PRESERVE; re-review after Stage 4. Class C.
- `/datum/surgery_step/extract_implant/success` (25): the upstream file did not change E to master. Requirement: PRESERVE; re-review after Stage 4. Class C.
- `/datum/nanite_program/nanite_sting/on_trigger` (20): the upstream file did not change E to master. Requirement: PRESERVE; re-review after Stage 4. Class C.
- `/datum/admins/one_click_antag` (19): the upstream file did not change E to master. Requirement: PRESERVE; re-review after Stage 4. Class C.
- `/obj/item/screwdriver/abductor/get_belt_overlay`, `/datum/species/ipc/post_death`, `/obj/machinery/gravity_generator/part/get_status` (2 lines each): the upstream file did not change E to master. Requirement: PRESERVE; re-review after Stage 4. Class C.

## B. Files changed by both Aquila and upstream (38): the only places Git can conflict

| Path | Kind | What each side did | Polish | AQ markers | Merge (dry run) | Stage | Requirement |
|---|---|---|---|---|---|---|---|
| `_maps/_basemap.dm` | CONFIG | Aquila map includes; upstream adds Von Neumann include (strip) | 0 | 1 | clean | 4 | verify Aquila lines/markers present |
| `_maps/map_files/Aetherwhisp/Aetherwhisp2.dmm` | MAP |  | 0 | 0 | clean text | 1 | revert to baseline blob (authoritative map) |
| `_maps/map_files/Hammerhead/Hammerhead.dmm` | MAP |  | 0 | 0 | clean text | 1 | revert to baseline blob (authoritative map) |
| `_maps/map_files/Serendipity/Serendipity1.dmm` | MAP |  | 0 | 0 | CONFLICT | 1 | revert to baseline blob (authoritative map) |
| `_maps/map_files/Serendipity/Serendipity2.dmm` | MAP |  | 0 | 0 | CONFLICT | 1 | revert to baseline blob (authoritative map) |
| `_maps/map_files/Shrike/Shrike2.dmm` | MAP |  | 0 | 0 | CONFLICT | 3 | revert to baseline blob (authoritative map) |
| `_maps/map_files/Tycoon/Tycoon2.dmm` | MAP |  | 0 | 0 | clean text | 4 | revert to baseline blob (authoritative map) |
| `_maps/map_files/generic/CentCom.dmm` | MAP |  | 0 | 0 | clean text | 3 | revert to baseline blob (authoritative map) |
| `code/__DEFINES/is_helpers.dm` | GAMEPLAY |  | 0 | 0 | clean | 1 | verify Aquila lines/markers present |
| `code/__HELPERS/_lists.dm` | GAMEPLAY | Aquila adds `list_clear_nulls`; upstream adds `unique_list_in_place` | 0 | 0 | CONFLICT | 1 | hand-resolve: upstream logic + Aquila strings |
| `code/__HELPERS/priority_announce.dm` | LOCALIZATION | Aquila Polish default titles; upstream adds `silent` arg to `minor_announce` | 2 | 0 | CONFLICT | 3 | hand-resolve: upstream logic + Aquila strings |
| `code/_globalvars/bitfields.dm` | GAMEPLAY |  | 0 | 0 | clean | 2 | verify Aquila lines/markers present |
| `code/game/gamemodes/traitor/traitor.dm` | MIXED |  | 2 | 0 | clean | 3 | verify Aquila lines/markers present |
| `code/modules/food_and_drinks/kitchen_machinery/processor.dm` | GAMEPLAY |  | 0 | 1 | clean | 1 | verify Aquila lines/markers present |
| `code/modules/library/lib_machines.dm` | GAMEPLAY |  | 0 | 0 | clean | 1 | verify Aquila lines/markers present |
| `code/modules/mob/camera/camera.dm` | GAMEPLAY |  | 0 | 1 | clean | 1 | verify Aquila lines/markers present |
| `code/modules/mob/living/carbon/carbon.dm` | GAMEPLAY | Aquila throw pacifism; upstream `throw_icon?.` | 0 | 1 | clean | 2 | verify Aquila lines/markers present |
| `code/modules/mob/living/carbon/human/emote.dm` | MIXED | Aquila fart/thirst emotes + Polish; upstream cat scream | 13 | 3 | clean | 1 | verify Aquila lines/markers present |
| `code/modules/mob/living/carbon/human/human_defines.dm` | GAMEPLAY |  | 0 | 1 | clean | 1 | verify Aquila lines/markers present |
| `code/modules/projectiles/guns/energy/laser.dm` | LOCALIZATION | Aquila desc tweaks; upstream rewrites descs | 3 | 5 | clean | 4 | verify Aquila lines/markers present |
| `code/modules/research/xenobiology/crossbreeding/_structures.dm` | GAMEPLAY |  | 0 | 0 | clean | 1 | verify Aquila lines/markers present |
| `code/modules/security_levels/security_levels.dm` | GAMEPLAY | Aquila comments out `toggle_gq_lights` timers; upstream Condition Z edits | 0 | 2 | clean | 3 | verify Aquila lines/markers present |
| `code/modules/tgs/v5/undefs.dm` | GAMEPLAY |  | 0 | 0 | clean | 1,3 | verify Aquila lines/markers present |
| `config/config.txt` | CONFIG | Aquila 20 lines; upstream additions | 0 | 0 | clean | 4 | verify Aquila lines/markers present |
| `config/game_options.txt` | MIXED | Aquila 117 lines (OVERFLOW_JOB Majtek, Polish); upstream additions | 7 | 0 | clean | 3 | verify Aquila lines/markers present |
| `config/maps.txt` | CONFIG | Aquila rotation; upstream adds `vonneumann` (must NOT be taken) | 0 | 0 | CONFLICT | 4 | hand-resolve: upstream logic + Aquila strings |
| `html/changelog.html` | INFRASTRUCTURE | changelog pipeline | 0 | 0 | clean | 1,3,4 | verify Aquila lines/markers present |
| `html/changelogs/.all_changelog.yml` | INFRASTRUCTURE | changelog pipeline | 0 | 0 | clean | 1,3,4 | verify Aquila lines/markers present |
| `nsv13.dme` | INFRASTRUCTURE | Aquila include last; upstream adds about 62 includes and removes 3 | 0 | 0 | clean | 1,2,3,4 | verify Aquila lines/markers present |
| `nsv13/code/controllers/subsystem/overmap_mode.dm` | LOCALIZATION | Aquila 12 Polish lines; upstream refactor/hard-mode edits | 12 | 0 | clean | 2 | verify Aquila lines/markers present |
| `nsv13/code/modules/munitions/ship_weapons/ballistic_weapons/revision2/automation.dm` | GAMEPLAY | Aquila duplicate ammo_sorter designs (+31); upstream ammo rack changes | 0 | 0 | clean | 1 | verify Aquila lines/markers present |
| `nsv13/code/modules/overmap/FTL/ftl_jump.dm` | LOCALIZATION |  | 5 | 0 | clean | 1 | verify Aquila lines/markers present |
| `nsv13/code/modules/overmap/ai-skynet.dm` | LOCALIZATION |  | 2 | 0 | clean | 1,2,3,4 | verify Aquila lines/markers present |
| `nsv13/code/modules/overmap/fleet_combat/combat_handling.dm` | LOCALIZATION | Aquila Polish combat-entry announcement; upstream passes `silent = TRUE` | 1 | 0 | CONFLICT | 3 | hand-resolve: upstream logic + Aquila strings |
| `nsv13/code/modules/overmap/overmap.dm` | LOCALIZATION |  | 2 | 0 | clean | 1,2,3,4 | verify Aquila lines/markers present |
| `nsv13/code/modules/overmap/weapons/damage.dm` | LOCALIZATION |  | 5 | 0 | clean | 2,3,4 | verify Aquila lines/markers present |
| `nsv13/code/modules/power/stormdrive.dm` | LOCALIZATION | Aquila Polish alerts + explosion signal; upstream `empulse(src,...)` + typo fix | 2 | 1 | clean | 1,4 | verify Aquila lines/markers present |
| `nsv13/code/modules/research/astrometrics.dm` | MIXED | Aquila Polish scan radio lines; upstream broadcast/mute rework | 1 | 0 | CONFLICT | 1 | hand-resolve: upstream logic + Aquila strings |

## C. Localization outside `aquila/` (222 files, about 2,816 Polish lines): significant entries

Top 45 by Polish lines; the full list is in the TSV. Modularized = NO for all of them.

| Path | Kind | Polish | Class | Upstream changed | Stage | Risk | Notes |
|---|---|---|---|---|---|---|---|
| `strings/ion_laws.json` | LOCAL | 322 | A | N | - | LOW | data file |
| `strings/names/last_female.txt` | LOCAL | 198 | A | N | - | LOW |  |
| `strings/names/last_male.txt` | LOCAL | 198 | A | N | - | LOW |  |
| `code/game/area/Space_Station_13_areas.dm` | MIXED | 163 | B | N | - | LOW | area names; B candidate |
| `nsv13/code/game/area/areas.dm` | LOCAL | 126 | B | N | - | LOW | area names; B candidate; maps reference these area types |
| `code/datums/traits/negative.dm` | LOCAL | 118 | B/C | N | - | LOW | quirk names/descriptions |
| `code/datums/ai_laws.dm` | LOCAL | 98 | B/C | N | - | LOW | AI law texts (aquila/ has an overlay file too) |
| `code/modules/mob/living/emote.dm` | LOCAL | 75 | B/C | N | - | LOW | emote messages |
| `code/modules/antagonists/blob/powers.dm` | LOCAL | 67 | C | N | - | LOW |  |
| `code/datums/traits/good.dm` | LOCAL | 64 | B | N | - | LOW | quirk names/descriptions |
| `strings/names/first_male.txt` | LOCAL | 64 | A | N | - | LOW |  |
| `code/datums/action.dm` | LOCAL | 57 | B/C | N | - | LOW |  |
| `code/game/gamemodes/objective.dm` | LOCAL | 56 | B/C | N | - | LOW |  |
| `strings/hallucination.json` | LOCAL | 55 | A | N | - | LOW |  |
| `strings/phobia.json` | LOCAL | 52 | A | N | - | LOW |  |
| `code/modules/client/preferences_toggles.dm` | LOCAL | 40 | C | N | - | LOW |  |
| `code/modules/mob/living/simple_animal/parrot.dm` | LOCAL | 37 | C | N | - | LOW |  |
| `strings/names/first_female.txt` | LOCAL | 36 | A | N | - | LOW |  |
| `strings/sillytips.txt` | LOCAL | 36 | A | N | - | LOW |  |
| `code/modules/food_and_drinks/food/snacks_pastry.dm` | LOCAL | 33 | B | N | - | LOW |  |
| `strings/wanted_message.json` | LOCAL | 33 | A | N | - | LOW |  |
| `code/modules/vending/wardrobes.dm` | MIXED | 32 | B | N | - | LOW |  |
| `code/datums/traits/neutral.dm` | LOCAL | 26 | B/C | N | - | LOW |  |
| `code/modules/events/ion_storm.dm` | LOCAL | 26 | C | N | - | LOW |  |
| `nsv13/code/modules/jobs/security/weapons.dm` | LOCAL | 22 | B | N | - | LOW | weapon-job text |
| `code/game/gamemodes/objective_items.dm` | LOCAL | 21 | B | N | - | LOW |  |
| `code/modules/events/shuttle_loan.dm` | LOCAL | 19 | C | N | - | LOW |  |
| `code/__DEFINES/jobs.dm` | MIXED | 15 | A | N | - | LOW |  |
| `code/modules/clothing/masks/hailer.dm` | LOCAL | 15 | C | N | - | LOW |  |
| `code/modules/crew_objectives/civilian_objectives.dm` | LOCAL | 15 | C | N | - | LOW |  |
| `strings/dreamstrings.txt` | LOCAL | 15 | A | N | - | LOW |  |
| `code/game/objects/items/melee/misc.dm` | LOCAL | 14 | C | N | - | LOW |  |
| `strings/numbers_as_words.txt` | LOCAL | 14 | A | N | - | LOW |  |
| `code/modules/mob/living/carbon/human/emote.dm` | MIXED | 13 | B | Y | 1 | MEDIUM |  |
| `code/modules/mob/living/carbon/life.dm` | LOCAL | 13 | C | N | - | LOW |  |
| `code/modules/mob/living/simple_animal/friendly/drone/_drone.dm` | LOCAL | 13 | B | N | - | LOW |  |
| `code/modules/antagonists/blob/overmind.dm` | LOCAL | 12 | C | N | - | LOW |  |
| `code/modules/antagonists/blob/structures/_blob.dm` | MIXED | 12 | C | N | - | LOW |  |
| `nsv13/code/controllers/subsystem/overmap_mode.dm` | LOCAL | 12 | C | Y | 2 | MEDIUM | overmap mode names/announcements; refactor stage 2 |
| `code/game/objects/items/storage/backpack.dm` | MIXED | 11 | B | N | - | LOW |  |
| `code/modules/mob/dead/new_player/new_player.dm` | MIXED | 10 | C | N | - | LOW |  |
| `config/ranks/corporate.txt` | MIXED | 10 | A | N | - | LOW |  |
| `config/ranks/military.txt` | MIXED | 10 | A | N | - | LOW |  |
| `config/ranks/royal_navy.txt` | MIXED | 10 | A | N | - | LOW |  |
| `html/antagtips/blob.html` | LOCAL | 10 | A | N | - | LOW |  |

Coupled definitions/data (class A, must change together): `code/__DEFINES/jobs.dm` (`JOB_NAME_*` incl. `Majtek`, `Dyrektor Naukowy`, `Bibliotekarz`), `config/jobs.txt`, `config/ranks/*.txt`, `config/game_options.txt` (`OVERFLOW_JOB Majtek`), `strings/*`, `html/antagtips/*`, `code/__DEFINES/DNA.dm` (`POLISH` mutation replaces `SWEDISH`).

## D. In-place gameplay hooks (`AQ EDIT`): 457 markers in 201 files

Distribution: `code/modules` 102 files, `code/game` 45, `nsv13/code` 16, `code/__DEFINES` 12, `code/datums` 11, `code/controllers` 5, `code/_globalvars` 5, `code/__HELPERS` 4. Also changed upstream (9, stage in brackets): `_maps/_basemap.dm` [4], `processor.dm` [1], `camera.dm` [1], `carbon.dm` [2], `human/emote.dm` [1], `human_defines.dm` [1], `laser.dm` [4], `security_levels.dm` [3], `stormdrive.dm` [1,4]. All have their Aquila lines present in the simulated merge. Requirement for every one: marker and logic preserved; no modularization attempted. Currently modularized: NO (PARTIAL where a signal/type lives in `aquila/`).

Notable hooks: `stormdrive.dm` sends `COMSIG_GLOB_STROMDRIVE_EXPLOSION` (consumed by `aquila/.../implant_explosive.dm`); `munitions_machinery.dm` vends `clothing/head/aquila/headband/orange`; `munitions_trolley.dm` defines `munitions_trolley_dummy` (used by Aquila maps); `starsystem_manager.dm` admin Delete; `code/__DEFINES/*` `CONTRACT_PLEASURE`, heretic defines, `COMSIG_*`.

## E. Upstream-owned files deleted or replaced by Aquila

| Path | Aquila action | Replacement | Upstream touched since base? | Requirement |
|---|---|---|---|---|
| `code/modules/antagonists/eldritch_cult/*` (11 files), `eldritch_demons.dm` | deleted (heretic rework from Bee) | `code/modules/antagonists/heretic/*` + hooks | no | keep deleted; make sure no stage re-adds them |
| `code/modules/antagonists/disease/disease_abilities.dm` | deleted | `aquila/code/modules/antagonists/disease/disease_abilities.dm` | no | keep deleted; DME includes the aquila copy |
| `code/game/machinery/dance_machine.dm` | deleted | `nsv13/code/game/machinery/dance_machine.dm` (modified) + `aquila/.../jukebox.dm` | no | preserve both |
| 7 title screens, 9 `sound/roundend/*.ogg` | deleted | `config/title_screens/images/aqg.dmi` | no | keep deleted |

## F. Weapon datum refactor (Stage 2, `84a268eb9e`)

| Item | Aquila state | Migration risk | Requirement |
|---|---|---|---|
| weapon datum names/descriptions | untouched (English upstream text) | nothing to lose | n/a |
| `FIRE_MODE_*`, fire modes | no Aquila usage; 0 hits in `aquila/`, in Aquila-modified files, and in the 733 maps (340 naval var-edits checked) | none | n/a |
| Aquila Polish in refactored files | `ai-skynet.dm` 2, `overmap.dm` 2, `damage.dm` 5, `overmap_mode.dm` 12; all survive the simulated merge | verify the strings are still reachable | PRESERVE; check in game |
| 108 new English literals in weapon files | n/a | localization debt | report only |
| `automation.dm` duplicate ammo_sorter designs | stale copy | duplicate design ids stay | keep; user decision later |

## G. Configuration, data, assets, maps (identified, not modified)

- `config/` (19 files vs base: `maps.txt`, `jobs.txt`, `ranks/*`, `admins.txt`, `config.txt`, `game_options.txt`, `roleplay_filter.txt`, `awaymissionconfig.txt`, title screens): PRESERVE; `maps.txt`, `game_options.txt` and `config.txt` overlap upstream.
- `strings/` (19 localized files) and `html/antagtips` (5): PRESERVE UNCHANGED (upstream never touches them).
- Map-embedded Polish text: 8 lines in `Atlas/atlas.dmm`, `atlas2.dmm`, `Aquila_Atlas/Aquila_Atlas.dmm`, `Aquila_Atlas2.dmm` (identified only; maps are authoritative).
- Binary assets outside `aquila/` changed vs base: 165 (icons/sounds/title screens). PRESERVE.
- Tooling: `tools/build/build.js` (`aquila/**` input) and `.github/workflows/*` (4 files, runner/guard overrides): PRESERVE.

## H. Feature ownership: Aquila commits by number of upstream-owned text files they touched (still differing from base)

| Commit | Author | Title | Outside files |
|---|---|---|---|
| `798eb7cf78` | Dejaku51 | Heretycy z bee (#90) | 130 |
| `f6ba7befbf` | srawek | Kompilacja dotychczasowych zmian Jokera (bez portow) (#5) | 108 |
| `68f6fed534` | Joker66613 | tweaks (#305) | 37 |
| `b5e858a7d6` | Joker66613 | Pakiet tłumaczeń (#73) | 37 |
| `5eb467af54` | Boletus | Tłumaczenie Bloba oraz resprite (#198) | 25 |
| `1556e9e232` | MACIEKBAKI | Dodaje małpi hełm, skiny borgów + jakąś tam krew (#78) | 24 |
| `983d8e4883` | Boletus | krwiste dźwięki (#48) | 22 |
| `c6c79490a7` | smorgli | ser (#202) | 21 |
| `a1ed6322cf` | MACIEKBAKI | tłumaczy bełkot naukowców + imiona (#58) | 20 |
| `45b6028725` | MACIEKBAKI | Dodaje potrzebe picia do gry (#215) | 17 |
| `da783fe989` | Boletus | tłumaczenia (#53) | 17 |
| `1ae7970a12` | srawek | Aktualizacja do NSV13 (#68) | 16 |
| `50d2a6ea4e` | MACIEKBAKI | Requested feature (#287) | 15 |
| `05f4c1eb7a` | Joker66613 | Space Dragon Rework (#238) | 14 |
| `4dc1cb3e9d` | MACIEKBAKI | reszta tłumaczeń i idk co jeszcze (#59) | 13 |
| `17bfd0190e` | Unibel | Bugfixy & tweaki 29-02-2024 - Unibel (#256) | 11 |
| `1791acf040` | Joker66613 | zmiany z dotychczasowych gier (#79) | 11 |
| `29cbf8284c` | MACIEKBAKI | tłumaczenia część kolejna [PORT 1062] (#42) | 11 |
| `6ad578ed2f` | Joker66613 | Cortex folding (#244) | 10 |
| `38e9c4938b` | Joker66613 | Polski dubbing, away missions, tweaki (#193) | 10 |
| `d3558d807f` | Joker66613 | Machanie przedmiotami i jakieś inne bajery (#235) | 9 |
| `6c0756af9e` | MACIEKBAKI | joker rozdaje drain bamage (#60) | 9 |
| `87de9f67b5` | Boletus | dodanie kostiumow, qol modularyzacji, ikony error dla braku sprite (#4 | 9 |
| `75c863e7c4` | srawek | Cele załogi - tłumaczenie [Port 540] (#66) | 8 |
| `61f714e5a2` | MACIEKBAKI | Ambiencja [port 605] (#34) | 8 |
