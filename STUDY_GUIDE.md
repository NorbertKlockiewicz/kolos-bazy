# Advanced Database Systems - Colloquium Study Guide

## Overview

This guide summarizes all 6 labs and provides quick reference for the test.

---

## Lab 1: OLAP/Star Schema/ETL

### Key Concepts:
- **Star Schema**: Fact table (center) + Dimension tables (surrounding)
- **Fact Table**: Contains measures (revenue, quantity) + foreign keys to dimensions
- **Dimension Tables**: Descriptive attributes (time, customer, product, location)
- **ETL**: Extract-Transform-Load using INSERT INTO...SELECT

### Schema Design Checklist:
✅ **Dimension tables must have:**
- SERIAL PRIMARY KEY (surrogate key)
- Natural key column(s) with UNIQUE constraint
- NOT NULL on important fields
- VARCHAR with length specified

✅ **Fact table must have:**
- PRIMARY KEY (often composite of FKs)
- FOREIGN KEYs referencing all dimensions
- NOT NULL on measures and FKs
- Only numeric measures (no descriptive attributes!)

### Common Query Pattern:
```sql
SELECT
    dim.attribute,
    SUM(fact.measure) AS total,
    AVG(fact.measure) AS average,
    COUNT(*) AS count
FROM fact_table fact
JOIN dim_table1 dim1 ON fact.dim1_key = dim1.dim_key
JOIN dim_table2 dim2 ON fact.dim2_key = dim2.dim_key
WHERE dim1.attribute = 'value'
  AND dim2.year = 2023
  AND dim2.month BETWEEN 4 AND 6  -- Q2
GROUP BY dim.attribute
ORDER BY total DESC;
```

### Time Periods:
- Q1: months 1-3
- Q2: months 4-6
- Q3: months 7-9
- Q4: months 10-12
- First half: months 1-6
- Second half: months 7-12

---

## Lab 2: Indexes

### Index Types:

| Type | When to Use | Syntax |
|------|-------------|--------|
| **B-tree** | =, <, >, BETWEEN, ORDER BY | `CREATE INDEX idx ON table(col);` |
| **Multi-column** | Queries on col1, or col1+col2, or col1+col2+col3 | `CREATE INDEX idx ON table(col1, col2, col3);` |
| **Partial** | Queries on subset of data (< 10% of rows) | `CREATE INDEX idx ON table(col) WHERE condition;` |
| **Expression** | Queries on functions (LOWER, EXTRACT, etc.) | `CREATE INDEX idx ON table(LOWER(col));` |
| **varchar_pattern_ops** | LIKE 'prefix%' pattern matching | `CREATE INDEX idx ON table(col varchar_pattern_ops);` |
| **GiST** | Geometric/spatial data | `CREATE INDEX idx ON table USING GIST(geom_col);` |

### Multi-Column Index Rules:
- **CAN use** if query filters on: col1, or col1+col2, or col1+col2+col3
- **CANNOT use** if query skips first column: col2, or col3, or col2+col3

### Pattern Matching:
- `LIKE 'abc%'` ✅ Can use index (with varchar_pattern_ops)
- `LIKE '%abc'` ❌ Cannot use index (need reverse() expression index)
- `LIKE '%abc%'` ❌ Cannot use index
- `ILIKE` ❌ Cannot use index (need LOWER() expression index)

### When Index Won't Be Used:
- Query returns > 10-20% of rows (seq scan faster)
- Table is too small
- Statistics are outdated (run ANALYZE)

---

## Lab 3: PostGIS/Spatial

### Important Functions:

| Function | Purpose | Example |
|----------|---------|---------|
| **ST_MakePoint(lon, lat)** | Create point | `ST_MakePoint(21.01, 52.23)` |
| **ST_SetSRID(geom, srid)** | Set coordinate system | `ST_SetSRID(point, 4326)` |
| **ST_Transform(geom, srid)** | Transform coordinates | `ST_Transform(geom, 2180)` |
| **ST_Distance(g1, g2)** | Distance between geometries | Returns units based on SRID |
| **ST_Area(geom)** | Area of polygon | Returns units based on SRID |
| **ST_Length(geom)** | Length of linestring | Returns units based on SRID |
| **ST_Contains(g1, g2)** | g1 contains g2? | Returns true/false |
| **ST_Within(g1, g2)** | g1 within g2? | Returns true/false |
| **ST_Intersects(g1, g2)** | Do they intersect? | Returns true/false |
| **ST_Buffer(geom, distance)** | Create buffer | `ST_Buffer(point, 100)` |
| **ST_Intersection(g1, g2)** | Intersection geometry | Used for clipping |

