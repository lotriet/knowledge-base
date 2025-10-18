# Faster Prefix Search in MySQL with a Normalized Column

**TL;DR:**
Make a cleaned, lowercased copy of your text in a new column, **index it**, and search that column with `LIKE 'term%'`. This turns slow scans into fast index lookups.

---

## When to use this
- Users type a name, address, or code and you only need **prefix** matching (e.g., `"jo"`, `"john s"`).
- Data may have mixed case, punctuation, or extra spaces.
- You want speed without introducing new infra (works on plain MySQL).

> If you need **contains** (`'%term%'`) or fuzzy search, consider **FULLTEXT** or a search engine. This guide is for **prefix** searches.

---

## MySQL version requirements

- **MySQL 8.0+** (recommended) - Uses `REGEXP_REPLACE` and `utf8mb4_0900_ai_ci` collation
- **MySQL 5.7** - Requires alternative approach (see below)

---

## The pattern (MySQL 8.0+)

1) **Add a normalized copy** of your source text
2) **Index** the normalized column (prefix index is fine)
3) **Query** using `LIKE 'prefix%'`

```sql
-- 1) Add normalized column
ALTER TABLE MyTable
  ADD COLUMN SourceText_Normalized VARCHAR(1000)
    GENERATED ALWAYS AS (
      LOWER(
        REGEXP_REPLACE(SourceText, '[^[:alnum:]]+', ' ')
      )
    ) STORED
    COLLATE utf8mb4_0900_ai_ci;  -- case & accent-insensitive compare

-- 2) Index it (prefix keeps the index smaller under utf8mb4)
CREATE INDEX ix_sourcetext_norm ON MyTable (SourceText_Normalized(256));

-- 3) Query (uses the index)
SELECT *
FROM MyTable
WHERE SourceText_Normalized LIKE 'john smi%';

-- 4) Verify the index is being used
EXPLAIN SELECT *
FROM MyTable
WHERE SourceText_Normalized LIKE 'john smi%';
```

### What does `[^[:alnum:]]+` mean?
- `[:alnum:]` = letters and digits.
- `[^ ... ]` = "not" that set.
- `+` = one or more.
So it **finds runs of non-alphanumeric characters** (punctuation, extra spaces, etc.) and replaces them with a **single space**.

---

## MySQL 5.7 alternative

For MySQL 5.7, use a trigger-based approach since `REGEXP_REPLACE` isn't available:

```sql
-- 1) Add normalized column (not generated)
ALTER TABLE MyTable
  ADD COLUMN SourceText_Normalized VARCHAR(1000)
    COLLATE utf8mb4_general_ci;

-- 2) Create trigger to populate on insert
DELIMITER $$
CREATE TRIGGER MyTable_BeforeInsert
BEFORE INSERT ON MyTable
FOR EACH ROW
BEGIN
  SET NEW.SourceText_Normalized = LOWER(TRIM(NEW.SourceText));
END$$

-- 3) Create trigger to populate on update
CREATE TRIGGER MyTable_BeforeUpdate
BEFORE UPDATE ON MyTable
FOR EACH ROW
BEGIN
  IF NEW.SourceText != OLD.SourceText THEN
    SET NEW.SourceText_Normalized = LOWER(TRIM(NEW.SourceText));
  END IF;
END$$
DELIMITER ;

-- 4) Backfill existing data
UPDATE MyTable
SET SourceText_Normalized = LOWER(TRIM(SourceText));

-- 5) Index it
CREATE INDEX ix_sourcetext_norm ON MyTable (SourceText_Normalized(256));
```

**Note:** The MySQL 5.7 approach doesn't remove punctuation automatically. For more aggressive cleaning, you'd need to chain multiple `REPLACE()` calls in the trigger.

---

## Multiple fields (optional one-box search)

Combine several columns, then normalize once.

```sql
-- MySQL 8.0+
ALTER TABLE MyTable
  ADD COLUMN SearchKey_Normalized VARCHAR(1000)
    GENERATED ALWAYS AS (
      LOWER(
        REGEXP_REPLACE(
          CONCAT_WS(' ', FieldA, FieldB, FieldC),
          '[^[:alnum:]]+', ' '
        )
      )
    ) STORED
    COLLATE utf8mb4_0900_ai_ci;

CREATE INDEX ix_searchkey_norm ON MyTable (SearchKey_Normalized(256));

SELECT *
FROM MyTable
WHERE SearchKey_Normalized LIKE 'some prefix%';
```

---

## Do / Don't

**Do**
- Use `CONCAT_WS(' ', ...)` so values are separated before cleaning.
- Use a **prefix index** (e.g., `(256)`) on large `VARCHAR` with `utf8mb4`.
- Keep queries **prefix-only**: `LIKE 'term%'`.
- Use `EXPLAIN` to verify the index is being used.
- Choose `STORED` over `VIRTUAL` for generated columns you want to index.

**Don't**
- Expect the index to help with `LIKE '%term%'` (leading wildcard).
- Store sensitive/PII field names in public code — keep them generic.
- Rely on triggers if you can use **generated columns** (less drift, easier maintenance).

---

## Common questions

**Q: Why `utf8mb4_0900_ai_ci`?**
It makes comparisons **accent-insensitive** (`ai`) and **case-insensitive** (`ci`), so `"Jöhn Smith"` matches `"john smith"`. This collation requires MySQL 8.0+. For MySQL 5.7, use `utf8mb4_general_ci` or `utf8mb4_unicode_ci`.

**Q: Will this speed up contains search (`'%smith%'`)?**
No. For contains/fuzzy, use **FULLTEXT** or a search service.

**Q: How big should the prefix index be?**
`(256)` is a practical default for names/addresses. If you know your typical prefix length, you can shrink it (e.g., `(50)` for short codes).

**Q: `STORED` vs `VIRTUAL` generated columns?**
- `STORED`: Consumes disk space, can be indexed, computed once on write.
- `VIRTUAL`: Computed on read, cannot be indexed directly.
For this pattern, use `STORED`.

---

## Copy-ready impact line (CV/README)

> Added a normalized column with prefix indexing, converting full table scans into indexed range scans for prefix `LIKE` searches in MySQL.

Or:

> Implemented prefix-indexed normalized columns, significantly improving query performance for user search features in MySQL.
