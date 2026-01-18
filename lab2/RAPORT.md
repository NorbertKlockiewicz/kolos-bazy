# Advanced Database Systems - Lab 2 - Norbert Klockiewicz

## 1. B-tree indexes

Creating Index:

```
CREATE INDEX employees_surname_idx ON employees (surname);
```

Query Plan (starts with A): 

```
 Bitmap Heap Scan on employees  (cost=13.65..905.28 rows=913 width=33)
   Recheck Cond: ((surname >= 'A'::text) AND (surname < 'B'::text))
   ->  Bitmap Index Scan on employees_surname_idx  (cost=0.00..13.42 rows=913 width=0)
         Index Cond: ((surname >= 'A'::text) AND (surname < 'B'::text))
```

Query Plan (starts with B, C etc.):

```
Seq Scan on employees  (cost=0.00..2091.00 rows=99087 width=33)
  Filter: (surname >= 'B'::text)
```

Query Plan after setting enable_seqscan to off:

```
Bitmap Heap Scan on employees  (cost=1152.22..3231.80 rows=99087 width=33)
  Recheck Cond: (surname >= 'B'::text)
    ->  Bitmap Index Scan on employees_surname_idx  (cost=0.00..1127.44 rows=99087 width=0)
            Index Cond: (surname >= 'B'::text)
```

Query Plan for employees with first name "Piotr":

```
Seq Scan on employees  (cost=0.00..2091.00 rows=648 width=33)
   Filter: (name = 'Piotr'::text)
```

### Analysis

**Q: Is the index used for surnames starting with 'A'?**
Yes. The plan shows "Bitmap Index Scan on employees_surname_idx" is used.

**Q: Why are there two steps (Bitmap Index Scan → Bitmap Heap Scan)?**
The Bitmap Index Scan builds a bitmap of matching row locations, then the Bitmap Heap Scan fetches actual rows in physical order. This reduces random I/O compared to fetching rows one-by-one.

**Q: Is the index used for surnames >= 'B'?**
No. Sequential Scan is used because the query returns ~99% of rows (99,087/100,000). Scanning the entire table is cheaper than reading most of the index plus most of the table.

**Q: Anything peculiar about the "Piotr" query costs?**
Yes. Despite returning only 648 rows (0.6%), the cost is 2091.00 - same as a full table scan. This is because there's no index on the `name` column, so all 100,000 rows must be scanned. The cost reflects scanning work, not result size.

## 2. Indexes and pattern matching

Case-sensitive
```
Bitmap Heap Scan on employees  (cost=13.73..906.12 rows=10 width=33)
   Filter: (surname ~~ 'A%'::text)
   ->  Bitmap Index Scan on employees_surname_name_idx  (cost=0.00..13.73 rows=944 width=0)
         Index Cond: ((surname ~>=~ 'A'::text) AND (surname ~<~ 'B'::text))
```

Case-insensitive
```
Seq Scan on employees  (cost=0.00..2091.00 rows=10 width=33)
   Filter: (surname ~~* 'A%'::text)
```

Ending with -ski
```
Seq Scan on employees  (cost=0.00..2091.00 rows=10 width=33) (actual time=7.297..7.297 rows=0 loops=1)
   Filter: (surname ~~ '%ski'::text)
   Rows Removed by Filter: 100000
 Planning Time: 0.341 ms
 Execution Time: 7.337 ms
```

## Analysis

Index is used only for the first query so surname starting with letter and not case-sensitive. For the rest of queries it's not using the index as expected. For the index to be used for the case insensitive and ending with... queries we would need to create index on expression.

## 3. Multi-column indexes

Searching for Melania Pindor

```
Index Scan using employees_surname_name_idx on employees  (cost=0.42..8.44 rows=1 width=33)
```

Searching for Pindors

```
Bitmap Heap Scan on employees  (cost=4.63..98.45 rows=27 width=33) (actual time=0.089..0.205 rows=27 loops=1)
   Recheck Cond: (surname = 'Pindor'::text)
   Heap Blocks: exact=25
   ->  Bitmap Index Scan on employees_surname_name_idx  (cost=0.00..4.62 rows=27 width=0) (actual time=0.078..0.079 rows=27 loops=1)
         Index Cond: (surname = 'Pindor'::text)
 Planning Time: 0.336 ms
 Execution Time: 0.257 ms
```

Searching for Melanias

```
Seq Scan on employees  (cost=0.00..2091.00 rows=648 width=33) (actual time=0.017..5.592 rows=638 loops=1)
   Filter: (name = 'Melania'::text)
   Rows Removed by Filter: 99362
 Planning Time: 0.320 ms
 Execution Time: 5.646 ms
```

### Analysis: Cost Differences Explanation

