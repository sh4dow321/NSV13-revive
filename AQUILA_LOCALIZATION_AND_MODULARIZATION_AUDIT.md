# Phase 1.5: Aquila localization and modularization audit

Analysis only. HEAD stays at the Phase 1 bridge `d253381c64`; no tracked file, config, asset or map was touched, nothing was committed or pushed. The companion checklist is `AQUILA_PRESERVATION_MANIFEST.md` (+ `.tsv` with every file).

Baselines used: effective upstream base `fa47d5fe77`; target `upstream/master` `69565b730fdf45c09861dfca6c669b67f8601a71`; Aquila side = `HEAD` (tree identical to `34fa11431f`).

## 0. Answer

> **Can modern BeeStation/NSV13 be merged while preserving Aquila localization simply by preserving `aquila/`? NO.**

Evidence:
- `aquila/` holds **368** Polish-bearing lines (53 of 306 `.dm` files, 9 of them mostly localization).
- **Outside `aquila/`** there are **222 text files with ~2,816 Polish lines** (strings/config/html included), spread over upstream-owned files such as `nsv13/code/game/area/areas.dm`, `code/game/area/Space_Station_13_areas.dm`, `code/datums/traits/*.dm`, `code/datums/ai_laws.dm`, `code/modules/mob/living/emote.dm`, `strings/*`, plus job-name defines and `config/*` that are coupled to them.
- Only part of the outside localization can be moved into `aquila/` without changing behaviour, and moving it is risky (§4). Stage 1-4 must therefore keep it in the upstream-owned files and protect it with a **mechanical preservation check** (§8), not rely on `aquila/`.

What *is* true: `aquila/` is a genuine, mostly well-behaved downstream **gameplay** layer (§1). It is just not where the localization lives.

## 1. `aquila/` audit

| Item | Value |
|---|---|
| Tracked files | **579**: 306 `.dm`, 202 `.ogg`, 71 `.dmi` |
| `.dm` code | 11,510 non-blank lines in 306 files (304 `#include`d, `aquila.dm` itself, 1 orphan: `recipes_pie.dm`) |
| Layout | `aquila/code/` mirrors `code/` (`modules` 175, `game` 79, `datums` 22, `__DEFINES` 19, `controllers` 4, `__HELPERS` 4, others), `aquila/sound` (128 voice, 29 machines, 17 misc, ...), `aquila/icons` (40 obj, 23 mob, ...) |
| Entry into build | one line at the very end of `nsv13.dme`: `// AQUILA` / `#include "aquila/aquila.dm"` / `// END_AQUILA`, **after** the `donator.dm` `#endif`; `tools/build/build.js` adds `'aquila/**'` as a DM input |
| Internal order | `__DEFINES` → `__HELPERS` → `_globalvars` → `_onclick` → `controllers` → `datums` → `game` → `modules` (alphabetical inside) |

### 1.1 Classification of the 306 `.dm` files

| Class | Files | Notes |
|---|---|---|
| **GAMEPLAY** | ~225 (remainder) | vampire gamemode (9 + HUD), infiltrator (7 + gamemode), demons (8), diseases (5, sentient disease), nanites (10), research designs (23 files under `modules/research`), 64 mob/species files, items (27), structures/effects (20), clothing (9), machinery (11), spells, surgery, mutations (4), martial arts, jukebox, metacoin, energy harvester, plort machine, space dragon rework, heretic hooks |
| **LOCALIZATION** (mostly) | 9 | `game/objects/effects/contraband.dm` (109 Polish lines), `datums/ai_laws.dm` (42), `modules/clothing/clothing.dm` (23), `signs_warning.dm` (18), `signs_departments.dm` (17), `structures/crucifix.dm` (15), `signs_maps.dm` (9), `spells/.../godhand.dm` (9), `drinks.dm` (8) |
| Mixed (some Polish strings in gameplay files) | 44 | e.g. `jukebox.dm`, `battery.dm`, `request.dm`, `gender_reassignment.dm`, `barsigns.dm`, `alert.dm` |
| **CONFIGURATION** | 2 | `controllers/configuration/entries/{game_options,general}.dm` (new config entries) |
| **INFRASTRUCTURE** | ~25 | `__DEFINES/*` (19, incl. `span.dm`, `traits.dm`, `uplink.dm`...), `__HELPERS/*` (4) |
| **ASSETS** | 273 non-`.dm` | 202 sounds (mostly Polish voice lines/ambience), 71 `.dmi` |
| OTHER | 1 | orphan `recipes_pie.dm` (never compiled) |

