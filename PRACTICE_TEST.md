# Practice Test - Advanced Database Systems

**Time: 40-50 minutes**
**Total Points: 40 (4 questions × 10 points each)**

**Instructions:**
- Write all queries on paper (simulate real test conditions)
- You can check syntax after finishing, but try to write correct queries first
- Focus on correctness and completeness
- Partial credit given for partially correct answers

---

## Question 1: PostGIS - Spatial Analysis (10 points)

### Given:

```sql
CREATE TABLE Countries (
    CountryID SERIAL PRIMARY KEY,
    Name VARCHAR(100),
    Boundary GEOMETRY(MULTIPOLYGON, 4326)
);

INSERT INTO Countries (Name, Boundary)
VALUES
('Poland', ST_SetSRID(ST_GeomFromText('MULTIPOLYGON(((14 49, 24 49, 24 54, 14 54, 14 49)))'), 4326)),
('Germany', ST_SetSRID(ST_GeomFromText('MULTIPOLYGON(((6 47, 15 47, 15 55, 6 55, 6 47)))'), 4326)),
('Czech Republic', ST_SetSRID(ST_GeomFromText('MULTIPOLYGON(((12 48, 19 48, 19 51, 12 51, 12 48)))'), 4326));
```

And:

```sql
CREATE TABLE Cities (
    CityID SERIAL PRIMARY KEY,
    Name VARCHAR(100),
    Population INTEGER,
    Location GEOMETRY(POINT, 4326)
);

INSERT INTO Cities (Name, Population, Location)
VALUES
('Warsaw', 1800000, ST_SetSRID(ST_MakePoint(21.0122, 52.2297), 4326)),
('Berlin', 3700000, ST_SetSRID(ST_MakePoint(13.4050, 52.5200), 4326)),
('Prague', 1300000, ST_SetSRID(ST_MakePoint(14.4378, 50.0755), 4326)),
('Krakow', 780000, ST_SetSRID(ST_MakePoint(19.9445, 50.0647), 4326));
```

### Tasks:

**a) [4 points]** Write a query that computes the area of each country in square kilometers. Use EPSG:3035 (Europe) for accurate measurements.

**b) [6 points]** Write a query that returns cities located in Poland with their population. Include only cities that are actually within Polish boundaries (use ST_Contains).

---

## Question 2: Neo4j - Graph Analysis (10 points)

### Scenario:

You need to model a software deployment pipeline. Components are deployed in sequence:

```
[App:Web]     [App:Mobile]
    |              |
    v              v
[Service:API]  [Service:Auth]
    |              |
    +------+-------+
           |
           v
    [Cache:Redis]
```

Each component has:
- `type`: App, Service, or Cache
- `name`: Web, Mobile, API, Auth, Redis
- `deploy_time`: how long it takes to deploy (in minutes)

Relationships show deployment dependencies (must deploy parent before child).

### Tasks:

**a) [4 points]** Write Cypher queries to create the complete graph with all nodes and :DEPENDS_ON relationships. Include deploy_time property for each node: Web=10, Mobile=8, API=15, Auth=12, Redis=5.

**b) [6 points]** Write a query that finds all components that depend on Cache:Redis (directly or indirectly) and calculates the total deployment time for the entire dependency chain starting from App:Web.

---

## Question 3: OLAP/Star Schema (10 points)

### Given this flawed star schema:

```sql
BEGIN;

CREATE TABLE time(
    month INTEGER,
    year INTEGER,
    full_date DATE
);

CREATE TABLE customer(
    customer_code VARCHAR,
    customer_name VARCHAR,
    country VARCHAR(50),
    registration_date DATE
);

CREATE TABLE product(
    prod_id INTEGER,
    name VARCHAR(100),
    category VARCHAR
);

CREATE TABLE orders(
    time_id DATE,
    cust_id VARCHAR,
    product_id INTEGER,
    order_amount NUMERIC(10,2),
    quantity INTEGER
);

COMMIT;
```

### Tasks:

**a) [4 points]** Identify ALL mistakes in this schema and write the corrected version. Keep it as a star model.

