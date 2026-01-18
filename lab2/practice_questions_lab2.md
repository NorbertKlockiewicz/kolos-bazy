# Lab 2: Indexes - Practice Questions

## Question 1: Create Indexes to Speed Up Queries (Similar to Test Question 4)

### Given the following table:

```sql
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    department VARCHAR(50),
    salary NUMERIC(10,2),
    hire_date DATE,
    city VARCHAR(50),
    notes TEXT
);
```

### And these queries:

```sql
-- Query 1
SELECT * FROM employees
WHERE hire_date BETWEEN '2023-01-01' AND '2023-12-31'
ORDER BY hire_date;

-- Query 2
SELECT * FROM employees
WHERE department = 'Engineering' AND salary > 80000;

-- Query 3
SELECT * FROM employees
WHERE last_name LIKE 'Kow%';
```

### Tasks:
1. **Create indexes** to speed up each query
   query 1:
   ```
   CREATE INDEX idx_name ON employees(hire_date);
   ```
   query 2:
   ```
   CREATE INDEX idx_name ON employees(department, salary) WHERE `salary` > 80000;
   ```
   query 3:
   ```
   CREATE INDEX idx_name ON table(last_name varchar_pattern_ops);
   ```

2. **Provide the CREATE INDEX statements**
3. **Explain why each index helps** the corresponding query

---

## Question 2: Multi-Column Index Behavior

### Given:

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    category VARCHAR(50),
    subcategory VARCHAR(50),
    brand VARCHAR(50),
    name VARCHAR(100),
    price NUMERIC(8,2)
);

CREATE INDEX idx_category_subcategory_brand
ON products(category, subcategory, brand);
```

### Questions:

**Which of these queries will use the index? Explain why or why not:**

1. `SELECT * FROM products WHERE category = 'Electronics';`
   will wokr

2. `SELECT * FROM products WHERE subcategory = 'Laptops';`
   won't work

3. `SELECT * FROM products WHERE category = 'Electronics' AND subcategory = 'Laptops';`
   will work

4. `SELECT * FROM products WHERE category = 'Electronics' AND brand = 'Apple';`
   will use but only for the category

5. `SELECT * FROM products WHERE brand = 'Apple';`
   won't work

**Write the rule** for when a multi-column index can be used.

---

## Question 3: Pattern Matching and Indexes

### Given:

```sql
CREATE TABLE books (
    book_id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    author VARCHAR(100),
    isbn VARCHAR(20)
);

CREATE INDEX idx_title ON books(title);
```

### Questions:

**For each query below, state whether the index will be used and why:**

1. `SELECT * FROM books WHERE title = 'Database Systems';`
   will work

2. `SELECT * FROM books WHERE title LIKE 'Database%';`
   won't work

3. `SELECT * FROM books WHERE title LIKE '%Database%';`
   won't work

4. `SELECT * FROM books WHERE title LIKE '%Systems';`
   won't work

5. `SELECT * FROM books WHERE LOWER(title) LIKE 'database%';`
   won't work

### Task:
**For queries that don't use the index, provide a solution** (either create a different index or modify the query).
   `CREATE INDEX idx_name ON books(title varchar_pattern_ops);` - query 2
   ``
   `CREATE INDEX idx_name ON books(lower(title) varchar_pattern_ops);` - query 4, 5



---

## Question 4: Partial Indexes

### Scenario:

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date DATE,
    status VARCHAR(20), -- 'pending', 'completed', 'cancelled'
    total_amount NUMERIC(10,2)
);
```

You frequently run queries to find pending orders, but pending orders are only 5% of all orders.

### Tasks:

1. **Create a partial index** to optimize queries for pending orders

2. **Write a query** that will use this partial index

3. **Write a query** that will NOT use this partial index (even though it queries the same column)

4. **Explain** when partial indexes are beneficial

---

## Question 5: Expression Indexes

### Given:

```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(100),
    full_name VARCHAR(100),
    created_at TIMESTAMP
);
```

### You frequently run these queries:

```sql
-- Query 1: Case-insensitive email search
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';

-- Query 2: Extract year from timestamp
SELECT * FROM users WHERE EXTRACT(YEAR FROM created_at) = 2023;

-- Query 3: Last name search (full_name = 'First Last')
SELECT * FROM users WHERE split_part(full_name, ' ', 2) = 'Smith';
```

### Tasks:

1. **Create expression indexes** to speed up each query

2. **Explain** what expression indexes are and when to use them

---

## Question 6: Index Selection Strategy

### Given the WeatherData table from the test:

```sql
CREATE TABLE WeatherData (
    WeatherID SERIAL PRIMARY KEY,
    City VARCHAR(50),
    MeasurementDate DATE,
    Temperature FLOAT,
    Humidity INT,
    WindSpeed FLOAT,
    Precipitation FLOAT,
    WeatherDescription VARCHAR(100)
);
```

### And these query patterns:

