# deep_dark_doorway
A Minecraft Data and Resource Pack that adds the Warden Heart item and Reinforced Deepslate functionality.

Use Reinforced Deepslate, Lodestone, and Warden Hearts to create Deep Dark Doorways to teleport quickly across the overworld!

# Installation
Drop the Data Pack Zip directly into your `.minecraft/saves/<WORLD_NAME>/datapacks/` folder for whichever world you want.

Place the Resource Pack Zip directly into your `.minecraft/resourcepacks/` folder.

# Deeper Datapack Explanation
Item Modifier/Loot Table Explanations
* item_modifiers/new_compass
  * new_compass.json
    * Set tag.ddd = ddd.tpDest
    * Sets name = Polarized Warden Heart
    * Sets CustomModelData = 2
    * Sets id = random
* loot_tables/
  * compass_dest.json
    * Returns compass with:
    * Set tag.ddd = ddd.tpDest
    * Set tag.LodstonePos = ddd.tpDest
    * Sets name = Polarized Warden Heart
    * Sets CustomModelData = 2
    * Sets id = random
  * compass_src.json
    * Returns compass with:
    * Set tag.ddd = ddd.tpDest
    * Sets name = Polarized Warden Heart
    * Sets CustomModelData = 2
    * Sets id = random
* minecraft/loot_tables/entities/warden.json
  * Returns compass with:
  * Sets CustomModelData = 1
  * Sets name = Warden Heart
  * Sets id = random

Advancement Explanations
* ddd/
  * used/
    * heart/
      * <NUMBER>.json
        * If compass{CustomModelData:1} in slot NUMBER, run entity/player/inventory/new/NUMBER
      * offhand.json
        * If compass not in mainhand and compass{CustomModelData:1} in offhand, run entity/player/inventory/new/offhand
    * paired_heart/<NUMBER>.json
      * <NUMBER>.json
        * If compass{CustomModelData:2} in slot NUMBER, run entity/player/inventory/old/NUMBER
      * offhand.json
        * If compass not in mainhand and compass{CustomModelData:2} in offhand, run entity/player/inventory/old/offhand
  * pair_doorway.json
    * Given for pairing two deep dark doorways
* minecraft/advancements/nether/use_lodestone.json
  * Given for using a compass without CustomModelData on a lodestone
* minecraft/advancements/end/enter_end_gateway.json
  * Given for using an End Gateway in the End.

MCFunction Explanations
* utils/
  * load.mcfunction 
    * Initialize "storage minecraft:ddd"
  * tick.mcfunction
    * Increment tick counter, every 10 ticks call utils/tick10.mcfunction
  * tick10
    * Foreach player[Inventory.contains(paired heart)] run entity/player/inventory/check/compass
    * Foreach Marker run entity/marker/tick/tick10
* entity/
  * marker/
    * tick/
      * break.mcfunction
        * Playsound warden.death
        * Setblock air, 
        * Kill markers[tag=ddd_portal,distance=..0.5]
      * portal.mcfunction
        * If @s[tag=ddd_unset] run entity/marker/unset
      	* If !isValidFrame run entity/marker/break
      * tick10.mcfunction
        * Foreach Marker[tag=ddd_portal, chunk=loaded] run entity/marker/tick/portal
        * Foreach Marker[tag=ddd_break, chunk=loaded] run entity/marker/break
        * Foreach Marker[tag=ddd_load] run entity/marker/load
    * tp/
      * break.mcfunction
        * Remove tag ddd_new
        * Tp @s to ddd.tpDest
      * portal.mcfunction
        * Remove tag ddd_new
        * Tp @s to ddd.tpDest
        * Kill @s[loadedChunk && !isValidFrame]
        * Set data.ddd = ddd.tpSrc
      * temp.mcfunction
        * Remove tag ddd_new
        * Tp @s to ddd.tpDest
        * For 4 adjacent blocks, run entity/marker/tp/load
        * Set ddd.valid = (!loadedChunk || isValidFrame)
        * Kill @s
      * load.mcfunction
	      * If unloaded chunk, forceload and summon Marker[tag=ddd_load]
    * break.mcfunction
      * Set ddd.tpDest = data.ddd
      * Summon x2 Marker[tag=ddd_break]
      * TP one to dest portal with entity/marker/tp/break
    * portal.mcfunction
      * Remove tag ddd_unset
      * Setblock end_gateway{ExitPortal:data.ddd}
      * Playsound warden.dig
    * load.mcfunction
      * Forceload remove chunk
      * Kill @s
  * player/
    * inventory/
      * check/
        * compass.mcfunction
          * Foreach inventory slot NUMBER, if paired heart run entity/player/inventory/check/<NUMBER>
        * <NUMBER>.mcfunction
          * Set ddd.tpDest = Inventory[NUMBER].tag.LodestonePos
          * Check frame with entity/marker/tp/temp, if invalid reset heart at Inventory[NUMBER]
      * new/<NUMBER>.mcfunction
        * Revoke acvancement used/heart/NUMBER
        * Stopsound lodestone_compass_lock
        * Set ddd.tpDest = Inventory[NUMBER].tag.LodestonePos
        * Item modify Inventory[NUMBER] to be Paired Heart
        * Check frame with entity/marker/tp/temp, if invalid reset heart at Inventory[NUMBER]
        * If ddd.valid, playsound warden.heartbeat
      * old/
        * dim/<NUMBER>.mcfunction
          * Revoke advancement used/paired_heart/NUMBER
          * Stopsound lodestone_compass_lock
          * If !in_overworld, reset Inventory[NUMBER].tag.LodestonePos
          * If in_overworld, run entity/player/inventory/old/eq/NUMBER
        * eq/<NUMBER>.mcfunction
          * Set ddd.tpSrc = Inventory[NUMBER].tag.ddd
          * Set ddd.tpDest = Inventory[NUMBER].tag.ddd
          * If ddd.tpSrc == ddd.tpDest, reset heart at Inventory[NUMBER]
          * If ddd.tpSrc != ddd.tpDest, run entity/player/inventory/old/old_frame/NUMBER
        * old_frame/<NUMBER>.mcfunction
          * Check frame is valid with entity/marker/tp/portal
          * If !isValidFrame, reset Inventory[NUMBER].tag.LodestonePos
          * If isValidFrame, run entity/player/inventory/old/new_frame/NUMBER
        * new_frame/<NUMBER>.mcfunction
          * Check frame is valid with entity/marker/tp/portal
          * If !isValidFrame, reset Inventory[NUMBER].tag.LodestonePos
          * If isValidFrame, replace Inventory[NUMBER] with air and grant advancement pair_doorway, run entity/player/inventory/check/compass
