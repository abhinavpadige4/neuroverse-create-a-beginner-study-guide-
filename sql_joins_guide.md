# SQL Joins — A Beginner Study Guide

A friendly, from-scratch walkthrough of the four most common SQL joins:
**INNER**, **LEFT**, **RIGHT**, and **FULL**. Each section has a short
explanation, one small worked example, and a note on what to watch out for.
A final section collects the mistakes beginners make most often.

---

## 1. Sample Tables

Every example in this guide uses the same two tiny tables so you can focus
on the join logic instead of the data.

### `employees`

| employee_id | name        | department_id |
|------------:|-------------|--------------:|
| 1           | Alice       | 10            |
| 2           | Bob         | 20            |
| 3           | Carol       | 30            |
| 4           | Dan         | NULL          |

### `departments`

| department_id | department_name |
|--------------:|-----------------|
| 10            | Engineering     |
| 20            | Sales           |
| 40            | Marketing       |

Notice two things that will matter later:

- **Dan** has `department_id = NULL` — he is not assigned to any department.
- **Marketing (40)** has no employees at all.

These two "orphan" rows are exactly what LEFT, RIGHT, and FULL joins are
designed to surface.

---

## 2. INNER JOIN

### What it does

`INNER JOIN` returns only the rows where the join condition is true on
**both** sides. If a row in the left table has no matching row in the right
table (or vice versa), it is dropped.

`INNER JOIN` is the default when you write `JOIN` without a qualifier.

### Example

```sql
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d
  ON e.department_id = d.department_id;
```

### Result

| name  | department_name |
|-------|-----------------|
| Alice | Engineering     |
| Bob   | Sales           |

### Why this result?

- Alice (10) matches Engineering (10) → kept.
- Bob (20) matches Sales (20) → kept.
- Carol (30) has no department with id 30 → dropped.
- Dan has `department_id = NULL`. `NULL = 30` is not true, so Dan is dropped.
- Marketing (40) has no employee with `department_id = 40` → dropped.

### When to use it

Use `INNER JOIN` when you only care about rows that have a valid match on
both sides — for example, "list every employee along with their department
name" where you don't want to include unassigned employees.

---

## 3. LEFT JOIN (LEFT OUTER JOIN)

### What it does

`LEFT JOIN` keeps **every row from the left table**. For each left row,
it attaches the matching right row if one exists; otherwise the right-side
columns are filled with `NULL`.

### Example

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d
  ON e.department_id = d.department_id;
```

### Result

| name  | department_name |
|-------|-----------------|
| Alice | Engineering     |
| Bob   | Sales           |
| Carol | NULL            |
| Dan   | NULL            |

### Why this result?

- Alice and Bob match as before.
- Carol has no matching department, but she is still on the left side, so
  she is kept with `department_name = NULL`.
- Dan's `department_id` is `NULL`, so no match is found — but he is still
  kept, again with `department_name = NULL`.
- Marketing (40) is on the right side and has no match, so it is dropped.

### When to use it

Use `LEFT JOIN` when you want "everything from the left, plus whatever
matches on the right." A classic pattern is finding rows that **don't**
have a match:

```sql
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
WHERE d.department_id IS NULL;   -- employees with no department
```

---

## 4. RIGHT JOIN (RIGHT OUTER JOIN)

### What it does

`RIGHT JOIN` is the mirror image of `LEFT JOIN`. It keeps **every row from
the right table** and fills in `NULL` for the left-side columns when there
is no match.

### Example

```sql
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d
  ON e.department_id = d.department_id;
```

### Result

| name  | department_name |
|-------|-----------------|
| Alice | Engineering     |
| Bob   | Sales           |
| NULL  | Marketing       |

### Why this result?

- Alice and Bob match as before.
- Marketing (40) has no matching employee, but it is on the right side, so
  it is kept with `name = NULL`.
- Carol and Dan are on the left side with no match, so they are dropped.

### When to use it

`RIGHT JOIN` is rarely needed — you can almost always rewrite it as a
`LEFT JOIN` by swapping the tables. Some teams ban `RIGHT JOIN` in code
review for that reason. Still, it's worth knowing because you'll see it in
legacy queries.

---

## 5. FULL JOIN (FULL OUTER JOIN)

### What it does

`FULL JOIN` keeps **every row from both tables**. When a row has no match
on the other side, the missing columns are filled with `NULL`. It is the
union of a `LEFT JOIN` and a `RIGHT JOIN`.

### Example

```sql
SELECT e.name, d.department_name
FROM employees e
FULL JOIN departments d
  ON e.department_id = d.department_id;
