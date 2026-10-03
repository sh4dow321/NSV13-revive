# Stage 4 report: final migration to `upstream/master`

**STAGE4_TARGET** (frozen after `git fetch upstream`): `69565b730fdf45c09861dfca6c669b67f8601a71`, "Automatic changelog compile [ci skip]", 2026-09-12 00:59 UTC. Upstream had **not moved** beyond the hash seen at the start of the project. 49 commits after `ff23bf114d` (27 non-changelog). Safety tag (local): `aquila-stage3-complete` = `dc630da07b`.

Range `ff23bf114d..69565b730f`: 50 files, 9 added, 41 modified, 0 deleted, 0 renamed; 27 `.dm`, 7 `.dmi`, 5 `.dmm` (Babylon2, Tycoon1, Tycoon2, plus the added `vnmk3.dmm` and `cargo_aiship.dmm`), tgui deps only (3 package.json/yarn.lock; no tgui interface changes), config 2 (`config.txt`, `maps.txt`), DME +1 include.

## Von Neumann handling (code in, map out)
| Upstream item | Decision |
|---|---|
| `_maps/map_files/vonneumann/vnmk3.dmm`, `_maps/shuttles/cargo/cargo_aiship.dmm` | **removed** (new Bee maps) |
| `_maps/vonneumann.json`, `_maps/vonneumann.dm` (`FORCE_MAP`), `_maps/map_files/vonneumann/job_changes.dm` | **removed** (map activation / map-bound metadata; upstream never included the job_changes file in `map_changes.dm`) |
| `_maps/_basemap.dm` include of `VNmk3.dmm`, `config/maps.txt` `map vonneumann ... votable` | **not taken** (Aquila versions kept; the only upstream change to both files was the Von Neumann block) |
| `/datum/map_template/shuttle/cargo/aiship` (name "cargo ferry (Von Neumann)") in `nsv13/code/datums/shuttles.dm` | **removed**: it points at the removed shuttle map; the file ends identical to Stage 3 |
| `nsv13/code/modules/overmap/types/solgov.dm` (`/solgov/vnc`, `vnc/starter`, `vnc/ai`, `apply_light_ai_weapons`), `fleet_types.dm` (adds `vnc/ai` to the SolGov fleet destroyer list), `projectiles_fx.dm`, `__DEFINES/overmap.dm`, new SolGov overmap sprites (`destroyer/frigate/frigate_old.dmi`), `solgov_floor.dmi`, decals | **kept**: generic overmap code and assets; it adds an AI SolGov frigate that can appear in SolGov fleets. It does not make the Von Neumann map playable. |
Result: 0 `vonneumann`/`VNmk3` references outside upstream-neutral code, `maps.txt` and `_basemap.dm` identical to Stage 3.

## Conflicts
One Git conflict, plus the policy handling above.
| File | Upstream | Aquila | Class | Resolution | Why |
|---|---|---|---|---|---|
| `config/maps.txt` | adds `map vonneumann ... votable` | Aquila rotation | map config | kept the Stage 3 (Aquila) file | the upstream change to this file is only the Von Neumann entry; nothing else needed |
Silent merges handled: `Babylon2.dmm`, `Tycoon1.dmm`, `Tycoon2.dmm` restored to the baseline blobs; `_basemap.dm` restored (Von Neumann include only).

