# Stage 3 localization debt (upstream `84a268eb9e..ff23bf114d`)

New upstream English text, **not translated** (per rules). Heuristic extraction: multi-word literals >=10 chars on added lines. Existing Aquila Polish strings in touched files were preserved.

100 literals on added lines; 15 are English text that already existed in the Stage 2 tree and was only moved or re-added; **85 are genuinely new** in 14 files. By flag (new only): {'player-facing': 69, 'admin/log': 1, 'player-facing?': 15}; by priority: {'MEDIUM': 62, 'LOW': 1, 'HIGH': 22}.

| Priority | Player-facing | Weapon/overmap | File | Type / proc | English text |
|---|---|---|---|---|---|
| HIGH | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | Allocated resources will not be recoverable when purging the coating tank |
| HIGH | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | Allocated resources will not be recoverable when purging the core tank |
| HIGH | player-facing? | yes | `HybridWeapons.js` | `` | I4 - Configuration: Canister Type |
| HIGH | player-facing | yes | `RailgunCanisterCharger.js` | `` | Canister Statistics |
| HIGH | player-facing? | yes | `RailgunCanisterCharger.js` | `` | Nil Canister Detected |
| HIGH | player-facing | yes | `RailgunCanisterCharger.js` | `` | Charger Settings |
| HIGH | player-facing | yes | `RailgunFiller.js` | `` | Canister Actions |
| HIGH | player-facing? | yes | `RailgunFiller.js` | `` | Nil Canister Detected |
| HIGH | player-facing? | yes | `RailgunFiller.js` | `` | Empty Canister |
| HIGH | player-facing? | yes | `RailgunFiller.js` | `` | Fill Canister |
| HIGH | player-facing? | yes | `RailgunFiller.js` | `` | Seal Canister |
| HIGH | player-facing | yes | `RailgunFiller.js` | `` | Canister Gases |
| HIGH | player-facing | yes | `RailgunFiller.js` | `` | Input Gases |
| HIGH | player-facing? | yes | `RailgunForge.js` | `` | Print Slug |
| HIGH | player-facing? | yes | `RailgunForge.js` | `` | Print Canister Round |
| HIGH | player-facing | yes | `RailgunForge.js` | `` | Coating Tank |
| HIGH | player-facing? | yes | `RailgunForge.js` | `` | Set Coating Tank Material |
| HIGH | player-facing? | yes | `RailgunForge.js` | `` | Process Tank 1 |
| HIGH | player-facing? | yes | `RailgunForge.js` | `` | Purge Tank 1 |
| HIGH | player-facing? | yes | `RailgunForge.js` | `` | Set Core Tank Material |
| HIGH | player-facing? | yes | `RailgunForge.js` | `` | Process Tank 2 |
| HIGH | player-facing? | yes | `RailgunForge.js` | `` | Purge Tank 2 |
| MEDIUM | player-facing | no | `shower.dm` | `/obj/structure/showerframe` | A shower frame, that needs 2 plastic sheets to finish construction into a basic shower, or can be made in |
| MEDIUM | player-facing | no | `standard.dm` | `/datum/outfit/assassin/post_equip` | Centcom Outfit basetype - Do Not Use |
| MEDIUM | player-facing | no | `custom_outfits.dm` | `/datum/outfit/sleeper` | Sleeper Outfit basetype - Do Not Use :) |
| MEDIUM | player-facing | no | `custom_outfits.dm` | `/obj/item/clothing/suit/ship/squad/military_police/general/Initialize` | Space Pirate Boarder outfit basetype - Do Not Use :) |
| MEDIUM | player-facing | yes | `railgun_ammo.dm` | `/obj/item/ship_weapon/ammunition/railgun_ammo/Initialize` | \improper Forged 400mm |
| MEDIUM | player-facing | yes | `railgun_ammo.dm` | `/obj/item/ship_weapon/ammunition/railgun_ammo/Initialize` | \improper Forged 400mm copper coated iron slug |
| MEDIUM | player-facing | yes | `railgun_ammo.dm` | `/obj/item/ship_weapon/ammunition/railgun_ammo/Initialize` | \improper Forged 800mm |
| MEDIUM | player-facing | yes | `railgun_ammo.dm` | `/obj/item/ship_weapon/ammunition/railgun_ammo/Initialize` | A gigantic cansiter that's designed to be fired out of a railgun. It's extremely heavy, containing an int |
| MEDIUM | player-facing | yes | `railgun_ammo.dm` | `/obj/item/ship_weapon/ammunition/railgun_ammo_canister/examine` | <span class='notice'>The canister has been permamently sealed.</span> |
| MEDIUM | player-facing | yes | `railgun_ammo.dm` | `/obj/item/ship_weapon/ammunition/railgun_ammo_canister/examine` | <span class='notice'>The canister isn't sealed.</span> |
| MEDIUM | player-facing | yes | `railgun_ammo.dm` | `/obj/item/ship_weapon/ammunition/railgun_ammo_canister/attack_hand` | <span class='notice'>Your hand tingles and feels warm when touching the [src].</span> |
| MEDIUM | player-facing | yes | `railgun_ammo.dm` | `/obj/item/ship_weapon/ammunition/railgun_ammo_canister/attack_hand` | \improper Forged 800mm copper coated iron canister |
| MEDIUM | player-facing | yes | `hybrid_railgun.dm` | `/obj/machinery/ship_weapon/hybrid_rail/examine` | <span class='danger'>The railgun is in a critical state and requires repairing to function!</span> |
| MEDIUM | player-facing | yes | `hybrid_railgun.dm` | `/obj/machinery/ship_weapon/hybrid_rail/examine` | <span class='notice'>Repair Status: [obj_integrity] / [max_integrity]</span> |
| MEDIUM | player-facing | yes | `hybrid_railgun.dm` | `/obj/machinery/ship_weapon/hybrid_rail/examine` | <span class='notice'>Selected Munition: Canister type</span> |
| MEDIUM | player-facing | yes | `hybrid_railgun.dm` | `/obj/machinery/ship_weapon/hybrid_rail/welder_act` | <span class='notice'>You start repairing the railgun...</span> |
| MEDIUM | player-facing | yes | `hybrid_railgun.dm` | `/obj/machinery/ship_weapon/hybrid_rail/welder_act` | <span class='notice'>You finishing repairing the railgun.</span> |
| MEDIUM | player-facing | yes | `hybrid_railgun.dm` | `/obj/machinery/ship_weapon/hybrid_rail/attackby` | <span class='notice'>Error: Unable to load ordnance while railgun is in a critical state.</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge` | Railgun Forge |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge` | Device for forging railgun munitions |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/proc/forge_slug` | \improper Forged 400mm [T1.material_selection] coated [T2.material_selection] slug |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/proc/forge_canister` | \improper Forged 800mm [T1.material_selection] coated [T2.material_selection] canister |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='notice'>Error: Coating/Core tanks not detected!</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='notice'>Error: Fabrication materials not selected</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='notice'>Error: Insufficent fabrication resources</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='notice'>Error: Fabrication materials not selected</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='notice'>Error: Insufficent fabrication resources</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='warning'>Purging coating tank</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='warning'>Purging core tank</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='notice'>Error: Coating tank must be purged before selecting another material</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='notice'>Error: Core tank must be purged before selecting another material</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='notice'>Coating Tank Processing: Disabled.</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='notice'>Error: Material selection must be locked before processing.</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='notice'>Coating Tank Processing: Enabled.</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='notice'>Core Tank Processing: Disabled.</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='notice'>Error: Material selection must be locked before processing.</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_act` | <span class='notice'>Core Tank Processing: Enabled.</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge/ui_data` | Railgun Forge Coating Tank |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge` | Device for forging railgun munitions |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge_coating_tank/RefreshParts` | Railgun Forge Core Tank |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge` | Device for forging railgun munitions |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge_core_tank/RefreshParts` | Railgun Canister Filler |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_forge_core_tank/RefreshParts` | Device for filling and sealing railgun canister munitions |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/atmospherics/components/binary/railgun_filler/attackby` | <span class='notice'>You start to load [I] into [src]...</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/atmospherics/components/binary/railgun_filler/attackby` | <span class='notice'>There is already an [F] in [src].</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/atmospherics/components/binary/railgun_filler/ui_act` | <span class='notice'>Error: Unable to fill canister as it has been permanently sealed shut.</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/atmospherics/components/binary/railgun_filler/ui_act` | <span class='notice'>Error: Unabled to empty canister as it has been permanently sealed shut.</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/atmospherics/components/binary/railgun_filler/ui_act` | <span class='notice'>Error: Canister has already been sealed shut.</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/atmospherics/components/binary/railgun_filler/ui_act` | <span class='notice'>Attention: [F.name] has been been permamently sealed |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/atmospherics/components/binary/railgun_filler/ui_act` | <span class='danger'>The canister vents its contents!</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/atmospherics/components/binary/railgun_filler/ui_data` | Railgun Canister Charger |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/atmospherics/components/binary/railgun_filler/ui_data` | Device for charging and discharging railgun canister munitions |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/atmospherics/components/binary/railgun_filler/attackby` | <span class='notice'>You start to load [I] into [src]...</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/atmospherics/components/binary/railgun_filler/attackby` | <span class='notice'>There is already an [F] in [src].</span> |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_charger/ui_data` | Railgun Forge (Machine Board) |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_charger/ui_data` | Railgun Charger (Machine Board) |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_charger/ui_data` | Railgun Filler (Machine Board) |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_charger/ui_data` | Railgun Forge Coating Tank (Machine Board) |
| MEDIUM | player-facing | yes | `railgun_forge.dm` | `/obj/machinery/railgun_charger/ui_data` | Railgun Forge Core Tank (Machine Board) |
| MEDIUM | player-facing | yes | `overmap.dm` | `/obj/structure/overmap/proc/railgun_bluespace_recoil` | <span class='warning'>You feel as the ship gets suddenly pulled away!</span> |
| MEDIUM | player-facing | yes | `physics.dm` | `/obj/structure/overmap/proc/collide` | <span class='danger'>Craft anchored by larger vessel, brakes have been engaged!</span> |
| MEDIUM | player-facing? | no | `drug_reagents.dm` | `/datum/reagent/drug/highjack` | Creates effects akin to mind-altering substances when processed by synthetics. |
| LOW | admin/log | no | `api.dm` | `/datum/tgs_api/v5/TriggerDeployment` | Interop version too low for triggering deployments! |

## Existing English text moved/re-added (not new)

`railgun_forge.dm` (5), `HybridWeapons.js` (4), `combat_handling.dm` (2), `dummy_pilot.dm` (1), `railgun_ammo.dm` (1), `hybrid_railgun.dm` (1), `projectiles_fx.dm` (1)

## Existing Aquila translations affected
- Case A preserved: `priority_announce.dm` (`minor_announce` default title `Uwaga:` kept; upstream's new `silent` argument added) and `combat_handling.dm` (Polish combat-entry announcement kept, now with `silent = TRUE`).
- Case C (meaning changed): none. Case D (obsolete): none.
