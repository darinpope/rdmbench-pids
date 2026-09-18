# RDM manufacturer PID tables

Names for the manufacturer-specific RDM parameters (PIDs `0x8000`–`0xFFDF`)
that fixtures list in `SUPPORTED_PARAMETERS` but do not describe through
`PARAMETER_DESCRIPTION`.

One JSON file per brand, in [`tables/`](tables). This is the library that
[RDMBench](https://github.com/darinpope/rdmbench) fetches, but nothing here is
specific to it — the format is plain JSON with a published schema, and any RDM
tool is welcome to read it.

`tables/` is flat and contains only the tables and their manifest: a bench
fetches that directory raw, its scan of it is non-recursive, and the manifest
can only name files sitting beside itself. Everything else in this repo is for
people, not benches.

**A fixture's own answers always win over anything in here.** Manufacturers
reuse PID numbers across product generations, and only the fixture in front of
you knows which generation it is. These tables are the fallback for a fixture
that lists a PID and won't describe it.

## Format

```json
{
  "manufacturer_id": "0x4D50",
  "manufacturer": "Martin Professional A/S",
  "source": "MAC Aura PXL user manual rev. C, §RDM, firmware 1.2",
  "pids": [
    { "pid": "0x8001", "name": "COLOUR_CALIBRATION", "data_type": "u8", "settable": true }
  ]
}
```

- `manufacturer_id` — the ESTA-assigned ID, the top 16 bits of the brand's
  UIDs. The registry is at
  <https://tsp.esta.org/tsp/working_groups/CP/mfctrIDs.php>. `7FF0h`–`7FFFh`
  is the prototyping/experimental block and does not belong in this repo.
- `source` — **required here.** Where the entries came from, specifically
  enough that someone else could check them: a manual with its revision, a
  fixture's own responses with the model and firmware, or your own capture.
  See [Sourcing rule](#sourcing-rule).
- `data_type` — free text, but use the E1.20 Table A-15 names as RDMBench
  spells them: `u8`, `i8`, `u16`, `i16`, `u32`, `i32`, `u64`, `i64`, `string`,
  `bool`, `bit field`, `group`, `uid`, `url`, `enum`.
- `labels` (optional) — `[{ "value": 0, "label": "Auto" }, …]`, names for an
  enumerated PID's values. **Only ever what a fixture said through
  `ENUM_LABEL`** (E1.20-2025 §10.4.3). Hand-written interpretation is not
  data and does not belong here.
- `metadata_version` / `metadata_json` (optional) — a fixture's E1.37-5 §5
  Parameter Metadata Language description of the PID, verbatim.
- `wanted` (optional) — `["0x8F02", …]`, PIDs this brand's fixtures **list
  and refuse to describe**, that no table names either. The library's own
  to-do list, per brand. See [What `wanted` is for](#what-wanted-is-for).
- `pid` and `manufacturer_id` accept `"0x8F02"`, `"8F02"` or a number.
- A PID outside `0x8000`–`0xFFDF` is rejected: these tables never override the
  standard ones, which E1.20 and E1.37-x define.
- Files load in name order; a later file's entry for the same manufacturer and
  PID replaces an earlier one. Two files claiming the same pair is legal but
  almost always a mistake, and the validator warns about it.
- Tables go directly in `tables/`, not in a subdirectory of it. A nested file
  is not organised, it is invisible.

The machine-readable contract is [`schema/manufacturer-pid-file.json`](schema/manufacturer-pid-file.json),
generated from RDMBench's own types — for anyone writing a tool against these
files. Do not hand-edit it.

What actually gates a merge here is stricter than the schema: CI runs
RDMBench's own loader over every table, so the check and the bench agree by
construction, and adds the rules a published library needs — a stated source,
no development fixtures, nothing hidden in a subdirectory.

## What `wanted` is for

A fixture that describes its private PIDs is the easy case. The common one
is a fixture that lists ten PIDs in `SUPPORTED_PARAMETERS` and refuses to
describe any of them — those rows show as raw hex in a report, and only a
manual will ever name them.

That refusal is information, and it used to be thrown away. `wanted`
records it: which PIDs a brand's fixtures have, that nobody has named yet.
It is the difference between "we have nothing for this brand" and "we know
exactly which eleven parameters to look up when someone finds the manual."

A bench ignores `wanted` entirely — it names nothing, so no report
changes. It is here for whoever is doing the curating, and it rides on the
table file so that a capture is one file and proposing it is one click.

A file may hold nothing but `wanted`. That is a perfectly good
contribution: the fixture told you which questions to ask.

## Sourcing rule

Entries must come from one of:

1. A manufacturer's manual, spec sheet or published PID table.
2. A fixture's own `PARAMETER_DESCRIPTION` / `ENUM_LABEL` / `METADATA_JSON`
   responses — see [Capturing from a fixture](#capturing-from-a-fixture).
3. Your own traffic capture from a manufacturer's control software.

**Never** the Open Lighting Architecture's `manufacturer_pids.proto` or the
rdm.openlighting.org PID store.

That data carries no licence grant anywhere — not in the file, not in OLA's
`LICENCE` (which allocates licences only to source components and never
mentions `data/rdm`), not in the `rdm-app` repository that maintains it, and
not on the site itself. More to the point, it is contributed by third parties
with no contributor licence agreement, so the Open Lighting Project is very
likely not in a position to grant rights to it even if asked. This is not a
criticism of a project that has served the industry well for years; it is
simply a gap that cannot be closed from the outside, so this repo stays clear
of it entirely and builds its own record with stated provenance.

That is why `source` is required, and why a pull request without a checkable
one will not be merged.

## Capturing from a fixture

A self-describing fixture on a bench is the best source for a brand nobody has
tabled yet. With RDMBench:

```
rdmbench-cli export-pids --share -o martin.json
rdmbench-cli export-pids --from snapshot.json
```

This writes a file in exactly this format from the fixture's own answers,
holding only the PIDs the already-loaded tables lack or have differently — a
proposed patch, not a dump — and reports on stderr which PIDs the fixture
lists but will not describe. For those, a manual is the only source.

`--share` withholds the unit's UID, for a file that leaves your shop. The
device label is never included. Nothing is uploaded anywhere: the file is
yours, and opening a pull request with it is a deliberate act.

The RDMBench app offers the same export from a report, and can open a
prefilled issue here with the file attached.

## How a bench gets these tables

`tables/manifest.json` carries the library version and a sha256 per file. A
bench does one GET of it, compares, and downloads only the files whose hash it
does not already have. The version is `YYYYMMDD.N` — the UTC date of the last
change and a same-day counter — **compared as that pair**, so `20260916.10` is
newer than `20260916.2`.

There is no server and no release process: a merged pull request is a release.

The manifest is generated, never hand-edited. Maintainers regenerate it with
RDMBench's CLI (`task manifest`); CI checks that it matches the files and that
the version moved when a table changed.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## Licence

[CC0 1.0 Universal](LICENSE) — public domain dedication. These are facts about
other people's products; no one should need permission to use them, and any
tool should be able to embed them without licence friction.