**Query 1: Melania Pindor**
- **Uses Index Scan** - the most efficient access method
- The multi-column index `employees_surname_name_idx` on (surname, name) is fully utilized
- Both columns in the WHERE clause match the index column order
- PostgreSQL can navigate directly to the matching row(s) using the index
- Lowest cost because it reads minimal pages: traverses B-tree index + fetches specific heap page(s)

**Query 2: Surname 'Pindor'**
- **Uses Bitmap Index Scan** - moderately efficient
- The multi-column index can still be used because `surname` is the leading column
- Bitmap approach is chosen because multiple rows (27) match
- Higher cost than Query 1 because:
  - Builds a bitmap of 27 matching row locations
  - Fetches 25 heap blocks to retrieve the actual row data

**Query 3: Name 'Melania'**
- **Uses Sequential Scan** - least efficient, full table scan
- The multi-column index **cannot be used** because `name` is the second column in the index
- A B-tree index on (surname, name) can only be used efficiently when surname is specified
- Highest cost because:
  - Must scan all 100,000 rows in the table
  - No index assistance available
  - Filters rows in memory (removed 99,362 rows to find 638 matches)

## 4. Indexes and sorting

Query 1: Sort by surname ASC, salary DESC

```
"Incremental Sort  (cost=2.82..10183.99 rows=100000 width=22)"
"  Sort Key: surname, salary DESC"
"  Presorted Key: surname"
"  ->  Index Scan using employees_surname_name_idx on employees  (cost=0.42..6480.07 rows=100000 width=22)"
```

Query 2: Sort by surname DESC, salary ASC (reversed)

```
"Incremental Sort  (cost=2.82..10183.99 rows=100000 width=22)"
"  Sort Key: surname DESC, salary"
"  Presorted Key: surname"
"  ->  Index Scan Backward using employees_surname_name_idx on employees  (cost=0.42..6480.07 rows=100000 width=22)"
```

### Analysis: Execution Plan Explanation

Both queries use **Incremental Sort**, an efficient sorting optimization introduced in PostgreSQL 13. Here's what's happening:

**Query 1: ORDER BY surname ASC, salary DESC**
- Uses **Index Scan** on `employees_surname_name_idx` (forward direction)
- The index provides data already sorted by `surname ASC`
- **Incremental Sort** is applied on top:
  - "Presorted Key: surname" indicates PostgreSQL knows data is already sorted by surname
  - Only needs to sort by `salary DESC` within each group of identical surnames
  - Much cheaper than sorting all 100,000 rows from scratch

**Query 2: ORDER BY surname DESC, salary ASC**
- Uses **Index Scan Backward** on `employees_surname_name_idx` (reverse direction)
- Reading the B-tree index backwards provides data sorted by `surname DESC`
- **Incremental Sort** applied the same way:
  - "Presorted Key: surname" (implicitly DESC due to backward scan)
  - Only sorts by `salary` within each surname group

**Did the plan change?**

Yes, the first query is using index scan and second one is using index scan backward

## 5. Partial indexes

Create partial index

```
CREATE INDEX employees_surname_partial_idx 
ON employees (surname) 
WHERE salary < 10000;
```

Query 1: Stricter condition (salary < 5000)

```
"Bitmap Heap Scan on employees  (cost=474.97..1944.00 rows=10 width=22)"
"  Recheck Cond: (salary < 10000)"
"  Filter: (salary < 5000)"
"  ->  Bitmap Index Scan on employees_surname_partial_idx  (cost=0.00..474.97 rows=50242 width=0)"
```

Query 2: Exact match (salary < 10000)

```
"Bitmap Heap Scan on employees  (cost=487.53..1956.56 rows=50242 width=22)"
"  Recheck Cond: (salary < 10000)"
"  ->  Bitmap Index Scan on employees_surname_partial_idx  (cost=0.00..474.97 rows=50242 width=0)"
```

Query 3: Less strict condition (salary < 15000):

```
"Seq Scan on employees  (cost=0.00..2091.00 rows=99990 width=22)"
"  Filter: (salary < 15000)"
```

### Analysis: Partial Index Behavior

A **partial index** only indexes rows that satisfy a specific condition (in this case, `salary < 10000`). This makes the index smaller and more efficient for queries that match the condition, but unusable for queries outside that range.

**Query 1: salary < 5000 (Stricter than index condition)**
- **Uses Bitmap Index Scan** on `employees_surname_partial_idx`
- The partial index IS used because salary < 5000 is **stricter** than salary < 10000
- All rows matching salary < 5000 are guaranteed to be in the index
- Process:
  1. Bitmap Index Scan reads the partial index (all rows with salary < 10000)
  2. Recheck Cond applies the index condition
  3. Filter further narrows to salary < 5000
