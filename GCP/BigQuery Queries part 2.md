# Frequently Asked in Senior Interviews

### CTE

```sql
WITH emp_cte AS ( SELECT * FROM dataset.emp ) SELECT * FROM emp_cte;
```

### Recursive CTE

```sql
WITH RECURSIVE nums AS ( SELECT 1 n UNION ALL SELECT n+1 FROM nums WHERE n<10 ) SELECT * FROM nums;
```

### Temporary Table

```sql
CREATE TEMP TABLE temp_emp AS SELECT * FROM dataset.emp;
```

### Temporary Function

```sql
CREATE TEMP FUNCTION full_name(f STRING,l STRING) AS ( CONCAT(f,' ',l) );

SELECT full_name('John','Doe');
```

### Variables

```sql
DECLARE v_count INT64;

SET v_count = 100;

SELECT v_count;
```

### IF Condition

```sql
DECLARE salary INT64 DEFAULT 50000;
IF salary > 40000 THEN SELECT 'High Salary';
ELSE  SELECT 'Low Salary';
END IF;
```

### CASE Statement

```sql
SELECT emp_name, CASE WHEN salary > 100000 THEN 'HIGH'
                      WHEN salary > 50000 THEN 'MEDIUM'
                 ELSE 'LOW' END AS salary_band
       FROM emp;
```

### Loops

```sql
DECLARE i INT64 DEFAULT 1;

WHILE i <= 5 DO
  SELECT i;
  SET i = i + 1;
END WHILE;
```

### Dynamic SQL

```sql
EXECUTE IMMEDIATE "SELECT * FROM dataset.emp";
```

### Dynamic SQL with Variable

```sql
DECLARE table_name STRING DEFAULT 'emp';

EXECUTE IMMEDIATE FORMAT("SELECT * FROM dataset.%s",table_name);
```

### Exception Handling

```sql
BEGIN
  SELECT 1/0;
EXCEPTION WHEN ERROR THEN
  SELECT 'Error Handled';
END;
```

### Transactions

```sql
BEGIN TRANSACTION;

INSERT INTO dataset.emp VALUES(1,'John',50000);

COMMIT TRANSACTION;
```

### Rollback

```sql
BEGIN TRANSACTION;

INSERT INTO dataset.emp VALUES(1,'John',50000);

ROLLBACK TRANSACTION;
```

### Pivot

```sql
SELECT * FROM (SELECT department,salary FROM emp)
PIVOT(SUM(salary) FOR department IN ('IT','HR','FINANCE'));
```

### Unpivot

```sql
SELECT * FROM emp
UNPIVOT (salary FOR quarter IN (q1,q2,q3,q4));
```

### Generate Series

```sql
SELECT * FROM UNNEST(GENERATE_ARRAY(1,10));

SELECT * FROM UNNEST(GENERATE_DATE_ARRAY('2025-01-01','2025-01-10'));
```

### Hash Functions

```sql
SELECT MD5('hello');

SELECT SHA1('hello');

SELECT SHA256('hello');
```

### Safe Functions

```sql
SELECT SAFE_CAST('123' AS INT64);

SELECT SAFE_DIVIDE(10,0);

SELECT SAFE_OFFSET(5);
```

### Approximate Functions

```sql
SELECT APPROX_COUNT_DISTINCT(emp_id) FROM emp;

SELECT APPROX_TOP_COUNT(emp_name,5) FROM emp;
```

### Sampling

```sql
SELECT * FROM emp TABLESAMPLE SYSTEM (10 PERCENT);
```

### Time Travel

```sql
SELECT * FROM dataset.emp FOR SYSTEM_TIME AS OF TIMESTAMP_SUB(CURRENT_TIMESTAMP(),INTERVAL 1 HOUR);
```

### Snapshot Table

```sql
CREATE SNAPSHOT TABLE dataset.emp_snapshot CLONE dataset.emp;
```

### Clone Table

```sql
CREATE TABLE dataset.emp_clone CLONE dataset.emp;
```

### Export Data

```sql
EXPORT DATA OPTIONS(uri='gs://bucket/output/*.csv',format='CSV',overwrite=true) AS SELECT * FROM dataset.emp;
```

### Load External Table

```sql
CREATE EXTERNAL TABLE dataset.ext_emp OPTIONS(format='CSV',uris=['gs://bucket/*.csv']);
```

### BigLake Table

```sql
CREATE EXTERNAL TABLE dataset.biglake_emp WITH CONNECTION `project.region.connection` OPTIONS(format='PARQUET',uris=['gs://bucket/*.parquet']);
```

### Search Index

```sql
CREATE SEARCH INDEX idx_emp_name ON dataset.emp(emp_name);
```

### Vector Index (GenAI)

```sql
CREATE VECTOR INDEX idx_embedding ON dataset.embeddings(embedding);
```

### Row Access Policy

```sql
CREATE ROW ACCESS POLICY emp_policy ON dataset.emp GRANT TO ("user:user@gmail.com") FILTER USING (department='HR');
```

### Column Level Security (Policy Tags)

