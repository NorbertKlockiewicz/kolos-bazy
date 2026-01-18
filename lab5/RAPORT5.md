# Lab5 - Neo4j - Norbert Klockiewicz

## Exercise 1: Creating the University Map

### Creating building nodes with labels and properties
```cypher
CREATE (S1:BLDG:UCO {name: 'S-1'})
CREATE (S2:BLDG:SRV {name: 'S-2'})
CREATE (D1:BLDG:RAT {name: 'D-1'})
CREATE (U2:BLDG:RAT {name: 'U-2'})
CREATE (A4:BLDG:RAT {name: 'A-4'})
CREATE (A3:BLDG:RAT {name: 'A-3'})
CREATE (A2:BLDG:RAT {name: 'A-2'})
CREATE (A1:BLDG:RAT {name: 'A-1'})
CREATE (A0:BLDG:RAT {name: 'A-0'})
CREATE (C1:BLDG:RAT {name: 'C-1'})
CREATE (C2:BLDG:RAT {name: 'C-2'})
CREATE (C3:BLDG:RAT {name: 'C-3'})
CREATE (C4:BLDG:RAT {name: 'C-4'})
CREATE (C5:BLDG:RAT {name: 'C-5'})
CREATE (C6:BLDG:RAT {name: 'C-6'})
CREATE (C7:BLDG:RAT {name: 'C-7'})
CREATE (U1:BLDG:RAT {name: 'U-1'})
CREATE (B1:BLDG:RAT {name: 'B-1'})
CREATE (B2:BLDG:RAT {name: 'B-2'})
CREATE (B3:BLDG:RAT {name: 'B-3'})
CREATE (B4:BLDG:RAT {name: 'B-4'})
CREATE (HA1:BLDG:RAT {name: 'H-A1'})
CREATE (HA2:BLDG:RAT {name: 'H-A2'})
CREATE (HB1B2:BLDG:RAT {name: 'H-B1B2'});
```

