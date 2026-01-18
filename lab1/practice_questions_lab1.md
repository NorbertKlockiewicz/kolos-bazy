# Lab 1: OLAP/Star Schema/ETL - Practice Questions

## Question 1: Fix Schema + Write Query (Similar to Test Question 3)

### Given the following flawed star schema:

```sql
BEGIN;

CREATE TABLE dim_date(
    date_id VARCHAR,
    day INTEGER,
    month INTEGER,
    year INTEGER,
    quarter INTEGER
);

CREATE TABLE dim_product(
    product_id INTEGER,
    product_name VARCHAR,
    category VARCHAR(50),
    unit_price NUMERIC(8,2)
);

CREATE TABLE dim_store(
    store_code CHAR(10),
    store_name VARCHAR(100),
    city VARCHAR,
    country VARCHAR(50)
);

CREATE TABLE fact_sales(
    date_id INTEGER,
    product_id INTEGER,
    store_id VARCHAR(10),
    units_sold INTEGER,
    total_revenue NUMERIC(12,2),
    discount_amount NUMERIC(8,2)
);

COMMIT;
```

### Tasks:
1. **Identify ALL mistakes** in the schema above (keep it as a star model)
 - a. no primary keys in dim tables
 - b. no primary key in fact_sales consisting of foreign keys from dim tables
 - c. date_id type mismatch
 - d. No FOREIGN KEY constraints in fact_sales
 - e. VARCHAR without length in dim_date (date_id), dim_product (product_name), dim_store (city)
 - f. No NOT NULL constraints on critical fields
 - g. No UNIQUE constraint on natural keys in dimensions
 - h. store_id data type mismatch (VARCHAR(10) in fact_sales, but store_code is CHAR(10) in dim_store)
2. **Write a query** that returns: Total revenue and average discount for products in category 'Electronics' sold in Germany during Q3 2023
   ```
    SELECT SUM(f.total_revenue) AS total_revenue, AVG(f.discount_amount) AS avg_discount FROM fact_sales f
    JOIN dim_date d ON f.date_id = d.date_id
    JOIN dim_product p ON f.product_id = p.product_id
    JOIN dim_store s ON f.store_id = s.store_code
    WHERE p.category = 'Electronics' AND s.country = 'Germany' AND d.year = 2023
    AND d.month BETWEEN 7 AND 9
   ```

---

## Question 2: Design Star Schema from OLTP

### Given the following OLTP tables:

```sql
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(50),
    country VARCHAR(50),
    registration_date DATE
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(customer_id),
    order_date DATE,
    shipping_address VARCHAR(200),
    total_amount NUMERIC(10,2),
    status VARCHAR(20)
);

CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id),
    product_name VARCHAR(100),
    quantity INTEGER,
    unit_price NUMERIC(8,2),
    category VARCHAR(50)
);
```

### Tasks:
1. **Design a star schema** with at least 3 dimensions (time must have monthly granularity)
2. **Write CREATE TABLE statements** for all dimension and fact tables
3. **Write ETL queries** (INSERT INTO...SELECT) to load:
   - Time dimension
   - Customer dimension
   - One fact table

---

## Question 3: Write Analytical Queries

### Given this star schema:

```sql
CREATE TABLE dim_time (
    time_key SERIAL PRIMARY KEY,
    date DATE NOT NULL UNIQUE,
    day INTEGER NOT NULL,
    month INTEGER NOT NULL,
    year INTEGER NOT NULL,
    quarter INTEGER NOT NULL,
    month_name VARCHAR(20) NOT NULL
);

CREATE TABLE dim_customer (
    customer_key SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL UNIQUE,
    customer_name VARCHAR(100) NOT NULL,
    city VARCHAR(50) NOT NULL,
    country VARCHAR(50) NOT NULL
);

CREATE TABLE dim_product (
    product_key SERIAL PRIMARY KEY,
    product_id VARCHAR(20) NOT NULL UNIQUE,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL,
    subcategory VARCHAR(50)
);

CREATE TABLE fact_orders (
    order_key SERIAL PRIMARY KEY,
    time_key INTEGER NOT NULL REFERENCES dim_time(time_key),
    customer_key INTEGER NOT NULL REFERENCES dim_customer(customer_key),
    product_key INTEGER NOT NULL REFERENCES dim_product(product_key),
    quantity INTEGER NOT NULL,
    revenue NUMERIC(10,2) NOT NULL,
    cost NUMERIC(10,2) NOT NULL,
    profit NUMERIC(10,2) NOT NULL
);
```

### Write queries for:

1. **Total revenue by category** for year 2023, ordered by revenue descending
   ```
   SELECT SUM(f.revenue) AS total_revenue, p.category FROM fact_orders f
   JOIN dim_product p ON f.product_key = p.product_key
   JOIN dim_time t ON f.time_key = t.time_key
   WHERE t.year = 2023 GROUP BY p.category ORDER BY total_revenue DESC
    ```

2. **Monthly profit trend** for Q1 2024 (show month name, total profit)
    ```
    SELECT t.month, SUM(f.profit) AS total_profit FROM fact_orders f
    JOIN dim_time t ON f.time_key = t.time_key
    WHERE t.year = 2024 AND t.month BETWEEN 1 AND 3
    GROUP BY t.month;
    ```

