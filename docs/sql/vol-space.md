# 📚 Volume Space

---

``` sql
/* Disk space report per volume
   Requires: VIEW SERVER STATE permission
*/
WITH vols AS
(
    SELECT DISTINCT
        vs.volume_mount_point,
        vs.total_bytes,
        vs.available_bytes
    FROM sys.master_files AS mf
    CROSS APPLY sys.dm_os_volume_stats(mf.database_id, mf.file_id) AS vs
)
SELECT
    CASE 
        WHEN vols.volume_mount_point LIKE '[A-Z]:\%' 
            THEN LEFT(vols.volume_mount_point, 2)      -- e.g. 'C:'
        ELSE vols.volume_mount_point                    -- mount path
    END AS [Drive],
    CAST(vols.total_bytes     / 1024.0 / 1024.0 / 1024.0 AS DECIMAL(18,2)) AS [TotalSpace],
    CAST(vols.available_bytes / 1024.0 / 1024.0 / 1024.0 AS DECIMAL(18,2)) AS [FreeSpace],
    CAST((vols.total_bytes - vols.available_bytes) / 1024.0 / 1024.0 / 1024.0 AS DECIMAL(18,2)) AS [UsedSpace],
    CAST(((vols.total_bytes - vols.available_bytes) * 100.0 / vols.total_bytes) AS DECIMAL(5,2)) AS [UsedPercentage],
    CAST((vols.available_bytes * 100.0 / vols.total_bytes) AS DECIMAL(5,2)) AS [FreePercentage],
    CASE 
        WHEN (vols.available_bytes * 100.0 / vols.total_bytes) < 10 THEN 'Low'
        WHEN (vols.available_bytes * 100.0 / vols.total_bytes) < 20 THEN 'Warn'
        ELSE 'Good'
    END AS [Status]
FROM vols
ORDER BY [Drive];

```

``` sql
GRANT VIEW SERVER STATE TO [YourLogin];
```
