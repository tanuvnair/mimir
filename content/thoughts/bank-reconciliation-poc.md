---
title: "Bank Reconciliation POC"
date: 2026-07-24
tags:
  - work
  - poc
  - brs
  - meeting
publish: false
---

# Bank Reconciliation POC

Notes from the June 29, 2026 meeting.

## Scope

- Company book / banking ledger with received and expenditure entries
- Match amount and the date it is reflected in the bank
- Different bank accounts → different statements; each bank account has a corresponding company ledger

## Open questions

- Full automation vs predefined formats per bank?

## Rules / approach

- Base rule: debit inwards, credit outwards
- Check typos / mismatches between company books and bank statement
- Treat the bank statement as the source of truth
- Fields to care about: date, description, amount, cheque number, NEFT description, UPI

## Research

- [ ] Research BRS
- [ ] Research Tally BRS

## Related

- [[dev-discussions]]