(`aquila/` classification of `.dm` is by dominant content: Polish-line ratio, declaration types, and the Aquila commit titles for those paths; I did not read all 11.5k lines.)

### 1.2 What it extends and what it overrides (DM semantics verified)
Scan of types/procs/vars (indent-based scanner) over `aquila/` vs `code/`, `nsv13/`, `goon/`, `interface/`:
- **827 types** declared, **205 of them re-opened** types that already exist outside `aquila/` (extensions);
- **72 hits** of "(type, proc) also defined outside `aquila/`", of which **65 are real** proc redefinitions (the rest are macros mis-parsed): **58 call `..()`** and so compose with whatever the earlier definition is; **7 are full copies** that silently pin an old implementation: `powersink/process`, `surgery_step/extract_implant/success`, `nanite_program/nanite_sting/on_trigger`, `admins/one_click_antag`, `screwdriver/abductor/get_belt_overlay`, `species/ipc/post_death`, `gravity_generator/part/get_status`;
- **267 var overrides** of vars declared elsewhere (`name`, `desc`, sounds, ...);
- **224 `aquila/` files sit at a path that also exists outside** (mirror layout); they are partial overlays (4-line files up to 380 lines), not whole-file copies;
- 264 `#define`s, 10 of them also defined outside (`CLAMPED_OFF`, `DISCONNECTED`, `OPERATING`, `SIN_*`, `VIABLE_MOB_CHECK`).

**Dream Maker semantics, tested here with BYOND 515.1633** (a 3-file toy project compiled and run, results reproduced both ways):

| Experiment | Result |
|---|---|
| Same-type proc redefined in a later-included file | compiles with **0 warnings**; the **later include wins** |
| Redefinition that calls `..()` | `..()` reaches the **earlier definition of the same type** (not only the parent type), i.e. it composes |
| Include order swapped | the other definition wins: **order is behaviour** |
| `var` default overridden by `name = "X"` in one file and declared `var/name = "Y"` in a later file | the **override wins regardless of order** (override vs declaration is order-independent; two *overrides* of one var are order-dependent) |
| `parent_type` | unaffected by include order for var defaults (child type saw the override) |

Consequences for Aquila: because `aquila/` is included **last**, its redefinitions always win over upstream's (good for preserving behaviour, but they also **hide upstream's later changes** when an overlay is a full copy), its var overrides are safe against include reordering, and a `#define` redefined there affects only code compiled after it.

### 1.3 Dependencies on upstream implementation details
- 58 `..()` overlays depend only on the existence of the proc signature (low).
- 7 full-copy procs depend on upstream bodies as of `fa47d5fe77` (list above). None of their files changed upstream since the base (**checked**), so no staleness risk in Stage 1-4, but any later upstream change will be silently hidden.
- `aquila/` contains **no** reference to the weapon-datum API (`FIRE_MODE_*`, `WEAPON_CLASS_*`, `/datum/ship_weapon`, `/datum/ams_mode`, `firemode`): 0 hits.
- Cross-link outside→inside: `COMSIG_GLOB_STROMDRIVE_EXPLOSION` (defined in `code/__DEFINES/dcs/...`, sent from `stormdrive.dm`, consumed by `aquila/.../implant_explosive.dm`), `munitions_trolley_dummy`, `/obj/item/clothing/head/aquila/headband/orange`, 201 files with **457 `AQ EDIT` markers** reaching into aquila logic.

### 1.4 Is `aquila/` a modular downstream layer?
**Partly.** For *added* features (vampires, infiltrators, nanites, mobs, items) yes: new types, composing overlays, includes last. It breaks down in three places: (1) **localization, which is in-place edits to upstream files**; (2) 7 full-copy overlays; (3) 201 files with `AQ EDIT` hooks in upstream files, plus about 200 further Aquila-modified upstream `.dm` gameplay files that carry no marker (413 gameplay files in total, heuristic).

## 2. Aquila localization and changes outside `aquila/`