### Creating connections between buildings with floor information
```cypher
MATCH (n:BLDG {name: 'A-0'}), (m: BLDG {name: 'A-1'})
CREATE (n)-[r:CONNECTED_TO {floor: 1}]->(m);

MATCH (n: BLDG {name: 'A-0'}), (m: BLDG {name: 'A-1'})
CREATE (n)-[r:CONNECTED_TO {floor: 2}]->(m);

MATCH (n: BLDG {name: 'A-1'}), (m: BLDG {name: 'A-2'})
CREATE (n)-[r:CONNECTED_TO {floor: 1}]->(m);

MATCH (n: BLDG {name: 'A-1'}), (m: BLDG {name: 'A-2'})
CREATE (n)-[r:CONNECTED_TO {floor: 2}]->(m);

MATCH (n: BLDG {name: 'A-1'}), (m: BLDG {name: 'H-A1'})
CREATE (n)-[r:CONNECTED_TO {floor: 1}]->(m);

MATCH (n: BLDG {name: 'A-1'}), (m: BLDG {name: 'H-A1'})
CREATE (n)-[r:CONNECTED_TO {floor: 2}]->(m);

MATCH (n: BLDG {name: 'A-2'}), (m: BLDG {name: 'H-A2'})
CREATE (n)-[r:CONNECTED_TO {floor: 1}]->(m);

MATCH (n: BLDG {name: 'A-2'}), (m: BLDG {name: 'H-A2'})
CREATE (n)-[r:CONNECTED_TO {floor: 2}]->(m);

MATCH (n: BLDG {name: 'H-A1'}), (m: BLDG {name: 'H-A2'})
CREATE (n)-[r:CONNECTED_TO {floor: 1}]->(m);

MATCH (n: BLDG {name: 'H-A1'}), (m: BLDG {name: 'H-A2'})
CREATE (n)-[r:CONNECTED_TO {floor: 2}]->(m);


WITH [
{from: 'A-1', to: 'C-1', floor: 1},
{from: 'C-1', to: 'C-2', floor: 0},
{from: 'C-1', to: 'C-2', floor: 1},
{from: 'C-1', to: 'C-2', floor: 2},
{from: 'C-1', to: 'C-2', floor: 3},
{from: 'C-1', to: 'C-2', floor: 4}
] AS connections

UNWIND connections AS c
MATCH (a: BLDG {name: c.from})
MATCH (b: BLDG {name: c.to})
CREATE (a)-[:CONNECTED_TO {floor: c.floor}]->(b);

WITH [
{from: 'C-2', to: 'C-3', floor: 0},
{from: 'C-2', to: 'C-3', floor: 1},
{from: 'C-2', to: 'C-3', floor: 2},
{from: 'C-2', to: 'C-3', floor: 3},
{from: 'C-2', to: 'C-3', floor: 4},
{from: 'C-3', to: 'C-7', floor: 0},
{from: 'C-3', to: 'C-7', floor: 1},
{from: 'C-3', to: 'C-7', floor: 2},
{from: 'C-3', to: 'C-7', floor: 3},
{from: 'C-3', to: 'C-7', floor: 4},
{from: 'C-7', to: 'C-6', floor: 0},
{from: 'C-7', to: 'C-6', floor: 1},
{from: 'C-7', to: 'C-6', floor: 2},
{from: 'C-7', to: 'C-6', floor: 3},
{from: 'C-7', to: 'C-6', floor: 4},
{from: 'C-6', to: 'C-4', floor: 0},
{from: 'C-6', to: 'C-4', floor: 1},
{from: 'C-6', to: 'C-4', floor: 2},
{from: 'C-6', to: 'C-4', floor: 3},
{from: 'C-6', to: 'C-4', floor: 4},
{from: 'C-4', to: 'A-4', floor: 1},
{from: 'C-4', to: 'A-4', floor: 2},
{from: 'C-4', to: 'A-4', floor: 3},
{from: 'C-4', to: 'A-4', floor: 4},
{from: 'A-4', to: 'A-3', floor: 1},
{from: 'A-4', to: 'A-3', floor: 2},
{from: 'A-4', to: 'A-3', floor: 3},
{from: 'A-4', to: 'A-3', floor: 4},
{from: 'A-4', to: 'D-1', floor: 0},
{from: 'A-4', to: 'D-1', floor: 1},
{from: 'A-3', to: 'U-2', floor: 0},
{from: 'A-3', to: 'U-2', floor: 1},
{from: 'D-1', to: 'U-2', floor: 0},
{from: 'C-6', to: 'C-5', floor: 2},
{from: 'C-6', to: 'C-5', floor: 3},
{from: 'C-6', to: 'C-5', floor: 4},
{from: 'C-7', to: 'C-5', floor: 2},
{from: 'C-7', to: 'C-5', floor: 3},
{from: 'C-7', to: 'C-5', floor: 4}
] AS connections

UNWIND connections AS c
MATCH (a: BLDG {name: c.from})
MATCH (b: BLDG {name: c.to})
CREATE (a)-[:CONNECTED_TO {floor: c.floor}]->(b);


WITH [
{from: 'B-1', to: 'B-2', floor: -1},
{from: 'B-2', to: 'B-3', floor: -1},
{from: 'B-3', to: 'B-4', floor: -1},
{from: 'B-1', to: 'H-B1B2', floor: 0},
{from: 'B-1', to: 'H-B1B2', floor: 1},
{from: 'B-2', to: 'H-B1B2', floor: 0},
{from: 'B-2', to: 'H-B1B2', floor: 1}
] AS connections

UNWIND connections AS c
MATCH (a: BLDG {name: c.from})
MATCH (b: BLDG {name: c.to})
CREATE (a)-[:CONNECTED_TO {floor: c.floor}]->(b);
```