### Important EPSG Codes:
- **4326**: WGS-84 (lat/lon in degrees) - GPS, global data
- **2180**: Poland (meters) - for area/distance in Poland
- **3035**: Europe (meters) - for area/distance in Europe

### Area Calculation Pattern:
```sql
-- Always transform to projected CRS first!
SELECT
    name,
    ST_Area(ST_Transform(boundary, 2180)) / 1000000 AS area_km2
FROM regions;
```

### Distance Calculation Pattern:
```sql
-- Transform both geometries to same projected CRS
SELECT
    ST_Distance(
        ST_Transform(geom1, 2180),
        ST_Transform(geom2, 2180)
    ) / 1000 AS distance_km
FROM ...;
```

### Spatial Join Pattern:
```sql
-- Find points within polygons
SELECT p.name, r.name
FROM points p
JOIN regions r ON ST_Contains(r.boundary, p.location);
```

### Buffer Pattern:
```sql
-- Count features within distance
SELECT COUNT(*)
FROM features f
WHERE ST_DWithin(
    f.location,
    ST_MakePoint(21.0, 52.0),
    1000  -- 1000 meters if using projected CRS
);
```

---

## Lab 4: Transactions & Concurrency

### Isolation Levels:

| Level | Prevents | Allows |
|-------|----------|--------|
| **READ COMMITTED** | Dirty reads | Non-repeatable reads, phantoms |
| **REPEATABLE READ** | Dirty reads, non-repeatable reads, phantoms (in PG) | Serialization errors, write skew |
| **SERIALIZABLE** | Everything | Nothing (but more errors) |

### Setting Isolation Level:
```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- or
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### Locking:

**Table-level lock:**
```sql
BEGIN;
LOCK TABLE accounts IN EXCLUSIVE MODE;
-- ... operations ...
COMMIT;
```

**Row-level locks:**
```sql
-- Exclusive (blocks reads and writes by others)
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- Shared (allows reads, blocks writes)
SELECT * FROM accounts WHERE id = 1 FOR SHARE;
```

### Common Problems:

**1. Lost Update:**
```sql
-- BAD: Both transactions read 100, both write 101
T1: SELECT count FROM table;  -- reads 100
T2: SELECT count FROM table;  -- reads 100
T1: UPDATE table SET count = 101;
T2: UPDATE table SET count = 101;  -- Lost T1's update!

-- GOOD: Use atomic operation
UPDATE table SET count = count + 1;  -- No lost update
```

**2. Non-Repeatable Read:**
- Same SELECT returns different values within transaction
- Solution: REPEATABLE READ isolation level

**3. Write Skew:**
- Two transactions read overlapping data, make decisions, both commit
- Solution: SELECT FOR UPDATE or SERIALIZABLE

### Error Messages:
```
ERROR: could not serialize access due to concurrent update
```
→ Serialization error at REPEATABLE READ/SERIALIZABLE, must retry

```
ERROR: deadlock detected
```
→ Deadlock between transactions, one automatically aborted

---

## Lab 5: Neo4j/Cypher

### Basic Syntax:

**Create nodes:**
```cypher
CREATE (n:Label {property: 'value'})
CREATE (a:Person {name: 'Alice', age: 30})
```

**Create relationships:**
```cypher
CREATE (a)-[:RELATIONSHIP_TYPE {property: 'value'}]->(b)
CREATE (a:Person {name: 'Alice'})-[:KNOWS]->(b:Person {name: 'Bob'})
```

**Match patterns:**
```cypher
MATCH (n:Label)
WHERE n.property = 'value'
RETURN n

