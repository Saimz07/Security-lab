# SQL Injection UNION Attacks — Column Count and Data Type Discovery

**Platform:** PortSwigger Web Security Academy
**Category:** SQL Injection / UNION Attacks
**Difficulty:** Practitioner
**Date:** 10/10/2026

## Objective
Learn the UNION attack technique: determine how many columns a
vulnerable query returns, then identify which of those columns can
hold string data — the groundwork needed before pulling real data out
of other tables.

## Approach

### Lab 1: Determining the number of columns
Same injectable parameter as earlier labs — the category filter
(`/filter?category=...`). Used `ORDER BY` to probe column count,
incrementing one at a time:

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
' ORDER BY 8--   -- Internal Server Error
```

`ORDER BY 8` broke the query, but so did a few numbers between 3 and
8 in a less obvious way — the error wasn't a clean, unambiguous
signal on its own. Confirmed the real count directly instead, using a
UNION with matching NULLs:

```
' UNION SELECT NULL,NULL,NULL--
```

This succeeded and produced a visible extra row in the product
listing, which is proof rather than inference — 3 columns confirmed.

### Lab 2: Finding a column containing text
With the count fixed at 3, tested each position for string
compatibility by swapping one NULL at a time for a literal string:

```
' UNION SELECT 'a',NULL,NULL--
```

This succeeded and displayed `a` as an extra row — proving column 1
accepts string data. Replaced `'a'` with the specific random value
the lab provided, in the correct position, to fully solve it:

```
' UNION SELECT NULL,'gV8wrr',NULL--
```

(Positioned in column 2, based on the lab's own instance — the exact
column varies per lab, which is why the one-at-a-time test matters
rather than assuming position 1 always works.)

## What was actually vulnerable
The category filter concatenated user input directly into a SQL
query with no parameterization, and the application displayed query
results directly in its response. That combination — injectable
input plus visible output — is exactly what makes UNION attacks
possible: an attacker can append an entirely separate `SELECT`
statement and have its results rendered on the page alongside the
legitimate data.

## How it would be fixed
Parameterized queries, as with the earlier SQLi labs — but also,
more specifically here, never reflect raw query output directly into
a page without controlling exactly what fields and tables that query
is allowed to touch.

## What I learned
- **ORDER BY and UNION-NULL aren't redundant.** ORDER BY gives a
  rough, fast signal but can be ambiguous (I got an error at 8 but
  hadn't cleanly confirmed 3 was the true boundary until I checked).
  UNION with matching NULLs is actual proof — it demonstrates a
  working injected row, not just an error message.
- **NULL is the right placeholder while counting** because it's
  compatible with virtually every data type in SQL, so it won't fail
  due to a type mismatch while you're still just establishing count.
- **Finding a string-compatible column requires testing positions
  one at a time** — you can't assume the first column is the one
  that accepts text; it depends entirely on the table's own schema.
- **SQL syntax varies by database engine.** Oracle requires
  `SELECT ... FROM DUAL` even for placeholder values, since Oracle
  doesn't allow a bare `SELECT` with no table. MySQL requires a
  trailing space after `--` for the comment to register, or `#` can
  be used instead. Neither issue appeared in these labs, which is
  itself informative — it suggests the backend isn't Oracle and
  tolerates the comment style used.

---
No credentials or sensitive data were retrieved in these labs beyond
the specific test values each lab required.
