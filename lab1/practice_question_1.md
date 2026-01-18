# Practice Question 1: Fix the Schema + Write Query

## Given the following flawed star schema:

```sql
BEGIN;

CREATE TABLE time_dim(
    day_id INTEGER,
    day INTEGER,
    month INTEGER,
    year INTEGER,
    full_date DATE
);

CREATE TABLE product_dim(
    prod_id VARCHAR,
    product_name VARCHAR,
    category VARCHAR,
    price NUMERIC(8,2)
);

CREATE TABLE customer_dim(
    cust_id INTEGER,
    customer_name VARCHAR(100),
    city VARCHAR(50),
    country VARCHAR(50)
);

CREATE TABLE fact_orders(
    time_id INTEGER,
    product_id CHAR(10),
    customer_id INTEGER,
    quantity INTEGER,
    total_amount NUMERIC(10,2)
);

COMMIT;
```

## Task:
1. Identify and fix ALL mistakes in the schema above (keep it as a star model)
2. Write a query that returns: **Total sales amount and average order quantity for customers from Poland in Q2 2023**

---

## Your Answer:

### Part 1 - Mistakes Identified:

✅ **Correct:**
1. Missing foreign keys in fact_orders
2. Data type mismatch (product_id vs prod_id)
3. Missing primary key in fact table

❌ **You Missed:**
4. **No PRIMARY KEY in any dimension tables** (day_id, prod_id, cust_id should be PKs)
5. **VARCHAR without length** in product_dim (prod_id, category, price columns)
6. **No UNIQUE constraints** on dimension tables
7. **No NOT NULL constraints** on critical fields

### Part 2 - Your Query:

```sql
Select SUM(total_amount), AVG(quantity)
FROM fact_orders f
JOIN customer_dim as c ON c.cust_id = f.customer_id
JOIN time_dim t ON t.day_id = f.time_id
WHERE t.year == 2023
  AND t.month BETWEEN 4 AND 6
  AND c.country == POLAND
GROUP BY t.year
```

**Errors:**
❌ `WHERE t.year == 2023` → Should be `=` (single equals)
❌ `c.country == POLAND` → Should be `=` and `'Poland'` (string needs quotes)
❌ `GROUP BY t.year` → **Wrong grouping!** You're asking for total/average across all Q2, not per year

---

## Correct Solutions:

### Part 1 - Fixed Schema:

```sql
BEGIN;

CREATE TABLE time_dim(
    day_id INTEGER PRIMARY KEY,
    day INTEGER NOT NULL,
    month INTEGER NOT NULL,
    year INTEGER NOT NULL,
    full_date DATE NOT NULL,
    UNIQUE(full_date)
);

CREATE TABLE product_dim(
    prod_id VARCHAR(10) PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(50),
    price NUMERIC(8,2)
);

CREATE TABLE customer_dim(
    cust_id INTEGER PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    city VARCHAR(50),
    country VARCHAR(50) NOT NULL
);

CREATE TABLE fact_orders(
    time_id INTEGER NOT NULL,
    product_id VARCHAR(10) NOT NULL,
    customer_id INTEGER NOT NULL,
    quantity INTEGER NOT NULL,
    total_amount NUMERIC(10,2) NOT NULL,
    PRIMARY KEY (time_id, product_id, customer_id),
    FOREIGN KEY (time_id) REFERENCES time_dim(day_id),
    FOREIGN KEY (product_id) REFERENCES product_dim(prod_id),
    FOREIGN KEY (customer_id) REFERENCES customer_dim(cust_id)
);

COMMIT;
```

### Part 2 - Fixed Query:

```sql
SELECT
    SUM(f.total_amount) AS total_sales,
    AVG(f.quantity) AS avg_quantity
FROM fact_orders f
JOIN customer_dim c ON c.cust_id = f.customer_id
JOIN time_dim t ON t.day_id = f.time_id
WHERE t.year = 2023
  AND t.month BETWEEN 4 AND 6
  AND c.country = 'Poland';
```

**Note:** No GROUP BY needed here because you want ONE total result for all of Q2 2023 Poland.

---

## Key Takeaways:

1. **Always check ALL tables** for PRIMARY KEYs, not just the fact table
2. **VARCHAR must have length** specified
3. **NOT NULL** on critical fields prevents data quality issues
4. **SQL uses single `=`** for comparison (not `==`)
5. **Strings need quotes** ('Poland', not POLAND)
6. **GROUP BY** is only needed when you want multiple rows in the result
