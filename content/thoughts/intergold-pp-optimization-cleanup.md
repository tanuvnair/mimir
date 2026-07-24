---
title: "Intergold PP Optimization Cleanup"
date: 2026-07-24
tags:
  - intergold
  - work
  - automation
publish: false
---

# Intergold PP Optimization Cleanup

## Plan

Configure these column indexes and their labels via environment variables: `PROD_END_DT_COL_INDEX`, `FIRST_BLOC_PROCESS_COL_INDEX`, `LAST_BLOC_PROCESS_COL_INDEX`, `BLOC_COL_INDEX`.

- `FIRST_BLOC_PROCESS_COL_INDEX` is column "ZCAD"
- `LAST_BLOC_PROCESS_COL_INDEX` is "Polish"

### Per-row flow

1. Store the value for the row in column `BLOC_COL_INDEX` and its corresponding column using the map (map stores which column name the value is associated with).
2. For each row, two cases:

#### Case A — `PROD_END_DT_COL_INDEX` < `CurrentDate`

- If value in `BLOC_COL_INDEX` is `"RPOL"`:
	- Set its corresponding column to `CurrentDate`
	- Add remark that it was updated
- Else:
	- Append remark `"extension is needed"`

#### Case B — otherwise

1. Go to the `BlocProcessCol` using the map.
2. Iterate from that column to `LAST_BLOC_PROCESS_COL_INDEX`.
3. For each process:
	- If `processDate` < `CurrentDate`:
		- First iteration: set `processDate = currentDate`, continue
		- Later iterations: if previous and current lack at least a one-day gap, set `processDate = previousProcessDate + 1`
	- Else:
		- If `processDate` < `previousProcessDate`: set `processDate = previousProcessDate + 1`
		- Else: stop

### Special rules

- All processes are sequential
- BlocProcessDate is only valid if between `currentYear - X`, `currentYear`, `currentYear + X` (X env-configurable)
- When setting a BlocProcessDate, it must not fall on a Sunday or a holiday from the holiday list
- When two or more processes share the same date, they keep the same updated date (e.g. Process3 and Process4 both set to CurrentDate)

## Needs-extension buffer logic

For rows with remark `"needs extension"`:

1. Add a 4 working-day buffer to `ProdEndDt`.
2. Check if the newly buffered `ProdEndDt` is today or a future date.
	- If yes: iterate usual fitting (set current running process to current date, then `prev + 1` for next processes) and check whether processes fit.
		- If it fits: add remark `"Updated: XYZ; 4 day buffer used"`.
		- If it does not fit: try fitting multiple processes on a single day by batching processes together (batches of 2).
	- Else: add remark `"added 4 day buffer; needs optimization"`.

## Validated rows

- SMC0001
- NTR0014
- JRP0004
- KIS0010
- HBR0003

## Related

- [[intergold]]
- [[intergold-dummy-mssql-db]]
- [[mssql]]
