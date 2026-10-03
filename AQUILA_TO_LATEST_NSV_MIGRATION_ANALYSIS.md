# Aquila → Latest BeeStation/NSV13: Migration Analysis

> ## REVISION 2: Authoritative-map policy and Phase 0 results (supersedes parts of this document)
>
> **Policy (from the project owner): Aquila maps are authoritative.** Target = latest BeeStation/NSV13 **code and architecture** + Aquila code/gameplay + **current Aquila maps, untouched**.
> - Existing Aquila `.dmm` files are never replaced, merged, tile-merged, "fixed" or redesigned from upstream. No upstream Shrike 2.0, no upstream Serendipity touch-ups, nothing "theirs".
> - The **only** permitted `.dmm` edit is a minimum, individually documented compatibility fix for an object/type path/API that new upstream code removed or renamed (see "Map compatibility log" below).
> - New upstream maps/ships (Von Neumann etc.) are **excluded** from `config/maps.txt`, rotation, voting, defaults and ship selection until separately approved.
> - A baseline inventory of all 733 `.dmm` files exists (`AQUILA_MAP_BASELINE.tsv`, tag `aquila-baseline-2025-09-09`). After every stage every baseline `.dmm` must be byte-identical (git blob) unless covered by a compatibility-log entry. An unexpected `.dmm` change is a migration failure.
>
> **Sections that are superseded by this policy** (kept below for the evidence, marked inline): the tile-level map-merge recommendations in §1 (items 6-7), §5 ("What simply accepting upstream maps would lose" and the "Accept upstream" cells), §8 rows 1, 2, 3, 8, 11, §10 step 4, §11 map rows, §12 phases, §14 map validation, §17 and GO/NO-GO. Their replacements are in "Revised plan" at the end of this document.
>
> **Phase 0 result: BASELINE FAIL** (details in `AQUILA_BASELINE_VALIDATION.md`). The unmodified Aquila HEAD:
> - compiles normally, passes dreamchecker (0 diagnostics), tgui build/test/lint/tsc, and boots DreamDaemon and passes all 40 unit tests;
> - **fails the `ALL_MAPS` compile**, because **`Tycoon2.dmm` contains committed merge-conflict markers**; `Serendipity2.dmm` (12 hunks) and `snake_upper.dmm` (1 hunk) contain them too. Serendipity and Snake are votable in `config/maps.txt`;
> - is not a "clean run": 5 runtime errors at init (`OVERFLOW_JOB Midshipman` vs renamed job "Majtek"; gulag room spawner re-entrant map load; two deck-turret core lookups on `Aquila_Atlas`).
>
> **Phase 0.5 update:** the baseline defects were repaired in the working tree (not committed, pending approval): see `AQUILA_PHASE_0_5_BASELINE_REPAIR.md` and the candidate map baseline `AQUILA_MAP_BASELINE_REPAIRED.*`. Result: REPAIRED BASELINE PASS WITH WARNINGS. Compatibility log entry C1 (deck_turret → deck_turret/core on 4 Aquila maps) is the first and so far only entry.
>
> New fact that matters for the policy: the three maps above are not just "Aquila maps", they are Aquila maps with unresolved upstream text inside. Which side is authoritative needs the owner's confirmation (my reading: the `HEAD` side of each hunk). They are the one place where a map edit is needed, and it is a repair of the baseline, not a migration step.

Analysis only. Nothing was merged, rebased, cherry-picked, committed, pushed or modified in the working tree.
Branch analysed: `update-do-najnowszego-nsv13` (working tree clean before and after).

Disclosure: one dry-run `git merge-tree --write-tree` (x2) was executed. It writes *unreferenced* objects into `.git/objects` only. It touches no refs, index or working tree, and `git gc` will prune the objects. Resulting dry-run tree for the recommended base: `7574c28f35b5d997739fa3702d100b5ff1066296`.

---

## 1. Executive summary

1. **Aquila is much closer to current upstream than Git's ancestry suggests.**
   Git's merge base with upstream is **2024-01-03** (`b6fc4ce967`), which implies 626 upstream commits to absorb and about 94 textual conflicts. That is misleading. Aquila has twice **copied upstream snapshots in as ordinary single-parent commits**, which broke ancestry:
   - `1ae7970a12` "Aktualizacja do NSV13 (#68)", 2024-01-30, which brought in upstream around 2024-01-26 (`d47b28bb5a`).
   - `f847d3caca` "Aquila powrot (#317)", **2025-09-09**, a 755-file commit. It brought `nsv13/`, `code/`, `tgui/` and most maps to the upstream state at **`fa47d5fe77` "Hard mode (#2764)", 2025-05-01**, via a plain commit.
