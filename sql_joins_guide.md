# SQL Joins — A Beginner Study Guide

SQL joins are how you combine rows from two (or more) tables based on a related
column. They are the single most important concept for writing useful queries
once you move beyond a single table.

This guide covers the four joins you will use 95% of the time:

- **INNER JOIN** — only matching rows from both tables
- **LEFT JOIN** — all rows from the left table, matches from the right (or NULL)
- **RIGHT JOIN** — all rows from the right table, matches from the left (or NULL)
- **FULL JOIN** — all rows from both tables, matched where possible

At the end you will find a **Common Mistakes** section that covers the pitfalls
that trip up almost every beginner.

---

## Sample Tables Used Throughout

Every example in this guide uses the same two small tables so you can follow
along easily.

### `employees`

| emp_id | name       | dept_id |
|-------:|------------|--------:|
|      1 | Alice      |       1 |
|      2 | Bob        |       2 |
|      3 | Carol      |       1 |
|      4 | Dave       |       9 |

### `departments`

| dept_id | dept_name   |
|--------:|-------------|
|       1 | Engineering |
|       2 | Sales       |
|       3 | Marketing   |

Notice two things that will matter for the examples:

- **Dave** has `dept_id = 9`, but there is no department 9. He is an employee
  with no matching department.
- **Marketing** (dept_id 3) has no employees at all. It is a department with
  no matching employee.

These two "orphan" rows are exactly what make LEFT, RIGHT, and FULL joins
interesting.

---

## 1. INNER JOIN

### What it does

An `INNER JOIN` returns **only the rows that have a match in both tables**.
If a row on either side has no match, it is dropped entirely.

### Example

```sql
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
```

### Result

| name  | dept_name   |
|-------|-------------|
| Alice | Engineering |
| Bob   | Sales       |
| Carol | Engineering |

### Why this result?

- Alice, Bob, and Carol all have a `dept_id` that exists in `departments`, so
  they appear.
- **Dave is missing** because there is no department with `dept_id = 9`.
- **Marketing is missing** because no employee has `dept_id = 3`.

### When to use it

Use `INNER JOIN` when you only care about rows that exist on **both** sides.
For example: "Give me every employee along with their department name" — you
don't want to see employees whose department is unknown.

> **Note:** `INNER JOIN` is the default join. Writing just `JOIN` is the same
> as writing `INNER JOIN`.

---

## 2. LEFT JOIN (LEFT OUTER JOIN)

### What it does

A `LEFT JOIN` returns **all rows from the left table**, plus the matching rows
from the right table. If there is no match on the right, the right-side columns
are filled with `NULL`.

### Example

```sql
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

### Result

| name  | dept_name   |
|-------|-------------|
| Alice | Engineering |
| Bob   | Sales       |
| Carol | Engineering |
| Dave  | NULL        |

### Why this result?

- Alice, Bob, and Carol behave exactly like in the INNER JOIN.
- **Dave is now included**, but because there is no matching department,
  `dept_name` is `NULL`.
- **Marketing is still missing** — LEFT JOIN only guarantees rows from the
  *left* table.

### When to use it

Use `LEFT JOIN` when you want **everything from the "main" table**, even if
there is no matching row on the other side. For example: "List every employee,
and show their department if we know it."

---

## 3. RIGHT JOIN (RIGHT OUTER JOIN)

### What it does

A `RIGHT JOIN` is the mirror image of a LEFT JOIN. It returns **all rows from
the right table**, plus the matching rows from the left. If there is no match
on the left, the left-side columns are filled with `NULL`.

### Example

```sql
SELECT e.name, d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;
```

### Result

| name  | dept_name   |
|-------|-------------|
| Alice | Engineering |
| Bob   | Sales       |
| Carol | Engineering |
| NULL  | Marketing   |

### Why this result?

- Alice, Bob, and Carol still match.
- **Marketing is now included**, with `name = NULL` because no employee belongs
  to it.
- **Dave is missing** — RIGHT JOIN only guarantees rows from the *right* table.

### When to use it

Use `RIGHT JOIN` when you want **everything from the right table**, even if
there is no matching row on the left. For example: "List every department, and
show one of its employees if there is one."

> **Tip:** Most developers prefer to write a `LEFT JOIN` and just swap the
> order of the tables instead of using `RIGHT JOIN`. The result is identical
> and the query is easier to read.

---

## 4. FULL JOIN (FULL OUTER JOIN)

### What it does

A `FULL JOIN` returns **all rows from both tables**. Where there is a match,
the rows are combined. Where there is no match on either side, the missing
columns are filled with `NULL`.

### Example

```sql
SELECT e.name, d.dept_name
FROM employees e
FULL JOIN departments d ON e.dept_id = d.dept_id;
```

### Result

| name  | dept_name   |
|-------|-------------|
| Alice | Engineering |
| Bob   | Sales       |
| Carol | Engineering |
| Dave  | NULL        |
| NULL  | Marketing   |

### Why this result?

- The three matched employees appear as before.
- **Dave** appears with `dept_name = NULL` (no matching department).
- **Marketing** appears with `name = NULL` (no matching employee).

### When to use it

Use `FULL JOIN` when you want to see **everything from both tables** and
identify mismatches on either side. It is very useful for reconciliation and
data-quality checks, e.g. "Show me every customer and every order, and I want
to see customers with no orders *and* orders with no customer."

> **Note:** Not every database supports `FULL JOIN` directly. MySQL, for
> example, does not — you can emulate it with a `LEFT JOIN ... UNION ...
> RIGHT JOIN`.

---

## Visual Summary

Imagine the two tables as two overlapping circles (a Venn diagram):

```
   INNER JOIN        LEFT JOIN         RIGHT JOIN        FULL JOIN
   ┌─────────┐      ┌─────────┐       ┌─────────┐       ┌─────────┐
   │         │      │         │       │         │       │         │
   │   ●●●   │      │ ●●●     │       │     ●●● │       │ ●●●     │
   │         │      │ ●●●     │       │     ●●● │       │ ●●●     │
   │         │      │         │       │         │       │         │
   └─────────┘      └─────────┘       └─────────┘       └─────────┘
   only overlap     left + overlap    overlap + right   everything
