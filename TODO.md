# TODO: Armor Plate Slots for Hardsuits and MODsuits

## Summary

Replace the current "tiny storage box" armor plate behavior on playable hardsuits and MODsuit chest parts with a dedicated whitelisted armor plate slot. The player should insert armor plates into an armor-plate-specific slot, not into generic item storage.

Default assumption: it is acceptable to change the shared armor-plate clothing base so non-spawning/test suits inherit the new behavior too, because they cannot appear in normal gameplay. If that is not acceptable, make an explicit active-spawn allowlist first.

## Implementation Changes

1. Inspect the current armor plate files:
   - `Content.Shared/_Mono/ArmorPlate/SharedArmorPlateSystem.cs`
   - `Content.Shared/_Mono/ArmorPlate/Components/ArmorPlateHolderComponent.cs`
   - `Resources/Prototypes/_Mono/Entities/Clothing/base_plating.yml`

2. Add a dedicated armor plate item slot to `ArmorPlateHolderComponent`.
   - Give it a stable container ID, for example `armor-plate-slot`.
   - Whitelist only entities with `ArmorPlateItemComponent`.
   - This prevents unrelated items from being inserted.

3. Register the slot in `SharedArmorPlateSystem`.
   - Inject/use `ItemSlotsSystem`.
   - On component startup, call `AddItemSlot(...)` for the armor plate slot.
   - This makes the slot exist on the suit entity.

4. Change plate detection to use the new slot container.
   - Replace checks against `StorageComponent.ContainerId`.
   - Use the holder's armor plate slot container ID instead.
   - When a plate is inserted into that slot, make it the active plate.
   - When removed, clear the active plate.
   - Since the slot only holds one plate, you no longer need to search a storage container for another plate.

5. Update `base_plating.yml`.
   - Remove the generic `Storage` component from `ClothingArmorPlate`.
   - Remove the `StorageBoundUserInterface` from armor-plate-only behavior if it is only there for the old storage box.
   - Keep required containers used by hardsuit/MODsuit systems, especially `toggleable-clothing`, if nearby parents rely on it.
   - Add the new armor plate slot holder setup through `ArmorPlateHolder`.

6. Update filled armor plate variants.
   - Existing `StorageFill` entries like `ClothingArmorPlateBlunt_Slash`, `Pierce`, `Heat`, and `Speed` probably need to become slot-based fills.
   - Use the repo's existing item-slot fill pattern if one exists.
   - If there is no existing slot-fill helper, leave those variants for later or add a small startup behavior that inserts the starting plate into the slot.

7. Confirm active gameplay coverage.
   - Check hardsuit base inheritance from `ClothingOuterHardsuitBase`.
   - Check MODsuit chestplate base inheritance from `ClothingModsuitChestplateBase`.
   - Confirm normal spawned/loadout/crate/uplink hardsuits and MODsuits still inherit armor plate support.
   - Do not manually add this to prototype-only, test-only, or impossible-to-spawn suits unless they already inherit the shared base.

## Test Plan

- Spawn an active hardsuit that supports armor plates.
- Verify it has an armor plate slot, not a normal generic storage box.
- Insert a valid armor plate.
- Confirm armor stats, speed modifiers, stamina modifiers, and damage absorption still apply.
- Remove the plate.
- Confirm the suit loses the plate bonuses.
- Try inserting a non-plate item.
- Confirm it is rejected.
- Test one MODsuit chest part the same way.
- Check prototype load/build after the YAML changes.

## Notes

- The important code change is switching armor plate handling from `StorageComponent.ContainerId` to a dedicated `ItemSlot`.
- The important YAML change is removing the generic storage behavior from armor plate clothing and replacing it with slot behavior.
- If you want strict "active gameplay only" behavior instead of changing the shared base, first build a list of actively spawned hardsuit/MODsuit prototypes, then apply the slot parent only to those prototypes.
