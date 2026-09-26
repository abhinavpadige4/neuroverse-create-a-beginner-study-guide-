# SQL Joins — A Beginner Study Guide

A beginner-friendly study guide covering the four most common SQL joins:
**INNER**, **LEFT**, **RIGHT**, and **FULL**. Each join type has a short
explanation, one small worked example using the same two sample tables,
and a note on when to use it. A final section collects the mistakes
beginners make most often.

## Files

- [`sql_joins_guide.md`](./sql_joins_guide.md) — the full markdown study guide (source of truth).
- [`index.html`](./index.html) — a styled HTML rendering of the same guide, served at the repo root.

## What's inside

1. **Sample Tables** — two tiny tables (`employees`, `departments`) with
   deliberate "orphan" rows (a `NULL` department and an empty department)
   so every join type has something interesting to show.
2. **INNER JOIN** — only matching rows from both sides.
3. **LEFT JOIN** — every left row, `NULL` on the right when no match.
4. **RIGHT JOIN** — every right row, `NULL` on the left when no match.
5. **FULL JOIN** — every row from both sides, `NULL` where there's no match.
6. **Visual Summary** — a small ASCII diagram of the four join shapes.
7. **Common Mistakes** — seven pitfalls:
   - `NULL = NULL` is not true
   - `WHERE` vs `ON` (turns a LEFT JOIN into an INNER JOIN)
   - Accidental Cartesian products
   - Joining on non-unique columns (row duplication)
   - Assuming `RIGHT JOIN` is portable
   - Forgetting `FULL JOIN` isn't universal (MySQL)
   - Mixing up which side is "kept"
8. **Quick Reference** — a one-glance table of what each join keeps.
9. **Practice Ideas** — five exercises to test your understanding.

## Viewing

Open `index.html` in a browser, or read `sql_joins_guide.md` in any
Markdown viewer.
