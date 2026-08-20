# TCR 01 — SQL

> The largest and most reliably scoreable chunk of the TCR section. SQL questions have crisp, defensible answers — which is exactly what the reasoning box rewards.

**Sample tables used throughout:**
```
Employees(emp_id, name, salary, dept_id, manager_id, hire_date)
Departments(dept_id, dept_name, location)
```

---

## 1. The order SQL actually executes (this explains most trick questions)

You **write**:
```
SELECT -> FROM -> WHERE -> GROUP BY -> HAVING -> ORDER BY -> LIMIT
```
The database **executes**:
```
FROM -> JOIN -> WHERE -> GROUP BY -> HAVING -> SELECT -> DISTINCT -> ORDER BY -> LIMIT
```

**Three consequences that are asked constantly:**
1. **You cannot use a `SELECT` alias in `WHERE`** — `WHERE` runs before `SELECT` exists.
   `SELECT salary*12 AS annual FROM emp WHERE annual > 100000;` -> **error**.
2. **You *can* use an alias in `ORDER BY`** — it runs after `SELECT`. (And in `GROUP BY` in MySQL/PostgreSQL, though not in every dialect.)
3. **`WHERE` filters rows before grouping; `HAVING` filters groups after aggregation.** So an aggregate like `COUNT(*)` can appear in `HAVING` but never in `WHERE`.

---

## 2. WHERE vs HAVING — the #1 exam question

| | WHERE | HAVING |
|---|---|---|
| Filters | individual **rows** | **groups** |
| Runs | before `GROUP BY` | after `GROUP BY` |
| Aggregates allowed? | **No** | **Yes** |
| Without GROUP BY | normal filter | treats the whole table as one group |

```sql
-- Departments where the average salary of employees hired after 2020 exceeds 50000
SELECT dept_id, AVG(salary) AS avg_sal
FROM Employees
WHERE hire_date > '2020-01-01'     -- filters ROWS first
GROUP BY dept_id
HAVING AVG(salary) > 50000;        -- then filters the resulting GROUPS
```
**Reasoning template:** *"WHERE is evaluated before rows are grouped, so it cannot reference aggregate values; HAVING is evaluated after grouping and therefore can. Filtering with WHERE first is also more efficient, since fewer rows reach the grouping stage."*

---

## 3. JOINS — know the exact result of each

```
Employees                     Departments
emp_id  name   dept_id        dept_id  dept_name
1       A      10             10       Sales
2       B      20             20       IT
3       C      NULL           30       HR
```

| Join | Returns | Result on the data above |
|---|---|---|
| `INNER JOIN` | only matching rows on both sides | A, B |
| `LEFT JOIN` | all left rows + matches (NULLs where none) | A, B, **C (NULL dept)** |
| `RIGHT JOIN` | all right rows + matches | A, B, **HR (NULL emp)** |
| `FULL OUTER JOIN` | everything from both sides | A, B, C, HR |
| `CROSS JOIN` | Cartesian product, m x n rows | 3 x 3 = 9 rows |
| `SELF JOIN` | a table joined to itself | employee/manager pairs |

```sql
SELECT e.name, d.dept_name
FROM Employees e
LEFT JOIN Departments d ON e.dept_id = d.dept_id;
```

### Self join — employees with their managers
```sql
SELECT e.name AS employee, m.name AS manager
FROM Employees e
LEFT JOIN Employees m ON e.manager_id = m.emp_id;
```
`LEFT` matters here: with `INNER`, the CEO (whose `manager_id` is NULL) disappears from the results. This is a favourite "why is a row missing?" question.

### Find rows with NO match (anti-join)
```sql
-- Employees not assigned to any department
SELECT e.name
FROM Employees e
LEFT JOIN Departments d ON e.dept_id = d.dept_id
WHERE d.dept_id IS NULL;             -- the LEFT JOIN + IS NULL idiom
```