### Adding faculty headquarters and linking them to buildings using MATCH and MERGE
```cypher
WITH [
  {label:'1', faculty:'Faculty of Geology, Geophysics, and Environmental Protection', building:'A-0'},
  {label:'2', faculty:'Centre for Cooperation and Technology Transfer', building:'C-1'},
  {label:'3', faculty:'Faculty of Drilling, Oil, and Gas', building:'A-1'},
  {label:'4', faculty:'Faculty of Non-Ferrous Metals', building:'A-2'},
  {label:'5', faculty:'AGH University Museum', building:'C-2'},
  {label:'6', faculty:'Main Library', building:'U-1'},
  {label:'7', faculty:'Faculty of Humanities', building:'C-7'},
  {label:'8', faculty:'Centre of Energy', building:'C-5'},
  {label:'9', faculty:'AGH University Doctoral School', building:'A-3'},
  {label:'10', faculty:'Admissions Centre', building:'U-2'},
  {label:'11', faculty:'Faculty of Geo-Data Science, Geodesy, and Environmental Engineering', building:'C-4'},
  {label:'12', faculty:'Centre of Excellence in Artificial Intelligence', building:'C-6'},
  {label:'13', faculty:'Faculty of Civil Engineering and Resource Management', building:'A-4'},
  {label:'14', faculty:'Faculty of Electrical Engineering, Automatics, Computer Science, and Biomedical Engineering', building:'B-1'},
  {label:'15', faculty:'Faculty of Mechanical Engineering and Robotics', building:'B-2'}
] AS items
UNWIND items AS it
MERGE (f:HQ {name: it.faculty, label: it.label})
WITH f, it
MATCH (b:BLDG {name: it.building})
MERGE (f)-[:LOCATED_IN]->(b);
```

### Adding classroom nodes and linking them to buildings using MATCH and MERGE
```cypher
MERGE (room:CLS {roomNumber: 315, floor: 3})
WITH room
MATCH (c2:BLDG {name: 'C-2'})
MERGE (room)-[:IS_IN]->(c2);

MERGE (room:CLS {roomNumber: 429, floor: 4})
WITH room
MATCH (c2:BLDG {name: 'C-2'})
MERGE (room)-[:IS_IN]->(c2);
```

---

## Exercise 2: Analytics Queries

### Finding buildings not connected to other buildings
```cypher
MATCH (b:BLDG)
WHERE NOT (b)-[:CONNECTED_TO]-()
RETURN b.name AS isolated_building;
```

**Result:**
```
╒═════════════════╕
│isolated_building│
╞═════════════════╡
│"S-1"            │
├─────────────────┤
│"S-2"            │
├─────────────────┤
│"U-1"            │
└─────────────────┘
```

### Counting service facilities
```cypher
MATCH (b:BLDG:SRV)
RETURN count(b) AS service_facilities_count;
```

**Result: 1**

### Counting connections per building
```cypher
MATCH (b:BLDG)
OPTIONAL MATCH (b)-[r:CONNECTED_TO]-()
RETURN b.name AS building, count(DISTINCT r) AS connections_count
ORDER BY connections_count DESC;
```

**Result:**
```
╒════════╤═════════════════╕
│building│connections_count│
╞════════╪═════════════════╡
│"C-6"   │13               │
├────────┼─────────────────┤
│"C-7"   │13               │
├────────┼─────────────────┤
│"A-4"   │10               │
├────────┼─────────────────┤
│"C-2"   │10               │
├────────┼─────────────────┤
│"C-3"   │10               │
├────────┼─────────────────┤
│"C-4"   │9                │
├────────┼─────────────────┤
│"A-1"   │7                │
├────────┼─────────────────┤
│"A-3"   │6                │
├────────┼─────────────────┤
│"C-1"   │6                │
├────────┼─────────────────┤
│"C-5"   │6                │
├────────┼─────────────────┤
│"A-2"   │4                │
├────────┼─────────────────┤
│"B-2"   │4                │
├────────┼─────────────────┤
│"H-A1"  │4                │
├────────┼─────────────────┤
│"H-A2"  │4                │
├────────┼─────────────────┤
│"H-B1B2"│4                │
├────────┼─────────────────┤
│"D-1"   │3                │
├────────┼─────────────────┤
│"U-2"   │3                │
├────────┼─────────────────┤
│"B-1"   │3                │
├────────┼─────────────────┤
│"A-0"   │2                │
├────────┼─────────────────┤
│"B-3"   │2                │
├────────┼─────────────────┤
│"B-4"   │1                │
├────────┼─────────────────┤
│"S-1"   │0                │
├────────┼─────────────────┤
│"S-2"   │0                │
├────────┼─────────────────┤
│"U-1"   │0                │
└────────┴─────────────────┘
```

### Finding buildings and floors connected to A-1
```cypher
MATCH (a:BLDG {name: 'A-1'})-[r:CONNECTED_TO]-(b:BLDG)
RETURN DISTINCT b.name AS connected_building, collect(DISTINCT r.floor) AS floors
ORDER BY b.name;
```

