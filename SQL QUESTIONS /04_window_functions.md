# Window Functions SQL Interview Questions

Assume the following table:

```sql
Employees
---------
emp_id      INT
emp_name    VARCHAR(100)
dept_id     INT
salary      INT
hire_date   DATE

Orders
------
order_id        INT
customer_id     INT
order_date      DATE
amount          DECIMAL(10,2)
```

---

## 31. Generate row numbers for employees ordered by salary

```sql
-- Solution 1: ROW_NUMBER()
SELECT emp_id,
       emp_name,
       salary,
       ROW_NUMBER() OVER(
           ORDER BY salary DESC
       ) AS row_num
FROM Employees;
```

```sql
-- Solution 2: Correlated Subquery
SELECT e1.emp_id,
       e1.emp_name,
       e1.salary,
       (
           SELECT COUNT(*)
           FROM Employees e2
           WHERE e2.salary > e1.salary
       ) + 1 AS row_num
FROM Employees e1;
```

---

## 32. Rank employees by salary

```sql
-- Solution 1: RANK()
SELECT emp_id,
       emp_name,
       salary,
       RANK() OVER(
           ORDER BY salary DESC
       ) AS salary_rank
FROM Employees;
```

```sql
-- Solution 2: Correlated Subquery
SELECT e1.emp_id,
       e1.emp_name,
       e1.salary,
       (
           SELECT COUNT(DISTINCT e2.salary)
           FROM Employees e2
           WHERE e2.salary > e1.salary
       ) + 1 AS salary_rank
FROM Employees e1;
```

---

## 33. Generate dense ranking for employees

```sql
-- Solution 1: DENSE_RANK()
SELECT emp_id,
       emp_name,
       salary,
       DENSE_RANK() OVER(
           ORDER BY salary DESC
       ) AS dense_rank_no
FROM Employees;
```

```sql
-- Solution 2
SELECT e1.emp_id,
       e1.emp_name,
       e1.salary,
       (
           SELECT COUNT(DISTINCT e2.salary)
           FROM Employees e2
           WHERE e2.salary > e1.salary
       ) + 1 AS dense_rank_no
FROM Employees e1;
```

---

## 34. Find previous employee salary

```sql
-- Solution 1: LAG()
SELECT emp_id,
       emp_name,
       salary,
       LAG(salary) OVER(
           ORDER BY salary
       ) AS previous_salary
FROM Employees;
```

```sql
-- Solution 2: Self Join
SELECT e1.emp_id,
       e1.emp_name,
       e1.salary,
       MAX(e2.salary) AS previous_salary
FROM Employees e1
LEFT JOIN Employees e2
ON e2.salary < e1.salary
GROUP BY e1.emp_id,
         e1.emp_name,
         e1.salary;
```

---

## 35. Find next employee salary

```sql
-- Solution 1: LEAD()
SELECT emp_id,
       emp_name,
       salary,
       LEAD(salary) OVER(
           ORDER BY salary
       ) AS next_salary
FROM Employees;
```

```sql
-- Solution 2: Self Join
SELECT e1.emp_id,
       e1.emp_name,
       e1.salary,
       MIN(e2.salary) AS next_salary
FROM Employees e1
LEFT JOIN Employees e2
ON e2.salary > e1.salary
GROUP BY e1.emp_id,
         e1.emp_name,
         e1.salary;
```

---

## 36. Find first hired employee in each department

```sql
-- Solution 1: FIRST_VALUE()
SELECT DISTINCT
       dept_id,
       FIRST_VALUE(emp_name) OVER(
           PARTITION BY dept_id
           ORDER BY hire_date
       ) AS first_employee
FROM Employees;
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
SELECT dept_id,
       emp_name
FROM RankedEmp
WHERE rn = 1;
```

---

## 37. Find latest hired employee in each department

```sql
-- Solution 1: LAST_VALUE()
SELECT DISTINCT
       dept_id,
       LAST_VALUE(emp_name) OVER(
           PARTITION BY dept_id
           ORDER BY hire_date
           ROWS BETWEEN UNBOUNDED PRECEDING
           AND UNBOUNDED FOLLOWING
       ) AS latest_employee
FROM Employees;
```

```sql
-- Solution 2: ROW_NUMBER()
WITH RankedEmp AS (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY dept_id
               ORDER BY hire_date DESC
           ) rn
    FROM Employees
)
SELECT dept_id,
       emp_name
FROM RankedEmp
WHERE rn = 1;
```

---

## 38. Calculate cumulative sales

```sql
-- Solution 1
SELECT order_date,
       amount,
       SUM(amount) OVER(
           ORDER BY order_date
       ) cumulative_sales
FROM Orders;
```

```sql
-- Solution 2
SELECT o1.order_date,
       o1.amount,
       (
           SELECT SUM(o2.amount)
           FROM Orders o2
           WHERE o2.order_date <= o1.order_date
       ) cumulative_sales
FROM Orders o1;
```

---

## 39. Calculate rolling 7-day average sales

```sql
-- Solution 1
SELECT order_date,
       AVG(amount) OVER(
           ORDER BY order_date
           ROWS BETWEEN 6 PRECEDING
           AND CURRENT ROW
       ) rolling_avg
FROM Orders;
```

```sql
-- Solution 2
SELECT o1.order_date,
       (
           SELECT AVG(o2.amount)
           FROM Orders o2
           WHERE o2.order_date
                 BETWEEN o1.order_date - INTERVAL '6 day'
                 AND o1.order_date
       ) rolling_avg
FROM Orders o1;
```

---

## 40. Calculate month-over-month sales growth

```sql
-- Solution 1: LAG()
WITH MonthlySales AS (
    SELECT DATE_TRUNC('month', order_date) month,
           SUM(amount) total_sales
    FROM Orders
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT month,
       total_sales,
       LAG(total_sales) OVER(
           ORDER BY month
       ) previous_month_sales,
       ROUND(
           (
             total_sales -
             LAG(total_sales) OVER(ORDER BY month)
           ) * 100.0
           /
           LAG(total_sales) OVER(ORDER BY month),
           2
       ) growth_pct
FROM MonthlySales;
```

```sql
-- Solution 2: Self Join
WITH MonthlySales AS (
    SELECT DATE_TRUNC('month', order_date) month,
           SUM(amount) total_sales
    FROM Orders
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT cur.month,
       cur.total_sales,
       prev.total_sales previous_month_sales,
       ROUND(
           (cur.total_sales - prev.total_sales)
           * 100.0 / prev.total_sales,
           2
       ) growth_pct
FROM MonthlySales cur
LEFT JOIN MonthlySales prev
ON prev.month = cur.month - INTERVAL '1 month';
```

---