### The trap: filter in WHERE vs in ON, for outer joins
```sql
-- A: LEFT JOIN behaves like an INNER JOIN, because the WHERE kills the NULL rows
SELECT e.name, d.dept_name
FROM Employees e LEFT JOIN Departments d ON e.dept_id = d.dept_id
WHERE d.location = 'NY';

-- B: keeps ALL employees, matching only NY departments
SELECT e.name, d.dept_name
FROM Employees e LEFT JOIN Departments d ON e.dept_id = d.dept_id AND d.location = 'NY';
```
**Reasoning:** *"In an outer join, a predicate in ON restricts what is matched, while a predicate in WHERE filters the joined result. Since unmatched rows carry NULLs, a WHERE condition on the right table discards them and silently converts the outer join into an inner join."* Learning this one distinction is worth several marks.

---

## 4. Aggregate functions and NULL handling

```sql
COUNT(*)          -- counts ROWS, including those with NULLs
COUNT(column)     -- counts NON-NULL values in that column
COUNT(DISTINCT c) -- counts distinct non-null values
SUM, AVG, MIN, MAX
```
**Critical NULL rules — expect at least one question:**
- Aggregates **ignore NULLs** (except `COUNT(*)`). `AVG(salary)` over `[100, NULL, 200]` is **150**, not 100.
- `NULL = NULL` is **not true** — it is *unknown*. Test with `IS NULL` / `IS NOT NULL`.
- Any arithmetic with NULL yields NULL: `100 + NULL = NULL`.
- `COALESCE(col, 0)` substitutes a default; `IFNULL`/`NVL` are the dialect-specific equivalents.
- `NULL` values are excluded by `NOT IN` in a surprising way: `WHERE x NOT IN (1, 2, NULL)` returns **no rows at all**, because the comparison against NULL is unknown. Use `NOT EXISTS` instead.

```sql
SELECT dept_id, COUNT(*) AS headcount, AVG(COALESCE(salary,0)) AS avg_sal
FROM Employees
GROUP BY dept_id;
```

**GROUP BY rule:** every non-aggregated column in `SELECT` must appear in `GROUP BY`. (MySQL historically allowed violations; standard SQL and `ONLY_FULL_GROUP_BY` mode reject them because the value chosen would be arbitrary.)

---

## 5. Subqueries

```sql
-- Scalar subquery: employees earning above the company average
SELECT name, salary FROM Employees
WHERE salary > (SELECT AVG(salary) FROM Employees);

-- IN: employees in NY departments
SELECT name FROM Employees
WHERE dept_id IN (SELECT dept_id FROM Departments WHERE location = 'NY');

-- Correlated subquery: employees earning more than their OWN department average
SELECT e.name, e.salary FROM Employees e
WHERE e.salary > (SELECT AVG(salary) FROM Employees WHERE dept_id = e.dept_id);

-- EXISTS: departments that have at least one employee
SELECT d.dept_name FROM Departments d
WHERE EXISTS (SELECT 1 FROM Employees e WHERE e.dept_id = d.dept_id);

-- Derived table (subquery in FROM)
SELECT dept_id, avg_sal FROM
  (SELECT dept_id, AVG(salary) AS avg_sal FROM Employees GROUP BY dept_id) t
WHERE avg_sal > 50000;
```

| | Non-correlated | Correlated |
|---|---|---|
| Depends on the outer query? | No | Yes (references an outer column) |
| Executed | once | conceptually once per outer row |
| Typical performance | faster | slower, though optimisers often rewrite it |

**`IN` vs `EXISTS`:** `EXISTS` stops at the first match and is safer with NULLs; `IN` materialises the full list. For large subquery results `EXISTS` usually wins; for small static lists `IN` is fine and clearer.

---

## 6. The Nth highest salary — asked in almost every written round

```sql
-- Method 1: LIMIT / OFFSET (MySQL, PostgreSQL) — 2nd highest
SELECT DISTINCT salary FROM Employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- Method 2: subquery — the highest salary below the maximum
SELECT MAX(salary) FROM Employees
WHERE salary < (SELECT MAX(salary) FROM Employees);

-- Method 3: correlated subquery — general Nth highest (N-1 salaries above it)
SELECT DISTINCT e1.salary FROM Employees e1
WHERE (N-1) = (SELECT COUNT(DISTINCT e2.salary) FROM Employees e2
               WHERE e2.salary > e1.salary);

-- Method 4: window function — the modern answer
SELECT salary FROM (
  SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
  FROM Employees
) t WHERE rnk = 2;
```
**Why `DENSE_RANK` and not `RANK`?** With salaries 100, 100, 90: `RANK` gives 1, 1, **3** (it skips), so "rank = 2" returns nothing. `DENSE_RANK` gives 1, 1, **2**, which is what "second highest distinct salary" means. Being able to explain this difference is a high-value TCR answer.