**Result:**
```
╒══════════════════╤══════╕
│connected_building│floors│
╞══════════════════╪══════╡
│"A-0"             │[2, 1]│
├──────────────────┼──────┤
│"A-2"             │[2, 1]│
├──────────────────┼──────┤
│"C-1"             │[1]   │
├──────────────────┼──────┤
│"H-A1"            │[2, 1]│
└──────────────────┴──────┘
```

### Finding all paths from C-4 to C-3
```cypher
MATCH path = (start:BLDG {name: 'C-4'})-[:CONNECTED_TO*..5]-(end:BLDG {name: 'C-3'})
WHERE ALL(node IN nodes(path) WHERE size([n IN nodes(path) WHERE n = node]) = 1)
WITH DISTINCT 
  [node IN nodes(path) | node.name] AS path_buildings,
  [rel IN relationships(path) | rel.floor] AS floors,
  length(path) AS hops
RETURN path_buildings, floors, hops
ORDER BY hops;
```

**Result:**
```
╒═══════════════════════════════════╤════════════╤════╕
│path_buildings                     │floors      │hops│
╞═══════════════════════════════════╪════════════╪════╡
│["C-4", "C-6", "C-7", "C-3"]       │[0, 0, 0]   │3   │
├───────────────────────────────────┼────────────┼────┤
│["C-4", "C-6", "C-7", "C-3"]       │[0, 0, 1]   │3   │
├───────────────────────────────────┼────────────┼────┤
│["C-4", "C-6", "C-7", "C-3"]       │[0, 0, 2]   │3   │
├───────────────────────────────────┼────────────┼────┤
│["C-4", "C-6", "C-7", "C-3"]       │[0, 0, 3]   │3   │
├───────────────────────────────────┼────────────┼────┤
│["C-4", "C-6", "C-7", "C-3"]       │[0, 0, 4]   │3   │
├───────────────────────────────────┼────────────┼────┤
│["C-4", "C-6", "C-7", "C-3"]       │[0, 1, 0]   │3   │
├───────────────────────────────────┼────────────┼────┤
...

Found 350 different paths

```

### Finding shortest path from C-3 to A-4
```cypher
MATCH path = shortestPath((start:BLDG {name: 'C-3'})-[:CONNECTED_TO*]-(end:BLDG {name: 'A-4'}))
RETURN [node IN nodes(path) | node.name] AS buildings,
       [rel IN relationships(path) | rel.floor] AS floors,
       length(path) AS path_length;
```

**Result:**

```
╒═══════════════════════════════════╤════════════╤═══════════╕
│buildings                          │floors      │path_length│
╞═══════════════════════════════════╪════════════╪═══════════╡
│["C-3", "C-7", "C-6", "C-4", "A-4"]│[4, 4, 2, 4]│4          │
└───────────────────────────────────┴────────────┴───────────┘
```

### Finding buildings with at least 3 direct connections
```cypher
MATCH (b:BLDG)-[:CONNECTED_TO]-(connected:BLDG)
WITH b, collect(DISTINCT connected) AS neighbors
WHERE size(neighbors) >= 3
RETURN b.name AS building, size(neighbors) AS neighbor_count
ORDER BY neighbor_count DESC;
```

**Result:**
```
╒════════╤══════════════╕
│building│neighbor_count│
╞════════╪══════════════╡
│"A-1"   │4             │
├────────┼──────────────┤
│"A-4"   │3             │
├────────┼──────────────┤
│"C-7"   │3             │
├────────┼──────────────┤
│"C-6"   │3             │
├────────┼──────────────┤
│"B-2"   │3             │
└────────┴──────────────┘
```

---

## Exercise 3: Modifications

### Changing HQ label to HEADQUARTER on all vertices
```cypher
MATCH (h:HQ)
SET h:HEADQUARTER
REMOVE h:HQ;
```

### Adding 1000 to each classroom number
```cypher
MATCH (c:CLS)
SET c.roomNumber = c.roomNumber + 1000;
```

### Removing all vertices representing classrooms
```cypher
MATCH (c:CLS)
DETACH DELETE c;
```