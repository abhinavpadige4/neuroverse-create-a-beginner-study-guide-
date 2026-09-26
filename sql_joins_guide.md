# SQL Joins — A Beginner Study Guide

> Learn how to combine rows from two tables using `INNER`, `LEFT`, `RIGHT`, and `FULL` joins. Each section includes a short explanation, a small worked example, and a visual intuition. A final section lists the most common mistakes beginners make.

---

## Table of Contents

1. [The Sample Tables](#1-the-sample-tables)
2. [INNER JOIN](#2-inner-join)
3. [LEFT JOIN (LEFT OUTER JOIN)](#3-left-join-left-outer-join)
4. [RIGHT JOIN (RIGHT OUTER JOIN)](#4-right-join-right-outer-join)
5. [FULL JOIN (FULL OUTER JOIN)](#5-full-join-full-outer-join)
6. [Quick Comparison](#6-quick-comparison)
7. [Common Mistakes](#7-common-mistakes)
8. [Practice Exercises](#8-practice-exercises)

---

## 1. The Sample Tables

We'll use two small tables throughout this guide so you can follow along easily.

### `employees`

| emp_id | name       | dept_id |
|-------:|------------|--------:|
|      1 | Alice      |       1 |
|      2 | Bob        |       2 |
|      3 | Carol      |       3 |
|      4 | Dave       |       9 |

### `departments`

| dept_id | dept_name     |
|--------:|---------------|
|       1 | Engineering   |
|       2 | Sales         |
|       3 | Marketing     |
|       4 | HR            |

Notice two interesting facts:

- **Dave** (`emp_id = 4`) has `dept_id = 9`, but there is **no department 9**.
- **HR** (`dept_id = 4`) exists, but **no employee** belongs to it.

These two "orphan" rows are exactly what make LEFT, RIGHT, and FULL joins interesting.

---

## 2. INNER JOIN

### What it does

An `INNER JOIN` returns **only the rows that have a matching value in both tables**. If a row in the left table has no match in the right table (or vice versa), it is dropped.

### Example

```sql
SELECT e.emp_id, e.name, d.dept_name
FROM employees e
INNER JOIN departments d
  ON e.dept_id = d.dept_id;
```

### Result

| emp_id | name  | dept_name   |
|-------:|-------|-------------|
|      1 | Alice | Engineering |
|      2 | Bob   | Sales       |
|      3 | Carol | Marketing   |

### Why Dave and HR are missing

- Dave's `dept_id = 9` has no match in `departments` → dropped.
- HR's `dept_id = 4` has no match in `employees` → dropped.

### Visual intuition

```
employees   ∩   departments
   ┌───┐         ┌───┐
   │ A │         │ E │
   │ B │         │ S │
   │ C │         │ M │
   │ D │         │ H │
   └───┘         └───┘
        INNER JOIN
        ┌───────┐
        │ A-E   │
        │ B-S   │
        │ C-M   │
        └───────┘
```

---

## 3. LEFT JOIN (LEFT OUTER JOIN)

### What it does

A `LEFT JOIN` returns **all rows from the left table**, plus the matching rows from the right table. If there is no match on the right, the right-side columns are filled with `NULL`.

### Example

```sql
SELECT e.emp_id, e.name, d.dept_name
FROM employees e
LEFT JOIN departments d
  ON e.dept_id = d.dept_id;
```

### Result

| emp_id | name  | dept_name   |
|-------:|-------|-------------|
|      1 | Alice | Engineering |
|      2 | Bob   | Sales       |
|      3 | Carol | Marketing   |
|      4 | Dave  | **NULL**    |

### Why Dave shows up with NULL

Dave is in the left table (`employees`), so he is always kept. There is no department 9, so `dept_name` becomes `NULL`.

### Visual intuition

```
employees   LEFT JOIN   departments
   ┌───┐              ┌───┐
   │ A │              │ E │
   │ B │              │ S │
   │ C │              │ M │
   │ D │              │ H │
   └───┘              └───┘
        ┌──────────┐
        │ A-E      │
        │ B-S      │
        │ C-M      │
        │ D-NULL   │  ← Dave kept, dept is NULL
        └──────────┘
```

> **Tip:** `LEFT JOIN` is the most common join you'll use in practice — for example, "list all customers, even those who have never placed an order."

---

## 4. RIGHT JOIN (RIGHT OUTER JOIN)

### What it does

A `RIGHT JOIN` returns **all rows from the right table**, plus the matching rows from the left table. If there is no match on the left, the left-side columns are filled with `NULL`.

### Example

```sql
SELECT e.emp_id, e.name, d.dept_name
FROM employees e
RIGHT JOIN departments d
  ON e.dept_id = d.dept_id;
```

### Result

| emp_id | name | dept_name |
|-------:|------|-----------|
|      1 | Alice| Engineering |
|      2 | Bob  | Sales     |
|      3 | Carol| Marketing |
| **NULL** | **NULL** | HR |

### Why HR shows up with NULLs

HR is in the right table (`departments`), so it is always kept. No employee has `dept_id = 4`, so `emp_id` and `name` become `NULL`.

### Visual intuition

```
employees   RIGHT JOIN   departments
   ┌───┐              ┌───┐
   │ A │              │ E │
   │ B │              │ S │
   │ C │              │ M │
   │ D │              │ H │
   └───┘              └───┘
        ┌──────────┐
        │ A-E      │
        │ B-S      │
        │ C-M      │
        │ NULL-H   │  ← HR kept, employee is NULL
        └──────────┘
```

> **Note:** `RIGHT JOIN` is rarely used because you can always rewrite it as a `LEFT JOIN` by swapping the tables. Some databases (e.g., MySQL) don't even support `RIGHT JOIN` directly.

---

## 5. FULL JOIN (FULL OUTER JOIN)

### What it does

A `FULL JOIN` returns **all rows from both tables**. Rows that match are combined; rows that don't match are kept with `NULL` on the side that has no match.

### Example

```sql
SELECT e.emp_id, e.name, d.dept_name
FROM employees e
FULL JOIN departments d
  ON e.dept_id = d.dept_id;
```

### Result

| emp_id | name | dept_name   |
|-------:|------|-------------|
|      1 | Alice| Engineering |
|      2 | Bob  | Sales       |
|      3 | Carol| Marketing   |
|      4 | Dave | **NULL**    |
| **NULL** | **NULL** | HR |

### Why both Dave and HR appear

- Dave has no matching department → kept with `dept_name = NULL`.
- HR has no matching employee → kept with `emp_id = NULL` and `name = NULL`.

### Visual intuition

```
employees   FULL JOIN   departments
   ┌───┐              ┌───┐
   │ A │              │ E │
   │ B │              │ S │
   │ C │              │ M │
   │ D │              │ H │
   └───┘              └───┘
        ┌──────────┐
        │ A-E      │
        │ B-S      │
        │ C-M      │
        │ D-NULL   │  ← Dave kept
        │ NULL-H   │  ← HR kept
        └──────────┘
```

> **Note:** Not every database supports `FULL JOIN`. MySQL, for example, does not — you'd simulate it with `LEFT JOIN ... UNION ... RIGHT JOIN`.

---

## 6. Quick Comparison

| Join Type   | Left rows kept? | Right rows kept? | Matched rows |
|-------------|:---------------:|:----------------:|:------------:|
| INNER       | Only if matched | Only if matched  | ✅           |
| LEFT        | **Always**      | Only if matched  | ✅           |
| RIGHT       | Only if matched | **Always**       | ✅           |
| FULL        | **Always**      | **Always**       | ✅           |

**Rule of thumb:**

- Use `INNER JOIN` when you only want rows that exist in **both** tables.
- Use `LEFT JOIN` when you want **all rows from the "main" table**, even if there's no match.
- Use `RIGHT JOIN` only when it reads more naturally (rare).
- Use `FULL JOIN` when you want to see **everything** from both sides, including orphans.

---

## 7. Common Mistakes

### Mistake 1: Forgetting that `NULL` is not equal to `NULL`

In SQL, `NULL = NULL` evaluates to `UNKNOWN`, not `TRUE`. This means a join condition like `ON a.x = b.x` will **never** match two `NULL` values.

```sql
-- This will NOT match rows where both x are NULL
SELECT * FROM a JOIN b ON a.x = b.x;

-- To match NULLs, use IS
SELECT * FROM a JOIN b ON a.x IS NOT DISTINCT FROM b.x;  -- PostgreSQL
-- or
SELECT * FROM a JOIN b ON (a.x = b.x OR (a.x IS NULL AND b.x IS NULL));
```

### Mistake 2: Confusing `ON` with `WHERE` in outer joins

With `INNER JOIN`, `ON` and `WHERE` often produce the same result. With outer joins, they do **not**.

```sql
-- WRONG: filters AFTER the join, turning LEFT JOIN into INNER JOIN
SELECT *
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_name = 'Engineering';

-- RIGHT: filters the right table BEFORE the join
SELECT *
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
                       AND d.dept_name = 'Engineering';
```

The first query drops Dave (because his `dept_name` is `NULL` and fails the `WHERE`). The second keeps Dave with `NULL` department columns.

### Mistake 3: Accidentally creating a Cartesian product

If you forget the `ON` clause (or write a condition that's always true), you get a **Cartesian product** — every row of the left table paired with every row of the right table.

```sql
-- BAD: no ON clause → 4 × 4 = 16 rows
SELECT * FROM employees, departments;

-- BAD: always-true condition → same Cartesian product
SELECT * FROM employees e JOIN departments d ON 1 = 1;
```

### Mistake 4: Joining on the wrong column

A very common typo: joining on `emp_id` instead of `dept_id`.

```sql
-- WRONG: joins employee id to department id
SELECT * FROM employees e
JOIN departments d ON e.emp_id = d.dept_id;

-- RIGHT: joins the foreign key to the primary key
SELECT * FROM employees e
JOIN departments d ON e.dept_id = d.dept_id;
```

### Mistake 5: Assuming `RIGHT JOIN` is supported everywhere

`RIGHT JOIN` is part of the SQL standard, but **MySQL does not support it**. Rewrite as a `LEFT JOIN` with the tables swapped:

```sql
-- Instead of:
SELECT * FROM employees e RIGHT JOIN departments d ON e.dept_id = d.dept_id;

-- Write:
SELECT * FROM departments d LEFT JOIN employees e ON e.dept_id = d.dept_id;
```

### Mistake 6: Forgetting to alias tables

When joining multiple tables, always use aliases and qualify column names. Otherwise you'll get "ambiguous column" errors or worse — silent bugs.

```sql
-- BAD: ambiguous 'id'
SELECT id FROM employees JOIN departments ON employees.id = departments.id;

-- GOOD: qualified with aliases
SELECT e.emp_id, d.dept_id
FROM employees e JOIN departments d ON e.dept_id = d.dept_id;
```

### Mistake 7: Using `JOIN` when you mean `CROSS JOIN` (or vice versa)

- `JOIN ... ON` requires a condition.
- `CROSS JOIN` produces every combination (Cartesian product) — useful for generating grids, but dangerous if used by accident.

### Mistake 8: Not checking for duplicate rows

If the join condition matches multiple rows on one side, you'll get duplicates. Use `DISTINCT` or aggregate carefully.

```sql
-- If two employees share dept_id, each dept row appears twice
SELECT d.dept_name, COUNT(*)
FROM departments d
JOIN employees e ON e.dept_id = d.dept_id
GROUP BY d.dept_name;
```

---

## 8. Practice Exercises

Try these on the sample tables above:

1. List all employees and their department names, including employees with no department.
2. List all departments and the number of employees in each, including departments with zero employees.
3. Find employees who are **not** in any department, and departments that have **no** employees.
4. Rewrite the `RIGHT JOIN` example from Section 4 as a `LEFT JOIN`.
5. Explain why `WHERE d.dept_name IS NULL` after a `LEFT JOIN` finds only unmatched left rows.

---

## Summary

| Join | Keeps | Use when |
|------|-------|----------|
| `INNER JOIN` | Only matched rows | You want only rows that exist in both tables |
| `LEFT JOIN` | All left rows + matches | You want all rows from the "main" table |
| `RIGHT JOIN` | All right rows + matches | Rarely — prefer `LEFT JOIN` with swapped tables |
| `FULL JOIN` | All rows from both | You want to see every row, matched or not |

**Remember:** joins are about **matching rows across tables**. The join type decides what happens to rows that **don't** match.

Happy querying! 🎉
