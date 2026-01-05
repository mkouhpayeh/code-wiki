# Index Rebuild

---

- Index = NULL → It's a heap (no clustered index).
``` sql title="Rebuild the heap"
CREATE CLUSTERED INDEX TempIndex ON dbo.YourTable(YourColumn);
DROP INDEX TempIndex ON dbo.YourTable;
```

- REBUILD WITH (ONLINE = ON)=> Only applies in SQL Server Enterprise edition


``` sql title="Rebuild Indexes of All Tables"
SET NOCOUNT ON;

DECLARE @dbName SYSNAME;
DECLARE @sql NVARCHAR(MAX);

DECLARE db_cursor CURSOR FOR
    SELECT name FROM sys.databases
    WHERE database_id > 4 AND state_desc = 'ONLINE'; -- Only user databases

OPEN db_cursor;
FETCH NEXT FROM db_cursor INTO @dbName;

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT 'Processing database: ' + @dbName;

    SET @sql = '
    USE [' + @dbName + '];

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
        FROM sys.dm_db_index_physical_stats (DB_ID(), NULL, NULL, NULL, ''LIMITED'') ips
        INNER JOIN sys.indexes i ON ips.[object_id] = i.[object_id] AND ips.index_id = i.index_id
        INNER JOIN sys.tables t ON ips.[object_id] = t.[object_id]
        INNER JOIN sys.schemas s ON t.schema_id = s.schema_id
        WHERE ips.index_id > 0 AND ips.alloc_unit_type_desc = ''IN_ROW_DATA'';

    OPEN index_cursor;
    FETCH NEXT FROM index_cursor INTO @objectId, @indexId, @frag, @indexName, @schemaName, @tableName;

    WHILE @@FETCH_STATUS = 0
    BEGIN
        SET @sqlCmd = NULL;

        IF @frag >= 30.0
            SET @sqlCmd = N''ALTER INDEX ['' + @indexName + N''] ON ['' + @schemaName + N''].['' + @tableName + N''] REBUILD WITH (ONLINE = ON);'';
        ELSE IF @frag >= 5.0
            SET @sqlCmd = N''ALTER INDEX ['' + @indexName + N''] ON ['' + @schemaName + N''].['' + @tableName + N''] REORGANIZE;'';

        IF @sqlCmd IS NOT NULL
        BEGIN
            PRINT @sqlCmd;
            EXEC sp_executesql @sqlCmd;
        END

        FETCH NEXT FROM index_cursor INTO @objectId, @indexId, @frag, @indexName, @schemaName, @tableName;
    END

    CLOSE index_cursor;
    DEALLOCATE index_cursor;
    ';

    EXEC (@sql); -- Switch to database and execute logic

    FETCH NEXT FROM db_cursor INTO @dbName;
END

CLOSE db_cursor;
DEALLOCATE db_cursor;
``` 


``` sql title="Rebuild Table's Indexes"
USE [Databasename];

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
    FROM sys.dm_db_index_physical_stats (DB_ID(), NULL, NULL, NULL, 'LIMITED') ips
    INNER JOIN sys.indexes i ON ips.[object_id] = i.[object_id] AND ips.index_id = i.index_id
    INNER JOIN sys.tables t ON ips.[object_id] = t.[object_id]
    INNER JOIN sys.schemas s ON t.schema_id = s.schema_id
    WHERE ips.index_id > 0 AND ips.alloc_unit_type_desc = 'IN_ROW_DATA';

OPEN index_cursor;
FETCH NEXT FROM index_cursor INTO @objectId, @indexId, @frag, @indexName, @schemaName, @tableName;

WHILE @@FETCH_STATUS = 0
BEGIN
    SET @sqlCmd = NULL;

    IF @frag >= 30.0
        SET @sqlCmd = N'ALTER INDEX [' + @indexName + N'] ON [' + @schemaName + N'].[' + @tableName + N'] REBUILD;';
    ELSE IF @frag >= 5.0
        SET @sqlCmd = N'ALTER INDEX [' + @indexName + N'] ON [' + @schemaName + N'].[' + @tableName + N'] REORGANIZE;';

    IF @sqlCmd IS NOT NULL
    BEGIN
        PRINT @sqlCmd;
        EXEC sp_executesql @sqlCmd;
    END

    FETCH NEXT FROM index_cursor INTO @objectId, @indexId, @frag, @indexName, @schemaName, @tableName;
END

CLOSE index_cursor;
DEALLOCATE index_cursor;
```

