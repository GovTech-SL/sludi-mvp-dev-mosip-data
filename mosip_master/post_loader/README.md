# Registration client UI spec restore (new environments only)

## Why

The masterdata loader reads `mosip_master/xlsx/ui_spec.xlsx`. An Excel cell holds at most 32,767
characters, and three registration client specs are longer: `newProcess`, `updateProcess` and
`lostProcess` (about 38 KB each). The loader writes them to `master.ui_spec` cut short, as invalid
JSON, so New, Update and Lost Registration do not work on the registration clients.
`restore_regclient_ui_specs.sql` writes the full specs back.

The specs in the script are the ones dev and QA serve (checked against both on 2026-11-03),
including the reviewed English, Sinhala and Tamil labels. Dev and QA are already correct.
**Run this only when you build a new environment.**

## When

Once, right after the masterdata loader (`mosip/masterdata-loader/install.sh`) has finished on the
new environment, and before registration clients are used there.

## How

The database is `mosip_master`. Run the whole file:

```
psql -X -v ON_ERROR_STOP=1 -h <db host> -U <user> -d mosip_master -f restore_regclient_ui_specs.sql
```

With DBeaver instead: open the file on the `mosip_master` connection and use Execute SQL Script
(Alt+X). If Auto-Commit is off, press Commit afterwards.

The script:

- changes only the three rows in `master.ui_spec`; it creates and deletes nothing;
- finds the rows itself, by spec type, so no ids need to be filled in;
- is a single update: all three specs are written, or on any error none are, and psql exits
  non-zero;
- can be run again safely.

## Check

The last query of the script returns one row per spec type. Expect **3 rows, all with
`up_to_date = true`**.

## Afterwards

Restart masterdata and syncdata, then sync the registration clients:

```
kubectl -n kernel rollout restart deployment masterdata syncdata
```

## If it fails

Nothing is written. Send the error message to the registration client team. The usual causes are
that the loader has not run yet (no `ui_spec` rows) or that there is no published identity schema.

## Keeping it current

When the registration client specs change on dev and QA, the registration client team regenerates
this script from the live specs and replaces it here.