```

- **INNER** = the overlap only.
- **LEFT** = the whole left circle (overlap + left-only).
- **RIGHT** = the whole right circle (overlap + right-only).
- **FULL** = both circles (everything).

---

## Common Mistakes

These are the pitfalls that show up in almost every beginner's SQL. If you
avoid these, you will write correct joins most of the time.

### 1. Confusing `ON` with `WHERE`

`ON` defines **how** rows are matched. `WHERE` filters rows **after** the join
has happened. Mixing them up changes the result dramatically.

```sql
-- WRONG: filters AFTER the join, so unmatched rows become NULL and are dropped
SELECT *
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_name = 'Engineering';

-- RIGHT: filters the right table BEFORE the join
SELECT *
FROM employees e
LEFT JOIN departments d
       ON e.dept_id = d.dept_id
      AND d.dept_name = 'Engineering';
```

The first query silently turns your LEFT JOIN into an INNER JOIN because the
`WHERE` clause removes every row where `dept_name` is `NULL`.

### 2. Forgetting that unmatched columns become `NULL`

After a LEFT, RIGHT, or FULL join, the columns from the side that had no match
are `NULL`. Beginners often write code that assumes those columns are always
populated, and then get surprised by `NULL`s in their results.

Use `COALESCE` or `IS NULL` checks when you need to handle this:

```sql
SELECT e.name, COALESCE(d.dept_name, 'Unassigned') AS dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

### 3. Comparing to `NULL` with `=`

`NULL = NULL` is **not** true in SQL — it is `UNKNOWN`. To test for NULL you
must use `IS NULL` or `IS NOT NULL`.

```sql
-- WRONG: never matches
SELECT * FROM employees WHERE dept_id = NULL;

-- RIGHT
SELECT * FROM employees WHERE dept_id IS NULL;
```

### 4. Accidentally creating a Cartesian product

If you forget the `ON` clause (or write a condition that is always true), you
get a **Cartesian product** — every row of the left table paired with every
row of the right table. With 1,000 employees and 10 departments that is
10,000 rows, and the numbers explode fast.

```sql
-- DANGEROUS: no ON clause, produces a Cartesian product
SELECT * FROM employees, departments;
```

Always include an `ON` clause that actually restricts the match.

### 5. Joining on a non-unique column

If you join on a column that has duplicates on one side, you will get
**duplicate rows** in the result. For example, if two employees share the same
`dept_id` and you join to a table that also has duplicates, you can end up
with far more rows than expected.

Join on a **primary key** or a **unique** column whenever possible.

### 6. Assuming `RIGHT JOIN` is required

Many developers avoid `RIGHT JOIN` entirely and just swap the tables in a
`LEFT JOIN`. Both produce the same result, but LEFT JOIN is easier to read
because the "main" table is always on the left.

```sql
-- These two queries return the same rows:
SELECT * FROM employees e RIGHT JOIN departments d ON e.dept_id = d.dept_id;
SELECT * FROM departments d LEFT JOIN employees e ON e.dept_id = d.dept_id;
```

### 7. Forgetting that INNER JOIN is the default

Writing `JOIN` without a qualifier is the same as `INNER JOIN`. This is fine
when you want only matches, but it silently drops unmatched rows when you
actually wanted a LEFT JOIN.

### 8. Mixing up which side is "left" and which is "right"

In `FROM a LEFT JOIN b`, **`a` is the left table** and **`b` is the right
table**. The LEFT JOIN keeps all rows from `a`. Beginners often write the
tables in the wrong order and then wonder why their "all employees" query is
missing employees.

### 9. Not knowing your database's support for FULL JOIN

`FULL JOIN` is standard SQL, but not every database implements it. MySQL, for
example, does not support it directly. If you need the behavior in MySQL,
emulate it:

```sql
SELECT * FROM employees e LEFT JOIN departments d ON e.dept_id = d.dept_id
UNION
SELECT * FROM employees e RIGHT JOIN departments d ON e.dept_id = d.dept_id;
```

### 10. Joining on the wrong column

A very common beginner error is joining on a column that looks similar but is
not the actual foreign key. For example, joining `employees.dept_id` to
`departments.dept_name` instead of `departments.dept_id`. The query may run
without error but return nonsense.

Always double-check that you are joining on the **primary key** of one table
to the **foreign key** in the other.

---

## Quick Reference Cheat Sheet

| Join Type    | Rows Returned                                        |
|--------------|------------------------------------------------------|
| INNER JOIN   | Only rows that match in both tables                  |
| LEFT JOIN    | All rows from the left table + matches from the right|
| RIGHT JOIN   | All rows from the right table + matches from the left|
| FULL JOIN    | All rows from both tables, matched where possible    |

## Key Takeaways

1. **INNER JOIN** is the strictest — only matches survive.
2. **LEFT JOIN** keeps everything from the left table; unmatched right columns
   become `NULL`.
3. **RIGHT JOIN** is the mirror of LEFT JOIN — most people just swap the
   tables and use LEFT JOIN instead.
4. **FULL JOIN** keeps everything from both tables.
5. Use `ON` to define the match, `WHERE` to filter after the match.
6. Always expect `NULL`s on the unmatched side of an outer join.
7. Join on primary keys / unique columns to avoid duplicate rows.

Master these four joins and you can answer almost any question that involves
more than one table.
