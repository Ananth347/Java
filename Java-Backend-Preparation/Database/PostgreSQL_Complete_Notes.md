# PostgreSQL & Database Complete Notes
### From Basics to Advanced — A Full Learning Roadmap

---

## Table of Contents

1. [Introduction to Databases](#1-introduction-to-databases)
2. [Database Design Fundamentals](#2-database-design-fundamentals)
3. [Normalization & Denormalization](#3-normalization--denormalization)
4. [Getting Started with PostgreSQL](#4-getting-started-with-postgresql)
5. [PostgreSQL Data Types](#5-postgresql-data-types)
6. [Database & Table DDL](#6-database--table-ddl)
7. [CRUD Operations (DML)](#7-crud-operations-dml)
8. [Filtering, Operators & Pattern Matching](#8-filtering-operators--pattern-matching)
9. [Sorting, Limiting & Pagination](#9-sorting-limiting--pagination)
10. [Joins](#10-joins)
11. [Aggregate Functions, GROUP BY & HAVING](#11-aggregate-functions-group-by--having)
12. [Subqueries](#12-subqueries)
13. [Common Table Expressions (CTEs) & Recursive Queries](#13-common-table-expressions-ctes--recursive-queries)
14. [Set Operations](#14-set-operations)
15. [Views & Materialized Views](#15-views--materialized-views)
16. [Keys & Constraints (Deep Dive)](#16-keys--constraints-deep-dive)
17. [Indexes](#17-indexes)
18. [Transactions & ACID](#18-transactions--acid)
19. [Concurrency Control, Locking & Isolation Levels](#19-concurrency-control-locking--isolation-levels)
20. [Functions & Stored Procedures (PL/pgSQL)](#20-functions--stored-procedures-plpgsql)
21. [Triggers](#21-triggers)
22. [Window Functions](#22-window-functions)
23. [JSON & JSONB](#23-json--jsonb)
24. [Arrays, Ranges & Composite Types](#24-arrays-ranges--composite-types)
25. [Full-Text Search](#25-full-text-search)
26. [Partitioning](#26-partitioning)
27. [Table Inheritance & Foreign Data Wrappers](#27-table-inheritance--foreign-data-wrappers)
28. [Performance Tuning & Query Optimization](#28-performance-tuning--query-optimization)
29. [Security: Roles, Privileges & Row-Level Security](#29-security-roles-privileges--row-level-security)
30. [Backup, Restore & Import/Export](#30-backup-restore--importexport)
31. [Replication & High Availability](#31-replication--high-availability)
32. [Extensions](#32-extensions)
33. [Connection Pooling](#33-connection-pooling)
34. [Monitoring, Logging & Maintenance](#34-monitoring-logging--maintenance)
35. [PostgreSQL with Application Code](#35-postgresql-with-application-code)
36. [Best Practices & Cheat Sheet](#36-best-practices--cheat-sheet)

---

## 1. Introduction to Databases

### 1.1 What is a Database?
A **database** is an organized collection of structured data stored electronically, designed for efficient storage, retrieval, and management.

### 1.2 What is a DBMS?
A **Database Management System (DBMS)** is software that lets users create, read, update, delete, and manage data in a database (e.g., PostgreSQL, MySQL, Oracle, SQL Server).

### 1.3 RDBMS
A **Relational Database Management System** stores data in tables (relations) made of rows and columns, and enforces relationships between tables. PostgreSQL is an **Object-Relational DBMS (ORDBMS)**.

### 1.4 Types of Databases
- **Relational (SQL):** PostgreSQL, MySQL, Oracle, SQL Server
- **Non-Relational (NoSQL):** MongoDB (document), Redis (key-value), Cassandra (column), Neo4j (graph)
- **NewSQL:** CockroachDB, Google Spanner

### 1.5 SQL vs NoSQL
| Aspect | SQL | NoSQL |
|---|---|---|
| Schema | Fixed/rigid | Flexible/dynamic |
| Scaling | Vertical (mostly) | Horizontal |
| Consistency | Strong (ACID) | Often eventual (BASE) |
| Use case | Structured, relational data | Unstructured/semi-structured, huge scale |

### 1.6 Why PostgreSQL?
Open-source, standards-compliant, extensible, supports JSON, full-text search, geospatial data (PostGIS), advanced indexing, and strong ACID guarantees — often called "the world's most advanced open source database."

---

## 2. Database Design Fundamentals

### 2.1 Entities, Attributes, Relationships
- **Entity:** A real-world object (e.g., Student, Order).
- **Attribute:** A property of an entity (e.g., name, age).
- **Relationship:** An association between entities (e.g., Student *enrolls in* Course).

### 2.2 Entity-Relationship (ER) Diagrams
Visual representation of entities, attributes, and relationships using ER notation (crow's foot, Chen notation).

### 2.3 Relationship Cardinality
- **One-to-One (1:1):** One row in Table A relates to exactly one row in Table B.
- **One-to-Many (1:N):** One row in Table A relates to many rows in Table B.
- **Many-to-Many (M:N):** Requires a junction/bridge table.

### 2.4 Schema Design
- **Conceptual schema:** High-level entities and relationships.
- **Logical schema:** Tables, columns, keys, without vendor-specific detail.
- **Physical schema:** Actual implementation (data types, indexes, partitions) in PostgreSQL.

---

## 3. Normalization & Denormalization

### 3.1 Why Normalize?
Reduce data redundancy and avoid update/insert/delete anomalies.

### 3.2 Normal Forms
- **1NF:** Atomic columns, no repeating groups.
- **2NF:** 1NF + no partial dependency on a composite key.
- **3NF:** 2NF + no transitive dependency (non-key attributes depend only on the key).
- **BCNF (Boyce-Codd):** Every determinant is a candidate key.
- **4NF:** No multi-valued dependencies.
- **5NF:** No join dependency anomalies (rarely needed in practice).

### 3.3 Denormalization
Deliberately introducing redundancy (e.g., duplicating a column, pre-aggregating) to improve read performance at the cost of write complexity — common in reporting/analytics systems.

---

## 4. Getting Started with PostgreSQL

### 4.1 Installation
- Linux: `sudo apt install postgresql postgresql-contrib`
- macOS: `brew install postgresql`
- Windows: official installer from postgresql.org
- Docker: `docker run --name pg -e POSTGRES_PASSWORD=pass -p 5432:5432 -d postgres`

### 4.2 Connecting via `psql`
```bash
psql -U postgres -h localhost -d mydatabase
```

### 4.3 Useful psql Meta-Commands
```
\l          -- list databases
\c dbname   -- connect to a database
\dt         -- list tables
\d table    -- describe table
\du         -- list roles/users
\dn         -- list schemas
\di         -- list indexes
\x          -- toggle expanded display
\q          -- quit
```

### 4.4 PostgreSQL Architecture Overview
- **Postmaster process:** Main server process managing connections.
- **Backend processes:** One per client connection.
- **Shared memory & buffers:** Shared buffer cache, WAL buffers.
- **WAL (Write-Ahead Log):** Ensures durability by logging changes before applying them.
- **Background workers:** Autovacuum, checkpointer, background writer, WAL writer.

### 4.5 Databases, Schemas, Tables Hierarchy
```
Cluster (server instance)
 └── Database
      └── Schema (e.g., public)
           └── Table / View / Function / Sequence
```

---

## 5. PostgreSQL Data Types

### 5.1 Numeric
`SMALLINT`, `INTEGER`, `BIGINT`, `DECIMAL`/`NUMERIC(p,s)`, `REAL`, `DOUBLE PRECISION`, `SERIAL`, `BIGSERIAL`, `SMALLSERIAL`

### 5.2 Character
`CHAR(n)`, `VARCHAR(n)`, `TEXT`

### 5.3 Date/Time
`DATE`, `TIME`, `TIMETZ`, `TIMESTAMP`, `TIMESTAMPTZ`, `INTERVAL`

### 5.4 Boolean
`BOOLEAN` (`true`/`false`/`null`)

### 5.5 Binary
`BYTEA`

### 5.6 UUID
`UUID` (often generated with `gen_random_uuid()` from `pgcrypto`, or `uuid-ossp`)

### 5.7 Special / Advanced Types
- `JSON` and `JSONB` (binary JSON, indexable)
- `ARRAY` (e.g., `INTEGER[]`, `TEXT[]`)
- `HSTORE` (key-value pairs)
- Range types: `INT4RANGE`, `NUMRANGE`, `TSRANGE`, `DATERANGE`
- `ENUM` (custom enumerated type)
- Geometric types: `POINT`, `LINE`, `POLYGON`, `CIRCLE`
- Network address types: `INET`, `CIDR`, `MACADDR`
- `XML`
- Composite types (user-defined row types)

### 5.8 Type Casting
```sql
SELECT '123'::INTEGER;
SELECT CAST('2024-01-01' AS DATE);
```

---

## 6. Database & Table DDL

### 6.1 Database Operations
```sql
CREATE DATABASE company_db;
DROP DATABASE company_db;
ALTER DATABASE company_db RENAME TO company_new;
```

### 6.2 Schema Operations
```sql
CREATE SCHEMA hr;
SET search_path TO hr, public;
DROP SCHEMA hr CASCADE;
```

### 6.3 Table Creation
```sql
CREATE TABLE employees (
    emp_id      SERIAL PRIMARY KEY,
    first_name  VARCHAR(50) NOT NULL,
    last_name   VARCHAR(50) NOT NULL,
    email       VARCHAR(100) UNIQUE,
    salary      NUMERIC(10,2) CHECK (salary > 0),
    dept_id     INTEGER REFERENCES departments(dept_id),
    hired_at    TIMESTAMPTZ DEFAULT now()
);
```

### 6.4 Altering Tables
```sql
ALTER TABLE employees ADD COLUMN phone VARCHAR(15);
ALTER TABLE employees DROP COLUMN phone;
ALTER TABLE employees ALTER COLUMN salary SET NOT NULL;
ALTER TABLE employees RENAME COLUMN email TO email_address;
ALTER TABLE employees RENAME TO staff;
```

### 6.5 Dropping / Truncating
```sql
DROP TABLE employees;
DROP TABLE IF EXISTS employees CASCADE;
TRUNCATE TABLE employees RESTART IDENTITY;
```

### 6.6 Sequences
```sql
CREATE SEQUENCE order_seq START 1 INCREMENT 1;
SELECT nextval('order_seq');
```

---

## 7. CRUD Operations (DML)

### 7.1 INSERT
```sql
INSERT INTO employees (first_name, last_name, salary)
VALUES ('Alice', 'Smith', 55000);

INSERT INTO employees (first_name, last_name, salary)
VALUES ('Bob', 'Jones', 60000)
RETURNING emp_id;
```

### 7.2 SELECT
```sql
SELECT * FROM employees;
SELECT first_name, salary FROM employees WHERE salary > 50000;
```

### 7.3 UPDATE
```sql
UPDATE employees SET salary = salary * 1.10 WHERE dept_id = 3;
```

### 7.4 DELETE
```sql
DELETE FROM employees WHERE emp_id = 10;
```

### 7.5 UPSERT (INSERT ... ON CONFLICT)
```sql
INSERT INTO employees (emp_id, email)
VALUES (1, 'alice@example.com')
ON CONFLICT (emp_id)
DO UPDATE SET email = EXCLUDED.email;
```

---

## 8. Filtering, Operators & Pattern Matching

### 8.1 Comparison Operators
`=`, `!=` / `<>`, `<`, `>`, `<=`, `>=`

### 8.2 Logical Operators
`AND`, `OR`, `NOT`

### 8.3 Range & Set Operators
```sql
SELECT * FROM employees WHERE salary BETWEEN 40000 AND 70000;
SELECT * FROM employees WHERE dept_id IN (1, 2, 3);
SELECT * FROM employees WHERE email IS NULL;
SELECT * FROM employees WHERE email IS NOT NULL;
```

### 8.4 Pattern Matching
```sql
SELECT * FROM employees WHERE first_name LIKE 'A%';   -- case-sensitive
SELECT * FROM employees WHERE first_name ILIKE 'a%';  -- case-insensitive
SELECT * FROM employees WHERE first_name ~ '^A.*n$';  -- regex match
SELECT * FROM employees WHERE first_name SIMILAR TO '(A|B)%';
```

### 8.5 DISTINCT
```sql
SELECT DISTINCT dept_id FROM employees;
SELECT DISTINCT ON (dept_id) * FROM employees ORDER BY dept_id, salary DESC;
```

---

## 9. Sorting, Limiting & Pagination

```sql
SELECT * FROM employees ORDER BY salary DESC, last_name ASC;
SELECT * FROM employees ORDER BY salary DESC LIMIT 10 OFFSET 20;
SELECT * FROM employees ORDER BY emp_id FETCH FIRST 5 ROWS ONLY;
```
> For large-offset pagination, prefer **keyset pagination** (`WHERE emp_id > last_seen_id`) over `OFFSET` for performance.

---

## 10. Joins

### 10.1 INNER JOIN
```sql
SELECT e.first_name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
```

### 10.2 LEFT (OUTER) JOIN
```sql
SELECT e.first_name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

### 10.3 RIGHT (OUTER) JOIN
```sql
SELECT e.first_name, d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;
```

### 10.4 FULL OUTER JOIN
```sql
SELECT e.first_name, d.dept_name
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.dept_id;
```

### 10.5 CROSS JOIN
```sql
SELECT * FROM employees CROSS JOIN departments;
```

### 10.6 SELF JOIN
```sql
SELECT a.first_name AS employee, b.first_name AS manager
FROM employees a
JOIN employees b ON a.manager_id = b.emp_id;
```

### 10.7 LATERAL JOIN
```sql
SELECT d.dept_name, top_emp.first_name
FROM departments d,
LATERAL (
    SELECT first_name FROM employees e
    WHERE e.dept_id = d.dept_id
    ORDER BY salary DESC LIMIT 1
) top_emp;
```

---

## 11. Aggregate Functions, GROUP BY & HAVING

### 11.1 Common Aggregates
`COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`, `ARRAY_AGG()`, `STRING_AGG()`

```sql
SELECT dept_id, COUNT(*), AVG(salary)
FROM employees
GROUP BY dept_id
HAVING AVG(salary) > 50000;
```

### 11.2 GROUPING SETS, ROLLUP, CUBE
```sql
SELECT dept_id, job_title, SUM(salary)
FROM employees
GROUP BY ROLLUP (dept_id, job_title);

SELECT dept_id, job_title, SUM(salary)
FROM employees
GROUP BY CUBE (dept_id, job_title);
```

---

## 12. Subqueries

### 12.1 Scalar Subquery
```sql
SELECT first_name FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### 12.2 Correlated Subquery
```sql
SELECT first_name FROM employees e
WHERE salary > (SELECT AVG(salary) FROM employees WHERE dept_id = e.dept_id);
```

### 12.3 EXISTS / NOT EXISTS
```sql
SELECT * FROM departments d
WHERE EXISTS (SELECT 1 FROM employees e WHERE e.dept_id = d.dept_id);
```

### 12.4 Subquery in FROM (Derived Table)
```sql
SELECT dept_id, avg_sal FROM (
    SELECT dept_id, AVG(salary) AS avg_sal FROM employees GROUP BY dept_id
) AS dept_avg
WHERE avg_sal > 50000;
```

---

## 13. Common Table Expressions (CTEs) & Recursive Queries

### 13.1 Basic CTE
```sql
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 80000
)
SELECT * FROM high_earners WHERE dept_id = 2;
```

### 13.2 Recursive CTE (e.g., org hierarchy)
```sql
WITH RECURSIVE org_chart AS (
    SELECT emp_id, first_name, manager_id, 1 AS level
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.emp_id, e.first_name, e.manager_id, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.emp_id
)
SELECT * FROM org_chart;
```

### 13.3 Multiple CTEs
```sql
WITH a AS (SELECT ...), b AS (SELECT ...)
SELECT * FROM a JOIN b ON ...;
```

---

## 14. Set Operations

```sql
SELECT name FROM current_employees
UNION
SELECT name FROM former_employees;      -- removes duplicates

SELECT name FROM current_employees
UNION ALL
SELECT name FROM former_employees;      -- keeps duplicates

SELECT name FROM table_a
INTERSECT
SELECT name FROM table_b;

SELECT name FROM table_a
EXCEPT
SELECT name FROM table_b;
```

---

## 15. Views & Materialized Views

### 15.1 Views (virtual table, always live)
```sql
CREATE VIEW dept_summary AS
SELECT dept_id, COUNT(*) AS headcount, AVG(salary) AS avg_salary
FROM employees GROUP BY dept_id;

CREATE OR REPLACE VIEW dept_summary AS ...;
DROP VIEW dept_summary;
```

### 15.2 Materialized Views (physically stored, needs refresh)
```sql
CREATE MATERIALIZED VIEW dept_summary_mv AS
SELECT dept_id, COUNT(*) AS headcount FROM employees GROUP BY dept_id;

REFRESH MATERIALIZED VIEW dept_summary_mv;
REFRESH MATERIALIZED VIEW CONCURRENTLY dept_summary_mv; -- needs unique index
```

---

## 16. Keys & Constraints (Deep Dive)

### 16.1 Key Types
- **Primary Key:** Uniquely identifies a row; implicitly `NOT NULL` + `UNIQUE`.
- **Foreign Key:** References a primary/unique key in another table, enforces referential integrity.
- **Candidate Key:** Any column set that could qualify as a primary key.
- **Composite Key:** Primary key made of multiple columns.
- **Unique Key:** Enforces uniqueness, allows one `NULL` in a single-column case.
- **Surrogate Key:** Artificial key (e.g., auto-increment ID) with no business meaning.
- **Natural Key:** Key derived from real-world data (e.g., email, SSN).

### 16.2 Constraints
```sql
CREATE TABLE orders (
    order_id   SERIAL PRIMARY KEY,
    cust_id    INTEGER NOT NULL REFERENCES customers(cust_id) ON DELETE CASCADE,
    quantity   INTEGER CHECK (quantity > 0),
    status     VARCHAR(20) DEFAULT 'pending',
    UNIQUE (cust_id, order_id)
);
```

### 16.3 Referential Actions
`ON DELETE CASCADE`, `ON DELETE SET NULL`, `ON DELETE RESTRICT`, `ON DELETE NO ACTION`, `ON UPDATE CASCADE`

### 16.4 Domain Constraints
```sql
CREATE DOMAIN positive_int AS INTEGER CHECK (VALUE > 0);
```

---

## 17. Indexes

### 17.1 Why Indexes?
Speed up lookups (`WHERE`, `JOIN`, `ORDER BY`) at the cost of extra storage and slower writes.

### 17.2 Index Types
- **B-tree** (default): equality & range queries.
- **Hash:** equality-only comparisons.
- **GIN** (Generalized Inverted Index): arrays, JSONB, full-text search.
- **GiST** (Generalized Search Tree): geometric data, full-text, ranges.
- **SP-GiST:** space-partitioned data (e.g., quad-trees).
- **BRIN** (Block Range Index): very large, naturally ordered tables (e.g., timestamps).

### 17.3 Creating Indexes
```sql
CREATE INDEX idx_emp_salary ON employees(salary);
CREATE UNIQUE INDEX idx_emp_email ON employees(email);
CREATE INDEX idx_emp_multi ON employees(dept_id, salary);
CREATE INDEX idx_emp_lower_name ON employees(LOWER(first_name)); -- expression index
CREATE INDEX idx_emp_active ON employees(emp_id) WHERE status = 'active'; -- partial index
CREATE INDEX CONCURRENTLY idx_emp_salary ON employees(salary); -- no table lock
```

### 17.4 Dropping / Inspecting
```sql
DROP INDEX idx_emp_salary;
\d employees   -- shows indexes in psql
```

### 17.5 Covering Index (INCLUDE)
```sql
CREATE INDEX idx_covering ON employees(dept_id) INCLUDE (salary, first_name);
```

---

## 18. Transactions & ACID

### 18.1 ACID Properties
- **Atomicity:** All or nothing.
- **Consistency:** Valid state transitions only.
- **Isolation:** Concurrent transactions don't interfere.
- **Durability:** Committed data survives crashes (via WAL).

### 18.2 Transaction Syntax
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
-- or ROLLBACK; to undo
```

### 18.3 Savepoints
```sql
BEGIN;
SAVEPOINT sp1;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
ROLLBACK TO SAVEPOINT sp1;
COMMIT;
```

---

## 19. Concurrency Control, Locking & Isolation Levels

### 19.1 Isolation Levels
- `READ UNCOMMITTED` (treated as Read Committed in Postgres)
- `READ COMMITTED` (default)
- `REPEATABLE READ`
- `SERIALIZABLE`

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### 19.2 Concurrency Anomalies
- **Dirty Read:** Reading uncommitted data.
- **Non-Repeatable Read:** Row changes between reads in same transaction.
- **Phantom Read:** New rows appear between reads.
- **Lost Update:** Two transactions overwrite each other's changes.

### 19.3 MVCC (Multi-Version Concurrency Control)
PostgreSQL's core concurrency mechanism — readers never block writers and vice versa, by keeping multiple row versions.

### 19.4 Locking
```sql
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;   -- row-level lock
SELECT * FROM accounts FOR SHARE;
LOCK TABLE accounts IN EXCLUSIVE MODE;
```

### 19.5 Deadlocks
Occur when two transactions wait on each other's locks; PostgreSQL auto-detects and aborts one transaction.

---

## 20. Functions & Stored Procedures (PL/pgSQL)

### 20.1 Basic Function
```sql
CREATE OR REPLACE FUNCTION get_full_name(fname TEXT, lname TEXT)
RETURNS TEXT AS $$
BEGIN
    RETURN fname || ' ' || lname;
END;
$$ LANGUAGE plpgsql;

SELECT get_full_name('John', 'Doe');
```

### 20.2 Function with Control Flow
```sql
CREATE OR REPLACE FUNCTION raise_salary(emp INTEGER, pct NUMERIC)
RETURNS VOID AS $$
BEGIN
    IF pct <= 0 THEN
        RAISE EXCEPTION 'Percentage must be positive';
    END IF;
    UPDATE employees SET salary = salary * (1 + pct/100) WHERE emp_id = emp;
END;
$$ LANGUAGE plpgsql;
```

### 20.3 Stored Procedures (with transaction control)
```sql
CREATE PROCEDURE transfer_funds(sender INT, receiver INT, amt NUMERIC)
LANGUAGE plpgsql AS $$
BEGIN
    UPDATE accounts SET balance = balance - amt WHERE id = sender;
    UPDATE accounts SET balance = balance + amt WHERE id = receiver;
    COMMIT;
END;
$$;

CALL transfer_funds(1, 2, 100);
```

### 20.4 Loops & Exception Handling
```sql
DO $$
DECLARE i INTEGER := 1;
BEGIN
    LOOP
        RAISE NOTICE 'i = %', i;
        i := i + 1;
        EXIT WHEN i > 5;
    END LOOP;
EXCEPTION WHEN OTHERS THEN
    RAISE NOTICE 'Error occurred: %', SQLERRM;
END $$;
```

### 20.5 Other Procedural Languages
PostgreSQL also supports `PL/Python`, `PL/Perl`, `PL/Tcl`, `PL/V8` (JavaScript) as extensions.

---

## 21. Triggers

### 21.1 Trigger Function
```sql
CREATE OR REPLACE FUNCTION log_salary_change()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO salary_audit(emp_id, old_salary, new_salary, changed_at)
    VALUES (OLD.emp_id, OLD.salary, NEW.salary, now());
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

### 21.2 Attaching a Trigger
```sql
CREATE TRIGGER trg_salary_update
BEFORE UPDATE OF salary ON employees
FOR EACH ROW
WHEN (OLD.salary IS DISTINCT FROM NEW.salary)
EXECUTE FUNCTION log_salary_change();
```

### 21.3 Trigger Types
- **Timing:** `BEFORE`, `AFTER`, `INSTEAD OF` (for views)
- **Level:** `FOR EACH ROW` vs `FOR EACH STATEMENT`
- **Events:** `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`

---

## 22. Window Functions

### 22.1 Syntax
```sql
SELECT first_name, dept_id, salary,
    RANK()        OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rank,
    DENSE_RANK()  OVER (PARTITION BY dept_id ORDER BY salary DESC) AS dense_rank,
    ROW_NUMBER()  OVER (PARTITION BY dept_id ORDER BY salary DESC) AS row_num,
    SUM(salary)   OVER (PARTITION BY dept_id) AS dept_total,
    AVG(salary)   OVER () AS overall_avg,
    LAG(salary)   OVER (PARTITION BY dept_id ORDER BY salary) AS prev_salary,
    LEAD(salary)  OVER (PARTITION BY dept_id ORDER BY salary) AS next_salary
FROM employees;
```

### 22.2 NTILE & Frame Clauses
```sql
SELECT first_name, NTILE(4) OVER (ORDER BY salary DESC) AS quartile FROM employees;

SELECT emp_id, salary,
    SUM(salary) OVER (ORDER BY emp_id ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS moving_sum
FROM employees;
```

---

## 23. JSON & JSONB

### 23.1 Storing JSON
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    details JSONB
);

INSERT INTO products (details) VALUES
('{"name": "Laptop", "specs": {"ram": "16GB", "cpu": "i7"}, "tags": ["electronics","computers"]}');
```

### 23.2 Querying JSON
```sql
SELECT details->'name' FROM products;              -- returns JSON
SELECT details->>'name' FROM products;             -- returns TEXT
SELECT details#>>'{specs,ram}' FROM products;       -- nested path as text
SELECT * FROM products WHERE details @> '{"name":"Laptop"}';   -- containment
SELECT * FROM products WHERE details ? 'tags';                 -- key exists
SELECT jsonb_array_elements(details->'tags') FROM products;
```

### 23.3 Indexing JSONB
```sql
CREATE INDEX idx_details_gin ON products USING GIN (details);
CREATE INDEX idx_details_path ON products USING GIN (details jsonb_path_ops);
```

### 23.4 JSON Functions
`jsonb_set()`, `jsonb_insert()`, `jsonb_build_object()`, `jsonb_agg()`, `row_to_json()`, `to_jsonb()`

---

## 24. Arrays, Ranges & Composite Types

### 24.1 Arrays
```sql
CREATE TABLE posts (id SERIAL, tags TEXT[]);
INSERT INTO posts (tags) VALUES (ARRAY['sql','postgres']);
SELECT * FROM posts WHERE 'sql' = ANY(tags);
SELECT unnest(tags) FROM posts;
```

### 24.2 Range Types
```sql
CREATE TABLE bookings (room_id INT, during TSRANGE);
INSERT INTO bookings VALUES (1, '[2024-01-01 10:00, 2024-01-01 12:00)');
SELECT * FROM bookings WHERE during && '[2024-01-01 11:00, 2024-01-01 13:00)';  -- overlap
```

### 24.3 Composite Types
```sql
CREATE TYPE address AS (street TEXT, city TEXT, zip TEXT);
CREATE TABLE customers (id SERIAL, home_address address);
```

### 24.4 ENUM Types
```sql
CREATE TYPE mood AS ENUM ('happy', 'sad', 'neutral');
CREATE TABLE person (name TEXT, current_mood mood);
```

---

## 25. Full-Text Search

### 25.1 Basics
```sql
SELECT to_tsvector('english', 'The quick brown foxes are jumping') @@ to_tsquery('english', 'fox');
```

### 25.2 Setting Up Search on a Table
```sql
ALTER TABLE articles ADD COLUMN search_vector TSVECTOR;
UPDATE articles SET search_vector = to_tsvector('english', title || ' ' || body);
CREATE INDEX idx_search ON articles USING GIN(search_vector);

SELECT * FROM articles
WHERE search_vector @@ to_tsquery('english', 'database & postgres');
```

### 25.3 Ranking Results
```sql
SELECT title, ts_rank(search_vector, query) AS rank
FROM articles, to_tsquery('english', 'database') query
WHERE search_vector @@ query
ORDER BY rank DESC;
```

### 25.4 Auto-Update via Trigger
```sql
CREATE TRIGGER tsvector_update
BEFORE INSERT OR UPDATE ON articles
FOR EACH ROW EXECUTE FUNCTION
tsvector_update_trigger(search_vector, 'pg_catalog.english', title, body);
```

---

## 26. Partitioning

### 26.1 Why Partition?
Split a huge table into smaller physical pieces for performance and manageability, while querying it as one logical table.

### 26.2 Range Partitioning
```sql
CREATE TABLE sales (
    id SERIAL,
    sale_date DATE NOT NULL,
    amount NUMERIC
) PARTITION BY RANGE (sale_date);

CREATE TABLE sales_2024 PARTITION OF sales
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

CREATE TABLE sales_2025 PARTITION OF sales
    FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
```

### 26.3 List Partitioning
```sql
CREATE TABLE orders (id SERIAL, region TEXT) PARTITION BY LIST (region);
CREATE TABLE orders_asia PARTITION OF orders FOR VALUES IN ('India','China','Japan');
```

### 26.4 Hash Partitioning
```sql
CREATE TABLE users (id SERIAL, name TEXT) PARTITION BY HASH (id);
CREATE TABLE users_p0 PARTITION OF users FOR VALUES WITH (MODULUS 4, REMAINDER 0);
```

### 26.5 Sub-partitioning & Attach/Detach
```sql
ALTER TABLE sales ATTACH PARTITION sales_2026 FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');
ALTER TABLE sales DETACH PARTITION sales_2024;
```

---

## 27. Table Inheritance & Foreign Data Wrappers

### 27.1 Table Inheritance
```sql
CREATE TABLE vehicles (id SERIAL, brand TEXT);
CREATE TABLE cars (doors INT) INHERITS (vehicles);
```

### 27.2 Foreign Data Wrappers (FDW)
Query external data sources (other Postgres servers, MySQL, files) as if they were local tables.
```sql
CREATE EXTENSION postgres_fdw;
CREATE SERVER remote_server FOREIGN DATA WRAPPER postgres_fdw
    OPTIONS (host 'remote_host', dbname 'remote_db', port '5432');
CREATE USER MAPPING FOR current_user SERVER remote_server
    OPTIONS (user 'remote_user', password 'secret');
CREATE FOREIGN TABLE remote_employees (emp_id INT, name TEXT)
    SERVER remote_server OPTIONS (schema_name 'public', table_name 'employees');
```

---

## 28. Performance Tuning & Query Optimization

### 28.1 EXPLAIN / EXPLAIN ANALYZE
```sql
EXPLAIN SELECT * FROM employees WHERE salary > 50000;
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM employees WHERE salary > 50000;
```
Look at: **Seq Scan vs Index Scan**, **estimated vs actual rows**, **cost**, **execution time**.

### 28.2 Query Planner Concepts
- Planner chooses between Sequential Scan, Index Scan, Index-Only Scan, Bitmap Heap Scan based on statistics.
- **`ANALYZE`** updates table statistics used by the planner.
- **`VACUUM`** reclaims dead tuple space from updates/deletes (MVCC bloat).
- **`VACUUM FULL`** rewrites the table (locks it, reclaims all space).
- **Autovacuum** runs these automatically in the background.

### 28.3 Common Optimization Techniques
- Add appropriate indexes (and drop unused ones).
- Avoid `SELECT *`; select only needed columns.
- Use `EXISTS` instead of `IN` for large subqueries.
- Batch large `INSERT`/`UPDATE`/`DELETE` operations.
- Use connection pooling for high-concurrency apps.
- Avoid functions on indexed columns in `WHERE` (breaks index usage) — use expression indexes instead.
- Use `LIMIT` with `ORDER BY` on indexed columns.
- Monitor and rewrite N+1 query patterns.

### 28.4 Configuration Tuning (postgresql.conf)
`shared_buffers`, `work_mem`, `maintenance_work_mem`, `effective_cache_size`, `max_connections`, `checkpoint_timeout`, `wal_buffers`

---

## 29. Security: Roles, Privileges & Row-Level Security

### 29.1 Roles & Users
```sql
CREATE ROLE readonly_user WITH LOGIN PASSWORD 'secret';
ALTER ROLE readonly_user WITH SUPERUSER;
CREATE ROLE app_group;
GRANT app_group TO readonly_user;
```

### 29.2 Privileges
```sql
GRANT SELECT, INSERT ON employees TO readonly_user;
GRANT ALL PRIVILEGES ON DATABASE company_db TO admin_user;
REVOKE INSERT ON employees FROM readonly_user;
```

### 29.3 Row-Level Security (RLS)
```sql
ALTER TABLE employees ENABLE ROW LEVEL SECURITY;

CREATE POLICY dept_isolation ON employees
    USING (dept_id = current_setting('app.current_dept')::INT);
```

### 29.4 SSL & Authentication
Configured via `pg_hba.conf` — supports `trust`, `md5`, `scram-sha-256`, `peer`, `ldap`, `cert` authentication methods.

---

## 30. Backup, Restore & Import/Export

### 30.1 Logical Backup
```bash
pg_dump -U postgres -d company_db -F c -f company_db.dump
pg_dumpall -U postgres -f all_databases.sql
```

### 30.2 Restore
```bash
pg_restore -U postgres -d company_db company_db.dump
psql -U postgres -d company_db -f all_databases.sql
```

### 30.3 CSV Import/Export
```sql
COPY employees TO '/tmp/employees.csv' WITH (FORMAT csv, HEADER true);
COPY employees FROM '/tmp/employees.csv' WITH (FORMAT csv, HEADER true);
```

### 30.4 Physical Backup
File-system-level backups (`pg_basebackup`) — used for point-in-time recovery (PITR) combined with WAL archiving.

---

## 31. Replication & High Availability

### 31.1 Streaming Replication
Primary server streams WAL records to one or more standby (replica) servers in near real-time.

### 31.2 Synchronous vs Asynchronous Replication
- **Asynchronous:** Primary doesn't wait for replica acknowledgment (default, faster, small data-loss risk).
- **Synchronous:** Primary waits for at least one replica to confirm (safer, slightly slower).

### 31.3 Logical Replication
Replicates specific tables/changes (not the whole cluster) using publications and subscriptions.
```sql
CREATE PUBLICATION my_pub FOR TABLE employees;
CREATE SUBSCRIPTION my_sub CONNECTION 'host=primary dbname=company_db' PUBLICATION my_pub;
```

### 31.4 Failover & Tools
Tools like **Patroni**, **repmgr**, **pgpool-II** manage automatic failover and load balancing across replicas.

---

## 32. Extensions

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";   -- UUID generation
CREATE EXTENSION IF NOT EXISTS pgcrypto;      -- encryption, gen_random_uuid()
CREATE EXTENSION IF NOT EXISTS pg_trgm;       -- trigram fuzzy text search
CREATE EXTENSION IF NOT EXISTS postgis;       -- geospatial data
CREATE EXTENSION IF NOT EXISTS pg_stat_statements; -- query performance stats
CREATE EXTENSION IF NOT EXISTS hstore;        -- key-value store type
```

---

## 33. Connection Pooling

### 33.1 Why Pooling?
Each PostgreSQL connection is a full OS process — expensive at scale. Poolers reuse connections across clients.

### 33.2 PgBouncer
Lightweight external connection pooler with modes: `session`, `transaction`, `statement`.

### 33.3 Application-Level Pooling
Most frameworks/ORMs (e.g., SQLAlchemy, HikariCP, node-postgres `pg.Pool`) provide built-in pooling.

---

## 34. Monitoring, Logging & Maintenance

### 34.1 Useful System Catalogs & Views
```sql
SELECT * FROM pg_stat_activity;      -- active connections/queries
SELECT * FROM pg_stat_user_tables;   -- table-level stats (scans, tuples)
SELECT * FROM pg_locks;              -- current locks
SELECT * FROM pg_stat_statements ORDER BY total_exec_time DESC LIMIT 10;
```

### 34.2 Vacuum & Analyze
```sql
VACUUM ANALYZE employees;
VACUUM VERBOSE;
```

### 34.3 Logging
Configure `log_min_duration_statement`, `log_statement`, `log_connections` in `postgresql.conf` to track slow/critical queries.

---

## 35. PostgreSQL with Application Code

### 35.1 Common Drivers
- **Python:** `psycopg2`, `asyncpg`, SQLAlchemy
- **Node.js:** `pg`, Prisma, Sequelize, TypeORM
- **Java:** JDBC, Hibernate
- **Go:** `pgx`, `database/sql`

### 35.2 ORMs vs Raw SQL
ORMs (SQLAlchemy, Prisma, Hibernate) speed up development and add safety, but raw SQL / query builders offer more control for performance-critical paths.

### 35.3 Prepared Statements & SQL Injection Prevention
Always use parameterized queries (`$1, $2` placeholders or ORM equivalents) — never string-concatenate user input into SQL.

---

## 36. Best Practices & Cheat Sheet

### 36.1 Design Best Practices
- Normalize first, denormalize only when performance demands it.
- Always define primary keys; prefer surrogate keys for stability.
- Use appropriate, narrowest data types (don't use `TEXT` for everything).
- Add `NOT NULL` and `CHECK` constraints wherever business rules allow.

### 36.2 Query Best Practices
- Always `EXPLAIN ANALYZE` slow queries before optimizing blindly.
- Index columns used in `WHERE`, `JOIN`, and `ORDER BY` — but don't over-index.
- Use transactions for multi-statement operations that must succeed/fail together.
- Prefer `JSONB` over `JSON` for anything you'll query/index.

### 36.3 Operational Best Practices
- Set up regular automated backups and test restores.
- Monitor autovacuum activity on high-write tables.
- Use connection pooling in production applications.
- Keep PostgreSQL updated to a supported major version.

### 36.4 Quick Command Reference
| Task | Command |
|---|---|
| List databases | `\l` |
| Connect to DB | `\c dbname` |
| List tables | `\dt` |
| Describe table | `\d tablename` |
| Backup | `pg_dump` |
| Restore | `pg_restore` / `psql -f` |
| Current connections | `pg_stat_activity` |
| Explain query | `EXPLAIN ANALYZE` |

---

## Suggested Learning Path
1. Sections 1–9: Foundations + core SQL (practice heavily with a sample database like `dvdrental` or `northwind`).
2. Sections 10–17: Joins, aggregation, views, keys, indexes.
3. Sections 18–22: Transactions, concurrency, PL/pgSQL, triggers, window functions.
4. Sections 23–27: JSON, arrays, full-text search, partitioning, FDWs.
5. Sections 28–35: Performance, security, backup/replication, real-world app integration.

**Practice tip:** Install PostgreSQL locally, load a sample dataset, and re-implement every query in this document yourself — active practice cements SQL far faster than reading alone.
