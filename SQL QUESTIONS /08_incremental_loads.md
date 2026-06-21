# Incremental Loads & CDC SQL Interview Questions

One of the most important topics for Data Engineers.

Typical Tables:

```sql
Source_Orders
-------------
order_id
customer_id
amount
created_date
updated_date

Target_Orders
-------------
order_id
customer_id
amount
created_date
updated_date

ETL_Audit
---------
job_name
last_successful_run
```

---

## 1. Load records created after the last successful run

```sql
-- Solution 1: Using Audit Table
SELECT *
FROM Source_Orders
WHERE created_date >
(
    SELECT MAX(last_successful_run)
    FROM ETL_Audit
    WHERE job_name = 'ORDER_LOAD'
);
```

```sql
-- Solution 2: Using Watermark Variable
SELECT *
FROM Source_Orders
WHERE created_date > :last_run_timestamp;
```

---

## 2. Load records updated after the last successful run

```sql
-- Solution 1
SELECT *
FROM Source_Orders
WHERE updated_date >
(
    SELECT MAX(last_successful_run)
    FROM ETL_Audit
    WHERE job_name = 'ORDER_LOAD'
);
```

```sql
-- Solution 2
SELECT *
FROM Source_Orders
WHERE updated_date > :watermark;
```

---

## 3. Find newly inserted records

```sql
-- Solution 1: NOT EXISTS
SELECT *
FROM Source_Orders s
WHERE NOT EXISTS (
    SELECT 1
    FROM Target_Orders t
    WHERE t.order_id = s.order_id
);
```

```sql
-- Solution 2: LEFT JOIN
SELECT s.*
FROM Source_Orders s
LEFT JOIN Target_Orders t
ON s.order_id = t.order_id
WHERE t.order_id IS NULL;
```

---

## 4. Find updated records

```sql
-- Solution 1
SELECT s.*
FROM Source_Orders s
JOIN Target_Orders t
ON s.order_id = t.order_id
WHERE s.updated_date > t.updated_date;
```

```sql
-- Solution 2
SELECT s.*
FROM Source_Orders s
JOIN Target_Orders t
ON s.order_id = t.order_id
WHERE HASH(
      s.customer_id,
      s.amount
      )
<>
HASH(
     t.customer_id,
     t.amount
     );
```

---

## 5. Find deleted records

```sql
-- Solution 1
SELECT *
FROM Target_Orders t
WHERE NOT EXISTS (
    SELECT 1
    FROM Source_Orders s
    WHERE s.order_id = t.order_id
);
```

```sql
-- Solution 2
SELECT t.*
FROM Target_Orders t
LEFT JOIN Source_Orders s
ON t.order_id = s.order_id
WHERE s.order_id IS NULL;
```

---

## 6. Upsert Data Using MERGE

```sql
-- Solution 1
MERGE INTO Target_Orders t
USING Source_Orders s
ON t.order_id = s.order_id

WHEN MATCHED THEN
UPDATE SET
    amount = s.amount,
    updated_date = s.updated_date

WHEN NOT MATCHED THEN
INSERT (
    order_id,
    customer_id,
    amount,
    created_date,
    updated_date
)
VALUES (
    s.order_id,
    s.customer_id,
    s.amount,
    s.created_date,
    s.updated_date
);
```

```sql
-- Solution 2

-- Update Existing
UPDATE Target_Orders t
SET amount = s.amount,
    updated_date = s.updated_date
FROM Source_Orders s
WHERE t.order_id = s.order_id;

-- Insert New
INSERT INTO Target_Orders
SELECT *
FROM Source_Orders s
WHERE NOT EXISTS (
    SELECT 1
    FROM Target_Orders t
    WHERE t.order_id = s.order_id
);
```

---

## 7. Maintain Watermark Table

```sql
-- Solution 1
INSERT INTO ETL_Audit
(
 job_name,
 last_successful_run
)
VALUES
(
 'ORDER_LOAD',
 CURRENT_TIMESTAMP
);
```

```sql
-- Solution 2
UPDATE ETL_Audit
SET last_successful_run = CURRENT_TIMESTAMP
WHERE job_name = 'ORDER_LOAD';
```

---

## 8. Load Only Latest Version of Records

```sql
-- Solution 1
WITH LatestOrder AS (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY order_id
               ORDER BY updated_date DESC
           ) rn
    FROM Source_Orders
)
SELECT *
FROM LatestOrder
WHERE rn = 1;
```

```sql
-- Solution 2
SELECT *
FROM Source_Orders s
WHERE updated_date =
(
    SELECT MAX(updated_date)
    FROM Source_Orders s2
    WHERE s2.order_id = s.order_id
);
```

---

## 9. Incremental Load Using Created and Updated Dates

```sql
-- Solution 1
SELECT *
FROM Source_Orders
WHERE created_date > :watermark
OR updated_date > :watermark;
```

```sql
-- Solution 2
SELECT *
FROM Source_Orders
WHERE GREATEST(
      created_date,
      updated_date
      ) > :watermark;
```

---

## 10. Capture CDC Using Timestamp

```sql
-- Solution 1
SELECT *
FROM Source_Orders
WHERE updated_date > :last_run;
```

```sql
-- Solution 2
SELECT *
FROM Source_Orders
WHERE updated_date BETWEEN
      :last_run
      AND
      :current_run;
```

---

## 11. Capture CDC Using Version Number

```sql
-- Solution 1
SELECT *
FROM Source_Orders
WHERE version_number > :last_version;
```

```sql
-- Solution 2
SELECT *
FROM Source_Orders
WHERE version_number BETWEEN
      :last_version + 1
      AND
      :current_version;
```

---

## 12. Detect Source-Target Differences

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
FULL OUTER JOIN Target_Orders t
ON s.order_id = t.order_id
WHERE s.amount <> t.amount
OR t.order_id IS NULL
OR s.order_id IS NULL;
```

---

## 13. Count Delta Records

```sql
-- Solution 1
SELECT COUNT(*)
FROM Source_Orders
WHERE updated_date > :watermark;
```

```sql
-- Solution 2
SELECT COUNT(*)
FROM (
    SELECT *
    FROM Source_Orders

    EXCEPT

    SELECT *
    FROM Target_Orders
) x;
```

---

## 14. Build Restartable ETL Logic

```sql
-- Solution 1
SELECT *
FROM Source_Orders
WHERE updated_date >
(
    SELECT last_successful_run
    FROM ETL_Audit
    WHERE job_name = 'ORDER_LOAD'
);
```

```sql
-- Solution 2
SELECT *
FROM Source_Orders
WHERE updated_date > :saved_checkpoint;
```

---

## 15. Handle Late Arriving Records

```sql
-- Solution 1
SELECT *
FROM Source_Orders
WHERE created_date < CURRENT_DATE - INTERVAL '1 day'
AND load_date = CURRENT_DATE;
```

```sql
-- Solution 2
SELECT *
FROM Source_Orders
WHERE event_date < load_date;
```