---

## 7. Window functions

```sql
SELECT name, dept_id, salary,
       ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn,
       RANK()       OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk,
       DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS drnk,
       AVG(salary)  OVER (PARTITION BY dept_id) AS dept_avg,
       LAG(salary)  OVER (ORDER BY hire_date)   AS prev_salary,
       LEAD(salary) OVER (ORDER BY hire_date)   AS next_salary
FROM Employees;
```

| Function | Values 100, 100, 90 |
|---|---|
| `ROW_NUMBER()` | 1, 2, 3 (always unique, ties broken arbitrarily) |
| `RANK()` | 1, 1, 3 (skips after a tie) |
| `DENSE_RANK()` | 1, 1, 2 (no gaps) |

**The key property:** unlike `GROUP BY`, a window function **does not collapse rows**. Every row is kept and gains an extra computed column. That single sentence answers most window-function MCQs.

```sql
-- Top earner per department
SELECT * FROM (
  SELECT name, dept_id, salary,
         ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn
  FROM Employees
) t WHERE rn = 1;

-- Running total
SELECT name, salary,
       SUM(salary) OVER (ORDER BY hire_date
                         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM Employees;
```

---

## 8. Set operations

| Operator | Behaviour |
|---|---|
| `UNION` | combines and **removes duplicates** (so it sorts/hashes — slower) |
| `UNION ALL` | combines and **keeps duplicates** — faster; prefer it when you know rows are distinct |
| `INTERSECT` | rows present in both |
| `EXCEPT` / `MINUS` | rows in the first but not the second |

All require the same number of columns with compatible types.

---

## 9. Keys and constraints

| Key | Meaning |
|---|---|
| **Super key** | any set of attributes that uniquely identifies a row |
| **Candidate key** | a minimal super key (no removable attribute) |
| **Primary key** | the chosen candidate key. **Unique + NOT NULL**, one per table |
| **Alternate key** | the candidate keys not chosen as primary |
| **Composite key** | a primary key made of two or more columns |
| **Foreign key** | references a primary key in another table; enforces **referential integrity**. Can be NULL, and can repeat |
| **Unique key** | enforces uniqueness but **allows NULLs** (usually one) and can exist multiple times per table |

**Primary key vs Unique key — a guaranteed question:** *"A primary key must be NOT NULL and there can be only one per table; a unique key permits NULLs and a table may have several. Both enforce uniqueness, but only the primary key serves as the row's identifying reference for foreign keys."*

**Constraints:** `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`.

```sql
CREATE TABLE Employees (
    emp_id     INT PRIMARY KEY,
    name       VARCHAR(50) NOT NULL,
    salary     DECIMAL(10,2) CHECK (salary > 0),
    dept_id    INT,
    hire_date  DATE DEFAULT (CURRENT_DATE),
    FOREIGN KEY (dept_id) REFERENCES Departments(dept_id)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);
```
**Referential actions:** `CASCADE` (propagate the delete/update), `SET NULL`, `RESTRICT`/`NO ACTION` (block it), `SET DEFAULT`.

---

## 10. SQL command categories

| Category | Commands | Auto-commit? |
|---|---|---|
| **DDL** — Data Definition | `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME` | Yes (implicit commit) |
| **DML** — Data Manipulation | `INSERT`, `UPDATE`, `DELETE`, (`SELECT`) | No |
| **DCL** — Data Control | `GRANT`, `REVOKE` | Yes |
| **TCL** — Transaction Control | `COMMIT`, `ROLLBACK`, `SAVEPOINT` | — |

### DELETE vs TRUNCATE vs DROP — asked constantly

| | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| Type | DML | DDL | DDL |
| Removes | selected rows (`WHERE`) | **all** rows | the whole table (structure included) |
| `WHERE` clause | Yes | No | No |
| Rollback-able | **Yes** | No (implicit commit) | No |
| Speed | slow (row-by-row, logged) | fast (deallocates pages) | fast |
| Fires triggers | Yes | No | No |
| Resets identity/auto-increment | No | Yes | n/a |
| Table remains | Yes | Yes (empty) | **No** |

**Reasoning template:** *"TRUNCATE is DDL: it deallocates the data pages rather than logging individual row deletions, which makes it much faster but non-rollbackable and unable to fire row triggers. DELETE is DML, logs each row, honours WHERE, and can be rolled back."*

---

## 11. Normalisation

**Purpose:** eliminate redundancy and the update/insert/delete anomalies it causes.

| Form | Rule | Violation looks like |
|---|---|---|
| **1NF** | atomic values only; no repeating groups or multi-valued cells | `phone = '111, 222'` |
| **2NF** | 1NF **+ no partial dependency** — no non-key attribute depends on only *part* of a composite key | key `(student_id, course_id)`, but `student_name` depends on `student_id` alone |
| **3NF** | 2NF **+ no transitive dependency** — non-key attributes must not depend on other non-key attributes | `emp_id -> dept_id -> dept_name`, so `dept_name` should move to its own table |
| **BCNF** | 3NF **+ every determinant is a candidate key** | a stricter 3NF; handles rare overlapping-candidate-key cases |
| **4NF** | BCNF + no multi-valued dependencies | |

**The one-line memory hook:** *"The key (1NF), the whole key (2NF), and nothing but the key (3NF) — so help me Codd."*

**Denormalisation** deliberately reintroduces redundancy to avoid expensive joins in read-heavy systems. It trades write complexity and storage for read speed — a very common "which would you choose?" TCR question. The answer is always about the read/write ratio.

---

## 12. Transactions and ACID

A **transaction** is a unit of work that succeeds or fails as a whole.

| Property | Meaning | Guaranteed by |
|---|---|---|
| **Atomicity** | all operations commit, or none do | undo log / rollback |
| **Consistency** | the database moves from one valid state to another, respecting all constraints | constraints + application logic |
| **Isolation** | concurrent transactions do not corrupt each other | locking / MVCC |
| **Durability** | once committed, it survives a crash | write-ahead log, flush to disk |

```sql
START TRANSACTION;
  UPDATE Accounts SET balance = balance - 500 WHERE id = 1;
  UPDATE Accounts SET balance = balance + 500 WHERE id = 2;
COMMIT;          -- or ROLLBACK to undo everything
```
**The canonical example:** a bank transfer. Atomicity is what prevents money leaving one account without arriving in the other.

### Concurrency problems

| Problem | What happens |
|---|---|
| **Dirty read** | you read data another transaction wrote but has not committed (it may roll back) |
| **Non-repeatable read** | you read the same **row** twice and get different values, because another transaction updated it in between |
| **Phantom read** | you run the same **query** twice and get a different **set of rows**, because another transaction inserted or deleted rows |
| **Lost update** | two transactions read then write the same row; one silently overwrites the other |

| Isolation level | Dirty read | Non-repeatable read | Phantom read |
|---|---|---|---|
| READ UNCOMMITTED | Possible | Possible | Possible |
| READ COMMITTED | Prevented | Possible | Possible |
| REPEATABLE READ | Prevented | Prevented | Possible (see note) |
| SERIALIZABLE | Prevented | Prevented | Prevented |

Note: MySQL InnoDB blocks most phantoms at REPEATABLE READ via next-key locking, but the SQL standard permits them at that level.

**Reasoning template:** *"Higher isolation eliminates more anomalies but reduces concurrency, because locks are held longer. READ COMMITTED is a common default since it prevents dirty reads at modest cost; SERIALIZABLE guarantees correctness but has the lowest throughput."*

---

## 13. Indexes

An index is an auxiliary structure (usually a **B+ tree**) that lets the engine find rows without scanning the whole table.

