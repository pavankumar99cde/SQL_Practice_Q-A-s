# SCD Type 1 & Type 2 SQL Interview Questions

One of the most frequently asked Data Engineering interview topics.

Typical Tables:

```sql
Source_Customer
---------------
customer_id
customer_name
city
email
updated_date

Dim_Customer
------------
customer_sk
customer_id
customer_name
city
email
effective_start_date
effective_end_date
is_current
```

---

## 1. Implement SCD Type 1 (Overwrite Existing Record)

```sql
-- Solution 1: MERGE
MERGE INTO Dim_Customer tgt
USING Source_Customer src
ON tgt.customer_id = src.customer_id

WHEN MATCHED THEN
UPDATE SET
    tgt.customer_name = src.customer_name,
    tgt.city          = src.city,
    tgt.email         = src.email

WHEN NOT MATCHED THEN
INSERT (
    customer_id,
    customer_name,
    city,
    email
)
VALUES (
    src.customer_id,
    src.customer_name,
    src.city,
    src.email
);
```

```sql
-- Solution 2: UPDATE + INSERT

-- Update Existing
UPDATE Dim_Customer d
SET
    customer_name = s.customer_name,
    city          = s.city,
    email         = s.email
FROM Source_Customer s
WHERE d.customer_id = s.customer_id;

-- Insert New
INSERT INTO Dim_Customer
(
 customer_id,
 customer_name,
 city,
 email
)
SELECT
 customer_id,
 customer_name,
 city,
 email
FROM Source_Customer s
WHERE NOT EXISTS (
    SELECT 1
    FROM Dim_Customer d
    WHERE d.customer_id = s.customer_id
);
```

---

## 2. Implement SCD Type 2

```sql
-- Solution 1: Expire + Insert

-- Step 1 Expire old row
UPDATE Dim_Customer d
SET effective_end_date = CURRENT_DATE,
    is_current = 'N'
FROM Source_Customer s
WHERE d.customer_id = s.customer_id
AND d.is_current = 'Y'
AND (
        d.city <> s.city
     OR d.email <> s.email
);
```

```sql
-- Solution 2: Insert new version

INSERT INTO Dim_Customer
(
 customer_id,
 customer_name,
 city,
 email,
 effective_start_date,
 effective_end_date,
 is_current
)
SELECT
 s.customer_id,
 s.customer_name,
 s.city,
 s.email,
 CURRENT_DATE,
 '9999-12-31',
 'Y'
FROM Source_Customer s
JOIN Dim_Customer d
ON s.customer_id = d.customer_id
WHERE d.is_current = 'N';
```

---

## 3. Find Changed Records

```sql
-- Solution 1
SELECT s.*
FROM Source_Customer s
JOIN Dim_Customer d
ON s.customer_id = d.customer_id
WHERE d.is_current = 'Y'
AND (
        s.city <> d.city
     OR s.email <> d.email
);
```

```sql
-- Solution 2
SELECT *
FROM (
    SELECT s.customer_id,
           s.city src_city,
           d.city tgt_city,
           s.email src_email,
           d.email tgt_email
    FROM Source_Customer s
    JOIN Dim_Customer d
    ON s.customer_id = d.customer_id
) x
WHERE src_city <> tgt_city
OR src_email <> tgt_email;
```

---

## 4. Find New Customers

```sql
-- Solution 1
SELECT *
FROM Source_Customer s
WHERE NOT EXISTS (
    SELECT 1
    FROM Dim_Customer d
    WHERE d.customer_id = s.customer_id
);
```

```sql
-- Solution 2
SELECT s.*
FROM Source_Customer s
LEFT JOIN Dim_Customer d
ON s.customer_id = d.customer_id
WHERE d.customer_id IS NULL;
```

---

## 5. Find Deleted Customers

```sql
-- Solution 1
SELECT *
FROM Dim_Customer d
WHERE NOT EXISTS (
    SELECT 1
    FROM Source_Customer s
    WHERE s.customer_id = d.customer_id
);
```

```sql
-- Solution 2
SELECT d.*
FROM Dim_Customer d
LEFT JOIN Source_Customer s
ON d.customer_id = s.customer_id
WHERE s.customer_id IS NULL;
```

---

## 6. Expire Existing Records

```sql
-- Solution 1
UPDATE Dim_Customer
SET effective_end_date = CURRENT_DATE,
    is_current = 'N'
WHERE customer_id IN (
    SELECT customer_id
    FROM Source_Customer
);
```

```sql
-- Solution 2
UPDATE d
SET d.effective_end_date = CURRENT_DATE,
    d.is_current = 'N'
FROM Dim_Customer d
JOIN Source_Customer s
ON d.customer_id = s.customer_id;
```

---

## 7. Find Current Active Records

```sql
-- Solution 1
SELECT *
FROM Dim_Customer
WHERE is_current = 'Y';
```

```sql
-- Solution 2
SELECT *
FROM Dim_Customer
WHERE effective_end_date = '9999-12-31';
```

---

## 8. Retrieve Full History for Customer

```sql
-- Solution 1
SELECT *
FROM Dim_Customer
WHERE customer_id = 1001
ORDER BY effective_start_date;
```

```sql
-- Solution 2
SELECT *
FROM Dim_Customer
WHERE customer_id = 1001
ORDER BY customer_sk;
```

---

## 9. Find Latest Version of Customer

```sql
-- Solution 1
SELECT *
FROM Dim_Customer
WHERE customer_id = 1001
AND is_current = 'Y';
```

```sql
-- Solution 2
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY customer_id
               ORDER BY effective_start_date DESC
           ) rn
    FROM Dim_Customer
) x
WHERE rn = 1;
```

---

## 10. Count Versions Per Customer

```sql
-- Solution 1
SELECT customer_id,
       COUNT(*) version_count
FROM Dim_Customer
GROUP BY customer_id;
```

```sql
-- Solution 2
SELECT DISTINCT
       customer_id,
       COUNT(*) OVER(
           PARTITION BY customer_id
       ) version_count
FROM Dim_Customer;
```

---

## 11. Find Customers Changed More Than Once

```sql
-- Solution 1
SELECT customer_id
FROM Dim_Customer
GROUP BY customer_id
HAVING COUNT(*) > 2;
```

```sql
-- Solution 2
WITH VersionCount AS (
    SELECT customer_id,
           COUNT(*) cnt
    FROM Dim_Customer
    GROUP BY customer_id
)
SELECT *
FROM VersionCount
WHERE cnt > 2;
```

---

## 12. Detect Attribute-Level Changes

```sql
-- Solution 1
SELECT customer_id
FROM Source_Customer s
JOIN Dim_Customer d
ON s.customer_id = d.customer_id
WHERE s.city <> d.city;
```

```sql
-- Solution 2
SELECT customer_id
FROM Source_Customer s
JOIN Dim_Customer d
ON s.customer_id = d.customer_id
WHERE HASH(s.city,s.email)
   <> HASH(d.city,d.email);
```