2. **The effective upstream base of the current Aquila HEAD is `fa47d5fe77`** (inferred from blob identity, see §2.3). Against that base:
   - Upstream has **214 commits / 235 files** to bring in (`fa47d5fe77..upstream/master`).
   - Only **38 files** were touched by both sides.
   - A dry-run three-way merge with that explicit base gives **8 textual conflicts** (vs 94 with Git's default base).
3. **Aquila's naval/overmap code is essentially vanilla NSV13 plus Polish localisation.** Aquila vs `fa47d5fe77` in `nsv13/code` is 45 files / ~870 changed lines. Nearly all of that is translated strings (`areas.dm`, job names, announcements) plus a handful of `AQ EDIT` hooks. The real Aquila divergence is elsewhere:
   - `aquila/` (306 `.dm`, 202 sounds, 70 icons), the large Bee/tg ports (vampires, infiltrators, genetics, heretics, borgs, etc.).
   - ~490 modified files in `code/`.
   - Maps (Atlas, Atlantis, Snake, Serendipity, Gulag, away missions).
4. **Upstream's one big architectural change is the "weapon datum refactor" (#2820, 2025-12-01, 74 files).** It deletes `nsv13/code/datums/weapon_types.dm`, `overmap/ai.dm` and `overmap/weapons/ship_weapon.dm`, and removes all `FIRE_MODE_*` defines. A static search found **no Aquila-original code or map that uses the removed API**. The residual risk is that Aquila's Polish strings sit inside code upstream rewrote, so the English text silently returns.
5. **Toolchain is identical**: BYOND 515.1633, rust_g 3.1.3, Node 18, auxmos 2.5.2-b, SpacemanDMM suite-1.8. Upstream changed only CI user-agent strings, Pillow/webpack versions and tgui deps. No BYOND upgrade is needed.
6. **Maps are the real danger**, not code.
   - 3 maps have genuine textual conflicts (Serendipity 1/2, Shrike 2).
   - Several more are "clean" but unsafe: DMM tile keys are positional aliases (e.g. upstream renamed key `Hr→jN`), so a textually clean merge can still corrupt a map.
   - The new upstream ship `vonneumann` will lack the Aquila `syndicate_*` docking ports.
7. **Recommendation: a hybrid (Option D).** Add an ancestry "bridge" (`merge -s ours fa47d5fe77`), then merge upstream in 4 staged, compilable checkpoints, with all `.dmm` handled by a tile-level three-way merge instead of text merge. Rebase and "start from upstream and forward-port" are both worse for this repo (§9).
8. **GO**, conditional on first establishing a baseline: today's HEAD has never been compiled in this analysis (no BYOND on this machine). See §17.

---

## 2. Exact Git ancestry

### 2.1 Refs (all fetched; no merge)

| Ref | Commit | Date | Note |
|---|---|---|---|
| local `HEAD` (`update-do-najnowszego-nsv13`) | `f847d3caca` | 2025-09-09 | "Aquila powrot (#317)" |
| `origin/master`, `origin/update-do-najnowszego-nsv13`, `origin/HEAD` | `f847d3caca` | | identical to HEAD |
| `aquila/master` (= `aquila/HEAD`) | `f847d3caca` | | **identical to HEAD** |
| `aquila/master-nsv13` | `590de2d7a9` | 2023-12-22 | frozen vanilla mirror, 0 Aquila commits |
| `aquila/updatensv` | `1c774003f4` | 2024-02-25 | real merge of upstream 2024-02-23, **not in master**, 27 commits ahead |
| `upstream/master` (= `upstream/HEAD`) | `69565b730f` | **2026-09-12** | "Automatic changelog compile" |

**NSV13-revive is currently byte-identical to Aquila master.** It has no additional changes. `git diff HEAD aquila/master` is empty.

### 2.2 Merge bases (Git's view)

| Pair | Merge base |
|---|---|
| HEAD ~ aquila/master | HEAD itself |
| HEAD ~ upstream/master | `b6fc4ce967` (2024-01-03) |
| aquila/master ~ aquila/master-nsv13 | `590de2d7a9` (2023-12-22) |
| aquila/updatensv ~ upstream/master | `71df05dfe9` (2024-02-23) |

`HEAD...upstream/master` = **177 Aquila-only / 626 upstream-only** commits.

### 2.3 Why Git's merge base is wrong (the central finding)

- Aquila history is linear (0 merge commits in the 177) and consists of squash-merged PRs.
- `1ae7970a12` (#68) and `f847d3caca` (#317) are *content imports* of upstream with no upstream parent.
- **Evidence for the real base.** I compared blob hashes of `nsv13/code`, `code`, `tgui/packages` at HEAD with every first-parent upstream commit from 2024-01 to 2026-09:

  | Upstream commit | Identical blobs with HEAD |
  |---|---|
  | `b6fc4ce967` (MB, 2024-01) | ~4531 |
  | 2024-12 | 4769–4790 |
  | **`fa47d5fe77`, 2025-05-01, through `8f9e64ee04`, 2025-05-09** (plateau; only changelog commits between) | **4345 / 4943 (best, the maximum in the scan)** |
  | `bb47a7cc4f`, 2025-05-23 | 4344 |
  | 2026-09 | 4916 |

  Restricted to `nsv13/code`: 419/464 files identical at `fa47d5fe77`.
- Similarly the #68 snapshot matches upstream `d47b28bb5a` (2024-01-26, 434/447 files in `nsv13/code`).
- **Confidence: high for `nsv13/` and `code/`; medium for maps** (maps were copied/forked by hand; see §5). `fa47d5fe77` is an *inferred* synthetic base; Git does not know about it.

### 2.4 Aquila-side commit profile (177 commits since MB)

- Jan 2024: 66; Feb: 81; Mar: 28; Apr: 1; **Sep 2025: 1 (#317)**.
- Authors: MACIEKBAKI 38, Joker66613 32, smorgli 27, Dejaku51 20, Boletus 18, srawek 14, Unibel 9, Szyszkrzyn 8, others 11.
- Content mix by commit title: Polish translation/localisation and string ports; "[Port ####]" ports from Bee/tg; gameplay/balance tweaks; Atlas/Atlantis/Snake/Serendipity/Gulag mapping; new antagonists (vampires, infiltrators); sound overhaul.
- Note: **Sept 2025's #317 is the only change after April 2024**; Aquila's main development stopped 2024-04-23.

---

## 3. Divergence statistics

| Measure | vs Git MB `b6fc4ce967` | vs inferred base `fa47d5fe77` |
|---|---|---|
| Aquila-changed files | 2106 (+1.13M/−68k lines) | **1470** (+1.09M/−37k) |
| Upstream-changed files | 864 | **235** (+95.8k/−16.6k) |
| Files changed by both | 771 | **38** |
| Dry-run textual conflicts | 94 | **8** |
| Upstream commits to absorb | 626 | **214** |

Aquila-vs-`fa47d5fe77` by area (changed lines):
- `_maps/map_files` ~801k (nearly all new Aquila maps).
- `_maps/RandomZLevels` 249k.
- `code/` 20k (563 files).
- `aquila/code` 12.5k (305 files).
- `strings/` 5.7k.
- `nsv13/code` 0.9k (45 files).
- `tgui` 0.4k.

Aquila vs base, file status: 718 A, 719 M, 32 D, 1 R.
- Deleted by Aquila: the old `eldritch_cult` tree (replaced by a ported `heretic` tree), `dance_machine.dm` (moved), roundend sounds, 7 title screens.

The 8 dry-run conflicts (explicit base): `Serendipity1.dmm`, `Serendipity2.dmm`, `Shrike2.dmm`, `config/maps.txt`, `code/__HELPERS/_lists.dm`, `code/__HELPERS/priority_announce.dm`, `nsv13/.../fleet_combat/combat_handling.dm`, `nsv13/.../research/astrometrics.dm`.

---

## 4. Aquila-specific change inventory

Classification key from the task. "Evidence" is `git diff fa47d5fe77 HEAD` unless stated.

### 4.1 AQUILA-ORIGINAL (new files, no upstream counterpart)
- `aquila/` module (included last in `nsv13.dme` via `aquila/aquila.dm`, lines 4191-4193):
  - **Vampire gamemode** (9 files + HUD defines), **Infiltrator antagonist and gamemode** (7 + `infiltration.dm`), demons (8), diseases (5, sentient disease), Space Dragon rework.
  - Nanite 2.0 additions (10), research designs (9), mob/species work (62 files in `modules/mob/living`), items (27), structures/effects, jukebox and wires, metacoin admin verb, energy harvester, bluespace miner, plort machine, gravity generator, power tools, etc.
  - **Not included in the DME:** `aquila/code/modules/food_and_drinks/recipes/tablecraft/recipes_pie.dm` (orphan).
- `aquila/sound` (202 files), `aquila/icons` (70).
- Also top-level `cheese.dmi`, `food.dmi`, `stack_objects.dmi`, `statue.dmi`, `data/replays/` placeholder.
- New `code/` files: 60 added (e.g. `heretic` tree; thirst system, floorpills, etc.).
- **Maps:** `Atlantis` (2 decks), `Aquila_Atlas`, `Aquila_Snake`, `Aquila_Serendipity` forks, `gulag.dmm`, Gulag ruins (7), RandomRooms (atlas engine variants ×3 and more), RandomZLevels (Academy, TheFactory, challenge, wildwest), `infiltrator_cutter` shuttle, heretic sacrifice template.
- Config: `roleplay_filter.txt`, `title_screens/images/aqg.dmi`, `maps.txt` rotation, `config.txt`, ranks, `jobs.txt`, `admins.txt`, `awaymissionconfig.txt`.
- Tooling: `tools/UpdatePaths/AQUILA_xenobiofun.txt`; `tools/build/build.js` adds `'aquila/**'` to DM inputs.

### 4.2 AQUILA-MODIFIED-UPSTREAM (edits to inherited files)
- **452 `AQ EDIT`/`AQUILA EDIT` markers in 199 files** (the project's own convention).
- `nsv13/code`: marker use is small (16 files), e.g.
  - `stormdrive.dm`: `SEND_GLOBAL_SIGNAL(COMSIG_GLOB_STROMDRIVE_EXPLOSION)` (hook for Aquila's explosive implant).
  - `munitions_machinery.dm` (vendor item), `munitions_trolley.dm` (`munitions_trolley_dummy`, used by Aquila maps).
  - `starsystem_manager.dm` (admin Delete command).
- Polish localisation in-place (the bulk): `areas.dm` (178 lines), `jobs/security/weapons.dm`, `dance_machine.dm`, `hammerhead/aetherwhisp/hammurabi/pegasus/boarding_areas` area names, job types, KNPC species names, announcements.
- `code/` (491 M): gameplay/strings/ports across `modules/mob`, `game/objects`, `reagents`, `jobs`, `projectiles`, `vending`, `research`, `food_and_drinks`, `controllers/subsystem`. (Not individually reviewed. See §16.)

### 4.3 AQUILA-MAP-CHANGE
See §5.

### 4.4 AQUILA-ASSET-CHANGE
Sounds (weapons, ambience, radio, voice, roundend; 7 roundend sounds deleted), sprites (`nsv13/icons` 43 files, `icons/*`, weapon icons swapped by #94), title screens, antag tips (`html/antagtips`), `html/img/blob*.png`.

### 4.5 AQUILA-CONFIG-CHANGE
`config/*` (above), `strings/*` (19 files; Polish names/tips/ion laws/etc.).

### 4.6 AQUILA-TOOLING-CHANGE
`.github/workflows` (4 files: `ubuntu-22.04→20.04` runner, repo guard commented out), `tools/build/build.js`, `tools/UpdatePaths/AQUILA_*`, `html/changelog*` (changelog pipeline retargeted).

### 4.7 INHERITED-NSV / ALREADY-PRESENT-IN-UPSTREAM / OBSOLETE
- `nsv13/code` files not in the 45 differing ones are identical to upstream `fa47d5fe77` → INHERITED-NSV.
- **Leftover duplicate:** `datum/design/board/ammo_sorter*` and `circuitboard/*/ammo_sorter` are defined in Aquila's `automation.dm` (+31 lines, from the #68-era snapshot) **and** in `nsv_circuitboard_designs.dm`. Upstream has them only in the latter. → OBSOLETE-OR-REPLACED-UPSTREAM (DM tolerates duplicate datum paths; include order decides vars).
- `code/__DEFINES/mobs.dm` DEFECATION values: #317 changed them from 30/60/90/120/160 to 40/80/100/160/200. This looks like it may be a regression of an Aquila (or Bee-port) tune. → PROVENANCE-UNCERTAIN.

### 4.8 PROVENANCE-UNCERTAIN
- 18 un-merged Aquila feature branches (`pituitary` 11, `symbiotic` 6, `miasma` 6, `updatensv` 27, etc. commits ahead of master). I did not check whether their content was re-done in master.
- Whether #317 silently dropped any pre-#317 Aquila edits: of 564 files Aquila-only-edited pre-#317 in `code/`/`nsv13/code`, **24 differ at HEAD** (eldritch tree deletion, `dance_machine`, config, `.dme`, DEFECATION values, a title-screen/string tweak). All explained *except* `mobs.dm` DEFECATION and `disease_abilities.dm` (blob differs but my line diff showed no content change, likely whitespace/line endings; not verified). Maps: see §5.

---

## 5. Aquila map inventory (dedicated audit)

> **SUPERSEDED by Revision 2 (authoritative-map policy):** the facts in this section (what differs, what upstream changed) remain valid evidence, but every "accept upstream", "re-apply by hand" or tile-level merge suggestion is void. Existing Aquila maps are kept byte-identical; see "Revised plan".

Method: blob comparison of every `.dmm` at HEAD / `fa47d5fe77` / `upstream/master`, then diff sizes. 735 `.dmm` files total. No `.dmm` was modified.

**Key structural facts (verified by blob hash):**
- `Aquila_Atlas/*.dmm` are byte-identical copies of the pre-#317 Aquila-edited `Atlas/atlas.dmm`/`atlas2.dmm`. Standard `Atlas/atlas.dmm` at HEAD is the same blob (Aquila edit), so two paths hold the same map.
- `Aquila_Snake/*` and `Aquila_Serendipity/*` are byte-identical to the **pre-#317** `Snake/*` and `Serendipity/*` (Aquila edits on a Jan 2024 upstream base). The standard `Snake/*` and `Serendipity/*` at HEAD are *different* blobs (Aquila edits re-applied on the May 2025 upstream version).
- So the Aquila_* forks are **frozen** on 2024-era upstream and will never receive upstream fixes.
- `config/maps.txt`: `aquila_atlas` is default and votable; `aquila_snake` and `aquila_serendipity` are votable; most upstream ships are commented out of `votable`.

Change size is "+added/−removed lines, HEAD vs `fa47d5fe77`" for Aquila and "`fa47d5fe77` vs `upstream/master`" for upstream.

| MAP | AQUILA MODIFIED? | UPSTREAM MODIFIED SINCE BASE? | BOTH? | CHANGE SIZE (AQ / UP) | CONFLICT RISK | NOTES |
|---|---|---|---|---|---|---|
| Atlas 1/2 (`atlas.dmm`, `atlas2.dmm`) | Yes, heavy | No | No | +14163/−10769; +11228/−6754 / 0 | **LOW** | Aquila-only; keep verbatim. Duplicate of `Aquila_Atlas`. |
| Aquila_Atlas 1/2 | Aquila-only (new) | n/a | n/a | +85k / +82k | LOW (textual) | Needs path/semantic validation only |
| Aquila_Snake upper/lower | Aquila-only (new) | n/a | n/a | +75k / +81k | LOW (textual) | Frozen 2024 base |
| Aquila_Serendipity 1/2 | Aquila-only (new) | n/a | n/a | +82k / +77k | LOW textual, **MED semantic** | Frozen 2024 base; will not get upstream Serendipity fixes |
| Atlantis 1/2 | Aquila-original | n/a (no upstream map) | No | +96k / +90k | LOW | `atlantis.json`; check `ams` etc. object paths |
| Snake lower | Yes, heavy | No | No | +3580/−3132 / 0 | LOW | Aquila-only |
| Snake upper | Yes | No | No | +361/−247 / 0 | LOW | Aquila-only |
| **Serendipity1** | Yes, heavy | Yes (touchups ×3) | **YES** | +3259/−2735 / +21/−20 | **HIGH** | **Textual conflict.** Upstream: turret control panel, camera, airalarm/newscaster swaps, deleted keys `zJ`,`Ot`, key renames (`Hr→jN`, `Ot→Rq`). |
| **Serendipity2** | Yes | Yes | **YES** | +400/−83 / +52/−7 | **HIGH** | **Textual conflict.** Upstream: scrubber/atmos pipes, cameras, tank ports. |
| **Shrike1** | No | Yes: "FROZENSTAR SHRIKE 2.0" | No | 0 / +6635/−5659 | MED | Accept upstream, but re-add Aquila docking ports (see Shrike2) |
| **Shrike2** | Yes (+72/−6: `syndicate_*` docking ports) | Yes (+7054/−6379, redesign) | **YES** | +72/−6 / +7054/−6379 | **HIGH** | **Textual conflict.** Upstream redesigned the deck; Aquila's small edit must be re-applied by hand |
| Tycoon1 | No | Yes | No | 0 / +18/−14 | LOW | Accept upstream |
| Tycoon2 | Yes (+46/−4) | Yes (+12/−13) | **YES** | / | MED | Textually clean in dry-run; **key-collision check required** |
| Hammerhead | Yes (+37/−4) | Yes (+4/−4, pump rotation) | **YES** | / | MED | Clean in dry-run; verify tile-level |
| Aetherwhisp1 | No | Yes (+48/−89) | No | 0 / | LOW | Accept upstream |
| Aetherwhisp2 | Yes (+73/−7) | Yes (+2/−3) | **YES** | / | LOW-MED | Clean in dry-run; verify |
| generic/CentCom | Yes, heavy (+4633/−2949) | Yes, tiny (+1/−3) | **YES** | / | MED | Clean in dry-run; map is heavily Aquila-edited, so verify tile-level |
| Eclipse1/2 | Yes (+1/−1; +72/−6) | No | No | / 0 | LOW | Aquila-only |
| Galactica1/2 | Galactica2 +60/−5 | No | No | / 0 | LOW | Aquila-only |
| Gladius1/2 | +1/−1; +62/−7 | No | No | / 0 | LOW | Aquila-only |
| Vago deck2 | +72/−6 | No | No | / 0 | LOW | Aquila-only |
| Babylon2 (Instanced) | No | Yes (+333/−781) | No | 0 / | LOW | Accept upstream |
| Syndicate boarding templates (`carrier`, `destroyer`, `mako`, `mako_carrier`, `marine_frigate`, `nukefrigate`) | No | Yes (new boarding mobs/layout) | No | 0 / 33–90 lines each | LOW | Accept upstream |
| `vonneumann/vnmk3.dmm` + `cargo_aiship.dmm` | n/a | **New upstream** | n/a | / +72.7k | **MED-HIGH (semantic)** | New ship lacks Aquila `syndicate_*` docking ports → Infiltrator shuttle cannot dock |
| Mining ruins (`mining5–29`) | `mining28` +4/−4 | `mining11` +1/−5 | No | / | LOW | |
| RandomZLevels, RandomRuins/Gulag, RandomRooms, `gulag.dmm`, holodeck extras | Aquila-only | No | No | | LOW | Aquila-original; keep |
| `_maps/_basemap.dm`, `*.json` (`aquila_*`, `atlantis`) | Yes | basemap +10/−1 upstream | **YES** for basemap | | MED | `vonneumann.json/.dm` new upstream; `config/maps.txt` conflicts |

**What simply accepting upstream maps would lose:**
- The whole Atlas/Snake/Serendipity/Gulag/Atlantis customisation.
- The `syndicate_*` docking-port pattern (+72/−6) on Aetherwhisp2, Eclipse2, Galactica2, Gladius2, Shrike2, Tycoon2, Vago2, whose use is in `code/modules/shuttle/syndicate.dm`.
- Oremagnet placements, `munitions_trolley_dummy`, Aquila rooms, `job_changes.dm` for Aquila maps.

**Textual merge of DMM is unsafe even when clean.** DMM uses per-file generated key aliases. Two independent edits can reuse a key for different tiles or rename keys, so a clean text merge can silently produce a broken or wrong map. Treat every map in "BOTH" as requiring a tile-level (coordinate → tile contents) three-way comparison, using `tools/mapmerge2` (already in the repo) as the parser basis.

**Map type-path check (done, shallow):** upstream removed 35 type paths E→master (weapon-datum and `/obj/structure/overmap` formatting churn). None is referenced by name in any HEAD `.dmm`. `/obj/machinery/computer/ams` was a false positive (moved file, still defined). A compile-time check with the real compiler is still required (§12).

---

## 6. Current upstream architecture vs Aquila

Upstream moved only 214 commits since the Aquila snapshot, so architecture mostly matches:

| Area | Aquila (HEAD) | Upstream master | Direct-copy safe? |
|---|---|---|---|
| BYOND | 515.1633 | 515.1633 (`MIN_COMPILER_VERSION 515 / BUILD 1633`) | Yes |
| Build | `tools/build` (Juke) + `BUILD.bat`, adds `aquila/**` input | same, minus Aquila input | Yes; keep the Aquila input |
| Rust/native | rust_g 3.1.3, auxmos 2.5.2-b | same | Yes |
| TGUI | Node 18, yarn 3.4.1, webpack ^5.94 | webpack ^5.104; new Railgun* UIs; Astrometrics/Starmap/HybridWeapons/TacticalConsole updates | Yes. Aquila has only 2 tgui deltas (`AntagInfoHeretic.tsx`, `CommunicationsConsole.js`) |
| Ship weapons | `/datum/ship_weapon`, `FIRE_MODE_*` | **Delta refactor** `delta_overmap_ship_weapons/*`, `weapon_datum_types.dm`, `FIRE_MODE_*` removed | **No** for anything touching weapons/AI. Aquila's translated weapon strings need re-application |
| Overmap AI | `overmap/ai.dm` + `ai-skynet.dm` | `ai.dm` deleted; autonomy in `autonomy.dm` | No for Aquila translations in `ai-skynet.dm` |
| Dummy pilots, `overmap_pointed`, fighter shields, hard mode | partial/absent | present | n/a (upstream gains) |
| VV | older | up-to-date VV port (#2798), `vv_ghost`, appearance vars | Aquila has no VV edits relative to base; clean |
| TGS DMAPI | older | updated twice (#2818, #2868) | Clean; Aquila `tgs` conflicts only vs Jan-2024 base |
| `minor_announce` | `(message,title,alert,from,html_encode)` | adds `silent` arg (#?) | **Conflict**: Aquila translates the default titles in the same proc |
| Astrometrics | Aquila translates radio lines | broadcast/mute rework (#2800) | **Conflict**: same lines |
| DB | schema unchanged | unchanged (no `SQL/` diff E→master) | Yes |
| Master controller / subsystems / QDEL / components | No upstream delta found in `code/controllers` beyond `input.dm`-adjacent edits that predate the base | | Not investigated deeper |

I could not compile either tree, so claims about compile-level incompatibility are by static evidence only. Not investigated in depth: atmos, movement, timers, traits and elements, because the E→master diff in `code/` is small (~45 files).

---

## 7. Aquila functionality already available upstream

| Aquila feature | Upstream equivalent | Behavioural difference | Architectural difference | Port still needed? |
|---|---|---|---|---|
| `ammo_sorter` designs/boards in `automation.dm` | Same designs in `nsv_circuitboard_designs.dm` + `ammo_rack.dm` | Possibly different material costs (Aquila copy has glass 2000/copper 1000/gold 500; upstream copy not diffed) | Duplicate definitions | **No** – verify then delete the stale copy (user decision) |
| Starsystem manager admin "Delete" | Already in upstream `starsystem_manager.dm` (lines 157, 226) | Upstream also returns result of `cmd_admin_delete` | none | No (take upstream) |
| `minor_announce` default title ("Uwaga:") | Upstream `silent` arg | Aquila only translated | same proc | **Yes** – re-apply translation onto new signature |
| VV tooling | Upstream VV port #2798 | Aquila had no VV changes | | No |
| Beepsky/borg/lore ports from Bee (genetics, nanites 2.0, heretics, borgs…) | Not in NSV upstream | n/a | Aquila-owned | Yes (no upstream version) |
| Boarding mobs for interiors | Upstream #2806 "boarding interiors with new mobs" | Aquila has own `syndicate_knpc`/`nanotrasen` mob files with translations | | **Verify** – possible overlap |

Everything else listed in the task as "already exists upstream" could not be confirmed, because Aquila has no naval-mechanics edits to compare. Nothing here is marked for removal.

---

## 8. Conflict forecast

> **SUPERSEDED by Revision 2 (authoritative-map policy):** rows 1 (Serendipity), 2 (Shrike), 3 (Von Neumann), 8 (other "both" maps) and 11 (frozen forks) no longer require merge work: those maps stay as they are and Von Neumann is excluded. They become *guard* items (verify the maps are unchanged) plus the compatibility check in "Revised plan". The non-map rows (4-7, 9, 10, 12-14) stand.

Textual (T), semantic (S), architectural (A), map (M). Based on the explicit-base dry run plus the static checks.

| # | Subsystem | Files | Aquila side | Upstream side | Type | Level | Resolution | Gameplay / map risk |
|---|---|---|---|---|---|---|---|---|
| 1 | Serendipity maps | `Serendipity1/2.dmm` | heavy rework | atmos/camera/turret fixes | T+M | **HIGH** | Tile-level 3-way merge; take Aquila layout, re-apply upstream fixes by coordinate | Broken pipes/powernet; map risk high |
| 2 | Shrike | `Shrike2.dmm` (+`Shrike1`) | 6 lines + 72 docking ports | complete 2.0 redesign | T+M | **HIGH** | Take upstream, re-add `syndicate_*` ports at correct coordinates | Infiltrator docking |
| 3 | Von Neumann (new) | `vnmk3.dmm`, `vonneumann.json` | n/a | new ship | S+M | **MED-HIGH** | Add Aquila docking ports + `job_changes.dm`; decide `maps.txt` entry | Infiltrator mode, rotation |
| 4 | Weapon datum refactor | 74 files in `nsv13/code`, tgui | Polish strings in `ai-skynet.dm`, `overmap.dm`, `damage.dm`, `ftl_jump.dm` etc. | rewrite | **S+A** | **MEDIUM** | Merge; then grep Polish-translated English literals; re-translate where upstream moved/rewrote | Silent English text returns; no logic loss found |
| 5 | Announcements/astrometrics | `priority_announce.dm`, `astrometrics.dm`, `combat_handling.dm` | translated | `silent` arg / broadcast rework | T | **MEDIUM** | Hand-merge: upstream logic + Aquila strings | Low |
| 6 | `_lists.dm` | helpers | adds `list_clear_nulls` | adds `unique_list_in_place` | T | LOW | Keep both | Low |
| 7 | Config | `config/maps.txt`, `config/game_options.txt`, `config.txt` | custom rotation/roles | `vonneumann`, `votable` changes | T | MEDIUM | Keep Aquila rotation; add new ship deliberately | Wrong default map |
| 8 | Other 'both' maps (Tycoon2, Hammerhead, Aetherwhisp2, CentCom) | | docking ports/edits | small tweaks | M | MEDIUM | Tile-level compare even though textual merge is clean | Hidden key collisions |
| 9 | `.dme` | `nsv13.dme` | `// AQUILA` include block | +18 lines of new includes | T | LOW | Auto-merges; re-verify the Aquila block stays last and every include exists | Compile failure if wrong |
| 10 | Aquila `code/` ports vs upstream `code/` fixes (14 files in both) | `is_helpers.dm`, `carbon.dm`, `camera.dm`, `traitor.dm`, `processor.dm`, `emote.dm`, `laser.dm`, `security_levels.dm`, … | ports/translations | fixes | S | LOW-MED | Review each of the 14 | Behavioural drift |
| 11 | Frozen forks | `Aquila_*` | 2024 base | no counterpart | S | MED | Decide: re-base fork on current map, or leave frozen | Fork misses fixes |
| 12 | Build/CI | `.github/workflows/*`, `tools/ci` | runner downgrade, guard removed | UA string changes | T | LOW | Keep Aquila overrides | None |
| 13 | tgui | none | none | Railgun UIs etc. | none | LOW | Take upstream | None |
| 14 | Database | none | none | none | none | LOW | | |

Textual vs semantic: items 4, 10, 11 are **not** visible as Git conflicts. A clean merge does not prove compatibility.

---

## 9. Migration strategy comparison

| Criterion | A. Merge upstream into Aquila (plain) | B. Rebase Aquila onto upstream | C. Start from upstream, forward-port | D. Hybrid (bridge + staged merge + tile-level maps) |
|---|---|---|---|---|
| Conflict volume | **94** (base `b6fc4ce967`; phantom conflicts from #68/#317 imports) | 177 commits replayed against 626; the #317 monolith (755 files) and #68 would conflict massively | Low textual; huge manual volume (1470 files) | **8** textual, plus targeted semantic review |
| Risk of losing Aquila functionality | Medium (conflict-resolution errors) | High | Medium-High (omissions) | **Low** (tree starts as Aquila) |
| Risk of restoring obsolete upstream code | Medium | Medium | Low | Low |
| Map preservation | Poor (text merge) | Poor | Manual copy; Aquila maps are 800k lines | **Good** with tile-level merge |
| History preservation | Good | Rewrites history | Loses history | Good (adds a bridge merge) |
| Testing difficulty | One giant checkpoint | Per-commit state may not compile | Whole app changes | **Incremental, compilable checkpoints** |
| Complexity | Medium | Very high | High | Medium |
| Future upstream updates | Still wrong base next time | Clean | Clean, but fork-heavy | **Clean** (bridge gives a correct base permanently) |

Why the history says D: Git sees the wrong base because both upstream syncs were content copies. A rebase would replay a 755-file import commit. Option C throws away 4,000+ working lines of `aquila/` integration for no benefit when only 8 text conflicts exist. A plain merge (A) would leave Git's wrong base in place and create ~86 phantom conflicts, each of which tempts a blind "ours/theirs".

---

## 10. Recommended strategy: D (hybrid)

> **SUPERSEDED by Revision 2 (authoritative-map policy):** step 4 ("Maps resolved only via tile-level compare") is replaced by the map-protection protocol in "Revised plan". The bridge + staged-merge structure otherwise stands.

1. **Preserve** `f847d3caca` with a local tag (no push by me).
2. **Bridge**: record `fa47d5fe77` as a parent with a no-op tree (`git merge -s ours fa47d5fe77`). This is one commit; it changes no files and gives the correct base from here on. The dry-run in §3 already proves the conflict count with that base.
3. **Merge upstream in stages** using first-parent squash boundaries (verified on the first-parent line):
   - Stage 1: up to `e09bf3db6d` (2025-11-29), i.e. just before the weapon refactor. Includes the Serendipity touchups `b558359d04`.
   - Stage 2: `84a268eb9e` (weapon datum refactor, 2025-12-01) alone.
   - Stage 3: through `ff23bf114d` (2026-06-06). Includes Shrike 2.0 `5204ba5a54`.
   - Stage 4: through `upstream/master` (`69565b730f`). Includes Von Neumann `76aaa53bab`.
4. **Maps** resolved only via tile-level compare; text-merge result for any `.dmm` is treated as untrusted.
5. **After each stage**: compile and boot before the next.

---

## 11. Preservation ledger

> **SUPERSEDED by Revision 2 (authoritative-map policy):** every map row below is now **PRESERVE UNCHANGED (authoritative)**, including Serendipity 1/2, Shrike 1/2, Tycoon, Hammerhead, Aetherwhisp, CentCom. "MAP MERGE REQUIRED" and "POSSIBLY OBSOLETE" for the `Aquila_*` forks are void: nothing is merged and nothing is declared obsolete. Von Neumann is **EXCLUDED pending approval**, not "BLOCKED".

Status vocabulary from the task. **Nothing is marked REMOVE.**

| Item | Status |
|---|---|
| `aquila/` module (vampires, infiltrators, demons, diseases, nanites, mob/species, items, jukebox, metacoin, etc.) | PRESERVE UNCHANGED; MANUAL GAMEPLAY VALIDATION REQUIRED |
| `// AQUILA` include block in `nsv13.dme` | PRESERVE UNCHANGED (verify position after merge) |
| In-place Polish localisation in `nsv13/code` (45 files) | PRESERVE — ADAPT TO NEW UPSTREAM ARCHITECTURE (weapon/AI/announce files) |
| `AQ EDIT` hooks (`stormdrive` signal, `munitions_trolley_dummy`, vendor item, `starsystem_manager` delete) | PRESERVE UNCHANGED; the starsystem delete is **UPSTREAM NOW FUNCTIONALLY EQUIVALENT** (verify) |
| `ammo_sorter` duplicate designs in `automation.dm` | POSSIBLY OBSOLETE — USER DECISION REQUIRED |
| `minor_announce` translation | PRESERVE — ADAPT (new `silent` signature) |
| Astrometrics radio translations | PRESERVE — ADAPT (broadcast rework) |
| Atlas, Atlantis, Snake, Gulag, away missions, RandomRooms, ruins | PRESERVE UNCHANGED; MANUAL MAP VALIDATION |
| Serendipity 1/2 | MAP MERGE REQUIRED |
| Shrike 1/2 | MAP MERGE REQUIRED |
| Tycoon2, Hammerhead, Aetherwhisp2, CentCom | MAP MERGE REQUIRED (verify) |
| Eclipse, Galactica, Gladius, Vago docking-port edits | PRESERVE UNCHANGED |
| `Aquila_*` forked maps | POSSIBLY OBSOLETE — USER DECISION REQUIRED (frozen on 2024 base) |
| Von Neumann ship + Aquila docking ports | MAP MERGE REQUIRED; BLOCKED until user decides whether the ship enters rotation |
| `config/maps.txt`, `game_options.txt`, `config.txt`, ranks, jobs, admins | PRESERVE UNCHANGED (hand-merge `maps.txt`) |
| Strings (`strings/*`), antag tips, title screens | PRESERVE UNCHANGED |
| `heretic` tree replacing `eldritch_cult` | PRESERVE UNCHANGED; MANUAL GAMEPLAY VALIDATION |
| `DEFECATION_*` constants | NOT YET UNDERSTOOD |
| `recipes_pie.dm` (orphan) | NOT YET UNDERSTOOD |
| 18 unmerged Aquila branches | NOT YET UNDERSTOOD |
| Aquila `code/` ports overlapping upstream (14 files) | MANUAL GAMEPLAY VALIDATION REQUIRED |
| CI/workflow overrides, `tools/build/build.js` | PRESERVE UNCHANGED |
| Persistence/DB | PRESERVE UNCHANGED (no schema change either side) |

---

## 12. Proposed migration phases

> **SUPERSEDED by Revision 2 (authoritative-map policy):** replaced by the phase list in "Revised plan".

Each phase ends in a compilable, bootable checkpoint, with a rollback to the tag from phase 0.

| Phase | Scope | Expected conflicts / map impact | Validation | Rollback |
|---|---|---|---|---|
| **0 Baseline** | Local tag at `f847d3caca`; install BYOND 515.1633 + Node 18; compile and boot **unmodified** HEAD; run `tools/ci` checks; record warnings and runtimes | none | Baseline compile log, DreamDaemon boot, `-DALL_MAPS` compile | n/a |
| **1 Bridge** | One `merge -s ours fa47d5fe77` commit | none (tree identical to HEAD) | `git diff HEAD~1 HEAD` empty | reset to tag |
| **2 Stage 1 merge** | Upstream up to `e09bf3db6d` | Serendipity1/2 (HIGH), `maps.txt`, `_lists.dm`, `priority_announce.dm`, `astrometrics.dm`, `combat_handling.dm` | Compile; map compile; Serendipity tile-level report; localisation grep | Phase 1 commit |
| **3 Stage 2 (weapon refactor)** | `84a268eb9e` only | No textual conflicts expected. Semantic: Polish strings in rewritten files | Compile; shooting/AMS/PDC/AI tests; grep of Polish-translated literals | Phase 2 commit |
| **4 Stage 3** | through `ff23bf114d` | Shrike (HIGH) | Compile; Shrike tile-level merge; docking ports | Phase 3 commit |
| **5 Stage 4** | through `69565b730f` | Von Neumann (MED-HIGH), Tycoon tweaks | Compile; full smoke suite | Phase 4 commit |
| **6 Localisation sweep** | Re-translate English text returned by upstream rewrites (user decision per string set) | none | String grep + in-game | Phase 5 commit |
| **7 Cleanups (optional, user-approved)** | Remove stale `ammo_sorter` duplicate, decide `Aquila_*` forks | none | Compile | Phase 6 commit |

Preferably do each stage in a throwaway integration branch, then fast-forward.

---

## 13. Build / toolchain

- **Required (same before and after):** BYOND **515.1633** (DreamMaker/DreamDaemon), rust_g 3.1.3 (`rust_g.dll` shipped), auxmos 2.5.2-b (`auxmos.dll`/`libauxmos.so` shipped), Node **18.14.2**, yarn 3.4.1, Python 3.11.2 (`tools/requirements.txt`: upstream Pillow 12.3.0 vs Aquila 10.3.0), SpacemanDMM suite-1.8 (dreamchecker), MySQL/MariaDB for DB-backed rounds.
- **Build:** `BUILD.bat` / `tools/build/build` (Juke). CI builds `-DCIBUILDING -DCITESTING -DALL_MAPS`.
- **Differences E→master:** CI download user-agent (`tools/ci/download_byond.sh`, `install_byond.sh`), Pillow 10.3.0→12.3.0, webpack ^5.94→^5.104, tgui yarn.lock. Nothing else.
- **Aquila overrides to keep:** `build.js` `aquila/**` input; workflow runner/guard edits; DME `// AQUILA` block.
- **This machine:** BYOND, `DreamMaker`, `node`, `yarn` are not on PATH and `Program Files (x86)/BYOND` was not found. Nothing was installed or upgraded.

---

## 14. Testing strategy

> **SUPERSEDED by Revision 2 (authoritative-map policy):** the map-related bullets that compare maps "before vs after" become exact byte comparison against `AQUILA_MAP_BASELINE.tsv`. Everything else stands.

**Automated validation**
- `tools/build/build --ci dm -DCIBUILDING -DCITESTING -DALL_MAPS`; dreamchecker (SpacemanDMM).
- `tools/ci/check_filedirs.sh nsv13.dme`, `check_grep.sh`, `check_changelogs.sh`, `json_verifier.py`.
- `mapmerge2.dmm_test`; custom script: for every `.dmm`, verify all type paths resolve and every key is used (map integrity).
- Script: every `#include` in `nsv13.dme` and `aquila/aquila.dm` exists (already true at HEAD).
- Script: diff of defined `datum/design` ids/paths for duplicates.
- tgui build (`yarn` in `tgui/`).
- Unit tests under `code/modules/unit_tests`.

**Headless runtime validation (DreamDaemon, `-DCITESTING`)**
- Boot and runtime initialisation, no runtimes in init; round start on each votable map (`aquila_atlas`, `aquila_snake`, `aquila_serendipity`, `atlas`, `atlantis`, `snake`, `serendipity`, `tycoon`, plus `shrike`, `vonneumann`); job assignment; ship spawning; DB connect.

**Real BYOND client validation**
- TGUI panels (Starmap, Astrometrics, Tactical, HybridWeapons, Railgun*, OrdnanceConsole); sound and icon assets; Polish text display; jukebox; vampire HUD.

**Manual gameplay/map validation**
- Overmap movement/collision; docking incl. Infiltrator shuttle at each ship; combat: broadsides, deck guns, railgun, PDC/AMS, torpedoes/VLS; DRADIS/sensors; KNPC and fighters; boarding and interiors; damage propagation; atmos on each Aquila map (Atlas engine variants, Serendipity pipes post-merge); power networks (Babylon fix); persistence/metacoin.
- For every map in the "BOTH" group: visual diff screenshots before vs after, plus tile-count and area-count comparison.

---

## 15. Highest-risk areas

1. Serendipity 1/2 and Shrike 2 tile-level merges.
2. Silent loss of Polish strings in the weapon-datum rewrite.
3. Infiltrator docking ports on new/redesigned ships.
4. The `Aquila_*` frozen forks diverging from supported content.
5. Unverified baseline: HEAD has not been shown to compile here; there may be pre-existing issues (e.g. duplicated designs, `DEFECATION` changes).
6. Hand-merged `config/maps.txt` (default map and rotation).
7. DMM key-aliasing corrupting "clean" merges.

---

## 16. Unknowns

- Whether HEAD compiles and boots, and its current runtime/warning count.
- `fa47d5fe77` is inferred. Spot-checks agree, but a different 1–2 snapshot dates would change a few diffs. Maps were copied manually, so their true base is less certain than code's.
- ~490 modified files in `code/` were not individually read; my statements about them rest on file counts, `AQ EDIT` markers and commit titles.
- Unmerged Aquila branches' content.
- Whether the `ammo_sorter` duplicates differ in cost/behaviour from upstream's.
- Intent behind #317's `DEFECATION_*` change and the unused `recipes_pie.dm`.
- Whether Von Neumann should be playable on Aquila.
- Gameplay equivalence of Aquila and upstream boarding-mob changes.
- Atmos, movement, QDEL/timer API deltas were not examined beyond the small `code/` diff.
- No BYOND client or server was run.

---

## 17. Exact recommended first implementation step

> **SUPERSEDED by Revision 2 (authoritative-map policy):** Phase 0 has now been run (result: BASELINE FAIL). See "Revised plan" and the final GO/NO-GO.

**Phase 0 (no repository changes beyond a local tag):**
1. Create a local annotated tag at `f847d3caca` (e.g. `aquila-baseline-2025-09-09`). Do not push it unless you choose to.
2. Install the pinned toolchain (BYOND 515.1633, Node 18.14.2, Python 3.11) per §13.
3. Compile the **unmodified** HEAD with `tools/build/build --ci dm -DCIBUILDING -DCITESTING -DALL_MAPS`, boot it headless, and save the output as the baseline log.
4. Only then do Phase 1 (the `-s ours` bridge merge).

---

# GO / NO-GO

> **SUPERSEDED by Revision 2 (authoritative-map policy):** see the new verdict at the end of this document.

**GO**, for Phase 0 and Phase 1, with these conditions.

Evidence supports starting safely:
- HEAD is identical to `aquila/master`, so the starting state is fully recoverable.
- The effective base (`fa47d5fe77`) reduces the problem to 214 commits, 38 overlapping files and 8 textual conflicts.
- The toolchain is unchanged.
- Aquila's naval-code divergence is small and mostly localisation.

**First implementation phase:** Phase 0 above (tag, toolchain, baseline compile/boot), then the bridge merge.

**Conditions before Phase 2 (the first real upstream merge):**
- The baseline must compile and boot. If it does not, fix or document that first.
- You decide whether Von Neumann enters rotation and what happens to the frozen `Aquila_*` fork maps (can be deferred to Phase 5/7).
- A tile-level map-merge tool/script must exist before touching `Serendipity` or `Shrike`.



---

# Revised plan (Revision 2): authoritative Aquila maps

## R1. What upstream map changes would have touched, and what happens now

Nothing from these is imported. They are listed so the guard in R3 knows what to expect in a raw upstream merge.

- Upstream-only `.dmm` changes in `fa47d5fe77..upstream/master` that a plain merge would apply silently, **even without conflicts** (Aquila did not touch them, so Git would take upstream): `Shrike1`, `Tycoon1`, `Aetherwhisp1`, `Babylon2` (Instanced), `mining11`, the six `_maps/templates/boarding/syndicate/*.dmm`. Under the policy these must be **reverted to the baseline** after each merge stage, just like the conflicting ones.
- Upstream-only **new** map files: `_maps/map_files/vonneumann/vnmk3.dmm`, `_maps/shuttles/cargo/cargo_aiship.dmm`, plus `_maps/vonneumann.json`, `_maps/vonneumann.dm`, `_maps/map_files/vonneumann/job_changes.dm` and the `config/maps.txt` / `_maps/_basemap.dm` entries. These must be **excluded**, not merely left out of the rotation (see R3).
- Conflicting `.dmm`: `Serendipity1/2`, `Shrike2` (resolve to baseline; do not look at upstream hunks).
- Textually clean but changed `.dmm`: `Tycoon2`, `Hammerhead`, `Aetherwhisp2`, `CentCom` (also revert to baseline).

## R2. Compatibility check (the only allowed reason to touch a map)

Evidence gathered so far, *before* any merge:
- Of the 35 type paths upstream removed between `fa47d5fe77` and master (all in the weapon-datum refactor: `/datum/ship_weapon/*`, `/datum/ams_mode/*`, plus formatting churn on `/obj/structure/overmap`), **none is named in any of the 733 baseline maps**. `/obj/machinery/computer/ams` appeared to be removed but is still defined (moved file).
- Of the 340 distinct (type, var) var-edits that baseline maps apply to naval types (ship weapons, overmap, munitions, deck turrets, fighters, computers/ship, FTL, PDC, railgun, AMS, helm, tactical, DRADIS), **all var names are still present somewhere in upstream master code**. (Heuristic: by name, not by type.)
- Therefore the *expected* number of required map compatibility edits is **0**. This is a prediction, not proof; the `ALL_MAPS` compile, `map_errors.log` and runtime tests per stage are the proof.

**Map compatibility log** (empty so far). Every future entry must contain: map, exact tile/coordinate or key, old path/var, new path/var, why upstream code requires it, upstream commit, and the resulting git blob.

| # | Map | Location | Old | New | Reason / upstream commit | New blob |
|---|---|---|---|---|---|---|
| none yet | | | | | | |

## R3. Map-protection protocol for every merge stage

A plain `git merge` cannot protect unconflicted upstream map edits (a merge driver or `-X ours` only acts on conflicts). So each stage is run with `--no-commit` and finished by an explicit restore step:

1. `git merge --no-commit --no-ff <stage-tip>` (never fast-forward, never commit before step 5).
2. Restore **every** baseline map to its baseline blob: for each path in `AQUILA_MAP_BASELINE.tsv`, `git checkout <baseline-tag> -- <path>` (or `git restore --source=<baseline-tag> --staged --worktree`). Also restore the map-adjacent files that define the map set: `_maps/*.json`, `_maps/_basemap.dm`, per-ship `job_changes.dm`, `config/maps.txt`.
3. Remove anything upstream added under `_maps/` that is not in the baseline (`vonneumann/*`, `cargo_aiship.dmm`, `vonneumann.json/.dm`, and so on) with `git rm`, and remove any `.dme` / `_basemap.dm` lines that include them.
4. Verify with the script in `AQUILA_MAP_BASELINE.md`: 0 `CHANGED`, 0 new `.dmm`, and `config/maps.txt` rotation identical.
5. Commit the merge only after the compile/test gate for that stage passes. Any compatibility-log entry is applied as a separate, labelled commit afterwards, and gets a documented "baseline v2" row for that file rather than silently regenerating the TSV.
6. Shared holodeck/template `.dmm` files are maps too and are protected the same way.

Caveat to settle in Phase 1: because step 2 deliberately makes the merge commit disagree with upstream on maps, a *later* merge will see those maps as "changed on our side" and conflict again whenever upstream touches them. That is expected and handled by the same protocol; it is not a sign of a failed merge.

## R4. Revised strategy

Still Option D (hybrid), now with the map-protection protocol:
1. Repair the baseline defects (decision needed, see R7), or explicitly accept them as known failures.
2. Ancestry bridge `merge -s ours fa47d5fe77` (tree-neutral, so maps cannot change).
3. Four staged upstream merges (boundaries unchanged: `e09bf3db6d` → `84a268eb9e` → `ff23bf114d` → `69565b730f`), each using R3.
4. Localisation sweep for strings that the weapon-datum rewrite brought back in English.
5. Optional, separately approved: stale `ammo_sorter` duplicate cleanup; any new upstream map evaluated as its own feature.

Expected textual conflicts under the new policy (explicit base `fa47d5fe77`): the 3 map conflicts disappear by construction. The remaining non-map conflicts are `config/maps.txt` (keep Aquila), `code/__HELPERS/_lists.dm`, `code/__HELPERS/priority_announce.dm`, `nsv13/.../combat_handling.dm`, `nsv13/.../astrometrics.dm`.

## R5. Revised phases

| Phase | Scope | Map impact | Gate | Rollback |
|---|---|---|---|---|
| 0 (done) | Tag, map baseline, toolchain, baseline build | none | BASELINE FAIL (known defects) | n/a |
| 0.5 (needs decision) | Repair baseline defects: resolve the 3 conflict-marker maps to the Aquila side; `OVERFLOW_JOB`; optionally the runtimes. Produce **baseline v2** TSV and tag | **The only allowed pre-migration map edit**: 3 maps, documented as baseline repair, not migration | `ALL_MAPS` compile passes; 40/40 unit tests; ideally 0 runtimes | `aquila-baseline-2025-09-09` |
| 1 | Ancestry bridge (`merge -s ours fa47d5fe77`) | none; tree identical | `git diff HEAD~1 HEAD` empty; map script 0 changes | previous commit |
| 2 | Stage 1 upstream (to `e09bf3db6d`) with R3 | 0 expected | compile + ALL_MAPS + tests + map script | Phase 1 commit |
| 3 | Stage 2 `84a268eb9e` (weapon-datum refactor) with R3 | 0 expected; **compat check matters most here** | same + naval runtime tests | Phase 2 commit |
| 4 | Stage 3 (to `ff23bf114d`) with R3 | 0 expected | same | Phase 3 commit |
| 5 | Stage 4 (to `69565b730f`) with R3 | 0 expected; Von Neumann must be absent | same | Phase 4 commit |
| 6 | Localisation sweep | none | grep + in-game | Phase 5 commit |

## R6. Revised testing additions

- After **every** stage: run the baseline-compare script (R3 step 4); `ALL_MAPS` compile; a DreamDaemon run with `-DCITESTING` for each votable map (`aquila_atlas`, `aquila_snake`, `aquila_serendipity`, `atlas`, `atlantis`, `snake`, `serendipity`, `tycoon`), checking `map_errors.log` and `runtime.log` against the Phase-0 runtime list (anything new is a regression; anything that disappears must be explained).
- For `deck_turret`: confirm whether the upstream rewrite changes the null-core behaviour that already fails on `Aquila_Atlas` at baseline.
- Static check that `config/maps.txt` has no `vonneumann` and that `_basemap.dm` / the `.dme` do not include it.

## R7. Decisions requested before Phase 1

1. **Three conflict-marker maps** (`Serendipity2`, `snake_upper`, `Tycoon2`): confirm that the `HEAD` side of each hunk is the authoritative Aquila content, and that repairing them (Phase 0.5) is allowed as a documented baseline repair. Without this the baseline stays broken for `ALL_MAPS`, and Serendipity/Snake stay unloadable in rotation. I have not touched them.
2. Fix or accept the config/runtime defects (`OVERFLOW_JOB` vs "Majtek", gulag spawner, deck-turret cores).

# GO / NO-GO (Revision 2)

**NO-GO for Phase 1 as originally scheduled; GO as soon as the two decisions in R7 are made.**

Reason: Phase 0 returned BASELINE FAIL. The failures are old, isolated and well understood (3 maps with committed conflict markers; 3 small runtime issues). They are not caused by the migration, but a migration compared against a broken baseline cannot prove it changed nothing. The ancestry bridge itself is tree-neutral and would be safe to run now, but doing it first would blur the "what was already broken" record you asked for.

**Updated recommendation for Phase 1:** after Phase 0.5 (baseline v2) is approved and done, run exactly one operation: `git merge -s ours fa47d5fe77` on the migration branch, then verify the tree is identical and every map blob matches the baseline (v2) inventory. No upstream content is imported in Phase 1. Not executed; waiting for your confirmation.


---

# Phase 1 record: ancestry bridge (done, not pushed)

| Item | Value |
|---|---|
| Original Aquila baseline | `f847d3caca` (tag `aquila-baseline-2025-09-09`, unchanged) |
| Repaired Aquila baseline (authoritative) | `34fa11431f` "Fix pre-migration Aquila baseline issues" |
| **Bridge commit** | `d253381c6417230c18213a2a82d95c018cc9c7ad` (`merge -s ours --no-ff fa47d5fe77`) |
| First parent / second parent | `34fa11431f` / `fa47d5fe77f9af12784e5eb37428c78d48067782` |
| Tree | `010189278a08ea49121ab9b87c1b6b9292d581fc` before and after: **0 tracked file changes** |
| Authoritative maps | 733 checked against `AQUILA_MAP_BASELINE_REPAIRED.tsv`, **0 mismatches**, 0 new `.dmm` |
| `upstream/master` (fetched, working tree untouched) | `69565b730fdf45c09861dfca6c669b67f8601a71` (2026-09-12) |
| `git merge-base HEAD upstream/master` | `fa47d5fe77` (was `b6fc4ce967`, 2024-01-03) |

## Effective base, re-verified independently
Scan of every first-parent upstream commit 2025-03-15..2025-07-15 against the 5,522 files of `nsv13/code`, `code`, `tgui/packages` at `34fa11431f`: `fa47d5fe77` ("Hard mode", 2025-05-01) and the three changelog-only commits after it tie for the best match (4,345 identical blobs); `bb47a7cc4f` (2025-05-23) is already worse (4,344). `fa47d5fe77` is on upstream's first-parent line.
**Refinement:** #317's own conflict markers name `d5afbe889c`, a changelog-only descendant of `fa47d5fe77` (two commits later; trees differ only in `html/changelog.html` and `html/changelogs/.all_changelog.yml`). Aquila's `.all_changelog.yml` equals `d5afbe889c`'s, not `fa47d5fe77`'s. The code content is identical, so the bridge parent `fa47d5fe77` is correct for code; the changelog files simply count as changed on both sides identically/trivially. Nothing contradicts the base.

## Recalculated migration situation (HEAD = bridge → upstream/master)
| Measure | Value |
|---|---|
| Upstream commits awaiting migration | **214** (0 merge commits) |
| Aquila-only commits | 179 (177 original + repair + bridge) |
| Files changed upstream (vs `fa47d5fe77`) | **235** (33 A, 3 D, 199 M; +95,827/−16,560) |
| Files changed on Aquila side (vs `fa47d5fe77`) | **1,471** (+1,087,802/−37,151) |
| Files changed by both | **38** (7 `.dmm`, 31 non-map) |
| Dry-run textual conflicts (`git merge-tree`, nothing written to the tree) | **8** |
| Non-map textual conflicts | **5**: `code/__HELPERS/_lists.dm`, `code/__HELPERS/priority_announce.dm`, `config/maps.txt`, `nsv13/.../fleet_combat/combat_handling.dm`, `nsv13/.../research/astrometrics.dm` |
| Map textual conflicts | **3**: `Serendipity1.dmm`, `Serendipity2.dmm`, `Shrike2.dmm` (Serendipity1 is new vs the earlier forecast because of the Phase 0.5 repair) |
| Both-changed `.dmm` (7) | Aetherwhisp2, Hammerhead, Serendipity1, Serendipity2, Shrike2, Tycoon2, CentCom |
| Upstream `.dmm` touched | 20 (18 M, 2 A) |

**Map policy for the migration (authoritative):** the repaired maps at `34fa11431f` stay as they are.
- The 18 modified upstream `.dmm` (of which only 3 conflict textually; 11 more would merge *silently*: Shrike1, Tycoon1, Aetherwhisp1, Babylon2, mining11, six boarding templates, plus the clean-but-changed Tycoon2, Hammerhead, Aetherwhisp2, CentCom) are all reverted to the repaired baseline after every stage via the `--no-commit` + restore protocol (R3).
- The 2 added upstream `.dmm` (`vonneumann/vnmk3.dmm`, `shuttles/cargo/cargo_aiship.dmm`) and `vonneumann.json/.dm`, plus their `maps.txt` / `_basemap.dm` entries are excluded.
- Map edits are allowed only as documented minimal compatibility edits (log C1 so far).

Predicted: 5 hand-resolved non-map conflicts (keep Aquila strings/config, take upstream logic), 0 map conflicts to resolve by content (all 3 resolved by "keep the baseline blob"), plus a semantic review of the weapon-datum rewrite for lost Polish strings (count unknown until the stage is merged).
