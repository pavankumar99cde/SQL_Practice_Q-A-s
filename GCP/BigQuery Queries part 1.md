For Bigquery interview revision
# DDL Queries

```sql
CREATE TABLE dataset.emp ( emp_id INT64, emp_name STRING, salary NUMERIC );

CREATE OR REPLACE TABLE dataset.emp ( emp_id INT64, emp_name STRING, salary NUMERIC );

CREATE TABLE dataset.emp_copy AS SELECT * FROM dataset.emp;

ALTER TABLE dataset.emp ADD COLUMN department STRING;

ALTER TABLE dataset.emp RENAME COLUMN emp_name TO employee_name;

TRUNCATE TABLE dataset.emp;

DROP TABLE dataset.emp;
```

# DML Queries

```sql
INSERT INTO dataset.emp (emp_id, emp_name, salary) VALUES (1,'John',50000);

UPDATE dataset.emp SET salary=60000 WHERE emp_id=1;

DELETE FROM dataset.emp WHERE emp_id=1;

MERGE dataset.target T USING dataset.source S
ON T.emp_id=S.emp_id WHEN MATCHED THEN
UPDATE SET salary=S.salary
WHEN NOT MATCHED THEN
INSERT (emp_id,salary) VALUES (S.emp_id,S.salary);
```


# Partitioning & Clustering

```sql
CREATE TABLE dataset.sales ( sale_id INT64, sale_date DATE, amount NUMERIC ) PARTITION BY sale_date;

CREATE TABLE dataset.sales ( sale_id INT64, sale_date DATE, amount NUMERIC, region STRING ) PARTITION BY sale_date CLUSTER BY region;
```

# Window Functions

```sql
SELECT
    *,
    ROW_NUMBER() OVER(PARTITION BY dept ORDER BY salary DESC) AS rn,
    RANK()       OVER(PARTITION BY dept ORDER BY salary DESC) AS rnk,
    DENSE_RANK() OVER(PARTITION BY dept ORDER BY salary DESC) AS drnk,
    LAG(salary)  OVER(PARTITION BY dept ORDER BY salary) AS prev_sal,
    LEAD(salary) OVER(PARTITION BY dept ORDER BY salary) AS next_sal
FROM emp
QUALIFY rn = 1;

# QUALIFY

SELECT * FROM emp QUALIFY ROW_NUMBER() OVER(PARTITION BY dept ORDER BY salary DESC)=1;
SELECT * FROM emp QUALIFY RANK() OVER(ORDER BY salary DESC)=2;
```

# Arrays

```sql
SELECT [1,2,3] arr;

SELECT ARRAY_LENGTH([1,2,3]);

SELECT * FROM UNNEST([1,2,3]); # help in flattening the data

SELECT ARRAY_AGG(emp_name) FROM emp;
```

# Structs

```sql
SELECT STRUCT(1 AS id,'John' AS name);

SELECT person.name FROM (SELECT STRUCT(1,'John') person);
```

# JSON Functions

```sql
      columns   : values
    "salary"    : 5000,
    "department": "IT",
    "skills"    : ["SQL","Python","GCP"],
    "address"   : {"city"    : "Hyderabad",
                   "country" : "India" }

-- Get Employee Name
SELECT JSON_VALUE(json_col,'$.name') AS name FROM emp; -- Output: John
-- Get Salary
SELECT JSON_VALUE(json_col,'$.salary') AS salary FROM emp; -- Output: 5000
-- Get Nested Field
SELECT JSON_VALUE(json_col,'$.address.city') AS city FROM emp; -- Output: Hyderabad
-- Get First Skill
SELECT JSON_VALUE(json_col,'$.skills[0]') AS first_skill FROM emp; -- Output: SQL
-- Get Entire Skills Array
SELECT JSON_QUERY(json_col,'$.skills') AS skills FROM emp; -- Output: ["SQL","Python","GCP"]
-- Get Skills as Array
SELECT JSON_VALUE_ARRAY(json_col,'$.skills') AS skills FROM emp; -- Output: [SQL, Python, GCP]
-- Count Number of Skills
SELECT ARRAY_LENGTH(JSON_VALUE_ARRAY(json_col,'$.skills')) AS skill_count FROM emp; -- Output: 3
-- Filter Employees with Salary > 4000
SELECT * FROM emp WHERE CAST(JSON_VALUE(json_col,'$.salary') AS INT64) > 4000;
-- Filter Employees from Hyderabad
SELECT * FROM emp WHERE JSON_VALUE(json_col,'$.address.city') = 'Hyderabad';
-- Check if JSON Path Exists
SELECT * FROM emp WHERE JSON_QUERY(json_col,'$.department') IS NOT NULL;
-- Expand Skills into Rows
SELECT skill FROM emp, UNNEST(JSON_VALUE_ARRAY(json_col,'$.skills')) AS skill;
```

