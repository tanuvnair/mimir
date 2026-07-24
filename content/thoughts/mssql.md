---
title: "MSSQL"
date: 2026-07-24
tags:
  - database
  - mssql
publish: false
---

# MSSQL

## Get current running queries

```SQL
SELECT   
    der.session_id AS [SPID],  
    des.login_name AS [Login_Name],  
    des.host_name AS [Client_Machine], 
    des.program_name AS [Application_Name], 
    DB_NAME(der.database_id) AS [Database_Name],  
    der.status AS [Status],  
    der.command AS [Command_Type],  

    dest.text AS [Full_Batch_Text], 
              
    der.start_time AS [Execution_Start_Time],  
    der.total_elapsed_time / 1000.0 AS [Duration_Seconds],
    der.percent_complete AS [Percent_Complete], 
    
    -- Resource & Memory Usage
    der.cpu_time AS [CPU_Time_Ms],  
    der.logical_reads AS [Logical_Reads],  
    der.reads AS [Physical_Reads], 
    der.writes AS [Logical_Writes], 
    (der.granted_query_memory * 8) / 1024 AS [Granted_Memory_MB], 
    
    -- Wait & Blocking Info
    der.wait_type AS [Current_Wait_Reason],  
    der.wait_time AS [Wait_Time_Ms], 
    der.wait_resource AS [Wait_Resource_Details], 
    der.blocking_session_id AS [Blocked_By_SPID],  
    der.open_transaction_count AS [Open_Transactions]
    
FROM sys.dm_exec_requests AS der  
JOIN sys.dm_exec_sessions AS des ON der.session_id = des.session_id  
CROSS APPLY sys.dm_exec_sql_text(der.sql_handle) AS dest 
OUTER APPLY sys.dm_exec_query_plan(der.plan_handle) AS qp 
WHERE des.is_user_process = 1 -- Filters out background system processes  
  AND der.session_id <> @@SPID; -- Excludes this query itself from the results
```

## Get queries and their performance stats for a given date range

```SQL
SELECT 
    -- Context & Identification
    DB_NAME(dest.dbid) AS [Database_Name],
    OBJECT_NAME(dest.objectid, dest.dbid) AS [Object_Name], -- e.g., Stored Procedure name
    
    -- Exact Statement Text (Crucial for multi-statement batches/procedures)
    SUBSTRING(dest.text, (deqs.statement_start_offset/2)+1, 
        (((CASE deqs.statement_end_offset 
            WHEN -1 THEN DATALENGTH(dest.text) 
            ELSE deqs.statement_end_offset END) - deqs.statement_start_offset)/2) + 1) AS [Exact_Query_Text],
            
    dest.text AS [Full_Batch_Text], 
    
    -- Execution Timings
    deqs.creation_time AS [Plan_Compiled_Time],
    deqs.last_execution_time AS [Last_Execution_Time],
    deqs.execution_count AS [Total_Executions],
    
    -- Duration (Wall-clock time)
    (deqs.total_elapsed_time / deqs.execution_count) / 1000.0 AS [Avg_Duration_Ms],
    deqs.max_elapsed_time / 1000.0 AS [Max_Duration_Ms],
    
    -- CPU Time (Actual processing time)
    (deqs.total_worker_time / deqs.execution_count) / 1000.0 AS [Avg_CPU_Time_Ms],
    deqs.max_worker_time / 1000.0 AS [Max_CPU_Time_Ms],
    
    -- I/O Activity (Disk/Memory Reads & Writes)
    (deqs.total_logical_reads / deqs.execution_count) AS [Avg_Logical_Reads_Pages],
    deqs.max_logical_reads AS [Max_Logical_Reads_Pages],
    (deqs.total_logical_writes / deqs.execution_count) AS [Avg_Logical_Writes_Pages],
    
    -- Rows Impacted
    (deqs.total_rows / deqs.execution_count) AS [Avg_Rows_Returned]

FROM sys.dm_exec_query_stats AS deqs
CROSS APPLY sys.dm_exec_sql_text(deqs.sql_handle) AS dest
OUTER APPLY sys.dm_exec_query_plan(deqs.plan_handle) AS qp
WHERE deqs.last_execution_time BETWEEN '2026-06-04 07:00:00' AND '2026-06-04 09:00:00'

ORDER BY deqs.last_execution_time ASC;
```

## Related

- [[intergold-dummy-mssql-db]]
- [[intergold]]
