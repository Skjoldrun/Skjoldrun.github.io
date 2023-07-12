---
layout: page
title: SQL - Delete data, keep certain num
parent: Other
---

# Delete data, keep certain num

This article describes a sql job to delete data that is out of a specified range of records to keep.

```sql
-- @NumDelete for how many old rows get deleted in batch
-- @NumKeep for how many newer rows are retained

DECLARE @NumDelete varchar(10)
SET @NumDelete = '10000'

DECLARE @NumKeep varchar(10)
SET @NumKeep = '1000000'

Declare @sql varchar(500)
SET @sql = N'DELETE TOP (' + @NumDelete + ') ' 
			+ 'FROM [ProductionLoggerDev].[dbo].[LogEvents] '  -- Change DB and Table 
			+ 'WHERE ID < (SELECT TOP (1) [Id] ' 
			+ '  FROM [ProductionLoggerDev].[dbo].[LogEvents]' -- Change DB and Table 
			+ '  ORDER BY [ID] DESC) - ' + @NumKeep

EXEC (@sql)

```

This script can be configured as a MS SQL job and run every night. Depending on how many new records are inserted into the DB, you may need to adjust the number variables. If this job would delete more existing records than new records added, the DB size can be frozen. This can be useful for logging DBs where you don't need to look too far back.