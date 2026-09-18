# Contributing

Thank you for adding to this. A few rules, and the reasons for them.

## The one rule that matters

**Every entry must have a source you can name, and it must not be OLA's.**

Acceptable:

- a manufacturer's manual, spec sheet or published PID table — cite the
  document and its revision;
- a fixture's own `PARAMETER_DESCRIPTION` / `ENUM_LABEL` / `METADATA_JSON`
  responses — cite the model and firmware version (RDMBench's `export-pids`
  fills this in for you);
- your own traffic capture from the manufacturer's control software — say so.

Not acceptable, for any reason:

- OLA's `data/rdm/manufacturer_pids.proto`, the rdm.openlighting.org PID
  store, or anything derived from either;
- guesses, inference from a PID's number, or "this is probably the same as on
  the other model";
- your own interpretation of what a parameter *means*. Names, types and the
  fixture's own value labels are data. "Setting this to 3 fixes the colour
  problem" is knowledge, and belongs in documentation, not in a table.

The reasoning behind the OLA prohibition is in the
[README](README.md#sourcing-rule). Short version: that data carries no licence
grant anywhere, and is third-party contributed with no CLA, so nobody is in a
position to give it away. It is not a slight on the project.

By opening a pull request you are asserting that your entries came from one of
the acceptable sources above, and that you are able to release them under
[CC0](LICENSE).

## What a pull request should contain

- **One file per brand**, named for the brand in lowercase with hyphens, in
  `tables/` — `tables/martin-professional.json`. Directly in it, not in a
  subdirectory: the loader's directory scan is deliberately non-recursive and
  the manifest cannot name a path, so a nested file is silently ignored by
  every bench.
- **A filled-in `source`.** A PR without one will be asked for one.
- **No `tables/manifest.json` change.** It is generated; a maintainer
  regenerates it when merging. Editing it by hand will conflict.

If you have RDMBench installed, check your file before you push:

```
rdmbench-cli validate-tables tables
```

If you don't, open the pull request anyway — CI runs exactly that command and
will tell you. The checker is RDMBench's own loader, the same code that reads
these files on a bench, so there is no second opinion to disagree with it.

## Adding a brand from a fixture you have

If you have the fixture on a bench, let it describe itself rather than typing
from a manual — it is faster and it cannot be mistyped:

```
rdmbench-cli export-pids --share -o <brand>.json
```

`--share` withholds your unit's UID. The device label is never included. Check
the file over before opening a PR; you are publishing it.

## A note on what is missing

If a fixture lists a PID and refuses to describe it, and no manual documents
it, the honest answer is to leave it *unnamed* — but not to leave it out. A
guessed name is a false statement about the fixture, and the next person to
read it will believe it. The PID number is a true one.

Put those in `wanted`:

```json
{
  "manufacturer_id": "0x4D50",
  "manufacturer": "Martin Professional A/S",
  "source": "MAC Aura, firmware 1.8.0 — fixture lists these and NACKs PARAMETER_DESCRIPTION",
  "pids": [],
  "wanted": ["0x8001", "0x8002", "0x8007"]
}
```

That is a complete, mergeable contribution with not a single name in it. It
tells whoever next has that manual open exactly what to look up, which is
more than anyone knew before you sent it. RDMBench's `export-pids` fills
`wanted` in for you.
