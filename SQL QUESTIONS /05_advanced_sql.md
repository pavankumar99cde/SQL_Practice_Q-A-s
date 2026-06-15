# Advanced SQL Interview Questions

Assume the following tables:

```sql
Employees
---------
emp_id      INT
emp_name    VARCHAR(100)
dept_id     INT
salary      INT
hire_date   DATE
manager_id  INT

Sales
-----
sale_id     INT
sale_date   DATE
amount      DECIMAL(10,2)

Numbers
-------
id          INT
```

---

## 41. Find duplicate records based on employee name

```sql
-- Solution 1: GROUP BY + HAVING
SELECT emp_name,
       COUNT(*) AS duplicate_count
FROM Employees
GROUP BY emp_name
HAVING COUNT(*) > 1;
```

```sql
-- Solution 2: Window Function
SELECT *
FROM (
    SELECT *,
           COUNT(*) OVER(
               PARTITION BY emp_name
           ) AS cnt
    FROM Employees
) t
WHERE cnt > 1;
```

---

## 42. Delete duplicate records while keeping the lowest emp_id

```sql
-- Solution 1: ROW_NUMBER()
WITH DuplicateRows AS (
    SELECT emp_id,
           ROW_NUMBER() OVER(
               PARTITION BY emp_name
               ORDER BY emp_id
           ) rn
    FROM Employees
)
DELETE FROM Employees
WHERE emp_id IN (
    SELECT emp_id
    FROM DuplicateRows
    WHERE rn > 1
);
```

```sql
-- Solution 2: Self Join
DELETE e1
FROM Employees e1
JOIN Employees e2
ON e1.emp_name = e2.emp_name
AND e1.emp_id > e2.emp_id;
```

---

## 43. Find missing IDs in a sequence

Example:

```text
1
2
3
5
6
9
```

Missing:

```text
4
7
8
```

```sql
-- Solution 1: Self Join
SELECT n1.id + 1 AS missing_id
FROM Numbers n1
LEFT JOIN Numbers n2
ON n1.id + 1 = n2.id
WHERE n2.id IS NULL;
```

```sql
-- Solution 2: Recursive CTE
WITH RECURSIVE Seq AS (
    SELECT MIN(id) id
    FROM Numbers

    UNION ALL

    SELECT id + 1
    FROM Seq
    WHERE id < (
        SELECT MAX(id)
        FROM Numbers
    )
)
SELECT id
FROM Seq
WHERE id NOT IN (
    SELECT id
    FROM Numbers
);
```

---

## 44. Find gaps between consecutive IDs

```sql
-- Solution 1: LEAD()
SELECT id,
       LEAD(id) OVER(
           ORDER BY id
       ) next_id
FROM Numbers
WHERE LEAD(id) OVER(
          ORDER BY id
      ) - id > 1;
```

```sql
-- Solution 2: Self Join
SELECT n1.id current_id,
       MIN(n2.id) next_id
FROM Numbers n1
JOIN Numbers n2
ON n2.id > n1.id
GROUP BY n1.id
HAVING MIN(n2.id) - n1.id > 1;
```

---

## 45. Find the Nth highest salary

```sql
-- Solution 1: DENSE_RANK()
WITH SalaryRank AS (
    SELECT *,
           DENSE_RANK() OVER(
               ORDER BY salary DESC
           ) rnk
    FROM Employees
)
SELECT *
FROM SalaryRank
WHERE rnk = N;
```

```sql
-- Solution 2: Correlated Subquery
SELECT *
FROM Employees e1
WHERE N - 1 =
(
    SELECT COUNT(DISTINCT salary)
    FROM Employees e2
    WHERE e2.salary > e1.salary
);
```

---

## 46. Convert rows into columns (Pivot)

Input:

```text
dept_id salary
1       10000
2       20000
3       30000
```

```sql
-- Solution 1: CASE WHEN
SELECT
SUM(CASE WHEN dept_id = 1 THEN salary END) Dept1,
SUM(CASE WHEN dept_id = 2 THEN salary END) Dept2,
SUM(CASE WHEN dept_id = 3 THEN salary END) Dept3
FROM Employees;
```

```sql
-- Solution 2: PIVOT (SQL Server)
SELECT *
FROM (
    SELECT dept_id,
           salary
    FROM Employees
) src
PIVOT (
    SUM(salary)
    FOR dept_id IN ([1],[2],[3])
) p;
```

---

## 47. Convert columns into rows (Unpivot)

```sql
-- Solution 1: UNION ALL
SELECT 'Dept1' dept_name,
       Dept1 salary
FROM SalaryTable

UNION ALL

SELECT 'Dept2',
       Dept2
FROM SalaryTable

UNION ALL

SELECT 'Dept3',
       Dept3
FROM SalaryTable;
```

```sql
-- Solution 2: UNPIVOT
SELECT dept_name,
       salary
FROM SalaryTable
UNPIVOT (
    salary FOR dept_name IN (
        Dept1,
        Dept2,
        Dept3
    )
) u;
```

---

## 48. Find consecutive records (3 consecutive days of sales)

```sql
-- Solution 1: LAG()
WITH SalesLag AS (
    SELECT sale_date,
           LAG(sale_date,1) OVER(ORDER BY sale_date) d1,
           LAG(sale_date,2) OVER(ORDER BY sale_date) d2
    FROM Sales
)
SELECT *
FROM SalesLag
WHERE sale_date = d1 + INTERVAL '1 day'
AND d1 = d2 + INTERVAL '1 day';
```

```sql
-- Solution 2: Self Join
SELECT s1.sale_date
FROM Sales s1
JOIN Sales s2
ON s2.sale_date = s1.sale_date - INTERVAL '1 day'
JOIN Sales s3
ON s3.sale_date = s1.sale_date - INTERVAL '2 day';
```

---

## 49. Display employee-manager hierarchy

```sql
-- Solution 1: Self Join
SELECT e.emp_name employee,
       m.emp_name manager
FROM Employees e
LEFT JOIN Employees m
ON e.manager_id = m.emp_id;
```

```sql
-- Solution 2: Recursive CTE
WITH RECURSIVE EmpHierarchy AS (
    SELECT emp_id,
           emp_name,
           manager_id,
           1 level_no
    FROM Employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT e.emp_id,
           e.emp_name,
           e.manager_id,
           h.level_no + 1
    FROM Employees e
    JOIN EmpHierarchy h
    ON e.manager_id = h.emp_id
)
SELECT *
FROM EmpHierarchy;
```

---

## 50. Find employees earning above company average salary

```sql
-- Solution 1: Subquery
SELECT *
FROM Employees
WHERE salary >
(
    SELECT AVG(salary)
    FROM Employees
);
```

```sql
-- Solution 2: Window Function
SELECT *
FROM (
    SELECT *,
           AVG(salary) OVER() avg_salary
    FROM Employees
) t
WHERE salary > avg_salary;
```

---

