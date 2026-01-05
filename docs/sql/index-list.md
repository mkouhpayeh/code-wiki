# 📚 Schema Index List

---

## Index list for each **schema**
``` sql title="SQL Server"
SELECT
    tablename,
    indexname,
    indexdef
FROM
    pg_indexes
WHERE
    schemaname = 'public'
ORDER BY
    tablename,
    indexname;
```

---