- Estimated 10 rows match (very selective)

**Query 2: salary < 10000 (Exact match with index condition)**
- **Uses Bitmap Index Scan** on `employees_surname_partial_idx`
- Perfect match with the partial index condition
- All 50,242 rows in the index match the query
- Most efficient use of the partial index

**Query 3: salary < 15000 (Less strict than index condition)**
- **Uses Sequential Scan** - index NOT used
- Why no index?
  - The partial index only contains rows with salary < 10000
  - Query needs rows with salary between 10000 and 15000 as well
  - These rows are NOT in the partial index
  - PostgreSQL must scan the entire table to find all matching rows
- Estimated 99,990 rows match (~100% of table)

## 6. Indexes on expressions

Create index on LOWER() expression

```
CREATE INDEX employees_surname_lower_idx ON employees (LOWER(surname) varchar_pattern_ops);
```

Create index on REVERSE() expression

```
CREATE INDEX employees_surname_reverse_idx ON employees (REVERSE(surname) varchar_pattern_ops);
```

Query: Starting with a%:

```
EXPLAIN
SELECT name, surname, salary
FROM employees
WHERE LOWER(surname) LIKE 'a%';


"Bitmap Heap Scan on employees  (cost=9.42..776.40 rows=500 width=22)"
"  Filter: (lower(surname) ~~ 'a%'::text)"
"  ->  Bitmap Index Scan on employees_surname_lower_idx  (cost=0.00..9.29 rows=500 width=0)"
"        Index Cond: ((lower(surname) ~>=~ 'a'::text) AND (lower(surname) ~<~ 'b'::text))"
```


Query: Ending with ski:

```
EXPLAIN
SELECT name, surname, salary
FROM employees
WHERE REVERSE(surname) LIKE 'iks%';

"Bitmap Heap Scan on employees  (cost=9.42..776.40 rows=500 width=22)"
"  Filter: (reverse(surname) ~~ 'iks%'::text)"
"  ->  Bitmap Index Scan on employees_surname_reverse_idx  (cost=0.00..9.29 rows=500 width=0)"
"        Index Cond: ((reverse(surname) ~>=~ 'iks'::text) AND (reverse(surname) ~<~ 'ikt'::text))"
```

### Analysis

In both cases by using the expression index we were able to make postgreSQL use those indexes while searching for rows that match conditions.


## 7. GiST indexes

Adding a location column:

```
ALTER TABLE employees
ADD COLUMN location point;
```

Find employees in top-right quarter:

```
SELECT *
FROM employees
WHERE location[0] > 0 AND location[1] > 0;
```

Create an GiST index:

```
CREATE INDEX employees_location_gist_idx
ON employees
USING gist (location);
```

Query that will use the index with the execution plan:

```
EXPLAIN
SELECT *
FROM employees
WHERE box '((0,0),(100,100))' @> location;

"Bitmap Heap Scan on employees  (cost=5.06..330.64 rows=100 width=49)"
"  Recheck Cond: ('(100,100),(0,0)'::box @> location)"
"  ->  Bitmap Index Scan on employees_location_gist_idx  (cost=0.00..5.03 rows=100 width=0)"
"        Index Cond: (location <@ '(100,100),(0,0)'::box)"
```

Selecting the employees not further than 20 units form (0, 0):

```
EXPLAIN
SELECT *
FROM employees
WHERE circle '((0,0),20)' @> location;

"Bitmap Heap Scan on employees  (cost=5.06..330.64 rows=100 width=49)"
"  Recheck Cond: ('<(0,0),20>'::circle @> location)"
"  ->  Bitmap Index Scan on employees_location_gist_idx  (cost=0.00..5.03 rows=100 width=0)"
"        Index Cond: (location <@ '<(0,0),20>'::circle)"
```

Distance operator: 

```
EXPLAIN
SELECT *
FROM employees
WHERE location <-> point(0,0) <= 20;

"Seq Scan on employees  (cost=0.00..3350.00 rows=33333 width=49)"
"  Filter: ((location <-> '(0,0)'::point) <= '20'::double precision)"
```

| Query                                      | Plan Type                        | Uses GiST | Notes                                     |
| ------------------------------------------ | -------------------------------- | --------- | ----------------------------------------- |
| `location[0] > 0 AND location[1] > 0`      | Sequential Scan                  | No        | Simple coordinate filters not indexable   |
| `box '((0,0),(100,100))' @> location`      | Bitmap Index Scan + Recheck Cond | Yes       | Uses bounding box containment             |
| `circle '((0,0),20)' @> location`          | Bitmap Index Scan + Recheck Cond | Yes       | Uses spatial index for circle containment |
| `location <-> point(0,0) <= 20`            | Sequential Scan                  | No        | Distance filter not indexable             |