MATCH (a:Person)-[:KNOWS]->(b:Person)
RETURN a.name, b.name
```

**MERGE (create if not exists):**
```cypher
MERGE (n:Person {name: 'Alice'})
ON CREATE SET n.created = timestamp()
ON MATCH SET n.accessed = timestamp()
RETURN n
```

### Common Patterns:

**Count relationships:**
```cypher
MATCH (n)-[r:REL_TYPE]->()
WITH n, count(r) AS rel_count
WHERE rel_count = 2
RETURN n
```

**Shortest path:**
```cypher
MATCH path = shortestPath((a:Label1)-[*]-(b:Label2))
WHERE a.name = 'Start' AND b.name = 'End'
RETURN path
```

**All paths:**
```cypher
MATCH path = (a:Label1)-[*1..5]-(b:Label2)
WHERE a.name = 'Start' AND b.name = 'End'
RETURN path
```

**Aggregation:**
```cypher
MATCH (p:Person)-[:WORKS_ON]->(proj:Project)
RETURN p.name, count(proj) AS num_projects
ORDER BY num_projects DESC
```

**Update properties:**
```cypher
MATCH (n:Label)
WHERE n.property = 'old'
SET n.property = 'new'
RETURN n
```

**Delete (with relationships):**
```cypher
MATCH (n:Label)
DETACH DELETE n
```

### Query Structure:
```cypher
MATCH (n:Label)-[r:REL]->(m:Label2)
WHERE n.property > 100
WITH n, count(r) AS rel_count
ORDER BY rel_count DESC
LIMIT 10
RETURN n.name, rel_count
```

---

## Lab 6: CouchDB/NoSQL

### Document Structure:
```json
{
  "_id": "unique_id",
  "_rev": "1-abc123",
  "field1": "value1",
  "field2": 123,
  "nested": {
    "field3": "value3"
  }
}
```

### Map Function:
```javascript
function(doc) {
  // Emit key-value pairs
  if (doc.type === "person") {
    emit(doc.name, doc.age);
  }
}
```

**Tips:**
- Always check if fields exist: `if (doc.field)`
- Can emit multiple times per document
- Key determines sort order

### Reduce Function:
```javascript
function(keys, values, rereduce) {
  if (rereduce) {
    // Reducing results from previous reduces
    return sum(values);
  } else {
    // First-level reduce
    return sum(values);
  }
}
```

**Built-in reduces:**
- `_count` - Count documents
- `_sum` - Sum values
- `_stats` - Statistics (sum, count, min, max)

### Average (requires custom reduce):
```javascript
function(keys, values, rereduce) {
  if (rereduce) {
    var sum = 0, count = 0;
    for (var i = 0; i < values.length; i++) {
      sum += values[i].sum;
      count += values[i].count;
    }
    return {sum: sum, count: count, avg: sum/count};
  } else {
    var sum = 0;
    for (var i = 0; i < values.length; i++) {
      sum += values[i];
    }
    return {sum: sum, count: values.length, avg: sum/values.length};
  }
}
```

### Query Parameters:

```
# Exact key
?key="value"

# Key range
?startkey="a"&endkey="z"

# Complex key (array)
?key=["store","category"]

# Range with complex key (all items from store)
?startkey=["store"]&endkey=["store",{}]

# Grouping
?group=true
?group_level=1

# Limit/skip
?limit=10&skip=20

# Descending
?descending=true
```

---

## Quick Reference for Test

### Most Likely Topics (based on previous test):
1. **PostGIS** (10 pts) - Area calculation, spatial containment
2. **Neo4j** (10 pts) - Create graph, path queries
3. **OLAP** (10 pts) - Fix schema, aggregation query
4. **Indexes** (10 pts) - Create indexes for queries

### Less Likely But Possible:
- **Transactions** - Isolation levels, locking scenarios
- **CouchDB** - Map/reduce views

### Time Management:
- Read all questions first (2 min)
- Start with easiest question (10 min)
- Do 2nd and 3rd questions (20 min each)
- Final question (15 min)
- Review all answers (5 min)

### Common Mistakes to Avoid:
1. ❌ Forgetting PRIMARY KEY in dimension tables
2. ❌ Forgetting FOREIGN KEY in fact table
3. ❌ VARCHAR without length
4. ❌ Using ST_Area on GEOMETRY without ST_Transform
5. ❌ Forgetting to divide by 1000000 for km²
6. ❌ SQL: Using == instead of = for comparison
7. ❌ SQL: Forgetting quotes around strings ('Poland', not POLAND)
8. ❌ Cypher: Using DELETE instead of DETACH DELETE
9. ❌ CouchDB: Not handling rereduce case
10. ❌ Indexes: Creating index on LIKE '%pattern%' (won't work)

### Write Syntax Carefully:
- SQL uses single `=` for comparison
- Strings need quotes: `'string'`
- Case matters in LIKE (use ILIKE for case-insensitive)
- PostgreSQL: `!=` or `<>` for not equal
- Cypher: Use `DETACH DELETE` to delete nodes with relationships
- CouchDB: Always check if fields exist before using them

---

## Study Strategy

### Day Before Test:
1. Review this study guide (30 min)
2. Practice [PRACTICE_TEST.md](PRACTICE_TEST.md) on paper (45 min)
3. Check your answers against answer key
4. Review mistakes in specific lab practice questions

### 1 Hour Before Test:
1. Review common mistakes list above
2. Review syntax for your weakest topic
3. Stay calm - you've practiced a lot!

### During Test:
1. Read all questions carefully
2. For schema questions: list ALL mistakes before fixing
3. For query questions: think about JOINs and WHERE conditions
4. Check syntax before moving on
5. If stuck, move to next question and come back

---

## Good Luck! 🍀

You have:
- ✅ 6 labs of practice questions (60+ questions total)
- ✅ 1 complete practice test with answers
- ✅ All your lab solutions to review
- ✅ This comprehensive study guide

**You're well prepared!**
