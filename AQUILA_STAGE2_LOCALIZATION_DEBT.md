# Stage 2 localization debt (weapon datum refactor `84a268eb9e`)

New upstream English text introduced by the refactor, **not translated** (per rules). Heuristic extraction (multi-word literals >=10 chars on added lines). Existing Aquila Polish strings in all touched files were preserved (21/21 verbatim).

The commit adds 110 such literals, but **70 of them are old English text that already existed in Stage 1 and was only moved** (e.g. weapon select/failure alerts from the deleted `weapon_types.dm`). Aquila had never translated any of those, so nothing regresses. Genuinely new English text: **40** in 12 files (39 weapon-facing).

## Genuinely new strings

| Priority | Weapon-facing | File | Type / proc | English text | Player-facing |
|---|---|---|---|---|---|
| HIGH | yes | `weapon_datum_types.dm` | `/datum/overmap_ship_weapon/mac/is_target_size_valid` | <span class='notice'>Charging hybrid railgun hardpoints...</span> | player-facing |
| HIGH | yes | `weapon_datum_types.dm` | `/datum/overmap_ship_weapon/flak/Destroy` | <span class='notice'>STS missile target acquisition systems: online.</span> | player-facing |
| HIGH | yes | `OrdnanceConsole.js` | `` | Expanded Info | player-facing? |
| HIGH | yes | `OrdnanceConsole.js` | `` | Filtered ammo:  | player-facing? |
| HIGH | yes | `TacticalConsole.js` | `` | Expanded Info | player-facing? |
| HIGH | yes | `TacticalConsole.js` | `` | Filtered ammo:  | player-facing? |
| MEDIUM | yes | `overmap.dm` | `/datum/keybinding/overmap/cycle_firemode/down` | Special Weapon Action | player-facing |
| MEDIUM | yes | `overmap.dm` | `/datum/keybinding/overmap/cycle_firemode/down` | Executes a selected weapon's special action, if one exists. | player-facing? |
| MEDIUM | yes | `overmap.dm` | `/datum/keybinding/overmap/special_weapon_action/down` | <span class='warning'>Current weapon has no special action!</span> | player-facing |
| MEDIUM | no | `plasma_loader.dm` | `/obj/machinery/atmospherics/components/unary/plasma_loader/multitool_act` | <span class='notice'>You connect [src] to [caster].</span> | player-facing |
| MEDIUM | yes | `overmap_ghosts.dm` | `/mob/living/verb/reassume_ship_control` | Reassume Ship Control | player-facing |
| MEDIUM | yes | `_overmap_ship_weapon.dm` | `/datum/overmap_ship_weapon` | Generic ship weapon. You shouldn't see this. | player-facing |
| MEDIUM | yes | `_overmap_ship_weapon.dm` | `/datum/overmap_ship_weapon/proc/cycle_ammo_filter` | <span class='notice'>No loaded ammunition detected, clearing filter.</span> | player-facing |
| MEDIUM | yes | `_overmap_ship_weapon.dm` | `/datum/overmap_ship_weapon/proc/cycle_ammo_filter` | <span class='notice'>Ammunition filter cleared.</span> | player-facing |
| MEDIUM | yes | `_overmap_ship_weapon.dm` | `/datum/overmap_ship_weapon/proc/cycle_ammo_filter` | <span class='notice'>Ammunition filter set to [initial(prototype_ammo.name)].</span> | player-facing |
| MEDIUM | yes | `_overmap_ship_weapon.dm` | `/datum/overmap_ship_weapon/proc/get_controller_string` | Operated Manually | player-facing? |
| MEDIUM | yes | `firing.dm` | `/datum/overmap_ship_weapon/proc/fire_proc_chain` | Error - unknown firing cycle failure. ([src]) | player-facing? |
| MEDIUM | yes | `firing_checks.dm` | `/datum/overmap_ship_weapon/proc/get_nonphysical_ammo` | the amount of shots left that a weapon can currently fire | player-facing? |
| MEDIUM | yes | `firing_checks.dm` | `/datum/overmap_ship_weapon/proc/can_fire` | Error - invalid target. ([src]) | player-facing? |
| MEDIUM | yes | `firing_checks.dm` | `/datum/overmap_ship_weapon/proc/can_fire` | Error - ammunition or charge depleted. ([src]) | player-facing? |
| MEDIUM | yes | `firing_checks.dm` | `/datum/overmap_ship_weapon/proc/can_fire` | Error - Target IFF friendly. ([src]) | player-facing? |
| MEDIUM | yes | `firing_checks.dm` | `/datum/overmap_ship_weapon/proc/can_fire` | Error - invalid angle. ([src]) | player-facing? |
| MEDIUM | yes | `firing_checks.dm` | `/datum/overmap_ship_weapon/proc/can_fire` | Error - target did not pass analysis. ([src]) | player-facing? |
| MEDIUM | yes | `firing_checks.dm` | `/datum/overmap_ship_weapon/proc/can_fire_physical` | Error - No ready weapons in control group. ([src]) | player-facing? |
| MEDIUM | yes | `firing_checks.dm` | `/datum/overmap_ship_weapon/proc/can_fire_physical` | Error - no weapon in control group ready to fire. ([src]) | player-facing? |
| MEDIUM | yes | `weapon_datum_types.dm` | `/datum/overmap_ship_weapon/railgun/is_target_size_valid` | Hybrid Railguns | player-facing |
| MEDIUM | yes | `weapon_datum_types.dm` | `/datum/overmap_ship_weapon/missile_launcher/is_target_size_valid` | Uh oh, this is a basetype, you shouldn't be seeing this! | player-facing |
| MEDIUM | yes | `weapons.dm` | `/obj/structure/overmap/proc/fire_weapon` | <span class='warning'>[report_list[1]]</span> | player-facing |
| LOW | yes | `phaser.dm` | `/obj/machinery/ship_weapon/energy/get_ammo_list` | Attempting to get physical ammo of an energy weapon. Check your proc chains. | admin/log |
| LOW | yes | `_fighters.dm` | `/obj/structure/overmap/small_craft/hardpoint_fire` | HEY hardpoint_fire is being called with an invalid mode ([osw_mode]) | admin/log |
| LOW | yes | `_fighters.dm` | `/obj/structure/overmap/small_craft/hardpoint_fire` | HEY hardpoint_fire is being called with an invalid mode ([fire_mode]) | admin/log |
| LOW | yes | `_fighters.dm` | `/obj/structure/overmap/small_craft/hardpoint_fire` | HEY hardpoint_fire is being called with an invalid mode ([fire_mode]) | admin/log |
| LOW | yes | `_overmap_ship_weapon.dm` | `/datum/overmap_ship_weapon/New` | Invalid weapon control flags. Must not be NONE. | admin/log |
| LOW | yes | `_overmap_ship_weapon.dm` | `/datum/overmap_ship_weapon/New` | Invalid weapon facing flags. Must not be NONE. | admin/log |
| LOW | yes | `_overmap_ship_weapon.dm` | `/datum/overmap_ship_weapon/Destroy` | Ship weapon deleting with nonstandard controller count ([controller_count]). This points to codeside issu | admin/log |
| LOW | yes | `_overmap_ship_weapon.dm` | `/datum/overmap_ship_weapon/proc/link_weapon` | [src] was already linked to [linked_overmap] when an attempt to link to [link_to] was made. This is not a | admin/log |
| LOW | yes | `_overmap_ship_weapon.dm` | `/datum/overmap_ship_weapon/proc/link_weapon` | [src] attempted to link weapon without a link target. This is not allowed. | admin/log |
| LOW | yes | `firing.dm` | `/datum/overmap_ship_weapon/proc/use_nonphysical_ammo` | Invalid nonphysical ammunition define used. ([used_nonphysical_ammo]) | admin/log |
| LOW | yes | `firing_checks.dm` | `/datum/overmap_ship_weapon/proc/get_ai_ammo_by_current_weapon_class` | Invalid nonphysical ammunition define used. ([used_nonphysical_ammo]) | admin/log |
| LOW | yes | `firing_checks.dm` | `/datum/overmap_ship_weapon/proc/get_ai_ammo_by_current_weapon_class` | Invalid nonphysical ammunition define used. ([used_nonphysical_ammo]) | admin/log |

## Relocated existing English text (not new; listed by file)

`weapon_datum_types.dm` (45), `_fighters.dm` (13), `autonomy.dm` (5), `tactical.dm` (2), `repair_kit.dm` (2), `_overmap_ship_weapon.dm` (2), `weapons.dm` (1)
