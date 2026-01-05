# 📚 Generate Sample Data

---

## Random Names
- Using a CTE (numbers table) is preferred in SQL Server because it is much faster (set-based, not row-by-row) and has cleaner & shorter code.

``` sql
;WITH Numbers AS
(
    SELECT TOP (500) ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n
    FROM sys.all_objects
)
INSERT INTO TableName (ColumnName)
SELECT 'Product ' + CAST(n AS nvarchar(10))
FROM Numbers;
```

---