```sql
ALTER TABLE dataset.emp ALTER COLUMN salary SET POLICY TAG 'projects/project/taxonomies/taxonomy/policyTags/tag';
```

### Table Functions

```sql
CREATE TABLE FUNCTION dataset.fn_emp(dept STRING) AS ( SELECT * FROM dataset.emp WHERE department=dept );

SELECT * FROM dataset.fn_emp('IT');
```

### Audit Query

```sql
SELECT * FROM region-us.INFORMATION_SCHEMA.JOBS_BY_PROJECT;
```

### Query Execution Plan

```sql
EXPLAIN SELECT * FROM dataset.emp WHERE emp_id=100;
```

# Session

```sql
CREATE TEMP TABLE session_table AS SELECT * FROM dataset.emp;
```

### Common Bash Commands

```bash
bq ls
bq ls project_id
bq show project:dataset.table
bq head dataset.table
bq cp dataset.table dataset.table_backup
bq extract dataset.table gs://bucket/output.csv
bq update dataset.table
bq query --dry_run --use_legacy_sql=false 'SELECT * FROM dataset.emp'
bq query --nouse_cache --use_legacy_sql=false 'SELECT * FROM dataset.emp'
bq mk --transfer_config
bq cancel JOB_ID
```
For interviews, you don't need every obscure `bq` option. Focus on the **20-30 commands used 95% of the time**.

# Dataset Commands

```bash
bq ls
bq ls project_id
bq mk --dataset project_id:dataset_name
bq show project_id:dataset_name
bq update project_id:dataset_name
bq rm -r -f project_id:dataset_name
```

# Table Commands

```bash
bq mk --table dataset.table id:INT64,name:STRING
bq show dataset.table
bq head dataset.table
bq cp dataset.table dataset.table_backup
bq cp -f dataset.table dataset.table_backup
bq rm -f dataset.table
bq update dataset.table
bq extract dataset.table gs://bucket/output.csv
bq extract --destination_format=PARQUET dataset.table gs://bucket/output.parquet
```

# Query Commands

```bash
bq query 'SELECT * FROM dataset.emp'
bq query --use_legacy_sql=false 'SELECT * FROM dataset.emp'
bq query --dry_run --use_legacy_sql=false 'SELECT * FROM dataset.emp'
bq query --nouse_cache --use_legacy_sql=false 'SELECT * FROM dataset.emp'
bq query --max_rows=100 'SELECT * FROM dataset.emp'
bq query --format=prettyjson 'SELECT * FROM dataset.emp'
bq query --destination_table=dataset.result_table 'SELECT * FROM dataset.emp'
bq query --replace --destination_table=dataset.result_table 'SELECT * FROM dataset.emp'
```

# Load Commands

```bash
bq load dataset.emp gs://bucket/file.csv
bq load --autodetect dataset.emp gs://bucket/file.csv
bq load --source_format=CSV dataset.emp gs://bucket/file.csv schema.json
bq load --source_format=PARQUET dataset.emp gs://bucket/file.parquet
bq load --source_format=AVRO dataset.emp gs://bucket/file.avro
bq load --skip_leading_rows=1 dataset.emp gs://bucket/file.csv schema.json
bq load --replace dataset.emp gs://bucket/file.csv schema.json
bq load --allow_quoted_newlines dataset.emp gs://bucket/file.csv schema.json
bq load --allow_jagged_rows dataset.emp gs://bucket/file.csv schema.json
bq load --max_bad_records=10 dataset.emp gs://bucket/file.csv schema.json
```

# Job Commands

```bash
bq ls -j
bq ls -j -a
bq show -j JOB_ID
bq wait JOB_ID
bq cancel JOB_ID
```

# Routine / Procedure Commands

```bash
bq show dataset.procedure_name
bq show dataset.function_name
```

# Transfer Service Commands

```bash
bq mk --transfer_config
bq ls --transfer_config
bq show --transfer_config TRANSFER_ID
bq update --transfer_config TRANSFER_ID
bq rm --transfer_config TRANSFER_ID
```

# IAM / Access Commands

```bash
bq show --format=prettyjson dataset

bq update --description "sales dataset" dataset
```

# Common Formatting Options

```bash
--format=pretty
--format=prettyjson
--format=json
--format=csv
```

# Common Query Options

```bash
--use_legacy_sql=false
--dry_run
--nouse_cache
--max_rows=100
--destination_table
--replace
--append_table
```

# Common Load Options

```bash
--autodetect
--replace
--source_format=CSV
--source_format=PARQUET
--source_format=AVRO
--source_format=JSON
--skip_leading_rows=1
--max_bad_records=10
--allow_jagged_rows
--allow_quoted_newlines
```

# Common Extract Options

```bash
--destination_format=CSV
--destination_format=PARQUET
--destination_format=AVRO
--compression=GZIP
```

# Most Asked Interview Commands

```bash
bq ls
bq show
bq mk --dataset
bq mk --table
bq load
bq query
bq cp
bq extract
bq rm
bq head
bq ls -j
bq show -j
bq cancel
bq wait
```