# Date Functions

```sql
SELECT CURRENT_DATE();

SELECT CURRENT_TIMESTAMP();

SELECT DATE_ADD(CURRENT_DATE(),INTERVAL 7 DAY);

SELECT DATE_SUB(CURRENT_DATE(),INTERVAL 7 DAY);

SELECT DATE_DIFF(CURRENT_DATE(),'2025-01-01',DAY);

SELECT FORMAT_DATE('%Y%m%d',CURRENT_DATE());

SELECT PARSE_DATE('%Y%m%d','20250621');
```

# String Functions

```sql
SELECT                                               -- Output
    UPPER('john') AS upper_name,                     -- JOHN
    LOWER('JOHN') AS lower_name,                     -- john
    INITCAP('john doe') AS proper_name,              -- John Doe
    TRIM(' John ') AS trimmed_name,                  -- John
    LTRIM('   John') AS left_trim,                   -- John
    RTRIM('John   ') AS right_trim,                  -- John
    SUBSTR('BigQuery',1,3) AS short_name,            -- Big
    LENGTH('BigQuery') AS str_length,                -- 8
    CONCAT('Hello',' ','World') AS message,          -- Hello World
    REPLACE('abc','a','x') AS replaced_text,         -- xbc
    REVERSE('abc') AS reversed_text,                 -- cba
    LEFT('BigQuery',3) AS left_chars,                -- Big
    RIGHT('BigQuery',3) AS right_chars,              -- ery
    STARTS_WITH('BigQuery','Big') AS starts_chk,     -- TRUE
    ENDS_WITH('BigQuery','ery') AS ends_chk,         -- TRUE
    STRPOS('BigQuery','Query') AS position_val,      -- 4
    SPLIT('A,B,C',',')[OFFSET(1)] AS split_value,    -- B
    LPAD('123',5,'0') AS left_pad,                   -- 00123
    RPAD('123',5,'0') AS right_pad,                  -- 12300
    REPEAT('Hi',3) AS repeated_text,                 -- HiHiHi
    REGEXP_CONTAINS('abc123', r'\d+') AS has_digit;  -- TRUE
```



# BigQuery Metadata Queries

```sql
SELECT * FROM dataset.INFORMATION_SCHEMA.TABLES;

SELECT * FROM dataset.INFORMATION_SCHEMA.COLUMNS;

SELECT * FROM region-us.INFORMATION_SCHEMA.JOBS;

SELECT * FROM dataset.INFORMATION_SCHEMA.PARTITIONS;
```

# Authorized View

```sql
CREATE VIEW dataset.authorized_view AS SELECT emp_id,emp_name FROM dataset.emp;
```

# UDF

```sql
CREATE TEMP FUNCTION get_bonus(salary INT64) AS (salary*0.1);

SELECT get_bonus(50000);
```
# Views

```sql
CREATE VIEW dataset.v_emp AS SELECT * FROM dataset.emp;

CREATE OR REPLACE VIEW dataset.v_emp AS SELECT * FROM dataset.emp;

CREATE MATERIALIZED VIEW dataset.mv_emp AS SELECT department,COUNT(*) cnt FROM dataset.emp GROUP BY department;
```

# Stored Procedure

```sql
CREATE PROCEDURE dataset.sp_emp() BEGIN SELECT * FROM dataset.emp; END;

CALL dataset.sp_emp();
```

# BigQuery Load Commands

```bash
bq load --source_format=CSV project:dataset.table gs://bucket/file.csv schema.json

bq load --autodetect project:dataset.table gs://bucket/file.csv

bq query --use_legacy_sql=false 'SELECT * FROM dataset.emp LIMIT 10'

bq mk --dataset project:dataset

bq rm -r dataset
```

# Joins

```sql
SELECT * FROM A INNER JOIN B      ON A.id=B.id;
SELECT * FROM A LEFT  JOIN B      ON A.id=B.id;
SELECT * FROM A RIGHT JOIN B      ON A.id=B.id;
SELECT * FROM A FULL OUTER JOIN B ON A.id=B.id;
SELECT * FROM A CROSS JOIN B;
```

# Aggregate functions
```sql
SELECT
    department,
    COUNT(*) AS total_employees,
    COUNT(1) AS total_employees_count1,
    SUM(salary) AS total_salary,
    AVG(salary) AS avg_salary,
    MAX(salary) AS max_salary,
    SUM(COUNT(*)) OVER() AS overall_employee_count
FROM emp
GROUP BY department;
```



