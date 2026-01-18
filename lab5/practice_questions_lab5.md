# Lab 5: Neo4j/Graph Databases - Practice Questions

## Question 1: Create Graph from Diagram (Similar to Test Question 2)

### Given this production workflow graph:

```
    [Machine:A]      [Machine:B]
    sn:101           sn:201
         |                |
         v                v
    [Machine:C]      [Machine:D]
    sn:301           sn:401
         |                |
         +-------+--------+
                 |
                 v
            [Machine:E]
            sn:501
```

Each machine has a serial number (sn). Resource flows from top to bottom.

### Tasks:

1. **Write Cypher queries to create all nodes** with:
   - Label: Machine
   - Properties: type (A, B, C, D, E), sn (serial number)

2. **Create all relationships** showing resource flow (use relationship type :PASSES_RESOURCE_TO)

3. **Write the complete graph creation** in one query using CREATE

---

## Question 2: Analytical Queries on Production Graph

### Using the graph from Question 1:

### Write Cypher queries to answer:

1. **Find all machines that pass resources to exactly 2 other machines**

2. **Find the complete path from Machine A to Machine E**

3. **Find all machines that receive resources from machines of type B**

4. **Find machines that are "leaf nodes"** (don't pass resources to anyone else)

5. **Find the longest production chain** (path with most machines)

---

## Question 3: University Campus Graph

### Scenario:

Create a graph representing AGH University campus:
- Buildings: A-0, A-1, A-2, B-1, B-2, C-1, C-2, D-1, U-1
- Building types: Teaching (A, C, D buildings), Administrative (B buildings), Student center (U-1)
- Connections between buildings (can walk without going outside)
- Floors where buildings connect (e.g., A-1 connects to A-2 on floors 0, 1, 2)
- Classrooms (room number, floor, building)
- Departments (name, building, floor)

### Tasks:

1. **Write CREATE statements** for all nodes with appropriate labels and properties

2. **Write queries to add connections** between buildings (use MERGE to add to existing nodes)

3. **Write analytical queries:**
   - Are there any isolated buildings (no connections)?
   - How many teaching buildings are there?
   - What buildings is A-1 connected to and at which floors?
   - Find all paths from C-4 to D-1
   - What is the shortest path from B-1 to U-1?

---

## Question 4: Modification Queries

### Given an existing graph with nodes labeled :Building and :Classroom:

### Write Cypher queries to:

1. **Change all :HQ labels to :HEADQUARTERS** (keep all properties)

2. **Add 1000 to every classroom's room number**
   - Example: room 123 becomes 1123

3. **Delete all nodes labeled :TempBuilding** and their relationships

4. **Add a new property 'capacity' to all classrooms** in building 'C-3' (set to NULL initially)

5. **Create a new relationship :NEARBY** between all buildings that are within 100m of each other**

---

## Question 5: Social Network Graph

### Scenario:

```
Users: Alice, Bob, Charlie, Diana, Eve
Relationships:
- Alice FOLLOWS Bob, Charlie
- Bob FOLLOWS Charlie, Diana
- Charlie FOLLOWS Diana
- Diana FOLLOWS Eve
- Eve FOLLOWS Alice
```

### Tasks:

1. **Create the complete graph**

2. **Write queries to find:**
   - Who does Alice follow?
   - Who follows Bob?
   - Users that Alice follows who also follow Diana (friends of friends)
   - Users with the most followers
   - Users that Alice and Bob both follow (mutual follows)
   - Cycles in the follow relationships

3. **Write a query** to suggest new people for Alice to follow (people followed by people Alice follows, but not Alice herself)

---

## Question 6: Pattern Matching

### Given a project collaboration graph:

```cypher
CREATE
  (alice:Person {name: 'Alice', role: 'Developer'}),
  (bob:Person {name: 'Bob', role: 'Developer'}),
  (charlie:Person {name: 'Charlie', role: 'Manager'}),
  (diana:Person {name: 'Diana', role: 'Designer'}),

  (proj1:Project {name: 'Website', status: 'active'}),
  (proj2:Project {name: 'Mobile App', status: 'completed'}),
  (proj3:Project {name: 'Database', status: 'active'}),

  (alice)-[:WORKS_ON {hours: 40}]->(proj1),
  (alice)-[:WORKS_ON {hours: 20}]->(proj3),
  (bob)-[:WORKS_ON {hours: 30}]->(proj1),
  (charlie)-[:MANAGES]->(proj1),
  (diana)-[:WORKS_ON {hours: 25}]->(proj2)
```

### Write queries to find:

1. **All active projects and their developers**

2. **Developers working on more than one project**

3. **Projects managed by Charlie with their total hours** (sum of all developers' hours)

4. **People working on the same project as Alice** (exclude Alice)

5. **Developers working on projects with no manager**

---

## Question 7: Aggregation and Statistics

### Using the project collaboration graph from Question 6:

### Write queries to:

1. **Count total number of projects per person**

2. **Find average hours worked per project**

3. **Find the person working the most total hours** across all projects

4. **List all roles and count how many people have each role**

5. **Find projects with more than 2 people working on them**

---

## Question 8: Path Finding Algorithms

### Given a transportation network:

```cypher
CREATE
  (warsaw:City {name: 'Warsaw'}),
  (krakow:City {name: 'Krakow'}),
  (wroclaw:City {name: 'Wroclaw'}),
  (poznan:City {name: 'Poznan'}),
  (gdansk:City {name: 'Gdansk'}),

  (warsaw)-[:ROAD {distance: 290}]->(krakow),
  (warsaw)-[:ROAD {distance: 350}]->(wroclaw),
  (warsaw)-[:ROAD {distance: 310}]->(poznan),
  (warsaw)-[:ROAD {distance: 340}]->(gdansk),
  (krakow)-[:ROAD {distance: 270}]->(wroclaw),
  (poznan)-[:ROAD {distance: 180}]->(wroclaw),
  (poznan)-[:ROAD {distance: 280}]->(gdansk)
```

### Write queries to:

1. **Find the shortest path** from Warsaw to Wroclaw (by number of hops)

2. **Find all paths** from Warsaw to Wroclaw with maximum 2 stops

3. **Find the shortest path by distance** from Warsaw to Gdansk

4. **Find all cities reachable from Warsaw within 2 hops**

5. **Find cities that are not directly connected to Warsaw**

---

## Question 9: Conditional Creation (MERGE vs CREATE)

### Scenario:

You're building a course enrollment system.

### Tasks:

1. **Explain the difference** between CREATE and MERGE

2. **Write a query** that creates a Student node if it doesn't exist, or returns existing node if it does
   - Use MERGE with ON CREATE and ON MATCH

3. **Write a query** to enroll a student in a course:
   - Create the enrollment relationship only if it doesn't exist
   - If it exists, update the enrollment_date property

4. **Why is MERGE important** in graph databases?

---

## Question 10: Resource Dependency Analysis (Test-like Question)

### Given this graph showing software components and their dependencies:

```
    [App:Frontend]  [App:Admin]
           |             |
           v             v
    [Service:API]   [Service:Auth]
           |             |
           +------+------+
                  |
                  v
           [Database:Main]
```

### Scenario:

Each component depends on the components below it. Components have:
- type (App, Service, Database)
- name (Frontend, Admin, API, Auth, Main)
- version (e.g., "1.2.0")

### Tasks:

1. **Write Cypher to create the complete graph** with :DEPENDS_ON relationships

2. **Write a query** to find all components that depend on Database:Main (directly or indirectly)

3. **Write a query** to find components that have dependencies on more than 2 different services

4. **Write a query** to find the complete dependency chain for App:Frontend (all transitive dependencies)

5. **Write a query** to identify potential issues: find components that have circular dependencies

---

# Tips for Lab 5 Questions:

## Cypher Syntax Basics:

### Creating Nodes:
```cypher
CREATE (n:Label {property: 'value'})
CREATE (a:Person {name: 'Alice', age: 30})

-- Multiple labels
CREATE (n:Label1:Label2 {property: 'value'})
```

### Creating Relationships:
```cypher
CREATE (a)-[:REL_TYPE {property: 'value'}]->(b)

-- With node creation
CREATE (a:Person {name: 'Alice'})-[:KNOWS]->(b:Person {name: 'Bob'})
```

### MATCH Pattern:
```cypher
MATCH (n:Label)
WHERE n.property = 'value'
RETURN n

-- Relationship pattern
MATCH (a:Person)-[:KNOWS]->(b:Person)
RETURN a, b

-- Variable length path
MATCH (a)-[:KNOWS*1..3]->(b)
RETURN a, b
```

### MERGE (Create if not exists):
```cypher
MERGE (n:Person {name: 'Alice'})
ON CREATE SET n.created = timestamp()
ON MATCH SET n.accessed = timestamp()
RETURN n
```

## Common Patterns:

### Find nodes with specific relationship count:
```cypher
MATCH (n)-[r:REL_TYPE]->()
WITH n, count(r) AS rel_count
WHERE rel_count = 2
RETURN n
```

### Find shortest path:
```cypher
MATCH path = shortestPath((a:Label1)-[*]-(b:Label2))
WHERE a.name = 'Start' AND b.name = 'End'
RETURN path
```

### Find all paths:
```cypher
MATCH path = (a:Label1)-[*]-(b:Label2)
WHERE a.name = 'Start' AND b.name = 'End'
RETURN path
```

### Aggregation:
```cypher
MATCH (p:Person)-[:WORKS_ON]->(proj:Project)
RETURN p.name, count(proj) AS project_count
ORDER BY project_count DESC
```

### Update properties:
```cypher
MATCH (n:Label)
WHERE n.property = 'old'
SET n.property = 'new'
RETURN n
```

### Delete nodes and relationships:
```cypher
MATCH (n:Label)
DETACH DELETE n  -- Deletes node and all its relationships
```

### Change labels:
```cypher
MATCH (n:OldLabel)
SET n:NewLabel
REMOVE n:OldLabel
RETURN n
```

## Important Cypher Functions:

### Aggregation:
- `count()` - Count items
- `sum()` - Sum values
- `avg()` - Average
- `min()`, `max()` - Min/max values
- `collect()` - Collect values into list

### Path Functions:
- `shortestPath()` - Find shortest path
- `allShortestPaths()` - Find all shortest paths
- `length()` - Length of path

### String Functions:
- `toLower()`, `toUpper()` - Case conversion
- `substring()` - Extract substring
- `split()` - Split string

### List Functions:
- `size()` - Size of list
- `head()`, `tail()` - First/rest of list
- `last()` - Last element

## Query Structure:

```cypher
// 1. MATCH - Find patterns
MATCH (n:Label)-[r:REL]->(m:Label2)

// 2. WHERE - Filter
WHERE n.property > 100

// 3. WITH - Pipeline results
WITH n, count(r) AS rel_count

// 4. ORDER BY - Sort
ORDER BY rel_count DESC

// 5. LIMIT - Limit results
LIMIT 10

// 6. RETURN - Return results
RETURN n, rel_count
```

## Common Mistakes:

❌ **Forgetting DETACH DELETE**
```cypher
DELETE n  -- Error if n has relationships
```
✅ Use:
```cypher
DETACH DELETE n  -- Deletes relationships too
```

❌ **Not using MERGE correctly**
```cypher
MERGE (a:Person)-[:KNOWS]->(b:Person)  -- Creates new nodes!
```
✅ Use:
```cypher
MERGE (a:Person {name: 'Alice'})
MERGE (b:Person {name: 'Bob'})
MERGE (a)-[:KNOWS]->(b)
```

❌ **Confusing direction**
```cypher
// Creates relationship in wrong direction
CREATE (a)<-[:FOLLOWS]-(b)
```

## Test Writing Tips:

1. ✅ **Use descriptive relationship types** (PASSES_RESOURCE_TO, not just CONNECTED)
2. ✅ **Always specify node labels** in MATCH (for performance)
3. ✅ **Use MERGE for idempotent operations** (safe to run multiple times)
4. ✅ **Use DETACH DELETE** when deleting nodes
5. ✅ **Filter with WHERE** for complex conditions
6. ✅ **Use variable length paths** `[*1..5]` for transitive queries
7. ✅ **Remember direction matters** in relationships (unless you use `-[]-`)

## Graph Modeling Tips:

1. **Nodes** = Entities (People, Buildings, Products)
2. **Relationships** = Actions/Connections (KNOWS, CONNECTED_TO, DEPENDS_ON)
3. **Properties** = Attributes (name, age, distance)
4. **Labels** = Categories (Person, Building, Machine)

**Good model:**
```cypher
(:Person {name: 'Alice'})-[:WORKS_ON {hours: 40}]->(:Project {name: 'DB'})
```

**Bad model:** (putting everything in properties)
```cypher
(:Node {type: 'Person', name: 'Alice', rel_type: 'WORKS_ON', project: 'DB'})
```
