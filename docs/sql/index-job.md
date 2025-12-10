# Index Rebuild Job

``` sql title="Job script"
USE msdb;
GO

-- Clean up if the job already exists
IF EXISTS (SELECT 1 FROM msdb.dbo.sysjobs WHERE name = N'IndexMaintenanceJob')
BEGIN
    EXEC msdb.dbo.sp_delete_job @job_name = N'IndexMaintenanceJob', @delete_unused_schedule = 1;
END
GO

-- Create the job (initially disabled)
EXEC msdb.dbo.sp_add_job
    @job_name = N'IndexMaintenanceJob',
    @enabled = 0,  -- Job is created as DISABLED
    @description = N'Maintenance job to rebuild or reorganize indexes in all user databases',
    @start_step_id = 1;
GO

-- Define the full job step command
DECLARE @JobCommand NVARCHAR(MAX);
SET @JobCommand =N'SET NOCOUNT ON;

DECLARE @dbName SYSNAME;
DECLARE @sql NVARCHAR(MAX);

DECLARE db_cursor CURSOR FOR
    SELECT name FROM sys.databases
    WHERE database_id > 4 AND state_desc = ''ONLINE''; -- Only user databases

OPEN db_cursor;
FETCH NEXT FROM db_cursor INTO @dbName;

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT ''Processing database: '' + @dbName;

    SET @sql = ''
    USE ['' + @dbName + ''];

    DECLARE @objectId INT, 
            @indexId INT, 
            @schemaName SYSNAME, 
            @tableName SYSNAME, 
            @indexName SYSNAME, 
            @frag FLOAT, 
            @sqlCmd NVARCHAR(MAX);

    DECLARE index_cursor CURSOR FOR
        SELECT 
            ips.[object_id], 
            ips.index_id,
            ips.avg_fragmentation_in_percent,
            i.name AS index_name,
            s.name AS schema_name,
            t.name AS table_name
        FROM sys.dm_db_index_physical_stats (DB_ID(), NULL, NULL, NULL, ''''LIMITED'''') ips
        INNER JOIN sys.indexes i ON ips.[object_id] = i.[object_id] AND ips.index_id = i.index_id
        INNER JOIN sys.tables t ON ips.[object_id] = t.[object_id]
        INNER JOIN sys.schemas s ON t.schema_id = s.schema_id
        WHERE ips.index_id > 0 AND ips.alloc_unit_type_desc = ''''IN_ROW_DATA'''';

    OPEN index_cursor;
    FETCH NEXT FROM index_cursor INTO @objectId, @indexId, @frag, @indexName, @schemaName, @tableName;

    WHILE @@FETCH_STATUS = 0
    BEGIN
        SET @sqlCmd = NULL;

        IF @frag >= 30.0
            SET @sqlCmd = N''''ALTER INDEX ['''' + @indexName + N''''] ON ['''' + @schemaName + N''''].['''' + @tableName + N''''] REBUILD WITH (ONLINE = ON);'''';
        ELSE IF @frag >= 5.0
            SET @sqlCmd = N''''ALTER INDEX ['''' + @indexName + N''''] ON ['''' + @schemaName + N''''].['''' + @tableName + N''''] REORGANIZE;'''';

        IF @sqlCmd IS NOT NULL
        BEGIN
            PRINT @sqlCmd;
            EXEC sp_executesql @sqlCmd;
        END

        FETCH NEXT FROM index_cursor INTO @objectId, @indexId, @frag, @indexName, @schemaName, @tableName;
    END

    CLOSE index_cursor;
    DEALLOCATE index_cursor;
    '';

    EXEC (@sql); -- Switch to database and execute logic

    FETCH NEXT FROM db_cursor INTO @dbName;
END

CLOSE db_cursor;
DEALLOCATE db_cursor;
'

-- Add the job step
EXEC msdb.dbo.sp_add_jobstep
    @job_name = N'IndexMaintenanceJob',
    @step_name = N'Check and Optimize Indexes',
    @subsystem = N'TSQL',
    @command = @JobCommand,
    @database_name = N'master',
    @on_success_action = 1, -- Quit with success
    @on_fail_action = 2;    -- Quit with failure
GO

-- Create a daily schedule at 02:00 AM
EXEC msdb.dbo.sp_add_schedule
    @schedule_name = N'Daily_2AM',
    @freq_type = 4,              -- daily {4: daily, 8: weekly, 16: Monthly}
    @freq_interval = 1,          -- every day {every day, Sunday, Day 1 of the month}
    @freq_recurrence_factor = 2, -- every 2 months
    @active_start_time = 020000; -- 2:00 AM
GO

-- Attach schedule to the job
EXEC msdb.dbo.sp_attach_schedule
    @job_name = N'IndexMaintenanceJob',
    @schedule_name = N'Daily_2AM';
GO

-- Attach the job to the current server
EXEC msdb.dbo.sp_add_jobserver
    @job_name = N'IndexMaintenanceJob';
GO
```

``` sql title="Enable Job"
EXEC msdb.dbo.sp_update_job @job_name = 'IndexMaintenanceJob', @enabled = 1;
```

*** 
| Day         | Value |
| ----------- | ----- |
| Sunday      | 1     |
| Tuesday     | 2     |
| Wednesday   | 4     |
| Thursday    | 8     |
| Friday      | 16    |
| Saturday    | 32    |
| Monday      | 64    |
These values can also be combined using addition (e.g. Monday + Thursday = 64 + 8 = 72).
*** 
