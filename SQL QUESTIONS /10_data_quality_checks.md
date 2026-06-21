# Data Quality & Validation SQL Interview Questions

Data Quality questions are extremely common in Data Engineering, ETL, Snowflake, Databricks, and Data Warehouse interviews.

---

## Sample Tables
```sql
                                                                        Customers
                                                                        ---------
                                                      Orders            customer_id
                                                      ------            customer_name
 Target_Orders             Source_Orders              order_id          email
 -------------             -------------              customer_id       city
 order_id                  order_id                   amount            created_date
 customer_id               customer_id                order_date        
 amount                    amount        
```


---

## 1. Find Duplicate Records

```sql
-- Solution 1: GROUP BY
SELECT customer_id,
       COUNT(*) duplicate_count
FROM Customers
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

```sql
-- Solution 2: Window Function
SELECT *
FROM (
    SELECT *,
           COUNT(*) OVER(
               PARTITION BY customer_id
           ) cnt
    FROM Customers
) x
WHERE cnt > 1;
```

---

## 2. Find Duplicate Emails

```sql
-- Solution 1
SELECT email,
       COUNT(*)
FROM Customers
GROUP BY email
HAVING COUNT(*) > 1;
```

```sql
-- Solution 2
SELECT *
FROM Customers
WHERE email IN (
    SELECT email
    FROM Customers
    GROUP BY email
    HAVING COUNT(*) > 1
);
```

---

## 3. Find NULL Values in Important Columns

```sql
-- Solution 1
SELECT *
FROM Customers
WHERE customer_name IS NULL
OR email IS NULL;
```

```sql
-- Solution 2
SELECT COUNT(*)
FROM Customers
WHERE customer_name IS NULL
OR email IS NULL;
```

---

## 4. Find Blank Values

```sql
-- Solution 1
SELECT *
FROM Customers
WHERE TRIM(customer_name) = '';
```

```sql
-- Solution 2
SELECT *
FROM Customers
WHERE customer_name IS NULL
OR LENGTH(TRIM(customer_name)) = 0;
```

---

## 5. Validate Email Format

```sql
-- Solution 1
SELECT *
FROM Customers
WHERE email NOT LIKE '%@%.%';
```

```sql
-- Solution 2
SELECT *
FROM Customers
WHERE email !~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$';
```

---

## 6. Find Orphan Records

Orders without Customers.

```sql
-- Solution 1
SELECT *
FROM Orders o
LEFT JOIN Customers c
ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```

```sql
-- Solution 2
SELECT *
FROM Orders o
WHERE NOT EXISTS (
    SELECT 1
    FROM Customers c
    WHERE c.customer_id = o.customer_id
);
```

---

## 7. Row Count Validation

```sql
-- Solution 1
SELECT COUNT(*)
FROM Source_Orders;
```

```sql
-- Solution 2
SELECT
(
    SELECT COUNT(*) FROM Source_Orders
) source_count,
(
    SELECT COUNT(*) FROM Target_Orders
) target_count;
```

---

## 8. Source vs Target Data Reconciliation

```sql
-- Solution 1
SELECT *
FROM Source_Orders

EXCEPT