**b) [6 points]** Write a query that returns total order amount and average quantity for products in category 'Electronics' sold to customers from Germany in Q2 2024 (April-June).

---

## Question 4: Indexes (10 points)

### Given:

```sql
CREATE TABLE Transactions (
    TransactionID SERIAL PRIMARY KEY,
    AccountNumber VARCHAR(20),
    TransactionDate DATE,
    Amount NUMERIC(10,2),
    TransactionType VARCHAR(10), -- 'debit' or 'credit'
    Description TEXT,
    Status VARCHAR(20) -- 'pending', 'completed', 'failed'
);

-- Table contains 10 million rows
-- 95% of transactions have status = 'completed'
-- 4% have status = 'pending'
-- 1% have status = 'failed'
```

### Common queries:

```sql
-- Query 1: Find transactions by date range
SELECT * FROM Transactions
WHERE TransactionDate BETWEEN '2024-01-01' AND '2024-01-31';

-- Query 2: Find all pending transactions (run very frequently)
SELECT * FROM Transactions
WHERE Status = 'pending'
ORDER BY TransactionDate;

-- Query 3: Pattern matching on description
SELECT * FROM Transactions
WHERE Description LIKE 'Payment to ABC%';
```

### Tasks:

**a) [6 points]** Create appropriate indexes to speed up all three queries. Provide CREATE INDEX statements and explain why each index helps.

**b) [4 points]** For Query 2, explain whether you would use a regular index or a partial index, and why. What's the trade-off?

---

# Answer Key & Grading Rubric

## Question 1: PostGIS

### a) Area calculation (4 points)
```sql
SELECT
    Name,
    ST_Area(ST_Transform(Boundary, 3035)) / 1000000 AS area_km2
FROM Countries;
```

**Grading:**
- ST_Transform to EPSG:3035 (2 points)
- ST_Area function (1 point)
- Division by 1000000 to convert to km² (1 point)

### b) Cities in Poland (6 points)
```sql
SELECT
    c.Name,
    c.Population
FROM Cities c
JOIN Countries co ON ST_Contains(co.Boundary, c.Location)
WHERE co.Name = 'Poland';
```

**Grading:**
- JOIN with Countries (2 points)
- ST_Contains correctly used (2 points)
- WHERE filter for Poland (1 point)
- SELECT correct columns (1 point)

---

## Question 2: Neo4j

### a) Create graph (4 points)
```cypher
CREATE
  (web:App {type: 'App', name: 'Web', deploy_time: 10}),
  (mobile:App {type: 'App', name: 'Mobile', deploy_time: 8}),
  (api:Service {type: 'Service', name: 'API', deploy_time: 15}),
  (auth:Service {type: 'Service', name: 'Auth', deploy_time: 12}),
  (redis:Cache {type: 'Cache', name: 'Redis', deploy_time: 5}),
  (web)-[:DEPENDS_ON]->(api),
  (mobile)-[:DEPENDS_ON]->(auth),
  (api)-[:DEPENDS_ON]->(redis),
  (auth)-[:DEPENDS_ON]->(redis);
```

**Grading:**
- All nodes created with correct labels (2 points)
- All properties included (1 point)
- All relationships correct (1 point)

### b) Dependencies and total time (6 points)
```cypher
MATCH path = (start:App {name: 'Web'})-[:DEPENDS_ON*]->(redis:Cache {name: 'Redis'})
WITH nodes(path) AS components
UNWIND components AS c
RETURN sum(c.deploy_time) AS total_deployment_time;
```

OR:

```cypher
MATCH (redis:Cache {name: 'Redis'})
MATCH (comp)-[:DEPENDS_ON*]->(redis)
RETURN comp.name, comp.type
UNION
MATCH (redis:Cache {name: 'Redis'})
RETURN redis.name, redis.type;

// And for total time from Web:
MATCH path = (:App {name: 'Web'})-[:DEPENDS_ON*]->(:Cache {name: 'Redis'})
WITH nodes(path) AS components
UNWIND components AS c
RETURN sum(c.deploy_time) AS total_time;
```

