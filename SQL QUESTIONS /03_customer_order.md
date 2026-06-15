# Customer & Order SQL Interview Questions

Assume the following tables:

```sql
Customers
---------
customer_id     INT
customer_name   VARCHAR(100)
city            VARCHAR(100)

Orders
------
order_id        INT
customer_id     INT
order_date      DATE
amount          DECIMAL(10,2)
```

---

## 21. Find customers who never placed an order

```sql
-- Solution 1: LEFT JOIN
SELECT c.*
FROM Customers c
LEFT JOIN Orders o
ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

```sql
-- Solution 2: NOT EXISTS
SELECT *
FROM Customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    WHERE o.customer_id = c.customer_id
);
```

---

## 22. Find customers who placed more than 5 orders

```sql
-- Solution 1: GROUP BY
SELECT customer_id,
       COUNT(*) AS total_orders
FROM Orders
GROUP BY customer_id
HAVING COUNT(*) > 5;
```

```sql
-- Solution 2: CTE
WITH CustomerOrders AS (
    SELECT customer_id,
           COUNT(*) AS total_orders
    FROM Orders
    GROUP BY customer_id
)
SELECT *
FROM CustomerOrders
WHERE total_orders > 5;
```

---

## 23. Find the customer with the highest spending

```sql
-- Solution 1: ORDER BY
SELECT customer_id,
       SUM(amount) AS total_spent
FROM Orders
GROUP BY customer_id
ORDER BY total_spent DESC
LIMIT 1;
```

```sql
-- Solution 2: RANK()
WITH CustomerSpend AS (
    SELECT customer_id,
           SUM(amount) AS total_spent,
           RANK() OVER(
               ORDER BY SUM(amount) DESC
           ) rnk
    FROM Orders
    GROUP BY customer_id
)
SELECT *
FROM CustomerSpend
WHERE rnk = 1;
```

---

## 24. Find top 3 customers by purchase amount

```sql
-- Solution 1: ORDER BY
SELECT customer_id,
       SUM(amount) AS total_spent
FROM Orders
GROUP BY customer_id
ORDER BY total_spent DESC
LIMIT 3;
```

```sql
-- Solution 2: DENSE_RANK()
WITH CustomerSpend AS (
    SELECT customer_id,
           SUM(amount) AS total_spent,
           DENSE_RANK() OVER(
               ORDER BY SUM(amount) DESC
           ) rnk
    FROM Orders
    GROUP BY customer_id
)
SELECT *
FROM CustomerSpend
WHERE rnk <= 3;
```

---

## 25. Calculate monthly sales

```sql
-- Solution 1
SELECT DATE_TRUNC('month', order_date) AS sales_month,
       SUM(amount) AS total_sales
FROM Orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY sales_month;
```

```sql
-- Solution 2
SELECT EXTRACT(YEAR FROM order_date) AS year,
       EXTRACT(MONTH FROM order_date) AS month,
       SUM(amount) AS total_sales
FROM Orders
GROUP BY EXTRACT(YEAR FROM order_date),
         EXTRACT(MONTH FROM order_date)
ORDER BY year, month;
```

---

## 26. Calculate running total sales

```sql
-- Solution 1: Window Function
SELECT order_id,
       order_date,
       amount,
       SUM(amount) OVER(
           ORDER BY order_date
       ) AS running_total
FROM Orders;
```

```sql
-- Solution 2: Correlated Subquery
SELECT o1.order_id,
       o1.order_date,
       o1.amount,
       (
           SELECT SUM(o2.amount)
           FROM Orders o2
           WHERE o2.order_date <= o1.order_date
       ) AS running_total
FROM Orders o1;
```

---

## 27. Find average order value per customer

```sql
-- Solution 1
SELECT customer_id,
       AVG(amount) AS avg_order_value
FROM Orders
GROUP BY customer_id;
```

```sql
-- Solution 2
SELECT DISTINCT
       customer_id,
       AVG(amount) OVER(
           PARTITION BY customer_id
       ) AS avg_order_value
FROM Orders;
```

---

## 28. Retrieve the latest order for each customer

```sql
-- Solution 1
SELECT *
FROM Orders o
WHERE order_date =
(
    SELECT MAX(order_date)
    FROM Orders
    WHERE customer_id = o.customer_id
);
```

```sql
-- Solution 2
WITH LatestOrder AS (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY customer_id
               ORDER BY order_date DESC
           ) rn
    FROM Orders
)
SELECT *
FROM LatestOrder
WHERE rn = 1;
```

---

## 29. Find customers ordering on consecutive days

```sql
-- Solution 1: LAG()
WITH ConsecutiveOrders AS (
    SELECT customer_id,
           order_date,
           LAG(order_date) OVER(
               PARTITION BY customer_id
               ORDER BY order_date
           ) prev_order_date
    FROM Orders
)
SELECT DISTINCT customer_id
FROM ConsecutiveOrders
WHERE order_date = prev_order_date + INTERVAL '1 day';
```

```sql
-- Solution 2: Self Join
SELECT DISTINCT o1.customer_id
FROM Orders o1
JOIN Orders o2
ON o1.customer_id = o2.customer_id
AND o1.order_date = o2.order_date + INTERVAL '1 day';
```

---

## 30. Find customers who purchased in every month of a year

```sql
-- Solution 1
SELECT customer_id
FROM Orders
WHERE EXTRACT(YEAR FROM order_date) = 2025
GROUP BY customer_id
HAVING COUNT(DISTINCT EXTRACT(MONTH FROM order_date)) = 12;
```

```sql
-- Solution 2
WITH MonthlyOrders AS (
    SELECT customer_id,
           COUNT(DISTINCT EXTRACT(MONTH FROM order_date)) AS months_active
    FROM Orders
    WHERE EXTRACT(YEAR FROM order_date) = 2025
    GROUP BY customer_id
)
SELECT customer_id
FROM MonthlyOrders
WHERE months_active = 12;
```

---
