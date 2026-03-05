# Utility commands

Quick-reference for the four read-only utility commands. None of these write any changes to
Home Assistant.

---

## check

Verify that zigporter can reach Home Assistant and Zigbee2MQTT before running a migration or
rename.

```bash
zigporter check
```

Runs four checks in sequence:

| Check | What it tests |
|---|---|
| Configuration | All required env vars (`HA_URL`, `HA_TOKEN`, `Z2M_URL`) are set |
| HA reachable | HTTP GET to `HA_URL/api/` returns 2xx |
| ZHA active | ZHA integration is configured and has at least one device |
| Z2M running | Z2M HTTP API at `Z2M_URL` is responding |

Each check prints a status icon (`✓` / `✗` / `!` / `–`) and a short message. If any blocking
check fails you are prompted to proceed anyway or abort.

`zigporter migrate` calls `check` automatically at the start of the wizard.

---

## inspect

Show a ZHA device's full dependency graph — its entities, automations, scripts, scenes, and
Lovelace dashboard cards.

```bash
zigporter inspect
```

The command opens an interactive device picker (area-grouped, matching the migrate wizard
style). Select a device to see a report like:

```
╭─ Living Room Ceiling ────────────────────────────────────────╮
│  IEEE: 0x00158d0001234567   Area: Living Room   Model: E1525 │
╰──────────────────────────────────────────────────────────────╯

Entities (3)
  light.living_room_ceiling
  sensor.living_room_ceiling_temperature
  sensor.living_room_ceiling_linkquality

Automations (1)
  □  Turn on at sunset
       light.living_room_ceiling

Dashboards (2 cards)
  □  Default dashboard › Living Room › entities card
       light.living_room_ceiling
```

Use `inspect` before migrating a device to understand what will need updating, or after
migrating to confirm everything looks correct.

---

## export

Snapshot your entire ZHA device inventory to a JSON file.

```bash
zigporter export
```

By default the file is written to `./zha_export.json`. Specify a different path with
`--output`:

```bash
zigporter export --output ~/backups/zha_$(date +%Y%m%d).json
```

Pass `--pretty` to write indented JSON (easier to diff in version control):

```bash
zigporter export --pretty
```

The export includes, for each ZHA device:

- IEEE address, device name, manufacturer, model
- Assigned area
- All entities (entity ID, platform, state, attributes)
- Automation references (which automations reference this device's entities)

The `migrate` command reads this export file as its primary input. Run `export` once before
starting a migration so you have a stable snapshot to migrate from.

---

## list-z2m

List all devices currently paired with Zigbee2MQTT.

```bash
zigporter list-z2m
```

Prints a table with one row per device (coordinator excluded):

```
                  Zigbee2MQTT Devices (12)
┌────────────────────────┬──────────────────────┬───────┬─────────┬────────┬──────────────┐
│ Friendly name          │ IEEE address         │ Type  │ Vendor  │ Model  │ Power source │
├────────────────────────┼──────────────────────┼───────┼─────────┼────────┼──────────────┤
│ Kitchen Ceiling        │ 0x00158d0001234567   │ Router│ IKEA    │ E1743  │ Mains        │
│ ...                    │ ...                  │ ...   │ ...     │ ...    │ ...          │
└────────────────────────┴──────────────────────┴───────┴─────────┴────────┴──────────────┘
```

Devices not supported by Z2M are shown in dim text.

Use `list-z2m` to confirm that a device appeared in Z2M after pairing, or to find a device's
IEEE address or friendly name for use with other commands.