```

### Result

| name  | department_name |
|-------|-----------------|
| Alice | Engineering     |
| Bob   | Sales           |
| Carol | NULL            |
| Dan   | NULL            |
| NULL  | Marketing       |

### Why this result?

- Alice and Bob match as before.
- Carol and Dan are kept from the left side with `department_name = NULL`.
- Marketing is kept from the right side with `name = NULL`.

### When to use it

Use `FULL JOIN` when you need a complete picture of both sides — for
example, reconciling two lists and finding rows that appear in only one of
them. Note that not every database supports `FULL JOIN` natively (MySQL
does not; PostgreSQL, SQL Server, Oracle, and BigQuery do).

---

## 6. Visual Summary

```
   LEFT TABLE          RIGHT TABLE
   ┌─────────┐        ┌─────────┐
   │  A  B  C│        │  B  C  D│
   └─────────┘        └─────────┘

   INNER JOIN  →  B  C
   LEFT JOIN   →  A  B  C  (D dropped)
   RIGHT JOIN  →  B  C  D  (A dropped)
   FULL JOIN   →  A  B  C  D
```

---

## 7. Common Mistakes

### Mistake 1: Forgetting that `NULL = NULL` is not true

In SQL, `NULL` compared to anything (including another `NULL`) evaluates
to `UNKNOWN`, not `TRUE`. So a join condition like
`ON e.department_id = d.department_id` will **never** match a `NULL`
`department_id`. That's why Dan disappears from the `INNER JOIN` result.

If you truly need to match `NULL`s, use `IS NOT DISTINCT FROM` (PostgreSQL)
or `IS NULL` checks, or `COALESCE` both sides to a sentinel value.

### Mistake 2: Putting the join condition in `WHERE` instead of `ON`

For `INNER JOIN` the two are usually equivalent, but for `LEFT`/`RIGHT`/
`FULL` joins they are **not**. Consider:

```sql
-- WRONG: filters AFTER the join, turning the LEFT JOIN into an INNER JOIN
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d
WHERE e.department_id = d.department_id;

-- RIGHT: filters DURING the join
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d
  ON e.department_id = d.department_id;
```

The first query drops Carol and Dan because their `department_id` doesn't
equal any `d.department_id` — the `WHERE` clause runs after the join and
removes the `NULL` rows. The second query keeps them.

**Rule of thumb:** put the join condition in `ON`; put filters that apply
to the joined result in `WHERE`.

### Mistake 3: Accidentally creating a Cartesian product

If you forget the `ON` clause (or write a condition that is always true,
like `ON 1 = 1`), you get a **Cartesian product** — every row of the left
table paired with every row of the right table. With 1,000 employees and
100 departments, that's 100,000 rows, most of which are meaningless.

Always double-check that your `ON` clause references real columns from
**both** tables.

### Mistake 4: Joining on non-unique columns and getting duplicated rows

If the right table has multiple rows that match one row on the left, the
left row is duplicated once per match. For example, if `departments` had
two rows with `department_id = 10`, Alice would appear twice in the
result.

Before joining, check that the join column is unique on the side you're
"expanding into." If it isn't, aggregate first or use `DISTINCT`.

### Mistake 5: Assuming `RIGHT JOIN` is portable

`RIGHT JOIN` is part of the SQL standard, but some databases (notably
MySQL) don't support it. If portability matters, rewrite as a `LEFT JOIN`
with the tables swapped.

### Mistake 6: Forgetting that `FULL JOIN` isn't universal either

MySQL has no `FULL JOIN` at all. The common workaround is:

```sql
SELECT ... FROM a LEFT JOIN b ON ...
UNION
SELECT ... FROM a RIGHT JOIN b ON ...;
```

Or, in MySQL specifically, `LEFT JOIN ... UNION ... LEFT JOIN` with the
tables swapped.

### Mistake 7: Mixing up which side is "kept"

A quick mental check: in `A LEFT JOIN B`, **A is always kept**. In
`A RIGHT JOIN B`, **B is always kept**. If you find yourself writing
`RIGHT JOIN` and then wondering why rows are missing, try swapping the
tables and using `LEFT JOIN` instead — it's easier to reason about.

---

## 8. Quick Reference

| Join type     | Left rows kept | Right rows kept | Missing side filled with |
|---------------|:--------------:|:---------------:|:------------------------:|
| `INNER JOIN`  | only matches   | only matches    | (rows dropped)           |
| `LEFT JOIN`   | all            | only matches    | `NULL` on right          |
| `RIGHT JOIN`  | only matches   | all             | `NULL` on left           |
| `FULL JOIN`   | all            | all             | `NULL` on either side    |

---

## 9. Practice Ideas

Try these on the sample tables above to test your understanding:

1. List every department, even those with no employees.
2. List every employee who is not assigned to a department.
3. Show every employee and department, marking which side each row came
   from (hint: check for `NULL` on one side).
4. Rewrite the `RIGHT JOIN` example as a `LEFT JOIN`.
5. Rewrite the `FULL JOIN` example using `UNION`.

If you can answer all five without looking back, you've got joins down.
