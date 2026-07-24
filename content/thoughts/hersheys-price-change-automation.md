---
title: "Hersheys Price Change Automation"
date: 2026-07-24
tags:
  - hersheys
  - work
  - automation
publish: false
---

# Hersheys Price Change Automation

## Updated requirement

- Change `CURRENT_YEAR` with `PREVIOUS_YEAR` and `NEXT_YEAR` with `CURRENT_YEAR`.
- The output will still have the previous-year / current-year output logic.
- Update current behavior from "Filters input rows by calendar year: only rows whose ValidFrom falls in the configured current year (CY) or next year (NY) are used" to "Consider all rows", but keep the rollover logic.

## Date rules (MRP / ValidFrom)

- MRP and ValidFrom
- Previous Year and Current Year
- ValidFrom till now
	- If before the 5th, considered in the same month
	- If on/after the 5th, considered from the next month

## Sample volume check

- HCO0001, Cases, Makson Total FY: `56078`
- Net Weight: `103.06`
- FY volumes in KGS: `5779398.7`

## Related

- [[hersheys]]
- [[hersheys-contract-management]]
- [[intergold-pp-optimization-cleanup]]