SELECT *
FROM Target_Orders;
```

```sql
-- Solution 2
SELECT s.*
FROM Source_Orders s
LEFT JOIN Target_Orders t
ON s.order_id = t.order_id
WHERE t.order_id IS NULL;
```

---

## 9. Checksum Validation

```sql
-- Solution 1
SELECT
SUM(HASH(order_id,customer_id,amount))
FROM Source_Orders;
```

```sql
-- Solution 2
SELECT
MD5(
STRING_AGG(
CAST(order_id AS VARCHAR),
''
)
)
FROM Source_Orders;
```

---

## 10. Detect Negative Amounts

```sql
-- Solution 1
SELECT *
FROM Orders
WHERE amount < 0;
```

```sql
-- Solution 2
SELECT COUNT(*)
FROM Orders
WHERE amount < 0;
```

---

## 11. Detect Outliers

```sql
-- Solution 1
SELECT *
FROM Orders
WHERE amount >
(
    SELECT AVG(amount) + 3 * STDDEV(amount)
    FROM Orders
);
```

```sql
-- Solution 2
WITH Stats AS (
    SELECT AVG(amount) avg_amt,
           STDDEV(amount) std_amt
    FROM Orders
)
SELECT o.*
FROM Orders o
CROSS JOIN Stats s
WHERE o.amount > s.avg_amt + 3*s.std_amt;
```

---

## 12. Detect Future Dates

```sql
-- Solution 1
SELECT *
FROM Orders
WHERE order_date > CURRENT_DATE;
```

```sql
-- Solution 2
SELECT COUNT(*)
FROM Orders
WHERE order_date > CURRENT_DATE;
```

---

## 13. Detect Missing Mandatory Columns

```sql
-- Solution 1
SELECT *
FROM Customers
WHERE customer_id IS NULL;
```

```sql
-- Solution 2
SELECT COUNT(*)
FROM Customers
WHERE customer_id IS NULL;
```

---

## 14. Check Uniqueness Constraint

```sql
-- Solution 1
SELECT customer_id,
       COUNT(*)
FROM Customers
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

```sql
-- Solution 2
SELECT
COUNT(*) total_rows,
COUNT(DISTINCT customer_id) unique_rows
FROM Customers;
```

---

## 15. Validate Business Rule

Amount must be > 0.

```sql
-- Solution 1
SELECT *
FROM Orders
WHERE amount <= 0;
```

```sql
-- Solution 2
SELECT COUNT(*)
FROM Orders
WHERE amount <= 0;
```

---

## 16. Detect Missing Dates

```sql
-- Solution 1
WITH RECURSIVE Dates AS (
    SELECT MIN(order_date) dt
    FROM Orders

    UNION ALL

    SELECT dt + INTERVAL '1 day'
    FROM Dates
    WHERE dt <
    (
        SELECT MAX(order_date)
        FROM Orders
    )
)
SELECT dt
FROM Dates
EXCEPT
SELECT order_date
FROM Orders;
```

```sql
-- Solution 2
SELECT c.calendar_date
FROM Calendar c
LEFT JOIN Orders o
ON c.calendar_date = o.order_date
WHERE o.order_date IS NULL;
```

---

## 17. Validate Data Completeness

```sql
-- Solution 1
SELECT COUNT(*) loaded_rows
FROM Target_Orders;
```

```sql
-- Solution 2
SELECT
(
 SELECT COUNT(*) FROM Source_Orders
)
-
(
 SELECT COUNT(*) FROM Target_Orders
) difference;
```

---

## 18. Detect Unexpected Growth

```sql
-- Solution 1
SELECT order_date,
       COUNT(*) order_count
FROM Orders
GROUP BY order_date;
```

```sql
-- Solution 2
WITH DailyCounts AS (
    SELECT order_date,
           COUNT(*) cnt
    FROM Orders
    GROUP BY order_date
)
SELECT *,
       LAG(cnt) OVER(
           ORDER BY order_date
       ) prev_cnt
FROM DailyCounts;
```

---

## 19. Detect Unexpected Null Growth

```sql
-- Solution 1
SELECT
COUNT(*) null_emails
FROM Customers
WHERE email IS NULL;
```

```sql
-- Solution 2
SELECT created_date,
       COUNT(*) null_count
FROM Customers
WHERE email IS NULL
GROUP BY created_date;
```

---

## 20. Generate Data Quality Dashboard Metrics

```sql
-- Solution 1
SELECT
COUNT(*) total_rows,
COUNT(DISTINCT customer_id) unique_customers,
SUM(
CASE
WHEN email IS NULL THEN 1
ELSE 0
END
) null_emails
FROM Customers;
```

```sql
-- Solution 2
SELECT
'Total Rows' metric,
COUNT(*) value
FROM Customers

UNION ALL

SELECT
'Unique Customers',
COUNT(DISTINCT customer_id)
FROM Customers

UNION ALL

SELECT
'Null Emails',
COUNT(*)
FROM Customers
WHERE email IS NULL;
```
