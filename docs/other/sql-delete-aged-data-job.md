---
layout: page
title: SQL - Delete aged data
parent: Other
---

# Delete aged data

This article describes a sql job to delete data of a specific age in your table. This job can help if you have a huge table and want to reduce the amount of data to clean it up. Maybe you only have to keep the data for up to ten years of age and older data can be deleted? Then this is what you need:

```sql
-- @NumDelete for how many old rows get deleted in batch
DECLARE @NumDelete varchar(10)
SET @NumDelete = '1000000'

-- @StartOfThisYear get the first day of this year
DECLARE @StartOfThisYear DATETIME
SET @StartOfThisYear = DATEADD(yy, DATEDIFF(yy, 0, GETDATE()), 0)

-- @KeepTenYearsDate subtract 2 years from @StartOfThisYear to keep 2 year old records
DECLARE @KeepTenYearsDate DATETIME
SET @KeepTenYearsDate = DATEADD(YEAR, -2, @StartOfThisYear)

-- declaring the sql statement to be executed with escaping ' with '' 
--  and converting the datetime to varchar for concatenating the strings
Declare @sql varchar(500)
SET @sql = N'DELETE TOP (' + @NumDelete + ')' 
			+ 'FROM [monitoring].[monitor].[anlagenmonitor]' 
			+ 'WHERE Zeitstempel < ''' + CONVERT(VARCHAR, @KeepTenYearsDate, 120) + ''''

EXEC (@sql)
```

This script can be configured as MS SQL Job and to run every night.