**Grading:**
- Finding components that depend on Redis (3 points)
- Path from Web to Redis (2 points)
- Calculating total deployment time (1 point)

---

## Question 3: OLAP

### a) Corrected schema (4 points)
```sql
BEGIN;

CREATE TABLE time(
    time_id SERIAL PRIMARY KEY,
    month INTEGER NOT NULL,
    year INTEGER NOT NULL,
    full_date DATE NOT NULL UNIQUE
);

CREATE TABLE customer(
    customer_id SERIAL PRIMARY KEY,
    customer_code VARCHAR(20) NOT NULL UNIQUE,
    customer_name VARCHAR(100) NOT NULL,
    country VARCHAR(50) NOT NULL,
    registration_date DATE
);

CREATE TABLE product(
    product_id SERIAL PRIMARY KEY,
    prod_id INTEGER NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL
);

CREATE TABLE orders(
    order_id SERIAL PRIMARY KEY,
    time_id INTEGER NOT NULL REFERENCES time(time_id),
    customer_id INTEGER NOT NULL REFERENCES customer(customer_id),
    product_id INTEGER NOT NULL REFERENCES product(product_id),
    order_amount NUMERIC(10,2) NOT NULL,
    quantity INTEGER NOT NULL
);

COMMIT;
```

**Key mistakes:**
- Missing PRIMARY KEYs in all tables
- Missing FOREIGN KEYs in orders table
- VARCHAR without length specification
- Missing NOT NULL constraints
- Data type inconsistencies (time_id DATE vs INTEGER)

**Grading:**
- All PKs added (1 point)
- All FKs added (1 point)
- VARCHAR lengths specified (0.5 points)
- NOT NULL constraints (0.5 points)
- Data type fixes (1 point)

### b) Query (6 points)
```sql
SELECT
    SUM(o.order_amount) AS total_amount,
    AVG(o.quantity) AS avg_quantity
FROM orders o
JOIN time t ON o.time_id = t.time_id
JOIN customer c ON o.customer_id = c.customer_id
JOIN product p ON o.product_id = p.product_id
WHERE p.category = 'Electronics'
  AND c.country = 'Germany'
  AND t.year = 2024
  AND t.month BETWEEN 4 AND 6;
```

**Grading:**
- Correct JOINs (2 points)
- SUM and AVG aggregates (2 points)
- WHERE filters correct (2 points)

---

## Question 4: Indexes

### a) Create indexes (6 points)
```sql
-- Query 1: Date range queries
CREATE INDEX idx_transaction_date ON Transactions(TransactionDate);

-- Query 2: Pending transactions with sorting
CREATE INDEX idx_pending_status_date ON Transactions(Status, TransactionDate)
WHERE Status = 'pending';
-- OR without partial:
CREATE INDEX idx_status_date ON Transactions(Status, TransactionDate);

-- Query 3: Pattern matching
CREATE INDEX idx_description ON Transactions(Description varchar_pattern_ops);
```

**Grading:**
- Index for Query 1 (1.5 points)
- Index for Query 2 (2 points - partial index gets full credit)
- Index for Query 3 (1.5 points - must include varchar_pattern_ops)
- Explanations (1 point)

### b) Partial index explanation (4 points)

**Answer:**
Use a **partial index** for Query 2 because:
- Only 4% of rows have status='pending' (low selectivity)
- Query is run very frequently on this small subset
- Partial index is much smaller (4% of full index size)
- Faster to scan and cheaper to maintain

**Trade-off:**
- Can only be used for queries with WHERE Status='pending'
- Cannot be used for queries on other statuses
- But this is acceptable since pending is the most queried status

**Grading:**
- Choosing partial index (1 point)
- Explaining low selectivity benefit (1.5 points)
- Explaining trade-offs (1.5 points)

---

# Total Time: ~45 minutes
# Total Points: 40

**Scoring:**
- 36-40: Excellent
- 30-35: Good
- 24-29: Satisfactory
- Below 24: Needs more practice
