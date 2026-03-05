# Fix a device after migration

Clean up stale ZHA device entries that remain in the Home Assistant registry after a device
has been migrated to Zigbee2MQTT.

```bash
zigporter fix-device
```

## When to use this

After a ZHA → Z2M migration, HA sometimes retains the old ZHA device entry alongside the new
MQTT-based Z2M entry for the same physical device (identified by IEEE address). This causes two
problems:

1. **Ghost entities** — stale ZHA entities (e.g. `sensor.device_temperature`) occupy the original
   entity IDs, so the new Z2M entities are registered with a numeric suffix instead
   (`sensor.device_temperature_2`).
2. **Duplicate device entries** — the device appears twice in the HA device registry.

`fix-device` detects and resolves both problems automatically.

!!! tip "Migration wizard does this for you"
    Step 5 of `zigporter migrate` runs this cleanup as part of the wizard. Use `fix-device`
    separately only for devices that were migrated before this step was added, or if you
    migrated through another path (e.g. manual pairing in the Z2M frontend).

## What it does

`fix-device` scans the HA device and entity registries, finds every device that has **both**
a stale ZHA entry and an active Z2M (MQTT) entry with the same IEEE address, and for each
device:

1. **Deletes stale ZHA entities** — removes every entity still attached to the old ZHA device
   entry.
2. **Removes the stale ZHA device entry** — tries the standard device registry API first; falls
   back to `zha.remove` for older HA versions. If neither works (e.g. ZHA was already removed),
   the entry is left for HA to prune automatically on the next restart.
3. **Renames suffixed Z2M entities** — any Z2M entity whose ID ends with `_2`, `_3`, etc.
   (because the original ID was occupied by the stale ZHA entity) is renamed back to its
   unsuffixed form.

## Usage

```bash
zigporter fix-device
```

The command is interactive. It:

1. Fetches the device and entity registries from HA.
2. Detects devices with stale ZHA + active Z2M entries.
3. If only one device needs fixing, it proceeds directly. If multiple devices are found, a
   picker lets you choose which one to fix.
4. Shows a plan table listing all entities to delete and all entity IDs to rename.
5. Asks for confirmation before writing any changes.

Example output:

```
Fetching registry data from Home Assistant... ✓

  Device: Living Room Ceiling  (IEEE 0x00158d0001234567)

  Action   Entity ID
  delete   sensor.living_room_ceiling_temperature
  delete   light.living_room_ceiling
  rename   light.living_room_ceiling_2 → light.living_room_ceiling

? Apply fix? (Y/n)

  ✓ Deleted stale entity  sensor.living_room_ceiling_temperature
  ✓ Deleted stale entity  light.living_room_ceiling
  ✓ Removed stale ZHA device entry  abc123
  ✓ light.living_room_ceiling_2 → light.living_room_ceiling

✓ Done.  Reload the HA page to confirm the device is clean.
```

## Notes

- Stale entity deletion **cannot be undone** from the CLI.
- If HA re-registers the ZHA device entry immediately after removal (because ZHA is still
  active for other devices), remove the device from within the ZHA integration settings instead.
- Reload the HA frontend after running this command to confirm the device registry is clean.
