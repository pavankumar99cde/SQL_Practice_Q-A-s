# Query Optimization & Performance Tuning SQL Interview Questions

One of the most important topics for Senior Data Engineer interviews.

---

## Sample Tables

```sql
Orders
------
order_id
customer_id
product_id
amount
order_date

Customers
---------
customer_id
customer_name
city

Products
--------
product_id
product_name
category
```

---

## 1. Query returns results slowly due to full table scan

```sql
-- Solution 1: Create Index
CREATE INDEX idx_orders_customer
ON Orders(customer_id);

SELECT *
FROM Orders
WHERE customer_id = 1001;
```

```sql
-- Solution 2: Composite Index
CREATE INDEX idx_orders_customer_date
ON Orders(customer_id, order_date);

SELECT *
FROM Orders
WHERE customer_id = 1001
AND order_date >= '2025-01-01';
```

---

## 2. Optimize a JOIN Query

```sql
-- Solution 1: Index Join Columns
CREATE INDEX idx_orders_customer
ON Orders(customer_id);

CREATE INDEX idx_customers_customer
ON Customers(customer_id);

SELECT *
FROM Orders o
JOIN Customers c
ON o.customer_id = c.customer_id;
```

```sql
-- Solution 2: Select Required Columns Only
SELECT
    o.order_id,
    o.amount,
    c.customer_name
FROM Orders o
JOIN Customers c
ON o.customer_id = c.customer_id;
```

---

## 3. Optimize GROUP BY Query

```sql
-- Solution 1: Index Grouping Column
CREATE INDEX idx_orders_product
ON Orders(product_id);

SELECT product_id,
       SUM(amount)
FROM Orders
GROUP BY product_id;
```

```sql
-- Solution 2: Pre-Aggregation
CREATE MATERIALIZED VIEW mv_product_sales AS
SELECT product_id,
       SUM(amount) total_sales
FROM Orders
GROUP BY product_id;
```

---

## 4. Avoid SELECT *

```sql
-- Solution 1: Bad
SELECT *
FROM Orders;
```

```sql
-- Solution 2: Good
SELECT
    order_id,
    amount
FROM Orders;
```

---

## 5. EXISTS vs IN

```sql
-- Solution 1: IN
SELECT *
FROM Customers
WHERE customer_id IN (
    SELECT customer_id
    FROM Orders
);
```

```sql
-- Solution 2: EXISTS
SELECT *
FROM Customers c
WHERE EXISTS (
    SELECT 1
    FROM Orders o
    WHERE o.customer_id = c.customer_id
);
```

---

## 6. Optimize DISTINCT

```sql
-- Solution 1
SELECT DISTINCT customer_id
FROM Orders;
```

```sql
-- Solution 2
SELECT customer_id
FROM Orders
GROUP BY customer_id;
```

---

## 7. Optimize Top N Query

```sql
-- Solution 1
SELECT *
FROM Orders
ORDER BY amount DESC
LIMIT 10;
```

```sql
-- Solution 2
CREATE INDEX idx_amount
ON Orders(amount DESC);

SELECT *
FROM Orders
ORDER BY amount DESC
LIMIT 10;
```

---

## 8. Optimize Date Filtering

```sql
-- Solution 1: Bad
SELECT *
FROM Orders
WHERE YEAR(order_date) = 2025;
```

```sql
-- Solution 2: Good
SELECT *
FROM Orders
WHERE order_date >= '2025-01-01'
AND order_date < '2026-01-01';
```

---

## 9. Optimize OR Conditions

```sql
-- Solution 1
SELECT *
FROM Orders
WHERE customer_id = 1001
OR customer_id = 1002;
```

```sql
-- Solution 2
SELECT *
FROM Orders
WHERE customer_id IN (1001,1002);
```

---

## 10. Optimize Large DELETE

```sql
-- Solution 1
DELETE
FROM Orders
WHERE order_date < '2020-01-01';
```

```sql
-- Solution 2: Batch Delete
DELETE
FROM Orders
WHERE order_id IN (
    SELECT order_id
    FROM Orders
    WHERE order_date < '2020-01-01'
    LIMIT 10000
);
```

---

## 11. Optimize Large UPDATE

```sql
-- Solution 1
UPDATE Orders
SET amount = amount * 1.1
WHERE product_id = 100;
```

```sql
-- Solution 2
UPDATE Orders
SET amount = amount * 1.1
WHERE product_id IN (
    SELECT product_id
    FROM Products
    WHERE category = 'Electronics'
);
```

---

## 12. Covering Index Example

```sql
-- Solution 1
CREATE INDEX idx_covering
ON Orders(customer_id, order_date, amount);
```

```sql
-- Solution 2
CREATE INDEX idx_covering2
ON Orders(order_date)
INCLUDE(amount, customer_id);
```

---

## 13. Optimize Window Function Query

```sql
-- Solution 1
SELECT *,
       ROW_NUMBER() OVER(
           PARTITION BY customer_id
           ORDER BY order_date DESC
       )
FROM Orders;
```

```sql
-- Solution 2
CREATE INDEX idx_window
ON Orders(customer_id, order_date DESC);
```

---

## 14. Remove Duplicate Rows Efficiently

```sql
-- Solution 1
WITH cte AS (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY customer_id,
                            order_date
               ORDER BY order_id
           ) rn
    FROM Orders
)
DELETE
FROM cte
WHERE rn > 1;
```

```sql
-- Solution 2
CREATE TABLE Orders_New AS
SELECT DISTINCT *
FROM Orders;
```

---

## 15. Partition Large Tables

```sql
-- Solution 1: Range Partition

CREATE TABLE Orders (
    order_id INT,
    order_date DATE
)
PARTITION BY RANGE(order_date);
```

```sql
-- Solution 2: Hash Partition

CREATE TABLE Orders (
    order_id INT,
    customer_id INT
)
PARTITION BY HASH(customer_id);
```

---

## Common Theory Questions

### 1. Clustered vs Non-Clustered Index

| Clustered | Non-Clustered |
|------------|--------------|
| Data physically sorted | Separate structure |
| One per table | Multiple allowed |
| Faster range scans | Faster point lookups |

---

### 2. Partitioning vs Indexing

| Partitioning | Indexing |
|-------------|----------|
| Splits data | Speeds lookups |
| Improves manageability | Improves query speed |
| Good for huge tables | Good for filters |

---

### 3. WHERE vs HAVING

| WHERE | HAVING |
|---------|--------|
| Filters before grouping | Filters after grouping |
| Faster | Usually slower |

---

### 4. UNION vs UNION ALL

| UNION | UNION ALL |
|---------|----------|
| Removes duplicates | Keeps duplicates |
| Slower | Faster |

---

### 5. DELETE vs TRUNCATE

| DELETE | TRUNCATE |
|---------|----------|
| Logged row by row | Minimal logging |
| Can use WHERE | No WHERE |
| Slower | Faster |

---

