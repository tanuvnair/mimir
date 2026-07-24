---
title: "Scheduler Optimization"
date: 2026-07-24
tags:
  - qubefini
  - scheduler
  - tm1
  - work
  - performance
publish: false
---

# Scheduler Optimization

Notes on cube schema versioning and avoiding unnecessary TM1 attribute fetches.

## Cube schema version optimization

There is no dependency on dimension attributes for the cube schema version logic. In `services/scheduler.go`, the flow is:

1. Call `ds.GetDatasetInfo(ctx, *scheduler.CubeName)` (line 734).
2. Use only `info.LastDataUpdate` and `info.LastSchemaUpdate` (lines 740–750).
3. Compare those to the latest `CubeSchemaVersion` and optionally insert a new row.

`CubeSchemaVersion` and the repo only store: `connection_id`, `cube_name`, `version`, `last_data_update`, `last_schema_update`, and related IDs/timestamps. Dimensions and attributes are not stored or used for versioning.

So for schema version logic, attributes (and even dimension names) are not required; only the cube’s last data/schema update times are.

## Who uses what from `GetDatasetInfo`

| Caller | Uses |
| --- | --- |
| Scheduler (schema version) | `LastDataUpdate`, `LastSchemaUpdate` only |
| `GetTM1CubeDimensionsHandler` | `Dimensions`, `CellCount` only |
| Dataset info API (returns full `datasetInfo`) | Full struct, including `DimensionsWithAttributes` |

The only place that currently needs `DimensionsWithAttributes` from `GetDatasetInfo` is the single API that returns the full dataset info. The schema version path and the “get dimensions” path do not need attributes.

## Recommendation

- For cube schema change version logic: avoid fetching attributes entirely. That path only needs cube metadata (e.g. from `Cubes('x')`) to get `LastDataUpdate` and `LastSchemaUpdate`; the existing dimension loop and all `GetDimensionAttributes` calls are unnecessary for this.
- To avoid calling attributes for every dimension at runtime:
	- **Option A** — Add a lighter method (e.g. `GetDatasetMetadata`) that returns only what the scheduler needs (last update times, and optionally dimension names from `$expand=Dimensions($select=Name)`), and use that in the scheduler instead of `GetDatasetInfo`. Leave `GetDatasetInfo` as-is for handlers that want full info.
	- **Option B** — Add an option to `GetDatasetInfo` (e.g. `includeAttributes bool`) and have the scheduler call it with `includeAttributes=false` so the TM1 implementation skips per-dimension `GetDimensionAttributes` calls when only schema version (and dimensions/cell count) is needed.

Either way, schema version logic does not depend on fetching attributes; it only depends on the cube’s last data/schema update times.

## Related

- [[qubefini]]
- [[scheduler-sql-snippets]]
- [[planning-analytics]]
