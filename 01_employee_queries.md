# Employee SQL Interview Questions

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

## 1. Find the employee(s) who earn the second highest salary

```sql
-- Solution 1: Using MAX()
SELECT *
FROM Employees
WHERE salary = (
    SELECT MAX(salary)
    FROM Employees
    WHERE salary < (
        SELECT MAX(salary)
        FROM Employees
    )
);
```

```sql
-- Solution 2: Using DENSE_RANK()
WITH SalaryRank AS (
    SELECT *,
           DENSE_RANK() OVER(ORDER BY salary DESC) AS rnk
    FROM Employees
)
SELECT *
FROM SalaryRank
WHERE rnk = 2;
```

---

## 2. Find the employee(s) who earn the third highest salary

```sql
-- Solution 1: Nested MAX()
SELECT *
FROM Employees
WHERE salary = (
    SELECT MAX(salary)
    FROM Employees
    WHERE salary < (
        SELECT MAX(salary)
        FROM Employees
        WHERE salary < (
            SELECT MAX(salary)
            FROM Employees
        )
    )
);
```

```sql
-- Solution 2: DENSE_RANK()
WITH SalaryRank AS (
    SELECT *,
           DENSE_RANK() OVER(ORDER BY salary DESC) AS rnk
    FROM Employees
)
SELECT *
FROM SalaryRank
WHERE rnk = 3;
```

---

## 3. Find employees whose salary is greater than their department average

```sql
-- Solution 1: Correlated Subquery
SELECT *
FROM Employees e
WHERE salary >
(
    SELECT AVG(salary)
    FROM Employees
    WHERE dept_id = e.dept_id
);
```

```sql
-- Solution 2: CTE
WITH DeptAvg AS (
    SELECT dept_id,
           AVG(salary) avg_salary
    FROM Employees
    GROUP BY dept_id
)
SELECT e.*
FROM Employees e
JOIN DeptAvg d
ON e.dept_id = d.dept_id
WHERE e.salary > d.avg_salary;
```

---

## 4. Retrieve the highest-paid employee from each department

```sql
-- Solution 1: Correlated Subquery
SELECT *
FROM Employees e
WHERE salary =
(
    SELECT MAX(salary)
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
               ORDER BY salary DESC
           ) rn
    FROM Employees
)
SELECT *
FROM RankedEmp
WHERE rn = 1;
```

---

## 5. Find departments having more than 5 employees

```sql
-- Solution 1
SELECT dept_id,
       COUNT(*) employee_count
FROM Employees
GROUP BY dept_id
HAVING COUNT(*) > 5;
```

```sql
-- Solution 2
WITH DeptCount AS (
    SELECT dept_id,
           COUNT(*) employee_count
    FROM Employees
    GROUP BY dept_id
)
SELECT *
FROM DeptCount
WHERE employee_count > 5;
```

---

## 6. Display total salary paid in each department

```sql
-- Solution 1
SELECT dept_id,
       SUM(salary) total_salary
FROM Employees
GROUP BY dept_id;
```

```sql
-- Solution 2
SELECT DISTINCT
       dept_id,
       SUM(salary) OVER(
           PARTITION BY dept_id
       ) total_salary
FROM Employees;
```

---

## 7. Find employees who earn more than their manager

```sql
-- Solution 1: Self Join
SELECT e.*
FROM Employees e
JOIN Employees m
ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;
```

```sql
-- Solution 2: Subquery
SELECT *
FROM Employees e
WHERE salary >
(
    SELECT salary
    FROM Employees
    WHERE emp_id = e.manager_id
);
```

---

## 8. Display employees without a manager

```sql
-- Solution 1
SELECT *
FROM Employees
WHERE manager_id IS NULL;
```

```sql
-- Solution 2
SELECT *
FROM Employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM Employees m
    WHERE m.emp_id = e.manager_id
);
```

---

## 9. Find duplicate employee names

```sql
-- Solution 1
SELECT emp_name,
       COUNT(*)
FROM Employees
GROUP BY emp_name
HAVING COUNT(*) > 1;
```

```sql
-- Solution 2
SELECT DISTINCT emp_name
FROM (
    SELECT emp_name,
           COUNT(*) OVER(
               PARTITION BY emp_name
           ) cnt
    FROM Employees
) t
WHERE cnt > 1;
```

---

## 10. Retrieve employees hired in the last 180 days

```sql
-- Solution 1 (PostgreSQL)
SELECT *
FROM Employees
WHERE hire_date >= CURRENT_DATE - INTERVAL '180 days';
```

```sql
-- Solution 2 (MySQL)
SELECT *
FROM Employees
WHERE DATEDIFF(CURDATE(), hire_date) <= 180;
```
