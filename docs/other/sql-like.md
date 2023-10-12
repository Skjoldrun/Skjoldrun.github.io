---
layout: page
title: SQL - LIKE
parent: Other
---

# LIKE Command

In SQL there is the `LIKE` command to filter results for string to match a pattern.

```SQL
match_expression [ NOT ] LIKE pattern [ ESCAPE escape_character ]
```

The `NOT` is optional here, as well as the `ESCAPE escape_character`. 

The pattern can contain following arguments:


| Wildcard | Description | Example |
| --- | --- | --- |
| `%` | Any string of zero or more characters. | WHERE title LIKE '%programming%' finds all book titles with the word 'programming' anywhere in the book title. |
| `_` (underscore) | Any single character. | WHERE name LIKE '_ean' finds all four-letter first names that end with ean (Dean, Sean, and so on). |
| `[]` | Any single character within the specified range ([a-f]) or set ([abcdef]). | WHERE name LIKE '[D-S]ean' finds all first names that start with one of these letters in range (Dean, Sean, and so on). |
| `[^]` | Any single character not within the specified range ([^a-f]) or set ([^abcdef]). | WHERE name LIKE '[C,L,^K]arsen' finds all first names that start with one of these letters in range, but not with K (Carsen, Larsen, but not Karsen). |


## use the custom escape_character

If your result string already contain the `[..]` as content you have to escape them. You maybe have some log statements in a DB and your logger writes strings like `"[SomeCategory][SomeId]": Logger is starting.`. If you want to filter these results you have to escape the brackets with a SQL like the following:

```SQL
SELECT *
FROM MyDB_Table
WHERE [Message] LIKE '"\[SomeCategory\]%' ESCAPE '\'
```

Now you escape the brackets with the `\` character like you would escape in C# strings.