## Preservation
- Polish lines outside `aquila/`: 2,816 before and after; `aquila/` untouched (0 files, 368); maps unchanged. 0 decrease.
- `AQ EDIT` markers: 459 before and after (201 files). Stage 4 touched two marker files: `laser.dm` (marker lines intact) and `stormdrive.dm` (Aquila Polish alerts and explosion signal intact).
- Aquila lines absent from the result: only 1 generated line of `html/changelog.html`.
- Manifest files touched: `laser.dm`, `config.txt`, changelog files, `nsv13.dme`, `ai-skynet.dm` (null-safety `QDELETED(OM)` checks), `overmap.dm` (thrust overlay moved), `damage.dm` (`LAZYINITLIST(proj.impacted)`), `stormdrive.dm` (`empulse(src, 3, 5)`, message wording). All coexist with the Aquila changes; none became redundant.
- **Full-copy overlays (final review)**, upstream implementation E (`fa47d5fe77`) → target: `powersink/process` UNCHANGED, `extract_implant/success` UNCHANGED, `nanite_sting/on_trigger` UNCHANGED, `one_click_antag` UNCHANGED, `screwdriver/abductor/get_belt_overlay` UNCHANGED, `species/ipc/post_death` UNCHANGED, `gravity_generator/part/get_status` UNCHANGED. None hides new upstream behaviour.
- `laser.dm`: upstream rewrote the `desc` of `retro/old` ("A modern lasergun, used by Solgov's ...") and `retro` (Solgov wording). The Aquila overlay `aquila/.../laser.dm` still sets the old English `desc` on `/obj/item/gun/energy/laser/retro/old/research`, so the **research variant keeps the old text** (cosmetic only; other variants use upstream's). Not changed (report only).
- Silent-risk files: `security_levels.dm` (changed upstream in Stage 3; Aquila's commented `toggle_gq_lights` timers verified intact at lines 58 and 77), `camera.dm` (changed in Stage 1), `human_defines.dm` (changed in Stage 1). No change in Stage 4.
- Weapon API residue: 0 references to `FIRE_MODE_*`, `MAX_POSSIBLE_FIREMODE`, `WEAPON_CLASS_*`, `/datum/ship_weapon`.
- DME: +`overmap_pointed.dm` only; no dangling or duplicate includes; `aquila/aquila.dm` is last; 304 Aquila includes resolve; `aquila/` has 0 changed files.
- Config/jobs: `OVERFLOW_JOB Majtek` resolves; `config.txt` gains upstream's `CLIENT_MAX_VERSION 515`. Known defects unchanged and untouched: `Dyrektor Naukowr`, `Kurator` (vs `Bibliotekarz`).
- Maps: 733 index blobs equal `AQUILA_MAP_BASELINE_REPAIRED.tsv`; 0 `.dmm` in the diff; 733 `.dmm` tracked.
- Localization debt: `AQUILA_STAGE4_LOCALIZATION_DEBT.md/.tsv`: 15 genuinely new English literals (+4 relocated; the Von Neumann cargo-ferry name was removed with its datum). Cumulative genuinely-new literals Stages 1-4 (heuristic): 79 + 40 + 85 + 15 = **219**.

## Toolchain
`dependencies.sh`, `Dockerfile`, CI scripts, rust_g, auxmos, SpacemanDMM, BYOND 515.1633, Node 18.14.2, yarn 3.4.1, Python 3.11.2: **unchanged in Stage 4**. Dependency pins that changed: `tools/requirements.txt` Pillow 12.2.0 → 12.3.0; `tgui/yarn.lock` and `tgui/packages/*/package.json` (dependabot bumps: ws, fast-uri, js-yaml, brace-expansion, browserslist, postcss, axios and similar). Installed into the scratch copy only: Pillow 12.3.0; tgui resolved webpack 5.105.0. TGS DMAPI: unchanged in Stage 4 (stays `7.4.0` from Stage 3). Deployment note, still open: the Aquila host's TGS server version is unknown; DMAPI 7.4.0 may need a matching server.

## Validation (BYOND 515.1633, scratch export of the final merge index)
Plain compile 0 errors / 0 warnings; `ALL_MAPS` 0 errors (1 pre-existing `loop_checks` warning); dreamchecker 0 diagnostics; tgui build (webpack 5.105.0), test, lint, tsc pass; dmi.test 1,302 files on Pillow 12.3.0; 733 maps parsed; `dmm_test` 733.
Runtime matrix (40 unit tests each, one run per map): default 40/40 clean_run; serendipity 1 `bad index`; snake clean; tycoon clean; aquila_serendipity 2; aquila_snake clean; atlantis 64 (pre-existing); atlas 2 (Syndicate Listening Post ruin pipe bug, random ruin, pre-existing). **No new Stage 4 regression.** `0xC0000005`: environmental. The "bad-map" regex hit on default is `Failed to load antag reputation` (fresh data directory).
Targeted smoke (scratch-only, passed): a Von Neumann AI frigate (`solgov/vnc/ai`) links 4 weapon datums and fires; the SolGov fleet lists it; the new tier-6 "compact shield generator array" fighter plating exists and a SolGov fighter spawns; `pointed()` on an overmap target runs; `CLIENT_MAX_VERSION` loads as 515. (A first smoke run hit `current_system.system_contents` null in `ai_process` because I spawned the ship outside the star system, a test artifact in unchanged code; with the ship registered there was 0 runtime.)

## Physical before/after examples (not maps)
1. **Overmap pointing (#2911, `8767d43eb8`)**: new `nsv13/code/modules/overmap_pointed/overmap_pointed.dm` (40 lines): heads of staff can mark overmap targets (`pointed()` override, red outline + `overmap_order` effect). DME +1.
2. **Dedicated fighter shield (#2909, `c2604b08fc`)**: before the SolGov fighter got `AddComponent(/datum/component/overmap_shields, 125, 125, 15)` in `Initialize` with `max_integrity = 25`; after `max_integrity = 125` and a new `armour_plating/tier6` "compact shield generator array" (shield instead of armor). (`_fighters.dm`, `fighter_components.dmi`.)
3. **Max client version (#2902, `18b472e608`)**: before only a build check; after `client_max_version` config entry, `CLIENT_MAX_VERSION 515`, and the warning compares version and build (`client_procs.dm`, `general.dm`, `config.txt`).
4. **Post April Fools fixes (#2884, `33bf006b30`)**: `ai-skynet.dm` null-safe target tracking (`QDELETED(OM) ||`), `overmap.dm` thrust overlay logic reordered, `damage.dm` `LAZYINITLIST(proj.impacted)`, `broadsides.dm` guard.
5. **Munition computer names (#2896, `bd4a93f282`)**: names/descriptions added to the munitions computers (`munitions.dm`, `deck_guns.dm`, `ammo_rack.dm`, `autonomy.dm`).
6. **Von Neumann AI frigate (#2656, code part)**: new `/obj/structure/overmap/nanotrasen/solgov/vnc` (+`starter`, `ai`) in `solgov.dm`, `apply_light_ai_weapons()`, SolGov fleet destroyer list now `list(solgov/ai, solgov/vnc/ai)` (`fleet_types.dm`), new sprites.
7. **Flak impact crash guard (#2891, `a3f98e8a10`)**: `flak.dm`/`lattice.dm`: openspace counts as space; `OM.relay_to_nearby` only when `isovermap(OM) && !QDELETED(OM)`.
8. **Stormdrive (#2891)**: `empulse(3, 5)` → `empulse(src, 3, 5)`; control-rod alert reworded ("removing" instead of "inserting") with an added `return` for the Reconsider branch.
9. **Plasmaglass smoothing (#2887, `d76133d212`)**: `window.dm` `canSmoothWith` list extended so reinforced plasma glass joins walls.
10. **Moth washing fix (#2884)**: `moth.reagents` null guard with `add_atom_colour("#404040", ...)` fallback.
11. **Dependencies (dependabot)**: Pillow 12.2.0 → 12.3.0; tgui lockfile +/- 500 lines (ws, fast-uri, js-yaml 4.3.2, postcss 8.5.13...).
Most important features between June 2026 and the target: Von Neumann AI SolGov frigate and fleets, overmap target pointing, fighter shield plating, client max-version warning, munition computer names, flak/relay robustness, many tgui dependency security bumps.

Diff totals: 39 files, +541/-254; 23 `.dm`, 3 tgui (dependencies only), 1 config, 7 assets (all `.dmi`, 3 of them new), 4 added (1 `.dm`, 3 `.dmi`), 1 DME, 0 `.dmm`, 0 `_maps` files.
