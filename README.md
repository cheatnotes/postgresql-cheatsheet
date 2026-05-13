# PostgreSQL Comprehensive Cheatsheet

> **Comprehensive PostgreSQL cheatsheet covering installation, connections, databases, tables, data types, CRUD, constraints, indexes, queries, joins, aggregations, CTEs, views, functions, triggers, transactions, users, backup/restore, performance tuning, monitoring, JSONB, full-text search, partitioning, system tables & more**

## Table of Contents

1. [Installation & Setup](#1-installation--setup)
2. [Connecting to Database](#2-connecting-to-database)
3. [Database Management](#3-database-management)
4. [Table Management](#4-table-management)
5. [Data Types](#5-data-types)
6. [CRUD Operations](#6-crud-operations)
7. [Constraints](#7-constraints)
8. [Indexes](#8-indexes)
9. [Queries & Filtering](#9-queries--filtering)
10. [Joins](#10-joins)
11. [Aggregation & Grouping](#11-aggregation--grouping)
12. [Subqueries & CTEs](#12-subqueries--ctes)
13. [Views](#13-views)
14. [Functions & Procedures](#14-functions--procedures)
15. [Triggers](#15-triggers)
16. [Transactions & Locking](#16-transactions--locking)
17. [Users & Permissions](#17-users--permissions)
18. [Backup & Restore](#18-backup--restore)
19. [Performance Tuning](#19-performance-tuning)
20. [Monitoring & Diagnostics](#20-monitoring--diagnostics)
21. [JSON/JSONB Operations](#21-jsonjsonb-operations)
22. [Full-Text Search](#22-full-text-search)
23. [Partitioning](#23-partitioning)
24. [Useful Functions](#24-useful-functions)
25. [System Tables & Views](#25-system-tables--views)

---

## 1. Installation & Setup

### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### macOS
```bash
brew install postgresql
brew services start postgresql
```

### Windows
Download installer from [postgresql.org](https://www.postgresql.org/download/windows/)

### Switch to postgres user
```bash
sudo -i -u postgres
# or
psql -U postgres
```

---

## 2. Connecting to Database

### psql commands
```bash
# Connect to database
psql -d database_name -U username -h host -p port

# Connect with default user
psql -U postgres

# Connect to specific database
psql -d mydb

# Execute single query
psql -d mydb -c "SELECT * FROM users;"

# Execute SQL file
psql -d mydb -f script.sql
```

### Inside psql meta-commands
```sql
\l                       -- List all databases
\c database_name         -- Connect to database
\dt                      -- List tables
\d table_name            -- Describe table
\du                      -- List users
\dn                      -- List schemas
\di                      -- List indexes
\dv                      -- List views
\df                      -- List functions
\dx                      -- List extensions
\?                       -- Help for meta-commands
\h                       -- Help for SQL commands
\q                       -- Quit psql
\!                       -- Execute shell command
\e                       -- Edit query in editor
\x                       -- Toggle expanded output
\timing                  -- Toggle query timing
\i filename.sql          -- Execute SQL from file
\o filename.txt          -- Export output to file
\copy (SELECT * FROM t) TO 'file.csv' CSV HEADER; -- Export query
```

---

## 3. Database Management

```sql
-- Create database
CREATE DATABASE database_name;
CREATE DATABASE mydb OWNER username;
CREATE DATABASE mydb ENCODING 'UTF8' LC_COLLATE 'en_US.UTF-8';

-- Drop database
DROP DATABASE database_name;
DROP DATABASE IF EXISTS mydb;  -- PostgreSQL 9.3+

-- Rename database
ALTER DATABASE old_name RENAME TO new_name;

-- List databases (in psql)
\l

-- Get database size
SELECT pg_database_size('mydb');
SELECT pg_size_pretty(pg_database_size('mydb'));

-- Current database
SELECT current_database();

-- Create schema
CREATE SCHEMA schema_name;
CREATE SCHEMA IF NOT EXISTS myschema;

-- Set search path
SET search_path TO myschema, public;

-- Drop schema
DROP SCHEMA schema_name CASCADE;
```

---

## 4. Table Management

```sql
-- Create table
CREATE TABLE table_name (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create table from existing table
CREATE TABLE new_table AS SELECT * FROM old_table WHERE false;  -- Structure only
CREATE TABLE new_table AS SELECT * FROM old_table;  -- With data

-- Drop table
DROP TABLE table_name;
DROP TABLE IF EXISTS table_name CASCADE;

-- Rename table
ALTER TABLE old_name RENAME TO new_name;

-- Add column
ALTER TABLE table_name ADD COLUMN column_name data_type;
ALTER TABLE table_name ADD COLUMN IF NOT EXISTS email VARCHAR(255);

-- Drop column
ALTER TABLE table_name DROP COLUMN column_name CASCADE;

-- Rename column
ALTER TABLE table_name RENAME COLUMN old_name TO new_name;

-- Change column data type
ALTER TABLE table_name ALTER COLUMN column_name TYPE new_data_type;

-- Set default value
ALTER TABLE table_name ALTER COLUMN column_name SET DEFAULT 0;

-- Drop default
ALTER TABLE table_name ALTER COLUMN column_name DROP DEFAULT;

-- Set NOT NULL
ALTER TABLE table_name ALTER COLUMN column_name SET NOT NULL;

-- Drop NOT NULL
ALTER TABLE table_name ALTER COLUMN column_name DROP NOT NULL;

-- Add comment
COMMENT ON TABLE table_name IS 'Description of table';
COMMENT ON COLUMN table_name.column_name IS 'Column description';

-- List tables
\d

-- Get table size
SELECT pg_size_pretty(pg_total_relation_size('table_name'));
```

---

## 5. Data Types

### Numeric Types
| Type | Size | Range |
|------|------|-------|
| `SMALLINT` | 2 bytes | -32,768 to 32,767 |
| `INTEGER` | 4 bytes | -2.1B to 2.1B |
| `BIGINT` | 8 bytes | -9.2Q to 9.2Q |
| `DECIMAL(p,s)` | variable | Exact numeric |
| `NUMERIC(p,s)` | variable | Exact numeric |
| `REAL` | 4 bytes | 6 decimal digits precision |
| `DOUBLE PRECISION` | 8 bytes | 15 decimal digits precision |
| `SERIAL` | 4 bytes | Auto-increment integer |
| `BIGSERIAL` | 8 bytes | Auto-increment bigint |

### Character Types
- `CHAR(n)` — Fixed-length, space-padded
- `VARCHAR(n)` — Variable-length with limit
- `TEXT` — Variable unlimited length

### Date/Time Types
- `DATE` — Year, month, day
- `TIME` — Hour, minute, second
- `TIMESTAMP` — Date and time (no timezone)
- `TIMESTAMPTZ` — Date and time with timezone
- `INTERVAL` — Time span

### Other Types
- `BOOLEAN` — true/false
- `UUID` — Universally unique identifier
- `JSON` — Textual JSON data
- `JSONB` — Binary JSON (indexable)
- `ARRAY` — Array of values (e.g., `TEXT[]`)
- `HSTORE` — Key-value pairs (extension)
- `INET` — IPv4/IPv6 address
- `CIDR` — CIDR network address
- `MACADDR` — MAC address
- `BYTEA` — Binary data
- `ENUM` — Enumeration type

```sql
-- Create custom ENUM type
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');

-- Create range type
CREATE TYPE temperature_range AS RANGE (subtype = NUMERIC);
```

---

## 6. CRUD Operations

### INSERT
```sql
-- Single row
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');

-- Multiple rows
INSERT INTO users (name, email) VALUES 
    ('Jane Doe', 'jane@example.com'),
    ('Bob Smith', 'bob@example.com');

-- With returning clause
INSERT INTO users (name, email) 
VALUES ('Alice', 'alice@example.com') 
RETURNING id, created_at;

-- Insert from another table
INSERT INTO archive_users SELECT * FROM users WHERE active = false;

-- Upsert (INSERT ON CONFLICT)
INSERT INTO users (id, name, email) 
VALUES (1, 'John', 'john@example.com')
ON CONFLICT (id) 
DO UPDATE SET name = EXCLUDED.name, email = EXCLUDED.email;

-- ON CONFLICT with nothing
INSERT INTO users (id, name) 
VALUES (1, 'John')
ON CONFLICT (id) DO NOTHING;
```

### SELECT
```sql
-- Basic select
SELECT * FROM users;
SELECT name, email FROM users;

-- Distinct
SELECT DISTINCT city FROM users;
SELECT DISTINCT ON (city) city, name FROM users ORDER BY city, created_at;

-- Limit and offset
SELECT * FROM users LIMIT 10 OFFSET 20;

-- Conditional selection (CASE)
SELECT name,
    CASE 
        WHEN age < 18 THEN 'Minor'
        WHEN age BETWEEN 18 AND 65 THEN 'Adult'
        ELSE 'Senior'
    END AS age_group
FROM users;
```

### UPDATE
```sql
-- Basic update
UPDATE users SET email = 'new@example.com' WHERE id = 1;

-- Update multiple columns
UPDATE users SET name = 'New Name', updated_at = NOW() WHERE id = 1;

-- Update with returning
UPDATE users SET active = true 
WHERE id = 5 
RETURNING id, name, email;

-- Update from another table
UPDATE users 
SET city = cities.default_city 
FROM cities 
WHERE users.city_id = cities.id;

-- Increment value
UPDATE products SET views = views + 1 WHERE id = 10;
```

### DELETE
```sql
-- Basic delete
DELETE FROM users WHERE id = 10;

-- Delete with returning
DELETE FROM users WHERE active = false RETURNING *;

-- Delete all rows
DELETE FROM users;  -- Slower, logs each row
TRUNCATE users;     -- Faster, cannot rollback in some cases

-- Delete using another table
DELETE FROM users 
USING inactive_users 
WHERE users.id = inactive_users.user_id;
```

---

## 7. Constraints

```sql
-- Primary Key
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    -- or
    user_id INTEGER,
    PRIMARY KEY (user_id)
);

-- Composite Primary Key
CREATE TABLE orders_products (
    order_id INTEGER,
    product_id INTEGER,
    PRIMARY KEY (order_id, product_id)
);

-- Foreign Key
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    -- or with options
    user_id INTEGER,
    FOREIGN KEY (user_id) REFERENCES users(id) 
        ON DELETE CASCADE 
        ON UPDATE CASCADE
);

-- Unique Constraint
CREATE TABLE users (
    email VARCHAR(255) UNIQUE,
    -- or
    UNIQUE (email, username)
);

-- Check Constraint
CREATE TABLE products (
    price NUMERIC CHECK (price > 0),
    quantity INTEGER CHECK (quantity >= 0),
    -- or named
    CONSTRAINT valid_dates CHECK (end_date > start_date)
);

-- NOT NULL Constraint
CREATE TABLE users (
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL
);

-- Exclusion Constraint (requires extension)
CREATE EXTENSION IF NOT EXISTS btree_gist;
CREATE TABLE reservations (
    room INTEGER,
    during TSRANGE,
    EXCLUDE USING gist (room WITH =, during WITH &&)
);

-- Add constraints to existing table
ALTER TABLE users ADD PRIMARY KEY (id);
ALTER TABLE users ADD UNIQUE (email);
ALTER TABLE users ADD FOREIGN KEY (role_id) REFERENCES roles(id);
ALTER TABLE users ADD CHECK (age >= 18);
ALTER TABLE users ALTER COLUMN name SET NOT NULL;

-- Drop constraints
ALTER TABLE users DROP CONSTRAINT users_pkey;
ALTER TABLE users DROP CONSTRAINT users_email_key;
ALTER TABLE users ALTER COLUMN name DROP NOT NULL;
```

---

## 8. Indexes

```sql
-- B-tree (default) - equality and range queries
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX CONCURRENTLY idx_users_name ON users(name);  -- No table lock

-- Unique index
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);

-- Composite index
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Partial index
CREATE INDEX idx_active_users ON users(email) WHERE active = true;

-- Expression index
CREATE INDEX idx_users_lower_email ON users(LOWER(email));
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';

-- Hash index (equality only, very fast for =)
CREATE INDEX idx_users_id_hash ON users USING HASH (id);

-- GiST index (geometric data, full-text search)
CREATE INDEX idx_posts_tsvector ON posts USING GIST (search_vector);

-- GIN index (JSONB, arrays, full-text)
CREATE INDEX idx_users_data ON users USING GIN (jsonb_data);
CREATE INDEX idx_posts_search ON posts USING GIN (to_tsvector('english', content));

-- BRIN index (large tables, naturally ordered data)
CREATE INDEX idx_logs_created_at ON logs USING BRIN (created_at);

-- Drop index
DROP INDEX idx_users_email;
DROP INDEX CONCURRENTLY idx_users_name;

-- List indexes
\d table_name
SELECT * FROM pg_indexes WHERE tablename = 'users';

-- Reindex
REINDEX INDEX idx_users_email;
REINDEX TABLE users;  -- All indexes on table

-- Index usage stats
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes 
ORDER BY idx_scan;
```

---

## 9. Queries & Filtering

```sql
-- WHERE clause operators
SELECT * FROM users WHERE age = 25;
SELECT * FROM users WHERE age != 25;
SELECT * FROM users WHERE age > 18;
SELECT * FROM users WHERE age BETWEEN 18 AND 30;
SELECT * FROM users WHERE name LIKE 'J%';     -- Case-sensitive
SELECT * FROM users WHERE name ILIKE 'j%';    -- Case-insensitive
SELECT * FROM users WHERE email IN ('a@b.com', 'c@d.com');
SELECT * FROM users WHERE email NOT IN ('spam@domain.com');
SELECT * FROM users WHERE city IS NULL;
SELECT * FROM users WHERE city IS NOT NULL;

-- Pattern matching
SELECT * FROM users WHERE name LIKE 'Jo_n';   -- _ matches single char
SELECT * FROM users WHERE name LIKE 'J%';     -- % matches any sequence
SELECT * FROM users WHERE name SIMILAR TO '(J|K)%';  -- SQL standard regex
SELECT * FROM users WHERE name ~ '^[A-Z]';    -- POSIX regex
SELECT * FROM users WHERE name ~* '^[a-z]';   -- Case-insensitive

-- Boolean logic
SELECT * FROM users 
WHERE age >= 18 
  AND city = 'New York' 
  AND (status = 'active' OR status = 'pending');

-- ORDER BY
SELECT * FROM users ORDER BY name;
SELECT * FROM users ORDER BY age DESC, name ASC;
SELECT * FROM users ORDER BY created_at NULLS LAST;  -- NULLs at end
SELECT * FROM users ORDER BY created_at NULLS FIRST;

-- GROUP BY
SELECT city, COUNT(*) FROM users GROUP BY city;
SELECT city, COUNT(*) FROM users GROUP BY city HAVING COUNT(*) > 5;

-- DISTINCT variations
SELECT DISTINCT city FROM users;
SELECT DISTINCT ON (city) city, name, created_at 
FROM users ORDER BY city, created_at DESC;

-- EXISTS
SELECT * FROM users u 
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- ANY / ALL (array operations)
SELECT * FROM users WHERE age > ANY(ARRAY[18, 21, 25]);
SELECT * FROM users WHERE age > ALL(ARRAY[18, 21, 25]);

-- UNION / INTERSECT / EXCEPT
SELECT name FROM employees
UNION  -- removes duplicates
SELECT name FROM contractors;

SELECT name FROM employees
UNION ALL  -- includes duplicates
SELECT name FROM contractors;

SELECT id FROM users
INTERSECT
SELECT user_id FROM orders;

SELECT id FROM users
EXCEPT
SELECT user_id FROM inactive_users;
```

---

## 10. Joins

```sql
-- INNER JOIN (default)
SELECT u.name, o.order_date
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT OUTER JOIN
SELECT u.name, o.order_date
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT OUTER JOIN
SELECT u.name, o.order_date
FROM orders o
RIGHT JOIN users u ON u.id = o.user_id;

-- FULL OUTER JOIN
SELECT u.name, o.order_date
FROM users u
FULL JOIN orders o ON u.id = o.user_id;

-- CROSS JOIN (Cartesian product)
SELECT * FROM users CROSS JOIN products;

-- NATURAL JOIN (joins on same column names)
SELECT * FROM users NATURAL JOIN orders;

-- SELF JOIN
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.id;

-- LATERAL join (subquery that references previous table)
SELECT u.name, recent.order_date
FROM users u,
LATERAL (
    SELECT order_date FROM orders 
    WHERE user_id = u.id 
    ORDER BY order_date DESC 
    LIMIT 2
) AS recent;

-- Multi-join
SELECT u.name, o.order_date, p.product_name
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

---

## 11. Aggregation & Grouping

```sql
-- Basic aggregate functions
SELECT 
    COUNT(*) AS total_count,
    COUNT(email) AS non_null_emails,
    COUNT(DISTINCT city) AS unique_cities,
    SUM(amount) AS total_amount,
    AVG(age) AS average_age,
    MIN(created_at) AS first_user,
    MAX(created_at) AS last_user,
    STDDEV(age) AS age_stddev,
    VARIANCE(age) AS age_variance
FROM users;

-- GROUP BY with multiple columns
SELECT 
    department,
    status,
    COUNT(*) AS count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department, status;

-- HAVING (filter after GROUP BY)
SELECT 
    department,
    COUNT(*) AS emp_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING COUNT(*) > 10 AND AVG(salary) > 50000;

-- GROUPING SETS
SELECT department, job_title, COUNT(*)
FROM employees
GROUP BY GROUPING SETS (
    (department, job_title),  -- individual groups
    (department),              -- department subtotal
    ()                         -- grand total
);

-- ROLLUP (hierarchy)
SELECT 
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    SUM(amount)
FROM orders
GROUP BY ROLLUP(year, month);

-- CUBE (all combinations)
SELECT department, job_title, COUNT(*)
FROM employees
GROUP BY CUBE(department, job_title);

-- String aggregation
SELECT 
    department,
    STRING_AGG(name, ', ' ORDER BY name) AS employees
FROM employees
GROUP BY department;

-- Array aggregation
SELECT 
    department,
    ARRAY_AGG(name) AS employees_array
FROM employees
GROUP BY department;

-- FILTER clause
SELECT 
    COUNT(*) AS total,
    COUNT(*) FILTER (WHERE status = 'active') AS active,
    COUNT(*) FILTER (WHERE status = 'inactive') AS inactive
FROM users;
```

---

## 12. Subqueries & CTEs

```sql
-- Subquery in WHERE
SELECT * FROM users 
WHERE id IN (SELECT user_id FROM orders WHERE amount > 100);

-- Subquery with EXISTS
SELECT * FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o 
    WHERE o.user_id = u.id AND o.amount > 100
);

-- Subquery in SELECT (scalar)
SELECT 
    name,
    (SELECT AVG(amount) FROM orders WHERE user_id = users.id) AS avg_order
FROM users;

-- Subquery in FROM (derived table)
SELECT user_id, SUM(amount) AS total
FROM (SELECT * FROM orders WHERE status = 'completed') AS completed_orders
GROUP BY user_id;

-- WITH (CTE - Common Table Expression)
WITH high_value_users AS (
    SELECT user_id, SUM(amount) AS total
    FROM orders
    GROUP BY user_id
    HAVING SUM(amount) > 1000
)
SELECT u.name, h.total
FROM users u
JOIN high_value_users h ON u.id = h.user_id;

-- Recursive CTE
WITH RECURSIVE employee_hierarchy AS (
    -- Base case
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case
    SELECT e.id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM employee_hierarchy ORDER BY level, name;

-- Multiple CTEs
WITH 
sales_data AS (
    SELECT product_id, SUM(amount) AS total_sales
    FROM orders
    WHERE status = 'completed'
    GROUP BY product_id
),
product_ranking AS (
    SELECT 
        product_id,
        total_sales,
        RANK() OVER (ORDER BY total_sales DESC) AS rank
    FROM sales_data
)
SELECT p.name, pr.total_sales, pr.rank
FROM products p
JOIN product_ranking pr ON p.id = pr.product_id
WHERE pr.rank <= 10;

-- CTE with data modification (PostgreSQL 9.1+)
WITH deleted AS (
    DELETE FROM users WHERE active = false
    RETURNING *
)
INSERT INTO users_archive SELECT * FROM deleted;
```

---

## 13. Views

```sql
-- Create simple view
CREATE VIEW active_users AS
SELECT id, name, email, created_at
FROM users
WHERE active = true;

-- Create view with options
CREATE OR REPLACE VIEW user_orders AS
SELECT u.name, o.order_date, o.amount
FROM users u
JOIN orders o ON u.id = o.user_id
WITH CHECK OPTION;  -- Prevents updates that would exclude row

-- Materialized View (cached result)
CREATE MATERIALIZED VIEW user_summary AS
SELECT 
    u.id,
    u.name,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;

-- Refresh materialized view
REFRESH MATERIALIZED VIEW user_summary;
REFRESH MATERIALIZED VIEW CONCURRENTLY user_summary;  -- No table lock

-- Drop view
DROP VIEW active_users;
DROP MATERIALIZED VIEW user_summary CASCADE;

-- Updateable views (simple views can be updated automatically)
INSERT INTO active_users (name, email) VALUES ('New User', 'new@example.com');

-- Create view with custom columns
CREATE VIEW user_emails (user_id, user_name, email_address) AS
SELECT id, name, email FROM users;

-- View with security barrier (Row Level Security)
CREATE VIEW public_users WITH (security_barrier) AS
SELECT id, name FROM users WHERE public_profile = true;
```

---

## 14. Functions & Procedures

### Functions (return value)
```sql
-- Scalar function
CREATE FUNCTION get_user_email(user_id INTEGER) 
RETURNS VARCHAR AS $$
DECLARE
    user_email VARCHAR;
BEGIN
    SELECT email INTO user_email FROM users WHERE id = user_id;
    RETURN user_email;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT get_user_email(1);

-- Table function (returns multiple rows)
CREATE FUNCTION get_users_by_city(city_name VARCHAR)
RETURNS TABLE(id INTEGER, name VARCHAR, email VARCHAR) AS $$
BEGIN
    RETURN QUERY
    SELECT u.id, u.name, u.email
    FROM users u
    WHERE u.city = city_name;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT * FROM get_users_by_city('New York');

-- Set-returning function
CREATE FUNCTION generate_series_custom(start_int INT, end_int INT)
RETURNS SETOF INT AS $$
BEGIN
    FOR i IN start_int..end_int LOOP
        RETURN NEXT i;
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- SQL function (simpler)
CREATE FUNCTION add_numbers(a INT, b INT) 
RETURNS INT AS $$
    SELECT a + b;
$$ LANGUAGE SQL IMMUTABLE;

-- Function with default parameters
CREATE FUNCTION greet(name TEXT, greeting TEXT DEFAULT 'Hello')
RETURNS TEXT AS $$
BEGIN
    RETURN greeting || ', ' || name || '!';
END;
$$ LANGUAGE plpgsql;

-- Drop function
DROP FUNCTION add_numbers(INT, INT);
DROP FUNCTION IF EXISTS get_user_email(INT);
```

### Procedures (no return value)
```sql
-- Stored procedure (PostgreSQL 11+)
CREATE PROCEDURE transfer_funds(
    from_account INT,
    to_account INT,
    amount DECIMAL
)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Withdraw
    UPDATE accounts SET balance = balance - amount WHERE id = from_account;
    
    -- Deposit
    UPDATE accounts SET balance = balance + amount WHERE id = to_account;
    
    -- Record transaction
    INSERT INTO transactions (from_account, to_account, amount, timestamp)
    VALUES (from_account, to_account, amount, NOW());
    
    COMMIT;
END;
$$;

-- Call procedure
CALL transfer_funds(1, 2, 100.00);
```

### Triggers & Trigger Functions
```sql
-- Create trigger function
CREATE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger
CREATE TRIGGER update_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

-- Drop trigger
DROP TRIGGER update_users_updated_at ON users;
```

---

## 15. Triggers

```sql
-- Basic trigger
CREATE TRIGGER trigger_name
    {BEFORE | AFTER | INSTEAD OF}
    {INSERT | UPDATE | DELETE | TRUNCATE}
    ON table_name
    [FOR EACH ROW | FOR EACH STATEMENT]
    [WHEN (condition)]
    EXECUTE FUNCTION function_name();

-- Example: Audit trigger
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name TEXT,
    operation TEXT,
    old_data JSONB,
    new_data JSONB,
    changed_by TEXT,
    changed_at TIMESTAMP DEFAULT NOW()
);

CREATE FUNCTION audit_trigger_function()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (table_name, operation, old_data, new_data, changed_by)
    VALUES (
        TG_TABLE_NAME,
        TG_OP,
        CASE WHEN TG_OP IN ('DELETE', 'UPDATE') THEN row_to_json(OLD) ELSE NULL END,
        CASE WHEN TG_OP IN ('INSERT', 'UPDATE') THEN row_to_json(NEW) ELSE NULL END,
        current_user
    );
    
    IF TG_OP = 'DELETE' THEN
        RETURN OLD;
    ELSE
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_audit
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW
    EXECUTE FUNCTION audit_trigger_function();

-- Constraint trigger (deferred)
CREATE CONSTRAINT TRIGGER check_order_total
    AFTER INSERT OR UPDATE ON order_items
    DEFERRABLE INITIALLY DEFERRED
    FOR EACH ROW
    EXECUTE FUNCTION validate_order_total();

-- Enable/disable triggers
ALTER TABLE users DISABLE TRIGGER users_audit;
ALTER TABLE users ENABLE TRIGGER users_audit;
ALTER TABLE users ENABLE TRIGGER ALL;

-- Drop trigger
DROP TRIGGER users_audit ON users;
```

---

## 16. Transactions & Locking

```sql
-- Basic transaction
BEGIN;  -- or START TRANSACTION
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
    -- Check something
    SELECT balance FROM accounts WHERE id = 1;
    -- If all good
COMMIT;  -- or ROLLBACK to undo

-- Transaction with savepoint
BEGIN;
    UPDATE products SET stock = stock - 10 WHERE id = 1;
    SAVEPOINT before_stock_check;
    UPDATE products SET stock = stock - 100 WHERE id = 2;
    -- Oops, not enough stock
    ROLLBACK TO SAVEPOINT before_stock_check;
    UPDATE products SET stock = stock - 5 WHERE id = 2;
COMMIT;

-- Isolation levels
-- Read Uncommitted (not allowed in PostgreSQL - defaults to Read Committed)
-- Read Committed (default)
-- Repeatable Read
-- Serializable

BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    -- critical operations
COMMIT;

-- Row-level locking
BEGIN;
    SELECT * FROM users WHERE id = 1 FOR UPDATE;
    -- Other transactions cannot modify this row
    UPDATE users SET balance = balance - 100 WHERE id = 1;
COMMIT;

-- Lock types
SELECT * FROM users WHERE id = 1 FOR UPDATE;           -- Row-level exclusive
SELECT * FROM users WHERE id = 1 FOR NO KEY UPDATE;    -- Weaker lock
SELECT * FROM users WHERE id = 1 FOR SHARE;            -- Share lock
SELECT * FROM users WHERE id = 1 FOR KEY SHARE;        -- Key share lock

-- Table locks
LOCK TABLE users IN ACCESS SHARE MODE;      -- SELECT only
LOCK TABLE users IN ROW SHARE MODE;         -- SELECT FOR UPDATE/SHARE
LOCK TABLE users IN ROW EXCLUSIVE MODE;     -- DML operations
LOCK TABLE users IN SHARE UPDATE EXCLUSIVE MODE;  -- VACUUM, ANALYZE
LOCK TABLE users IN SHARE MODE;             -- Read-only, no concurrent DML
LOCK TABLE users IN SHARE ROW EXCLUSIVE MODE; -- Like SHARE but no concurrent updates
LOCK TABLE users IN EXCLUSIVE MODE;         -- Block reads and writes
LOCK TABLE users IN ACCESS EXCLUSIVE MODE;  -- Full table lock (default for DDL)

-- Advisory locks (application-level)
SELECT pg_advisory_lock(12345);
SELECT pg_advisory_unlock(12345);
SELECT pg_try_advisory_lock(12345);  -- Returns true/false

-- Deadlock detection (auto-detected and resolved)
-- View locks
SELECT * FROM pg_locks;
SELECT relation::regclass, locktype, mode, granted 
FROM pg_locks 
WHERE pid = pg_backend_pid();
```

---

## 17. Users & Permissions

```sql
-- Create user/role
CREATE USER username WITH PASSWORD 'password';
CREATE ROLE role_name WITH LOGIN;  -- LOGIN needed for connection
CREATE ROLE readonly;  -- No login, for group role

-- Alter user
ALTER USER username WITH PASSWORD 'new_password';
ALTER USER username VALID UNTIL '2025-12-31';
ALTER USER username SET search_path TO myschema, public;

-- Drop user
DROP USER username;
DROP ROLE role_name;

-- Grant privileges
GRANT CONNECT ON DATABASE mydb TO username;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO username;
GRANT USAGE ON SCHEMA public TO username;
GRANT ALL PRIVILEGES ON users TO username;
GRANT SELECT ON TABLE users TO readonly_role;

-- Grant on sequences (for SERIAL columns)
GRANT USAGE ON SEQUENCE users_id_seq TO username;

-- Grant schema privileges
GRANT CREATE ON SCHEMA public TO username;
GRANT USAGE ON SCHEMA public TO username;

-- Grant membership (group roles)
GRANT readonly TO john_doe;  -- john_doe inherits readonly privileges
GRANT admin TO jane_doe WITH ADMIN OPTION;  -- Can grant admin role to others

-- Revoke privileges
REVOKE INSERT ON users FROM username;
REVOKE ALL PRIVILEGES ON DATABASE mydb FROM username;
REVOKE readonly FROM john_doe;

-- Default privileges (for future objects)
ALTER DEFAULT PRIVILEGES IN SCHEMA public 
GRANT SELECT ON TABLES TO readonly;
ALTER DEFAULT PRIVILEGES FOR USER creator_user
GRANT INSERT ON TABLES TO app_user;

-- Row Level Security (RLS)
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
CREATE POLICY user_access ON users
    USING (id = current_setting('app.current_user_id')::INT);

-- Role attributes
CREATE ROLE admin WITH 
    SUPERUSER 
    CREATEDB 
    CREATEROLE 
    LOGIN 
    PASSWORD 'admin123';

-- List users/roles
\du
SELECT * FROM pg_roles;
SELECT * FROM pg_user;
```

---

## 18. Backup & Restore

### pg_dump (logical backup)
```bash
# Backup single database (plain SQL format)
pg_dump -U postgres -d mydb > mydb.sql

# Backup with custom format (compressed)
pg_dump -U postgres -Fc -d mydb > mydb.dump

# Backup specific tables
pg_dump -U postgres -d mydb -t users -t orders > users_orders.sql

# Backup schema only
pg_dump -U postgres -d mydb --schema-only > schema.sql

# Backup data only
pg_dump -U postgres -d mydb --data-only > data.sql

# Backup with INSERT statements (instead of COPY)
pg_dump -U postgres -d mydb --inserts > mydb.sql

# Backup all databases
pg_dumpall -U postgres > all_databases.sql

# Backup globals (roles, tablespaces)
pg_dumpall -U postgres --globals-only > globals.sql
```

### pg_restore
```bash
# Restore custom/directory format
pg_restore -U postgres -d mydb mydb.dump

# Restore single table
pg_restore -U postgres -d mydb -t users mydb.dump

# List contents of dump file
pg_restore -l mydb.dump

# Restore plain SQL format
psql -U postgres -d mydb < mydb.sql
```

### psql restore
```bash
# Execute SQL file
psql -U postgres -d mydb -f backup.sql

# With error handling
psql -U postgres -d mydb -v ON_ERROR_STOP=1 -f backup.sql
```

### File-based backup (physical)
```bash
# For continuous archiving (WAL)
# Configure postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /archive/%f'

# Take base backup
pg_basebackup -D /backup/base -Fp -P -U replication
```

---

## 19. Performance Tuning

### Query Analysis
```sql
-- EXPLAIN (query plan without execution)
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';

-- EXPLAIN ANALYZE (with execution)
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'john@example.com';

-- Verbose output
EXPLAIN (ANALYZE, BUFFERS, VERBOSE) SELECT * FROM users;

-- JSON format for parsing
EXPLAIN (FORMAT JSON) SELECT * FROM users;

-- Timing and cost
EXPLAIN (ANALYZE, TIMING, COSTS) SELECT * FROM users;
```

### Configuration Parameters
```sql
-- View configuration
SHOW all;
SHOW work_mem;
SHOW shared_buffers;
SHOW effective_cache_size;
SHOW max_connections;

-- Set configuration (session level)
SET work_mem = '256MB';
SET enable_seqscan = off;  -- Temporary, for testing

-- Set configuration (database level)
ALTER DATABASE mydb SET work_mem = '256MB';

-- Set configuration (user level)
ALTER USER postgres SET work_mem = '512MB';

-- Key parameters to tune
-- postgresql.conf settings:
shared_buffers = '25% of RAM'          -- Typically 25%
effective_cache_size = '75% of RAM'    -- 50-75% of RAM
work_mem = '16-64MB'                   -- Per sort/hash operation
maintenance_work_mem = '512MB-2GB'     -- For vacuum, index creation
wal_buffers = '16MB'                   
max_connections = 100-500
random_page_cost = 1.1-4.0             -- 1.1 for SSD, 4 for HDD
autovacuum = on
```

### Vacuum & Analyze
```sql
-- Analyze table (update statistics)
ANALYZE users;
ANALYZE;  -- All tables in current DB

-- Vacuum (clean dead rows)
VACUUM users;
VACUUM FULL users;  -- Reclaim space (locks table)
VACUUM FREEZE;      -- Prevent transaction ID wraparound

-- Verbose vacuum
VACUUM VERBOSE users;

-- Auto-vacuum status
SELECT schemaname, relname, last_vacuum, last_autovacuum, 
       last_analyze, last_autoanalyze, n_dead_tup
FROM pg_stat_user_tables;

-- Manual vacuum of all databases (as superuser)
vacuumdb -U postgres --all --analyze
```

### Performance Queries
```sql
-- Identify missing indexes
SELECT schemaname, tablename, seq_scan, seq_tup_read,
       idx_scan, seq_tup_read / seq_scan AS avg_seq_rows
FROM pg_stat_user_tables
WHERE seq_scan > 0
ORDER BY seq_tup_read DESC LIMIT 10;

-- Identify unused indexes
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read
FROM pg_stat_user_indexes
WHERE idx_scan = 0 AND idx_tup_read = 0;

-- Identify slow queries (requires pg_stat_statements)
CREATE EXTENSION pg_stat_statements;
SELECT query, calls, total_time, mean_time, rows
FROM pg_stat_statements
ORDER BY mean_time DESC LIMIT 10;

-- Table and index sizes
SELECT 
    relname,
    pg_size_pretty(pg_total_relation_size(relid)) as total_size,
    pg_size_pretty(pg_relation_size(relid)) as table_size,
    pg_size_pretty(pg_indexes_size(relid)) as index_size
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(relid) DESC;

-- Cache hit ratio
SELECT 
    sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) AS cache_hit_ratio
FROM pg_statio_user_tables;
```

---

## 20. Monitoring & Diagnostics

```sql
-- Current activity
SELECT pid, usename, application_name, state, query, 
       wait_event_type, wait_event, 
       now() - query_start AS query_duration
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY query_duration DESC;

-- Cancel/kill query
SELECT pg_cancel_backend(pid);   -- Cancel query
SELECT pg_terminate_backend(pid); -- Kill connection

-- Database size
SELECT datname, pg_size_pretty(pg_database_size(datname)) as size
FROM pg_database;

-- Table bloat (estimated)
SELECT schemaname, tablename, 
       pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as size,
       pg_stat_get_live_tuples(schemaname||'.'||tablename) as live_tuples,
       pg_stat_get_dead_tuples(schemaname||'.'||tablename) as dead_tuples
FROM pg_tables
WHERE schemaname = 'public';

-- Connection stats
SELECT datname, numbackends, xact_commit, xact_rollback
FROM pg_stat_database;

-- Lock monitoring
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocking_locks.pid AS blocking_pid,
    blocked_activity.query AS blocked_query
FROM pg_locks blocked_locks
JOIN pg_locks blocking_locks ON blocked_locks.locktype = blocking_locks.locktype
JOIN pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
WHERE blocked_locks.granted = false;

-- Table access stats
SELECT schemaname, relname, 
       seq_scan, seq_tup_read, 
       idx_scan, idx_tup_fetch,
       n_tup_ins, n_tup_upd, n_tup_del
FROM pg_stat_user_tables;

-- Index usage stats
SELECT schemaname, relname, indexrelname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes;

-- WAL stats (PostgreSQL 10+)
SELECT location, xlog_position, redo_location, redo_wal_file
FROM pg_control_checkpoint();

-- Log settings
SHOW log_destination;    -- stderr, csvlog, syslog
SHOW logging_collector;  -- on/off
SHOW log_directory;      -- pg_log
SHOW log_filename;       -- postgresql-%Y-%m-%d_%H%M%S.log
SHOW log_statement;      -- none, ddl, mod, all
SHOW log_min_duration_statement;  -- logs queries slower than ms
```

---

## 21. JSON/JSONB Operations

```sql
-- Create table with JSON column
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    attributes JSONB
);

-- Insert JSON data
INSERT INTO products (name, attributes) VALUES 
('Laptop', '{"brand": "Dell", "ram": 16, "ssd": 512, "color": "black"}'),
('Phone', '{"brand": "Apple", "ram": 8, "color": "silver", "model": "iPhone 14"}');

-- JSON operators
SELECT attributes->'brand' AS brand FROM products;           -- Returns JSON
SELECT attributes->>'brand' AS brand FROM products;         -- Returns TEXT
SELECT attributes->'ram' FROM products;                     -- JSON number
SELECT attributes->>'ram' FROM products;                    -- TEXT number

-- Path access (nested)
SELECT attributes#>'{brand, name}' FROM products;           -- JSON path
SELECT attributes#>>'{brand, name}' FROM products;          -- TEXT path

-- JSONB containment (@>)
SELECT * FROM products WHERE attributes @> '{"brand": "Apple"}';
SELECT * FROM products WHERE attributes @> '{"ram": 16}';
SELECT * FROM products WHERE attributes @> '{"brand": "Dell", "ram": 16}';

-- JSONB existence (?)
SELECT * FROM products WHERE attributes ? 'color';  -- Has key 'color'

-- JSONB array operations
SELECT * FROM products WHERE attributes->'tags' ? 'electronics';  -- Array has value

-- Update JSONB
UPDATE products 
SET attributes = attributes || '{"warranty": "2 years"}' 
WHERE name = 'Laptop';

UPDATE products 
SET attributes = jsonb_set(attributes, '{ram}', '32') 
WHERE name = 'Laptop';

-- Delete key from JSONB
UPDATE products 
SET attributes = attributes - 'color' 
WHERE name = 'Phone';

-- JSONB functions
SELECT jsonb_pretty(attributes) FROM products;
SELECT jsonb_typeof(attributes->'ram') FROM products;  -- number
SELECT jsonb_extract_path(attributes, 'brand', 'name') FROM products;

-- JSONB indexing
CREATE INDEX idx_products_attributes ON products USING GIN (attributes);
CREATE INDEX idx_products_brand ON products ((attributes->>'brand'));  -- Expression

-- JSON aggregation
SELECT json_agg(products) FROM products;
SELECT row_to_json(products) FROM products;
SELECT json_build_object('id', id, 'name', name, 'attrs', attributes) FROM products;

-- JSONB functions (PostgreSQL 12+)
SELECT jsonb_path_query(attributes, '$.brand') FROM products;
SELECT jsonb_path_exists(attributes, '$.ram ? (@ > 8)') FROM products;
```

---

## 22. Full-Text Search

```sql
-- Basic full-text search
SELECT * FROM posts 
WHERE to_tsvector('english', title || ' ' || content) 
      @@ to_tsquery('english', 'database & performance');

-- Create tsvector column
ALTER TABLE posts ADD COLUMN search_vector tsvector;
UPDATE posts SET search_vector = 
    setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
    setweight(to_tsvector('english', coalesce(content, '')), 'B');

-- Index for full-text search
CREATE INDEX idx_posts_search ON posts USING GIN (search_vector);

-- Search using GIN index
SELECT title, content
FROM posts
WHERE search_vector @@ to_tsquery('english', 'database & performance')
ORDER BY ts_rank(search_vector, to_tsquery('database')) DESC;

-- Phrase search (PostgreSQL 12+)
SELECT * FROM posts
WHERE search_vector @@ phraseto_tsquery('english', 'database performance');

-- Websearch support (PostgreSQL 11+)
SELECT * FROM posts
WHERE search_vector @@ websearch_to_tsquery('english', 'database OR performance');

-- Highlighting
SELECT 
    title,
    ts_headline('english', content, to_tsquery('database & performance'))
FROM posts
WHERE search_vector @@ to_tsquery('database & performance');

-- Trigger to maintain tsvector
CREATE FUNCTION posts_tsvector_update() RETURNS trigger AS $$
BEGIN
    NEW.search_vector :=
        setweight(to_tsvector('english', coalesce(NEW.title, '')), 'A') ||
        setweight(to_tsvector('english', coalesce(NEW.content, '')), 'B');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER tsvector_update 
    BEFORE INSERT OR UPDATE ON posts
    FOR EACH ROW EXECUTE FUNCTION posts_tsvector_update();

-- Dictionary management
CREATE TEXT SEARCH DICTIONARY my_dict (
    TEMPLATE = pg_catalog.simple,
    STOPWORDS = english
);

-- Chinese/Japanese/Korean support (requires extension)
CREATE EXTENSION pg_jieba;  -- Chinese word segmentation
```

---

## 23. Partitioning

```sql
-- Range Partitioning (PostgreSQL 10+)
-- Create parent table
CREATE TABLE orders (
    id SERIAL,
    order_date DATE NOT NULL,
    customer_id INT,
    amount DECIMAL
) PARTITION BY RANGE (order_date);

-- Create partitions
CREATE TABLE orders_2024_q1 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');
    
CREATE TABLE orders_2024_q2 PARTITION OF orders
    FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

-- List Partitioning
CREATE TABLE users (
    id SERIAL,
    country_code CHAR(2)
) PARTITION BY LIST (country_code);

CREATE TABLE users_us PARTITION OF users
    FOR VALUES IN ('US', 'CA');
    
CREATE TABLE users_eu PARTITION OF users
    FOR VALUES IN ('UK', 'FR', 'DE');

-- Hash Partitioning
CREATE TABLE logs (
    id SERIAL,
    log_data TEXT
) PARTITION BY HASH (id);

CREATE TABLE logs_p1 PARTITION OF logs
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);
    
CREATE TABLE logs_p2 PARTITION OF logs
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);

-- Default partition
CREATE TABLE orders_other PARTITION OF orders DEFAULT;

-- Attach existing table as partition
ALTER TABLE orders ATTACH PARTITION orders_2024_q3
    FOR VALUES FROM ('2024-07-01') TO ('2024-10-01');

-- Detach partition
ALTER TABLE orders DETACH PARTITION orders_2024_q1;

-- Query partitions directly
SELECT * FROM orders_2024_q1;

-- Trigger-based partitioning (pre-10)
-- Not covered: use native partitioning for PostgreSQL 10+
```

---

## 24. Useful Functions

### String Functions
```sql
-- Basic
LENGTH('text'); CONCAT('a', 'b'); UPPER('text'); LOWER('text');
INITCAP('hello world');  -- Hello World
TRIM(' text '); LTRIM(); RTRIM();
SUBSTRING('text' FROM 2 FOR 2);  -- 'ex'
LEFT('text', 2);  -- 'te'
RIGHT('text', 2);  -- 'xt'
REPLACE('text', 't', 'p');  -- 'pexp'
POSITION('e' IN 'text');  -- 2
STRPOS('text', 'x');  -- 3
REVERSE('text');  -- 'txet'
SPLIT_PART('a,b,c', ',', 2);  -- 'b'
REGEXP_REPLACE('text123', '[0-9]', '', 'g');  -- 'text'
REGEXP_MATCHES('text123', '([a-z]+)([0-9]+)');  -- {'text','123'}
```

### Date/Time Functions
```sql
-- Current time
NOW(); CURRENT_TIMESTAMP; CURRENT_DATE; CURRENT_TIME;
-- Extract components
EXTRACT(YEAR FROM NOW());
EXTRACT(MONTH FROM NOW());
EXTRACT(DOW FROM NOW());  -- Day of week (0-6, Sunday=0)
EXTRACT(EPOCH FROM NOW());  -- Unix timestamp
-- Date arithmetic
NOW() + INTERVAL '1 day';
NOW() - INTERVAL '2 hours';
AGE('2024-12-31', '2024-01-01');  -- 11 mons 30 days
DATE_TRUNC('month', NOW());  -- First day of month
TO_CHAR(NOW(), 'YYYY-MM-DD HH24:MI:SS');
TO_DATE('2024-01-15', 'YYYY-MM-DD');
TO_TIMESTAMP('2024-01-15 10:30:00', 'YYYY-MM-DD HH24:MI:SS');
MAKE_DATE(2024, 1, 15);
MAKE_TIMESTAMP(2024, 1, 15, 10, 30, 0);
JUSTIFY_DAYS(INTERVAL '35 days');  -- 1 mon 5 days
```

### Numeric Functions
```sql
ABS(-10); CEIL(10.1); FLOOR(10.9); ROUND(10.123, 2);
TRUNC(10.456, 2);  -- 10.45 (truncate)
POWER(2, 3); SQRT(16); MOD(10, 3);  -- 1
RANDOM();  -- 0-1
SETSEED(0.5);
DIV(10, 3);  -- 3 (integer division)
EXP(1); LN(2.71828); LOG(100); -- 2 (base 10)
GREATEST(1, 5, 3); LEAST(1, 5, 3);
WIDTH_BUCKET(3.5, 0, 10, 5);  -- Bucket number
```

### Array Functions
```sql
array_length(ARRAY[1,2,3], 1);  -- 3
array_ndims(ARRAY[[1],[2]]);  -- 2
array_to_string(ARRAY[1,2,3], ',');  -- '1,2,3'
string_to_array('a,b,c', ',');  -- {'a','b','c'}
array_cat(ARRAY[1,2], ARRAY[3,4]);  -- {1,2,3,4}
array_append(ARRAY[1,2], 3);  -- {1,2,3}
array_prepend(1, ARRAY[2,3]);  -- {1,2,3}
array_remove(ARRAY[1,2,3,2], 2);  -- {1,3}
array_position(ARRAY[1,2,3], 2);  -- 2
array_positions(ARRAY[1,2,3,2], 2);  -- {2,4}
unnest(ARRAY[1,2,3]);  -- Expand to rows
```

### Conditional Functions
```sql
-- CASE
CASE WHEN condition THEN result END;
COALESCE(value, default);  -- First non-null
NULLIF(value1, value2);  -- NULL if equal
GREATEST(value1, value2, ...);  -- Largest
LEAST(value1, value2, ...);     -- Smallest

-- ISNULL/IFNULL not available in PostgreSQL
-- Use COALESCE instead
```

### Aggregate Functions (Custom)
```sql
-- Create custom aggregate
CREATE FUNCTION sum_square_state(state INT, next INT) 
RETURNS INT AS $$ SELECT state + next * next; $$ LANGUAGE SQL;

CREATE AGGREGATE sum_square(INT) (
    SFUNC = sum_square_state,
    STYPE = INT,
    INITCOND = '0'
);

SELECT sum_square(amount) FROM orders;
```

### UUID Functions
```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

uuid_generate_v1();  -- Time-based
uuid_generate_v4();  -- Random
uuid_generate_v3();  -- MD5 namespace
uuid_generate_v5();  -- SHA-1 namespace

SELECT gen_random_uuid();  -- PostgreSQL 13+
```

---

## 25. System Tables & Views

### Key System Catalogs
```sql
-- Database info
pg_database        -- Databases
pg_tablespace      -- Tablespaces
pg_user            -- Database users

-- Schema info
pg_namespace       -- Schemas
pg_class           -- Tables, indexes, sequences, views
pg_attribute       -- Table columns
pg_type            -- Data types

-- Object info
pg_index           -- Indexes
pg_constraint      -- Constraints  
pg_trigger         -- Triggers
pg_proc            -- Functions/procedures

-- Privileges
pg_authid          -- Authentication IDs (roles)
pg_auth_members    -- Role membership
pg_shadow          -- Password info (superuser only)

-- Extensions
pg_extension       -- Installed extensions
pg_available_extensions  -- Available extensions

-- Statistics
pg_statistic       -- Column statistics (internal)
pg_stats           -- Human-readable statistics
```

### Useful Queries on System Tables
```sql
-- Find tables by column name
SELECT table_schema, table_name, column_name
FROM information_schema.columns
WHERE column_name = 'user_id';

-- Find tables with no primary key
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_schema = 'public'
AND table_name NOT IN (
    SELECT table_name 
    FROM information_schema.table_constraints 
    WHERE constraint_type = 'PRIMARY KEY'
);

-- Constraint info
SELECT * FROM information_schema.constraint_column_usage;
SELECT * FROM information_schema.key_column_usage;
SELECT * FROM information_schema.referential_constraints;

-- Find large unused indexes
SELECT 
    schemaname,
    indexname,
    tablename,
    pg_size_pretty(pg_relation_size(indexname::regclass))
FROM pg_indexes
WHERE indexname NOT IN (
    SELECT indexrelname::text 
    FROM pg_stat_user_indexes 
    WHERE idx_scan > 0
);

-- Search function source code
SELECT proname, prosrc 
FROM pg_proc 
WHERE proname LIKE '%search%';

-- Dependency chains
SELECT * FROM pg_constraint WHERE conrelid = 'users'::regclass;
SELECT * FROM pg_depend WHERE objid = 'users'::regclass;

-- Table inheritance
SELECT inhrelid::regclass, inhparent::regclass 
FROM pg_inherits;
```

---

## Quick Reference

### Command-line Shortcuts
```bash
psql -l                           # List databases
psql -d mydb -c "\dt"            # List tables
pg_dump --help                   # Backup help
pg_restore --help                # Restore help
vacuumdb --all --analyze         # Vacuum all databases
createdb mydb                    # Create database
dropdb mydb                      # Drop database
createuser username              # Create user
dropuser username                # Drop user
```

### Common Snippets
```sql
-- Generate random data
INSERT INTO users (name, email) 
SELECT 
    'User ' || generate_series,
    'user' || generate_series || '@example.com'
FROM generate_series(1, 1000);

-- Find duplicate emails
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;

-- Pivot table (crosstab)
CREATE EXTENSION tablefunc;
SELECT * FROM crosstab(
    'SELECT product, month, sales FROM sales ORDER BY 1,2'
) AS ct(product TEXT, jan INT, feb INT, mar INT);

-- Find overlapping date ranges
SELECT * FROM events e1, events e2
WHERE e1.id != e2.id 
  AND tsrange(e1.start_date, e1.end_date) && tsrange(e2.start_date, e2.end_date);

-- Recursive query (tree structure)
WITH RECURSIVE tree AS (
    SELECT id, name, parent_id, 1 AS depth
    FROM categories WHERE parent_id IS NULL
    UNION ALL
    SELECT c.id, c.name, c.parent_id, t.depth + 1
    FROM categories c JOIN tree t ON c.parent_id = t.id
)
SELECT * FROM tree;

-- Generate series of dates
SELECT generate_series(
    '2024-01-01'::date,
    '2024-12-31'::date,
    '1 day'::interval
) AS date;

-- Moving average
SELECT 
    date,
    amount,
    AVG(amount) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg_7d
FROM sales;
```

---

## Version Information

```sql
SELECT version();
SHOW server_version;
SELECT current_setting('server_version_num');
SELECT pg_config;
```

---