```sql
CREATE INDEX idx_emp_dept ON Employees(dept_id);
CREATE UNIQUE INDEX idx_emp_email ON Employees(email);
CREATE INDEX idx_composite ON Employees(dept_id, salary);     -- column order matters
DROP INDEX idx_emp_dept ON Employees;
```

| | Clustered index | Non-clustered index |
|---|---|---|
| Determines physical row order? | **Yes** | No |
| Number per table | **one** | many |
| Leaf node holds | the actual data row | a key/pointer to the row |
| Speed | slightly faster (no extra lookup) | needs a second lookup to fetch the row |
| Usually | the primary key | secondary indexes |

**The trade-off, stated for the reasoning box:** *"An index turns a full table scan into a B+ tree lookup, roughly O(n) to O(log n), but every INSERT, UPDATE and DELETE must also maintain the index and it consumes storage. Indexes therefore help read-heavy workloads and hurt write-heavy ones."*

**When an index is NOT used (classic MCQ material):**
- The column is wrapped in a function: `WHERE YEAR(hire_date) = 2023`. Rewrite as a range: `WHERE hire_date >= '2023-01-01' AND hire_date < '2024-01-01'`.
- A leading wildcard: `WHERE name LIKE '%son'` cannot use a B-tree prefix; `LIKE 'son%'` can.
- The column has low cardinality (a yes/no flag) — a scan is cheaper than the lookup.
- On a composite index `(a, b)`, a query filtering only on `b` cannot use it. **Leftmost prefix rule:** `(a)` and `(a,b)` work, `(b)` alone does not.
- The query would return most of the table anyway — the optimiser prefers a sequential scan.

---

## 14. Views, stored procedures, triggers

```sql
-- VIEW: a stored query; a virtual table with no data of its own
CREATE VIEW HighEarners AS
SELECT emp_id, name, salary FROM Employees WHERE salary > 100000;
```
Views provide **security** (expose only chosen columns/rows), **simplicity** (hide complex joins), and **abstraction** (the underlying schema can change without breaking callers). A **materialised view** does store its results and must be refreshed — faster reads, possibly stale data.

```sql
-- STORED PROCEDURE: precompiled reusable SQL stored in the database
CREATE PROCEDURE GetDeptEmployees(IN dept INT)
BEGIN
    SELECT * FROM Employees WHERE dept_id = dept;
END;
CALL GetDeptEmployees(10);
```
Benefits: precompiled execution plan, fewer network round trips, centralised logic, and parameterisation that helps prevent SQL injection.

```sql
-- TRIGGER: fires automatically in response to a data event
CREATE TRIGGER audit_salary
AFTER UPDATE ON Employees
FOR EACH ROW
BEGIN
    INSERT INTO SalaryAudit(emp_id, old_sal, new_sal, changed_at)
    VALUES (OLD.emp_id, OLD.salary, NEW.salary, NOW());
END;
```
Triggers combine `BEFORE`/`AFTER` with `INSERT`/`UPDATE`/`DELETE`.
**Procedure vs trigger:** a procedure is invoked explicitly with `CALL`; a trigger fires implicitly when the data event occurs.

---

## 15. Queries you should be able to write on sight

```sql
-- Find duplicate values
SELECT email, COUNT(*) FROM Employees GROUP BY email HAVING COUNT(*) > 1;

-- Delete duplicates, keeping the lowest id
DELETE e1 FROM Employees e1
JOIN Employees e2 ON e1.email = e2.email AND e1.emp_id > e2.emp_id;

-- Department-wise employee count, including empty departments
SELECT d.dept_name, COUNT(e.emp_id) AS cnt
FROM Departments d
LEFT JOIN Employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_name;
-- NOTE: COUNT(e.emp_id), not COUNT(*). COUNT(*) would return 1 for an empty
-- department, because the LEFT JOIN emits one all-NULL row for it.

-- Employees earning more than their manager
SELECT e.name FROM Employees e
JOIN Employees m ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;

-- Top 3 salaries per department
SELECT * FROM (
  SELECT name, dept_id, salary,
         DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS r
  FROM Employees
) t WHERE r <= 3;

-- Copy structure and data into a new table
CREATE TABLE EmpBackup AS SELECT * FROM Employees;

-- Conditional aggregation (a pivot)
SELECT dept_id,
       SUM(CASE WHEN salary >  50000 THEN 1 ELSE 0 END) AS high_earners,
       SUM(CASE WHEN salary <= 50000 THEN 1 ELSE 0 END) AS others
FROM Employees GROUP BY dept_id;
```