```sql
-- Pattern A: Frequent queries by date range
SELECT City, AVG(Temperature), AVG(Humidity)
FROM WeatherData
WHERE MeasurementDate BETWEEN '2025-01-01' AND '2025-01-31'
GROUP BY City;

-- Pattern B: Frequent queries with multiple conditions
SELECT *
FROM WeatherData
WHERE WindSpeed > 15 AND Precipitation > 0.2;

-- Pattern C: Frequent pattern matching
SELECT *
FROM WeatherData
WHERE WeatherDescription LIKE '%Cloudy%';

-- Pattern D: Frequent sorting
SELECT * FROM WeatherData
ORDER BY City, MeasurementDate DESC;
```

### Tasks:

1. **Design an index strategy** for this table (create 2-4 indexes)
   A - classic b-tree index
   B - multicolumn index
   C - full text search
   D - classic B-tree index

2. **Justify each index** (which queries it helps, why)

3. **Explain any trade-offs** (why you didn't index everything)

---

## Question 7: GiST Indexes for Spatial Data

### Given:

```sql
CREATE TABLE restaurants (
    restaurant_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    location POINT, -- coordinates (x, y)
    cuisine VARCHAR(50),
    rating NUMERIC(2,1)
);

-- Random locations between -100 and 100
-- Example: location = '(45.5, -73.6)'
```

### Tasks:

1. **Create a GiST index** on the location column

2. **Write a query** to find all restaurants within 10 units of point (0, 0)

3. **Write a query** to find all restaurants in the northeast quadrant (x > 0 AND y > 0)

4. **Explain** when to use GiST vs B-tree indexes

---

## Question 8: Index Analysis with EXPLAIN

### Given:

```sql
CREATE TABLE sales (
    sale_id SERIAL PRIMARY KEY,
    product_id INTEGER,
    customer_id INTEGER,
    sale_date DATE,
    amount NUMERIC(10,2)
);

-- 1 million rows
-- 10,000 distinct product_id values
-- 50,000 distinct customer_id values
```

### Scenario:
You run: `SELECT * FROM sales WHERE product_id = 12345;`

The query returns 100 rows out of 1 million (0.01% selectivity).

### Questions:

1. **Should you create an index** on product_id? Why or why not?

2. If you run: `SELECT * FROM sales WHERE product_id > 5000;` (returns 50% of rows), **will the index be used?** Why or why not?

3. **What PostgreSQL configuration** can you change to force the planner to use the index?

---

# Tips for Lab 2 Questions:

## Index Types:

### B-tree Index (default)
- **Use for:** Equality (=), comparison (<, >, <=, >=), BETWEEN, IN, IS NULL
- **Use for:** Sorting (ORDER BY)
- **Pattern matching:** Only prefix searches (LIKE 'abc%') in C locale or with varchar_pattern_ops

### GiST Index
- **Use for:** Geometric data, spatial queries
- **Operators:** @>, <@, &&, <->, etc.
- **Use for:** Box containment, point-in-polygon, nearest neighbor

### Expression Index
- **Use for:** Queries on functions/expressions
- **Example:** Index on LOWER(column) for case-insensitive search

### Partial Index
- **Use for:** Queries on subset of data
- **Example:** WHERE status = 'active' when active is only 5% of data

## Multi-Column Index Rules:

✅ **Can use index:**
- Query uses first column(s) in index order
- `WHERE col1 = ... AND col2 = ...` (uses both)
- `WHERE col1 = ...` (uses first column only)

❌ **Cannot use index:**
- Query skips first column
- `WHERE col2 = ...` (skips col1)
- `WHERE col1 = ... AND col3 = ...` (skips col2)

## Pattern Matching with Indexes:

| Pattern | Index Used? | Notes |
|---------|-------------|-------|
| `LIKE 'abc%'` | ✅ Yes (with varchar_pattern_ops or C locale) | Prefix search |
| `LIKE '%abc'` | ❌ No | Suffix search - needs expression index on reverse() |
| `LIKE '%abc%'` | ❌ No | Contains - consider full-text search |
| `ILIKE 'abc%'` | ❌ No | Case-insensitive - needs expression index on LOWER() |

## When Planner Won't Use Index:

1. **High selectivity** - Query returns > 10-20% of rows (seq scan faster)
2. **Table too small** - Seq scan faster than index scan for small tables
3. **Statistics outdated** - Run ANALYZE to update statistics
4. **Cost estimation** - Adjust random_page_cost or enable_seqscan settings

## CREATE INDEX Syntax Examples:

```sql
-- Simple B-tree
CREATE INDEX idx_name ON table(column);

-- Multi-column
CREATE INDEX idx_name ON table(col1, col2, col3);

-- Partial
CREATE INDEX idx_name ON table(column) WHERE condition;

-- Expression
CREATE INDEX idx_name ON table(LOWER(column));
CREATE INDEX idx_name ON table(EXTRACT(YEAR FROM date_column));

-- Pattern matching
CREATE INDEX idx_name ON table(column varchar_pattern_ops);

-- GiST
CREATE INDEX idx_name ON table USING GIST(geometry_column);
```
