# Stage 4 localization debt (upstream `ff23bf114d..69565b730f`)

New upstream English text, **not translated** (per rules). Heuristic extraction (multi-word literals >=10 chars on added lines in `nsv13/code`, `code`, `tgui/packages`, `strings`, `config`, `html/admin`, `interface`). Existing Aquila Polish strings were preserved.

20 literals on added lines; 4 are English text that already existed in the Stage 3 tree; **15 genuinely new** (the 16th, the Von Neumann cargo-ferry name in `shuttles.dm`, was removed with the map) in 9 files. By flag (new): {'player-facing': 16}; by priority: {'MEDIUM': 15, 'HIGH': 1}.

| Priority | Player-facing | File | Type / proc | English text |
|---|---|---|---|---|
| HIGH | player-facing | `stormdrive.dm` | `/obj/machinery/atmospherics/components/binary/stormdrive_reactor/attackby` | [src] is not in maintenance mode! Manually removing a control rod from an active nuclear reaction would p |
| MEDIUM | player-facing | `laser.dm` | `/obj/item/gun/energy/laser/retro` | A modern lasergun, used by Solgov's security or military forces. it is quite deadly and easy to maintain, |
| MEDIUM | player-facing | `laser.dm` | `/obj/item/gun/energy/laser/retro/old` | First generation lasergun, designed by Nanotrasen and produced by Solgov. Suffers from ammo issues but it |
| MEDIUM | player-facing | `munitions.dm` | `/obj/machinery/computer/ship/munitions_computer` | This console allows you to control and monitor a linked ship weapon. |
| MEDIUM | player-facing | `deck_guns.dm` | `/obj/machinery/ship_weapon/deck_turret/proc/rack_load` | Deck Gun Control Console |
| MEDIUM | player-facing | `deck_guns.dm` | `/obj/machinery/ship_weapon/deck_turret/proc/rack_load` | A control console for a deck gun. |
| MEDIUM | player-facing | `ammo_rack.dm` | `/obj/machinery/computer/ammo_sorter` | This console allows you to manage and control linked ammo racks. From here, you can monitor the status of |
| MEDIUM | player-facing | `ammo_rack.dm` | `/obj/machinery/computer/ammo_sorter/proc/unlinkSorter` | A machine that allows you to compartmentalise the ship's ammo stores, controlled by a central console. |
| MEDIUM | player-facing | `_fighters.dm` | `/obj/item/fighter_component/armour_plating/welder_act` | compact shield generator array |
| MEDIUM | player-facing | `_fighters.dm` | `/obj/item/fighter_component/armour_plating/welder_act` | A rather heavy, and highly illegal shield generator, paired with a fine, lightly armored self-applicating |
| MEDIUM | player-facing | `_fighters.dm` | `/obj/item/fighter_component/armour_plating/tier6/on_install` | <span class='notice'>\The [src] starts up, humming loudly.</span> |
| MEDIUM | player-facing | `_fighters.dm` | `/obj/item/fighter_component/armour_plating/tier6/remove_from` | <span class='warning'>\The [src] shuts down, melting into slag.</span> |
| MEDIUM | player-facing | `solgov.dm` | `/obj/structure/overmap/nanotrasen/solgov/vnc` | Von Neumann class patrol frigate |
| MEDIUM | player-facing | `solgov.dm` | `/obj/structure/overmap/nanotrasen/solgov/vnc` | A long-range SolGov patrol boat, usually used for clearing asteroid fields. |
| MEDIUM | player-facing | `autonomy.dm` | `/datum/ams_mode/countermeasures/acquire_targets` | The Anti-Missile System (AMS) is a computer system that can automatically target and fire upon threats to |

## Existing English moved/re-added
`client_procs.dm` (1), `energy_gun.dm` (1), `_fighters.dm` (1), `autonomy.dm` (1)

## Existing Aquila translations affected
Case A/C/D: none. Stage 4 did not touch any Aquila-localized string (the only manifest files touched were `laser.dm` (descriptions, see below), `stormdrive.dm`, `ai-skynet.dm`, `overmap.dm`, `damage.dm`, `config.txt`, changelog files).

## Cumulative genuinely-new English strings, Stages 1-4 (heuristic counts)
| Stage | New literals | Notes |
|---|---|---|
| 1 | 79 (about 33 admin/log) | VV port, KNPC, astrometrics, closets, signs |
| 2 | 40 (70 more were relocated existing text) | weapon datum files |
| 3 | 85 (15 relocated) | Railgun Forge |
| 4 | 15 (4 relocated) | Von Neumann cargo-ferry name excluded (removed) |
| **Total** | **219** | |
