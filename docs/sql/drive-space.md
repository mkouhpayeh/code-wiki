``` sql
-- Disk Space Report for SQL Server
EXEC xp_fixeddrives;

-- Detailed version:
DECLARE @drives TABLE
(
    DriveLetter CHAR(1),
    FreeSpaceMB INT
);

INSERT INTO @drives
EXEC xp_fixeddrives;

-- Get total size using WMI via xp_cmdshell
CREATE TABLE #DriveSpace
(
    DriveLetter CHAR(1),
    TotalGB DECIMAL(10,2),
    FreeGB DECIMAL(10,2),
    UsedGB DECIMAL(10,2),
    UsedPercentage DECIMAL(5,2),
    FreePercentage DECIMAL(5,2),
    Status NVARCHAR(10)
);

DECLARE @cmd NVARCHAR(4000);
INSERT INTO #DriveSpace (DriveLetter, TotalGB, FreeGB, UsedGB, UsedPercentage, FreePercentage, Status)
SELECT 
    d.DriveLetter,
    CAST(s.TotalSize / 1024.0 / 1024.0 / 1024.0 AS DECIMAL(10,2)) AS TotalGB,
    CAST(d.FreeSpaceMB / 1024.0 AS DECIMAL(10,2)) AS FreeGB,
    CAST((s.TotalSize / 1024.0 / 1024.0) - (d.FreeSpaceMB / 1024.0) AS DECIMAL(10,2)) AS UsedGB,
    CAST(((s.TotalSize - (d.FreeSpaceMB * 1024 * 1024)) * 100.0 / s.TotalSize) AS DECIMAL(5,2)) AS UsedPercentage,
    CAST((d.FreeSpaceMB * 1024 * 1024 * 100.0 / s.TotalSize) AS DECIMAL(5,2)) AS FreePercentage,
    CASE 
        WHEN (d.FreeSpaceMB * 1.0 / (s.TotalSize / 1024 / 1024)) < 0.1 THEN 'Low'
        ELSE 'Good'
    END AS Status
FROM @drives d
CROSS APPLY (
    SELECT 
        CAST(f.TotalBytes AS FLOAT) AS TotalSize
    FROM OPENROWSET('MSDASQL', 'Driver={SQL Server};Server=(local);Trusted_Connection=yes;',
        'SELECT volume_mount_point AS Drive, total_bytes AS TotalBytes FROM sys.dm_os_volume_stats(DB_ID(), 1)'
    ) f
    WHERE LEFT(f.Drive, 1) = d.DriveLetter
) s;

SELECT 
    DriveLetter AS [Drive],
    TotalGB AS [TotalSpace],
    FreeGB AS [FreeSpace],
    UsedGB AS [UsedSpace],
    UsedPercentage,
    FreePercentage,
    Status
FROM #DriveSpace
ORDER BY DriveLetter;

DROP TABLE #DriveSpace;

```
