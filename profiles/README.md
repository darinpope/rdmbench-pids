# Fixture profiles

What a *model* is, captured from one unit of it: identity, product category
and details, firmware, **every** DMX personality (name, footprint, and the
E1.37-5 stable ID where the fixture has one), every sensor's definition, and
the PIDs it lists. One JSON file per model, one directory per brand:

```
profiles/martin-professional/mac-aura.json
```

Harvested self-description, like the PID tables — the fixture's own
DEVICE_INFO, DMX_PERSONALITY_DESCRIPTION and SENSOR_DEFINITION answers — and
so free, public and CC0. What it is *not*: readings, the unit's current
mode, or per-personality slot tables (reading those means switching the
fixture's mode, so they are left out on purpose; the footprint is what a
profile records about a mode).

Nothing fetches this directory. Unlike `tables/`, which a bench downloads by
manifest and must stay flat, `profiles/` is organised for people and any
depth is fine. RDMBench does not consume profiles yet — the bench has the
fixture in front of it. This is a contribution to the ecosystem first, and
the feature it makes possible ("what does channel 9 do in the mode the desk
is patched for?", answerable without the fixture) comes after.

## Capturing one

```
rdmbench-cli export-profile --share -o mac-aura.json
```

or **Contribute Fixture…** in the RDMBench report. Either reads every
personality from the fixture (a snapshot deliberately reads only the current
one), prints where the file belongs under `profiles/`, and — in the app —
opens a prefilled pull request. `--share` withholds the unit's UID; the
device label is never included.

## Format

The contract is [`schema/fixture-profile.json`](../schema/fixture-profile.json),
generated from RDMBench's types. In short:

```json
{
  "manufacturer_id": "0x4D50",
  "manufacturer": "Martin Professional",
  "model_id": "0x0110",
  "model": "MAC Aura",
  "product_category": "Fixture (moving yoke)",
  "product_category_id": "0x0102",
  "software_version_id": "0x00010800",
  "software_version": "1.8.0",
  "protocol_version": "1.0",
  "sub_device_count": 0,
  "personalities": [
    { "number": 1, "name": "Basic (8-bit)", "footprint": 14 },
    { "number": 2, "name": "Standard (16-bit)", "footprint": 25, "personality_id": "0002.0000" }
  ],
  "sensors": [
    { "number": 0, "name": "Head temperature", "kind": "temperature", "unit": "°C",
      "range": [-10, 90], "normal": [0, 70],
      "records_lowest_highest": true, "records_value": false }
  ],
  "supported_parameters": ["0x0080", "0x0081", "…"],
  "source": "Captured by RDMBench 0.1.0 from the fixture's own DEVICE_INFO / DMX_PERSONALITY_DESCRIPTION / SENSOR_DEFINITION: MAC Aura, software 1.8.0, 2026-09-18"
}
```

- `source` is required, as for the tables — see the [sourcing rule](../README.md#sourcing-rule).
- Personalities and sensors change between firmware versions. A profile is
  true of the firmware in `source`; a newer capture of the same model
  replaces the file, with the version moving in `source`.
- One file per manufacturer + model ID. Two files for one model is a
  warning from the validator, since a reader cannot tell which to use.

CI runs `rdmbench-cli validate-profiles profiles` on every pull request:
each file parses as a profile, names a source, comes from a real ESTA
registration (no prototyping-block IDs), and no personality or sensor is
defined twice.