Method: `git diff fa47d5fe77 HEAD` excluding `aquila/` (892 files: 456 text with no Polish, 216 text with Polish, 165 binary assets, 55 maps); Polish detection = Polish diacritics **or** ≥2 strong Polish function words per line (so English→Polish rewrites without diacritics are caught but a short single-word rename like `Midshipman`→`Majtek` is only found through the coupling checks below); every file was also attributed to its Aquila commits (176 pre-#317 commits).

| Area outside `aquila/` | Polish-bearing files | Polish lines |
|---|---|---|
| `strings/` | 19 | 1,028 (ion laws 322, names, hallucination, phobia, tips, wanted_message...) |
| `code/modules/` | 112 | 677 |
| `code/datums/` | 13 | 389 (traits `negative`/`good`, `ai_laws`, `action`) |
| `code/game/` | 33 | 346 (`area/Space_Station_13_areas.dm` 163, `objective.dm`) |
| `nsv13/code/` | 27 | 245 (`area/areas.dm` 126, `jobs/security/weapons.dm` 22, `overmap_mode.dm` 12, KNPC species, courier/shakedown, announcements) |
| `config/` | 7 | 56 |
| `html/` | 5 | 43 (antag tips) |
| `code/__DEFINES/`, `__HELPERS/`, `controllers/` | 6 | 32 (`JOB_NAME_*` etc.) |
| **Total** | **222** (142 mostly-localization, 80 mixed) | **~2,816** |
| `.dmm` maps | 4 | 8 lines (`atlas.dmm`, `Aquila_Atlas.dmm` and their second decks). Identified only, not touched. |

By code location (Polish `.dm` lines at HEAD, 1,682 total):
- **1,031 type-level variable assignments** (`name = "…"`, `desc = "…"` inside type blocks). Overridable by a `var` override (class B).
- **522 inside proc bodies** (`to_chat`, `priority_announce`, `say`, `visible_message`, `send_alert`). Only editable in place or by copying the proc (class C).
- 23 on `#define` lines (job names, class A), 103 in lists/data, 3 comments.

Major localized systems: job titles and ranks, area names, traits/quirks, AI laws, emotes, objectives, blob powers, announcements (`priority_announce`, `minor_announce`, overmap/ship/stormdrive alerts), ion laws and name lists, hallucinations/phobias, antag tips, Beepsky/robots speech, mob examine texts, vending text, config job lists.

**Localization is also coupled to data files**: confirmed example `OVERFLOW_JOB`; this audit found two more of the same class in `config/jobs.txt` (**pre-existing, not fixed here**): `Dyrektor Naukowr` (typo; the define is `Dyrektor Naukowy`) and `Kurator` (the define is now `Bibliotekarz`), so their slot counts from config never apply.

### 2.1 Non-localization Aquila modifications outside `aquila/` (≈420 text files)
Classified from content and the 176 Aquila commits (heuristic, not line-by-line reading): GAMEPLAY 413 files (+9 BUGFIX), 18 CONFIG, 10 INFRASTRUCTURE, 165 ASSET, 55 MAP. 199 of the GAMEPLAY files are "significant" (≥60 added lines or an `AQ EDIT` marker). The biggest items:
- Bee/tg **ports** of whole systems: heretic rework (`code/modules/antagonists/heretic/*`, +~5k lines, replacing the deleted `eldritch_cult` tree), Space Dragon rework, genetics 2.0 / mutations, borgs, nanites 2.0, jukebox (moved from `dance_machine.dm` to nsv13), sentient disease (deleted `disease_abilities.dm` replaced by `aquila/.../disease_abilities.dm`);
- balance/gameplay tweaks in place (`security_levels.dm` comments out `toggle_gq_lights` timers; Midshipman slots 5→100; `DEFECATION_*` values; `camera.dm`; `carbon.dm` throw pacifism; `human_defines.dm`);
- sound/asset hooks, drinks/thirst system, ERT/random-role pool, config rotation (`maps.txt`, `jobs.txt`, ranks, `roleplay_filter.txt`).

## 3. Localization vs gameplay, per significant item
Per-file classification (LOCALIZATION / MIXED / GAMEPLAY / BUGFIX / CONFIG / ASSET / MAP / INFRASTRUCTURE) is in `AQUILA_PRESERVATION_MANIFEST.tsv` (column `kind`). Stage 1-4 must preserve both independently: localization is the **line-level** content of 222 files; gameplay is the **457 markers** plus the ~420 gameplay files.

## 4. Modularization safety classes (A/B/C/D/E)

| Class | Meaning | Count (LOCALIZATION+MIXED files) | Examples |
|---|---|---|---|
| **A** must remain in the upstream-owned file | data files, `#define`s, anything read by code at compile time or from a data path | 36 | `strings/*` (19), `html/antagtips`, `config/*`, `code/__DEFINES/jobs.dm` |
| **B** could safely be implemented from `aquila/` | file consists of type-level `name/desc = "…"` overrides only | 64 | `nsv13/code/game/area/areas.dm`, `code/game/area/Space_Station_13_areas.dm`, `traits/good.dm`, `snacks_pastry.dm`, `wardrobes.dm` |
| **B/C** mostly B with a few strings in procs | | 13 | `traits/negative.dm`, `ai_laws.dm`, `emote.dm`, `action.dm`, `objective.dm` |
| **C** needs architectural work | strings inside proc bodies | 109 | `blob/powers.dm`, `preferences_toggles.dm`, `parrot.dm`, `ion_storm.dm`, `shuttle_loan.dm`, `priority_announce.dm`, overmap/stormdrive announcements |
| **D** obsolete/dead | none confirmed in the rewritten files | 0 | (`recipes_pie.dm` in `aquila/`, never compiled; counted outside this table) |
| **E** uncertain | | 0 | |

(Class is assigned per file from where its Polish lines sit; `…/modularization_class` in the TSV.)

Why B is only "could", not "do":
1. **Type must exist.** A var override on a type upstream renames/deletes silently creates a new empty type (no error, the translation is lost). Area names are keyed by type paths that maps reference.
2. **Overrides are order-independent but not exclusive.** Two overrides of the same var are order-dependent; moving translations into `aquila/` also disables upstream's own later value without notice (`laser.dm` below is a live example).
3. **Strings inside procs (class C) cannot be overridden from outside**: the only way is a full copy of the proc (the 7 stale-copy pattern) or a refactor to a named string/define in the upstream file, which creates *more* conflict surface.
4. **Defines (class A)**: a redefinition in a later file changes only code compiled after it. `JOB_NAME_*` is used in earlier files (`GetJob(JOB_NAME_…)`), so the value would differ per file. Job names must stay in the define file.
5. **Data coupling**: `config/jobs.txt`, `config/ranks/*`, `game_options.txt`, `maps.txt` and `strings/*` are matched by exact string to the localized names; they must stay consistent with the define values regardless of where code lives.
I am not claiming anything is movable beyond class B, and even B should wait until after the migration.

## 5. Upstream migration risk (vs `upstream/master` 69565b730f, base `fa47d5fe77`)

A simulated merge (`git merge-tree`, nothing written to the tree) was inspected line by line for the 31 non-map files both sides changed:
- **Aquila-added/changed lines lost in the merged result: 0** (also 0 Polish lines lost). Of 31 files, 26 merge clean, 5 conflict (`_lists.dm`, `priority_announce.dm`, `maps.txt`, `combat_handling.dm`, `astrometrics.dm`); for the conflicted ones both sides' lines are present in the markers.
- **English leakage** (Aquila-replaced English line reappearing): 0 in the 31 files.
- **Upstream file deletions** (3: `weapon_types.dm`, `overmap/ai.dm`, `weapons/ship_weapon.dm`): Aquila never modified them, so no modify/delete conflicts and no lost Aquila content. **No relocation** of any Aquila-modified file by upstream (git rename detection over E..master finds none affecting them).
- **Renamed type/proc**: the 35 removed type paths are all weapon-datum API and the 18 removed proc names are almost all weapon/AI procs (§6); none is used by `aquila/`, by any Aquila-modified outside file, or by the 733 maps (340 var-edits checked).
- **Duplicate definitions**: pre-existing `ammo_sorter` designs (automation.dm vs `nsv_circuitboard_designs.dm`) stay; upstream changes `automation.dm` in Stage 1 (clean).

### 5.1 Clean merge but silently wrong (the dangerous ones)
| # | Case | Why Git can't see it | Effect |
|---|---|---|---|
| 1 | `aquila/.../projectiles/guns/energy/laser.dm` overrides `desc` of `/obj/item/gun/energy/laser/retro/old/research` (old English text); upstream rewrites that desc (Solgov wording) | override beats declaration regardless of order | upstream's new description is permanently hidden (cosmetic) |
| 2 | **224 upstream-new English multi-word string literals** will arrive untranslated (108 in the weapon-datum files, 71 other `nsv13/code`, 36 `code/modules`, 5 tgui) | new lines, no overlap | English text in a Polish client; a localization debt list, not a loss |
| 3 | 7 full-copy overlays pin old bodies | none of their files changed upstream so far | future silent staleness |
| 4 | Aquila `security_levels.dm` hunks that comment out `toggle_gq_lights` sit next to upstream's Condition Z edits | clean 3-way merge, yet behaviour must be re-verified in game | possible changed alert flow |
| 5 | `OVERFLOW_JOB`-style data coupling (two more found) | config is not code | slot counts / ranks ignored |
| 6 | `human_defines.dm` (upstream `can_buckle = FALSE`) vs Aquila `gender_ambiguous` var; `camera.dm` (`movement_type = FLYING` vs `return FALSE`) | disjoint hunks | likely benign, needs a play test |
| 7 | Upstream ships `config/maps.txt` and `_basemap.dm` entries for `vonneumann` | clean in `_basemap.dm` | would add a new ship to ALL_MAPS compile if not stripped (map policy) |

## 6. Weapon datum refactor (`84a268eb9e`, 74 files, Stage 2)
- Aquila **does not** modify weapon datum names, descriptions, fire modes, `FIRE_MODE_*` usage, weapon UI strings, weapon configuration or behaviour: `weapon_types.dm`, `ship_weapon.dm`, `ai.dm`, all `delta_overmap_ship_weapons/*`, `ship_weapons/**` weapons, tgui weapon UIs have **no Aquila Polish and no `AQ EDIT`**; `aquila/` has 0 references to the API.
- Aquila-modified files that the refactor also touches (7): `bitfields.dm`, `carbon.dm`, `nsv13.dme`, `overmap_mode.dm` (12 Polish lines), `ai-skynet.dm` (2), `overmap.dm` (2), `damage.dm` (5). In the simulated merge **all 21 Polish lines survive** (2/2, 2/2, 5/5, 12/12).
- Must be **reapplied/adapted**: nothing; must be **checked**: that the translated announcements (`ai-skynet.dm`: Typhoon signature / SolGov plea; `overmap.dm`: reactor supercritical / lurch; `damage.dm`: 5 structural-integrity announcements) still sit in the code paths the refactor kept.
- Risk is localization debt: 108 new English literals in the weapon files (`railgun_forge.dm` 36, `weapon_datum_types.dm` 31, `_fighters.dm` 18, `_overmap_ship_weapon.dm` 10, `firing_checks.dm` 9).
- Also touched by the refactor and relevant to Aquila maps: `FIRE_MODE_*` defines removed; no Aquila map references them (checked).

## 7. DME / include architecture
- One include line at the end of `nsv13.dme`; `aquila/aquila.dm` pulls 304 files. Upstream adds 62 DME lines (new files) and removes the 3 weapon files; the merge of `nsv13.dme` is clean in the dry run, but after each stage the Aquila block must still be **last** (position is behaviour, §1.2) and all 304 includes must resolve.
- Do not add `aquila/` earlier; do not reorder inside `aquila.dm`.
- A static include-existence check passes on HEAD and should run per stage.

## 8. Recommended preservation strategy (what Stage 1-4 must use in addition to `aquila/`)
1. **Merge normally; do not move translations.** Git preserves the in-place localization in every file only Aquila changed (all but the 38 overlapping files). Only 38 files overlap and 8 conflict; resolve the 5 non-map ones by hand (upstream logic + Aquila strings) and the 3 map ones by the authoritative-map rule.
2. **Add a mechanical preservation check run after every stage** (to be written in Stage 1, read-only, not part of the repo's CI):
   a. every file in the manifest: all lines that Aquila added/changed vs `fa47d5fe77` are still present (multiset line check; I simulated it: 0 lost);
   b. the `AQ EDIT` marker count (457 in 201 files) is unchanged and every marker file still has its marker;
   c. total Polish-line count is not lower than the baseline (2,816 outside + 368 inside + 8 maps);
   d. config/data coupling: every `config/jobs.txt`, `ranks/*`, `OVERFLOW_JOB` value resolves to a `JOB_NAME_*` define (it currently fails for `Dyrektor Naukowr`, `Kurator`);
   e. `nsv13.dme` ends with the `aquila/aquila.dm` include and all includes exist;
   f. a **localization-debt report**: upstream-added multi-word literals (224 expected) listed for the owner, not auto-translated;
   g. map baseline comparison (733 blobs).
3. Treat the 7 full-copy overlays and the `laser.dm` desc override as items to re-review at the end of Stage 4 (not to fix during migration).
4. After the migration, a *separate* project can modularize class B (and maybe C via named strings); not before.

## 9. Final state
HEAD `d253381c64` (bridge); no tracked file changed; 733 maps match `AQUILA_MAP_BASELINE_REPAIRED.tsv`; nothing committed or pushed. New untracked documents only.
