# Advanced Employee SQL Interview Questions

Assume the following table:

```sql
Employees
---------
emp_id      INT
emp_name    VARCHAR(100)
dept_id     INT
salary      INT
hire_date   DATE
manager_id  INT
```

---

## 11. Find the top 3 highest-paid employees from each department

```sql
-- Solution 1: DENSE_RANK()
WITH RankedEmp AS (
    SELECT *,
           DENSE_RANK() OVER(
               PARTITION BY dept_id
               ORDER BY salary DESC
           ) rnk
    FROM Employees
)
SELECT *
FROM RankedEmp
WHERE rnk <= 3;
```

```sql
-- Solution 2: ROW_NUMBER()
WITH RankedEmp AS (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY dept_id
               ORDER BY salary DESC
           ) rn
    FROM Employees
)
SELECT *
FROM RankedEmp
WHERE rn <= 3;
```

---

## 12. Rank employees by salary within each department

```sql
-- Solution 1: RANK()
SELECT emp_id,
       emp_name,
       dept_id,
       salary,
       RANK() OVER(
           PARTITION BY dept_id
           ORDER BY salary DESC
       ) AS rank_no
FROM Employees;
```

```sql
-- Solution 2: DENSE_RANK()
SELECT emp_id,
       emp_name,
       dept_id,
       salary,
       DENSE_RANK() OVER(
           PARTITION BY dept_id
           ORDER BY salary DESC
       ) AS rank_no
FROM Employees;
```

---

## 13. Find employees sharing the same salary

```sql
-- Solution 1: GROUP BY
SELECT salary,
       COUNT(*)
FROM Employees
GROUP BY salary
HAVING COUNT(*) > 1;
```

```sql
-- Solution 2: Window Function
SELECT *
FROM (
    SELECT *,
           COUNT(*) OVER(
               PARTITION BY salary
           ) cnt
    FROM Employees
) t
WHERE cnt > 1;
```

---

## 14. Calculate salary difference from department average

```sql
-- Solution 1: Correlated Subquery
SELECT emp_id,
       emp_name,
       salary,
       salary -
       (
           SELECT AVG(salary)
           FROM Employees e2
           WHERE e2.dept_id = e1.dept_id
       ) AS salary_diff
FROM Employees e1;
```

```sql
-- Solution 2: Window Function
SELECT emp_id,
       emp_name,
       salary,
       salary -
       AVG(salary) OVER(
           PARTITION BY dept_id
       ) AS salary_diff
FROM Employees;
```

---

## 15. Find the earliest hired employee in each department

```sql
-- Solution 1: Correlated Subquery
SELECT *
FROM Employees e
WHERE hire_date =
(
    SELECT MIN(hire_date)
    FROM Employees
    WHERE dept_id = e.dept_id
);
```

```sql
-- Solution 2: ROW_NUMBER()
WITH RankedEmp AS (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY dept_id
               ORDER BY hire_date
           ) rn
    FROM Employees
)
SELECT *
FROM RankedEmp
WHERE rn = 1;
```

---

## 16. Find departments where average salary exceeds ₹50,000

```sql
-- Solution 1
SELECT dept_id,
       AVG(salary) avg_salary
FROM Employees
GROUP BY dept_id
HAVING AVG(salary) > 50000;
```

```sql
-- Solution 2
WITH DeptAvg AS (
    SELECT dept_id,
           AVG(salary) avg_salary
    FROM Employees
    GROUP BY dept_id
)
SELECT *
FROM DeptAvg
WHERE avg_salary > 50000;
```

---

## 17. Find employees belonging to departments where nobody earns more than ₹100,000

```sql
-- Solution 1
SELECT *
FROM Employees
WHERE dept_id IN (
    SELECT dept_id
    FROM Employees
    GROUP BY dept_id
    HAVING MAX(salary) <= 100000
);
```

```sql
-- Solution 2
WITH DeptMax AS (
    SELECT dept_id,
           MAX(salary) max_salary
    FROM Employees
    GROUP BY dept_id
)
SELECT e.*
FROM Employees e
JOIN DeptMax d
ON e.dept_id = d.dept_id
WHERE d.max_salary <= 100000;
```

---

## 18. Find employees having the same manager as employee 101

```sql
-- Solution 1
SELECT *
FROM Employees
WHERE manager_id =
(
    SELECT manager_id
    FROM Employees
    WHERE emp_id = 101
);
```

```sql
-- Solution 2
SELECT e1.*
FROM Employees e1
JOIN Employees e2
ON e1.manager_id = e2.manager_id
WHERE e2.emp_id = 101;
```

---

## 19. Display count of employees hired each year

```sql
-- Solution 1
SELECT EXTRACT(YEAR FROM hire_date) hire_year,
       COUNT(*) employee_count
FROM Employees
GROUP BY EXTRACT(YEAR FROM hire_date)
ORDER BY hire_year;
```

```sql
-- Solution 2
SELECT YEAR(hire_date) hire_year,
       COUNT(*) employee_count
FROM Employees
GROUP BY YEAR(hire_date)
ORDER BY hire_year;
```

---

## 20. Find employees whose salary is among the top 10% salaries

```sql
-- Solution 1: NTILE()
WITH SalaryBucket AS (
    SELECT *,
           NTILE(10) OVER(
               ORDER BY salary DESC
           ) bucket
    FROM Employees
)
SELECT *
FROM SalaryBucket
WHERE bucket = 1;
```

```sql
-- Solution 2: Percent Rank
WITH SalaryRank AS (
    SELECT *,
           PERCENT_RANK() OVER(
               ORDER BY salary
           ) pct_rank
    FROM Employees
)
SELECT *
FROM SalaryRank
WHERE pct_rank >= 0.90;
```

---