3. **Top 5 customers** by total revenue in France during 2023
    ```
    SELECT c.customer_name, SUM(f.revenue) AS total_revenue FROM fact_orders f
    JOIN dim_customer c ON f.customer_key = c.customer_key
    JOIN dim_time t ON f.time_key = t.time_key
    WHERE t.year = 2023 AND c.country = 'France'
    GROUP BY c.customer_name
    ORDER BY total_revenue DESC LIMIT 5
    ```


4. **Average order value** by country and quarter for 2023
    ```
    SELECT c.country, t.quarter, AVG(f.revenue) AS average_order_value FROM fact_orders f
    JOIN dim_customer c ON f.customer_key = c.customer_key
    JOIN dim_time t ON f.time_key = t.time_key
    WHERE t.year = 2023
    GROUP BY c.country, t.quarter
    ORDER BY t.quarter
    ```

5. **Products with negative profit** (loss) in any month of 2023, show product name, month, and profit
    ```
    SELECT t.month_name, p.product_name, SUM(f.profit) AS monthly_profit FROM fact_orders f
    JOIN dim_product p ON f.product_key = p.product_key
    JOIN dim_time t ON f.time_key = t.time_key
    WHERE t.year = 2023
    GROUP BY p.product_name, t.month_name
    HAVING monthly_profit < 0
    ```
---

## Question 4: Fix ETL Query

### Given this incorrect ETL query:

```sql
-- Loading fact table
INSERT INTO fact_sales (time_key, customer_key, product_key, amount, quantity)
SELECT
    dt.time_key,
    dc.customer_key,
    dp.product_key,
    o.total_amount,
    o.quantity
FROM orders o
LEFT JOIN dim_time dt ON o.order_date = dt.date
LEFT JOIN dim_customer dc ON o.customer_id = dc.customer_id
LEFT JOIN dim_product dp ON o.product_id = dp.product_id
ORDER BY o.order_id;
```

### Tasks:
1. **Identify what's wrong** with this ETL query
   We should use INNER JOIN instead of LEFT JOIN
2. **Provide the corrected version**
   ```
    INSERT INTO fact_sales (time_key, customer_key, product_key, amount, quantity)
    SELECT
        dt.time_key,
        dc.customer_key,
        dp.product_key,
        o.total_amount,
        o.quantity
    FROM orders o
    INNER JOIN dim_time dt ON o.order_date = dt.date
    INNER JOIN dim_customer dc ON o.customer_id = dc.customer_id
    INNER JOIN dim_product dp ON o.product_id = dp.product_id
    ORDER BY o.order_id;
```

---

## Question 5: Schema Design Mistakes Checklist

### For the schema below, list ALL mistakes:

```sql
CREATE TABLE time_dimension(
    day INTEGER,
    month INTEGER,
    year INTEGER,
    full_date DATE
);

CREATE TABLE customer_dimension(
    cust_key INTEGER PRIMARY KEY,
    cust_id VARCHAR,
    name VARCHAR,
    country VARCHAR(50)
);

CREATE TABLE sales_fact(
    time_id DATE,
    customer_id INTEGER,
    product_id INTEGER,
    revenue NUMERIC(10,2),
    FOREIGN KEY (customer_id) REFERENCES customer_dimension(cust_key)
);
```

### Task:
Write the **completely corrected version** of all three tables.

```
CREATE TABLE time_dimension(
    time_id DATE PRIMARY KEY,
    day INTEGER,
    month INTEGER,
    year INTEGER,
    full_date DATE
);

CREATE TABLE customer_dimension(
    cust_key INTEGER PRIMARY KEY,
    cust_id VARCHAR,
    name VARCHAR,
    country VARCHAR(50)
);

CREATE TABLE sales_fact(
    time_id DATE,
    customer_id INTEGER,
    product_id INTEGER,
    revenue NUMERIC(10,2),
    FOREIGN KEY (customer_id) REFERENCES customer_dimension(cust_key)
    FOREIGN KEY (time_id) REFERENCES time_dimension(time_id)

    PRIMARY KEY (time_id, customer_id, product_id)
);
```

---

# Tips for Lab 1 Questions:

## Common Schema Mistakes to Look For:
1. ❌ Missing PRIMARY KEY in dimension tables
2. ❌ Missing FOREIGN KEY in fact table
3. ❌ Data type mismatches between fact FK and dimension PK
4. ❌ VARCHAR without length specified
5. ❌ Missing NOT NULL constraints on critical fields
6. ❌ Missing UNIQUE constraints on natural keys in dimensions
7. ❌ No primary key in fact table (or wrong composite key)

## Query Writing Tips:
1. ✅ Always JOIN fact table with dimension tables
2. ✅ Use WHERE for filtering (city, country, date ranges)
3. ✅ Use aggregate functions: SUM(), AVG(), COUNT(), MIN(), MAX()
4. ✅ Use GROUP BY when you want results per category/month/country
5. ✅ For quarters: `WHERE quarter = 3` or `WHERE month BETWEEN 7 AND 9`
6. ✅ For time ranges: `WHERE year = 2023 AND month BETWEEN 1 AND 6`

## ETL Tips:
1. ✅ Use INNER JOIN (not LEFT JOIN) to avoid NULLs in fact table
2. ✅ Load dimensions FIRST, then fact table
3. ✅ Use INSERT INTO...SELECT pattern
4. ✅ Join on natural keys (customer_id, product_id) not surrogate keys
5. ✅ Filter out NULLs in critical fields with WHERE clauses