---

## 16. Rapid-fire facts

- `DISTINCT` applies to the **entire selected row**, not just the first column.
- `ORDER BY` defaults to **ASC**; where NULLs sort is dialect-dependent.
- `LIKE`: `%` matches any sequence of characters, `_` matches exactly one.
- `BETWEEN a AND b` is **inclusive** of both endpoints.
- `CHAR(n)` is fixed length and space-padded; `VARCHAR(n)` is variable length and stores only what you use.
- SQL is **declarative** — you state *what* you want and the optimiser decides *how*.
- **SQL injection** is prevented by **parameterised queries / prepared statements**, not by hand-escaping strings.
- A `JOIN` with no `ON` clause degenerates into a `CROSS JOIN` — the usual cause of an accidental million-row result.
- `UPDATE` or `DELETE` without a `WHERE` clause affects **every row**. Write the `WHERE` first, then the rest.
- **OLTP** = many small transactions on a normalised schema (order entry). **OLAP** = few large analytical queries on a denormalised star schema (reporting warehouse).
- **SQL vs NoSQL:** relational databases give a fixed schema, joins and strong ACID guarantees; NoSQL stores (document, key-value, column, graph) give flexible schemas and horizontal scaling, often with eventual consistency. Choose SQL for structured, relational, transactional data; NoSQL for high-volume, schema-fluid, horizontally-scaled workloads.

---

## 17. Reasoning phrases you can reuse verbatim

- *"WHERE filters rows before aggregation while HAVING filters groups after it, which is why aggregate functions are valid only in HAVING."*
- *"A LEFT JOIN preserves every row of the left table, filling unmatched columns with NULL; a WHERE condition on the right table then discards those rows, effectively converting it into an inner join."*
- *"An index converts a full table scan into a B+ tree lookup, improving read performance at the cost of slower writes and additional storage."*
- *"Normalisation removes redundancy and update anomalies but adds joins; denormalisation trades storage and write consistency for read speed, so the right choice depends on the read/write ratio."*
- *"TRUNCATE is DDL and deallocates data pages without logging individual rows, so it is faster than DELETE but cannot be rolled back or fire row-level triggers."*
- *"DENSE_RANK is correct here because RANK skips numbers after ties, so filtering on rank = N can return no rows when duplicate values exist."*
- *"Isolation levels trade correctness against concurrency: stricter levels hold locks longer and reduce throughput."*
- *"A primary key is NOT NULL and unique with one per table, whereas a unique key allows NULLs and a table may have several; only the primary key identifies rows for foreign key references."*

---

## 18. Self test

1. Can you use a `SELECT` alias in `WHERE`? In `ORDER BY`? -> *No; yes. WHERE runs before SELECT, ORDER BY after.*
2. `COUNT(*)` vs `COUNT(col)`? -> *Rows including NULLs, versus non-NULL values in that column.*
3. Why does `WHERE x NOT IN (1, 2, NULL)` return nothing? -> *The comparison against NULL evaluates to UNKNOWN, so no row can satisfy it. Use NOT EXISTS.*
4. Difference between `UNION` and `UNION ALL`? -> *UNION removes duplicates (and pays a sort/hash cost); UNION ALL keeps them and is faster.*
5. What does 3NF add over 2NF? -> *It eliminates transitive dependencies — non-key attributes must not determine other non-key attributes.*
6. Which ACID property does a rollback implement? -> *Atomicity.*
7. Why would an index be ignored for `WHERE UPPER(name) = 'X'`? -> *Wrapping the column in a function prevents the engine from matching the stored index keys; a functional/expression index would be required.*
8. When does a LEFT JOIN behave like an INNER JOIN? -> *When a WHERE clause filters on a column of the right table, discarding the NULL-extended unmatched rows.*

Next: [02_OOPS_THEORY.md](02_OOPS_THEORY.md)
