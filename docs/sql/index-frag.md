# Check Index Fragmentation

``` sql title="SQL Server"
SELECT 
    dbschemas.[name] AS [Schema],
    dbtables.[name] AS [Table],
    dbindexes.[name] AS [Index],
    indexstats.avg_fragmentation_in_percent,
    indexstats.page_count
FROM 
    sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') AS indexstats
INNER JOIN 
    sys.tables dbtables ON indexstats.object_id = dbtables.object_id
INNER JOIN 
    sys.schemas dbschemas ON dbtables.schema_id = dbschemas.schema_id
INNER JOIN 
    sys.indexes AS dbindexes ON dbindexes.object_id = indexstats.object_id 
                              AND indexstats.index_id = dbindexes.index_id
WHERE 
    indexstats.database_id = DB_ID()
    AND indexstats.page_count > 100 -- skip small indexes
ORDER BY 
    avg_fragmentation_in_percent DESC;
```

```
| Fragmentation % | Action                           |
| --------------- | -------------------------------- |
| 0–5%            | No action needed                 |
| 5–30%           | **Reorganize** index (light fix) |
| >30%            | **Rebuild** index (full fix)     |

```

``` sql title="Reorganize (lightweight)"
ALTER INDEX [IndexName] ON [TableName] REORGANIZE;
```
``` sql title="Rebuild (heavier, can update stats)"
ALTER INDEX [IndexName] ON [TableName] REBUILD WITH (ONLINE = ON);
```
