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

A bench fetches this directory with the tables. `tables/manifest.json` has
a `profiles` map, the sha256 of every file here keyed by its path under
`profiles/`. It is the same manifest with one version, so a changed profile
is a release just as a changed table is. Unlike `tables/`, which must stay
flat, `profiles/` is organised for people and any depth is fine. A bench
reads a profile only for its `quirks` and its personalities' `load` plans —
for everything else the bench has the fixture in front of it. The feature the rest makes possible ("what does channel 9 do in the
mode the desk is patched for?", answerable without the fixture) comes after.

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
- `quirks` is optional and never written by a capture: it records what a
  model is known to get **wrong**, which the wire cannot show. Each entry
  is a `kind` (today only `identify-inverted`) and a required `source`
  giving the bench, the method and the date — an observation someone can
  repeat, not an opinion. See [`adj/5px-12px.json`](adj/5px-12px.json).
- A personality's `load` is optional and never written by a capture: the
  channels a load test must hold at a level of their own because full on
  them is not output — a colour macro, a program, a dimmer mode. The ADJ
  5PX in 12-channel mode with every slot at 255 lights **UV only**, since
  its channel 9 at 255 is a macro that overrides the emitters. Each slot
  is a `channel` (1 for the personality's first), a `value`, and an
  optional `note` saying what the channel is; the plan's `source` is
  required and should name the manual page and the bench run that
  confirmed it. RDMBench's `load` applies it without being asked, so an
  assistant driving the fixture gets it right with no manual.
- One file per manufacturer + model ID. Two files for one model is a
  warning from the validator, since a reader cannot tell which to use.

CI runs `rdmbench-cli validate-profiles profiles` on every pull request:
each file parses as a profile, names a source, comes from a real ESTA
registration (no prototyping-block IDs), and no personality or sensor is
defined twice, and every `load` plan is sourced, not empty, and names only
channels inside its personality's footprint, each once.
