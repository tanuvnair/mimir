---
title: "PostgreSQL Restore"
date: 2026-07-24
tags:
  - database
  - postgresql
  - qubefini
  - work
publish: false
---

# PostgreSQL Restore Snippets

Commands for restoring Qubefini / MiniETL staging data locally.

## Reset DB to staging data (Qubefini)

```zsh
psql -h localhost -p 5432 -U postgres -d postgres -c "DROP DATABASE IF EXISTS minietl;"

psql -h localhost -p 5432 -U postgres -d postgres -c "CREATE DATABASE minietl;"

pg_restore -h localhost -p 5432 -U postgres -d minietl --no-owner --no-acl -v 03_04_2026_qubefini_data.dump
```

Credentials and dump paths for other databases live in [[quick-notes-credentials]].

## Related

- [[qubefini]]
- [[scheduler-sql-snippets]]
- [[quick-notes-credentials]]
