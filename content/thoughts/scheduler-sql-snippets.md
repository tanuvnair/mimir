---
title: "Scheduler SQL Snippets"
date: 2026-07-24
tags:
  - qubefini
  - postgresql
  - scheduler
  - work
  - database
publish: false
---

# Scheduler SQL Snippets

Operational queries for Qubefini schedulers. Connection GUIDs and credentials live in [[quick-notes-credentials]].

Related: [[qubefini]], [[scheduler-optimization]], [[postgresql-restore]], [[planning-analytics]]

## Recent schedulers

```sql
SELECT id, name, export_time_old, export_time, start_time_old, start_time,
       end_time_old, end_time, created_at
FROM "schedulers"
WHERE created_at >= NOW() - INTERVAL '2 hours';
```

```sql
SELECT id, name, header_labels, is_active
FROM "schedulers";
```

## Toggle active flag

```sql
-- Prefer scoping with WHERE id = '...' or connection_id = '...'
UPDATE schedulers SET is_active = false WHERE id = '<scheduler-id>';

UPDATE schedulers
SET is_active = true, header_labels = null
WHERE id = '<scheduler-id>';
```

## Find a scheduler

```sql
SELECT id, name, cube_name, start_date, end_date, export_time, cron_expression
FROM schedulers
WHERE connection_id = '<connection-id>'; -- see [[quick-notes-credentials]]
```

## Update one scheduler by id

```sql
UPDATE schedulers
SET
  start_date = '2026-04-10 14:30:00',
  end_date = '2026-04-11 14:30:00',
  export_time = '2026-04-10 14:30:00',
  cron_expression = '30 14 * * *'
WHERE id = '<scheduler-id>';
```

## Update all schedulers for a connection

```sql
UPDATE schedulers
SET
  start_date = '2026-04-10 14:30:00',
  end_date = '2026-04-11 14:30:00',
  export_time = '2026-04-10 14:30:00',
  cron_expression = '30 14 * * *'
WHERE connection_id = '<connection-id>';
```

## Retry settings

```sql
UPDATE schedulers
SET
  maximum_retry_attempts = 2,
  retry_delay_minutes = 10
WHERE connection_id IN ('<connection-id-a>', '<connection-id-b>');
```

## Rows per file

```sql
SELECT id, name, cube_name, start_date, end_date, export_time, rows_per_file
FROM schedulers;

UPDATE schedulers
SET rows_per_file = 1000000;
```

## Retry columns

```sql
SELECT id, name, cube_name, start_date, end_date, export_time,
       maximum_retry_attempts, retry_delay_minutes
FROM schedulers;
```

## Local job-run query (after 18:30)

```sql
SELECT
    jr.id,
    jr.scheduler_id,
    jr.created_at,
    s.name
FROM job_runs jr
LEFT JOIN schedulers s
    ON jr.scheduler_id = s.id
WHERE jr.created_at::time >= TIME '18:30:00';
```

## Related

- [[qubefini]]
- [[scheduler-optimization]]
- [[postgresql-restore]]
- [[planning-analytics]]
