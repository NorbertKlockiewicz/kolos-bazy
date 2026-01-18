# Lab 6: CouchDB/NoSQL Document Databases - Practice Questions

## Question 1: Basic Document Operations

### Task: Create a shopping list in CouchDB

### Create these documents:

```json
{"product": "milk", "price": 3.50, "store": "Tesco", "quantity": 2, "category": "dairy"}
{"product": "bread", "price": 2.20, "store": "Biedronka", "quantity": 1, "category": "bakery"}
{"product": "eggs", "price": 8.00, "store": "Tesco", "quantity": 1, "category": "dairy"}
{"product": "tomato", "price": 2.00, "store": "Biedronka", "category": "produce"}
{"product": "tomato", "price": 3.00, "store": "Lidl", "category": "produce"}
{"product": "cheese", "price": 12.50, "store": "Auchan", "category": "dairy"}
{"product": "laptop", "price": 2500, "store": "MediaMarkt", "category": "electronics", "warranty": "2 years"}
```

### Questions:

1. **Write the HTTP request** (or curl command) to insert each document

2. **Write a request** to update the "tomato" document from Lidl to add "quantity": 3

3. **Write a request** to delete the laptop document

4. **Explain** why the laptop document has a "warranty" field that others don't (schema flexibility)

---

## Question 2: Map/Reduce Views - Sorting

### Given the shopping list from Question 1:

### Task 1: Create a view that sorts all items by store name

```javascript
// Map function
function(doc) {
  // Your code here
}
```

### Questions:

1. **Write the complete map function**

2. **What is the URL** to query this view?

3. **What does the output look like?** (show example with 2-3 items)

4. **Write the URL** to get only items from "Biedronka"

---

## Question 3: Map/Reduce Views - Counting

### Given the shopping list from Question 1:

### Task: Create a view that counts the number of items to buy at each store

### Questions:

1. **Write the map function**

2. **Write the reduce function** (use built-in `_count` or custom)

3. **What is the URL** to query this view with grouping?

4. **What does the output look like?**

5. **How would you get just the count for "Tesco"?** (provide URL)

---

## Question 4: Map/Reduce Views - Average Price

### Given the shopping list from Question 1:

### Task: Create a view that calculates the average price of each product across all stores

**Important:** Must handle reduce and rereduce properly!

### Questions:

1. **Write the map function** (what should it emit?)

2. **Write the reduce function** (handle both reduce and rereduce)

3. **Explain the difference** between reduce and rereduce

4. **Why can't you use** `_sum` or `_count` directly for average?

---

## Question 5: Map/Reduce Views - Complex Aggregation

### Given the shopping list from Question 1:

### Create views for:

1. **Total cost per store** (price × quantity, where quantity exists)

2. **Products available at multiple stores**

3. **Most expensive item in each category**

### For each view, provide:
- Map function
- Reduce function (if needed)
- Example output
- Query URL

---

## Question 6: Querying Views with Parameters

### Given a view that emits: `emit([doc.store, doc.category], doc.price);`

### Questions:

1. **What does this view's output look like?** (show sample)

2. **Write the URL** to get all dairy products from Tesco

3. **Write the URL** to get all products from Biedronka (any category)

4. **Explain** the difference between `key`, `startkey`, and `endkey` parameters

5. **Write the URL** to get products sorted by category, then by store (would this view work? why/why not?)

---

## Question 7: Document Design Patterns

### Scenario: You're designing a blog system in CouchDB

**Option A: Embedded comments**
```json
{
  "_id": "post1",
  "title": "My First Post",
  "content": "...",
  "author": "Alice",
  "comments": [
    {"author": "Bob", "text": "Great post!", "date": "2024-01-15"},
    {"author": "Charlie", "text": "Thanks!", "date": "2024-01-16"}
  ]
}
```

**Option B: Separate comment documents**
```json
// Post
{"_id": "post1", "title": "My First Post", "content": "...", "author": "Alice"}

// Comments
{"_id": "comment1", "post_id": "post1", "author": "Bob", "text": "Great post!"}
{"_id": "comment2", "post_id": "post1", "author": "Charlie", "text": "Thanks!"}
```

### Questions:

1. **List pros and cons** of each approach

2. **Which approach would you choose if:**
   - Posts typically have 0-5 comments
   - Posts can have hundreds of comments
   - You need to query all comments by a specific author

3. **Write a map function** for Option B to get all comments for a specific post

---

## Question 8: Schema Flexibility

### Given these documents in the same database:

```json
{"_id": "1", "type": "person", "name": "Alice", "age": 30, "email": "alice@example.com"}
{"_id": "2", "type": "person", "name": "Bob", "phone": "+48123456789"}
{"_id": "3", "type": "company", "name": "ACME Corp", "employees": 100, "industry": "Tech"}
{"_id": "4", "type": "person", "name": "Charlie", "age": 25, "address": {"city": "Warsaw", "country": "Poland"}}
```

### Tasks:

1. **Write a map function** to get all people with their contact info (email or phone)

2. **Write a map function** to handle the nested address field safely

3. **Explain** how you would handle missing fields in map functions (null checks)

4. **Is schema flexibility** a good or bad thing? Discuss trade-offs.

---

## Question 9: View Collation and Sorting

### Given these emitted keys in a view:

```javascript
emit(["Biedronka", "dairy"], ...);
emit(["Biedronka", "produce"], ...);
emit(["Lidl", "dairy"], ...);
emit(["Tesco", "bakery"], ...);
emit(["Tesco", "dairy"], ...);
```

### Questions:

1. **In what order** will these appear in the view output? (write them in order)

2. **Write the query parameters** to get:
   - All items from Tesco (any category)
   - All dairy products (any store)
   - Items from Biedronka in produce category only

3. **Can you get "all dairy products"** easily with this view design? Why or why not?

4. **Suggest a better key structure** if you frequently query by category across all stores

---

## Question 10: Real-World Scenario - Inventory System

### Design a CouchDB database for a multi-store inventory system:

**Requirements:**
- Multiple stores (Biedronka, Lidl, Tesco)
- Products with SKU, name, category, price
- Stock quantity per store (can be different at each store)
- Need to query:
  - Total stock for a product across all stores
  - Products low in stock (quantity < 10) at any store
  - Average price per category
  - All products available at a specific store

### Tasks:

1. **Design the document structure** (show example documents)

2. **Create at least 3 map/reduce views** to support the query requirements

3. **Write complete map and reduce functions** for each view

4. **Explain your design choices** (why this structure? why these views?)

---

# Tips for Lab 6 Questions:

## CouchDB Basics:

### HTTP API:
```bash
# Create document
curl -X PUT http://localhost:5984/dbname/docid -d '{"key":"value"}'

# Get document
curl http://localhost:5984/dbname/docid

# Update document (need _rev)
curl -X PUT http://localhost:5984/dbname/docid -d '{"_rev":"...","key":"new_value"}'

# Delete document
curl -X DELETE http://localhost:5984/dbname/docid?rev=...

# Create database
curl -X PUT http://localhost:5984/dbname
```

### Every document has:
- `_id` - Unique identifier (you provide or auto-generated)
- `_rev` - Revision (for MVCC, prevents conflicts)

## Map/Reduce Views:

### Map Function:
```javascript
function(doc) {
  // Emit key-value pairs
  emit(key, value);
}
```

**Key points:**
- Runs once per document
- Can emit 0, 1, or multiple times per document
- Keys determine sort order
- Values are passed to reduce

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

**Key points:**
- Optional (only for aggregation)
- Must be pure function (deterministic)
- Must handle rereduce case
- Output must be same type as values

### Built-in Reduce Functions:
- `_count` - Count rows
- `_sum` - Sum values
- `_stats` - Statistics (sum, count, min, max, sumsqr)

## View Queries:

### URL Parameters:

```
# Basic query
/db/_design/designdoc/_view/viewname

# With key
?key="exact_key"

# With key range
?startkey="a"&endkey="z"

# With complex key
?key=["store","category"]

# With grouping (for reduce views)
?group=true
?group_level=1

# Limit results
?limit=10

# Descending order
?descending=true
```

### Key Ranges with Arrays:

```javascript
// Get all items from "Tesco"
?startkey=["Tesco"]&endkey=["Tesco",{}]

// Get all dairy products
?startkey=[null,"dairy"]&endkey=[{},"dairy"]  // Won't work!

// For this you need a different view that emits [category, store]
```

## Map/Reduce Patterns:

### 1. Simple Listing (Sort):
```javascript
// Map only, no reduce
function(doc) {
  emit(doc.store, doc);  // Sort by store
}
```

### 2. Count:
```javascript
// Map
function(doc) {
  emit(doc.store, 1);
}
// Reduce
_count
```

### 3. Sum:
```javascript
// Map
function(doc) {
  emit(doc.store, doc.price);
}
// Reduce
_sum
```

### 4. Average (Complex):
```javascript
// Map
function(doc) {
  emit(doc.category, doc.price);
}
// Reduce
function(keys, values, rereduce) {
  if (rereduce) {
    var sum = 0;
    var count = 0;
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

### 5. Group By Multiple Keys:
```javascript
// Map
function(doc) {
  emit([doc.store, doc.category], doc.price);
}
// Query with group_level=1 for store-level, group_level=2 for store+category
```

## Common Patterns:

### Check if field exists:
```javascript
function(doc) {
  if (doc.price !== undefined && doc.price !== null) {
    emit(doc._id, doc.price);
  }
}
```

### Handle nested fields safely:
```javascript
function(doc) {
  if (doc.address && doc.address.city) {
    emit(doc.address.city, doc.name);
  }
}
```

### Filter by type:
```javascript
function(doc) {
  if (doc.type === "person") {
    emit(doc.name, doc);
  }
}
```

### Emit multiple times:
```javascript
function(doc) {
  if (doc.tags) {
    for (var i = 0; i < doc.tags.length; i++) {
      emit(doc.tags[i], doc.title);
    }
  }
}
```

## Key Concepts:

### 1. Schema Flexibility
- ✅ Different documents can have different fields
- ✅ Easy to evolve schema over time
- ❌ No enforcement, can lead to inconsistency
- ❌ Must handle missing fields in map functions

### 2. MVCC (Multi-Version Concurrency Control)
- Every update creates new revision (_rev)
- Need current _rev to update (prevents lost updates)
- Conflicts detected automatically
- No locking needed

### 3. Map/Reduce Materialized Views
- Views are indexes, not stored queries
- Computed incrementally (only new/changed docs)
- Results are cached (fast reads)
- Views are eventually consistent

### 4. No JOINs
- CouchDB is document-oriented, not relational
- Either embed related data or denormalize
- Or query multiple views and join in application

## Common Mistakes:

❌ **Forgetting null checks in map:**
```javascript
emit(doc.price, doc.name);  // Error if price is undefined!
```
✅ Fix:
```javascript
if (doc.price) {
  emit(doc.price, doc.name);
}
```

❌ **Not handling rereduce:**
```javascript
function(keys, values, rereduce) {
  return sum(values);  // Breaks on rereduce!
}
```
✅ Fix:
```javascript
function(keys, values, rereduce) {
  if (rereduce) {
    return sum(values);  // Values are already sums
  } else {
    return sum(values);  // Values are raw values
  }
}
```

❌ **Using wrong query parameters:**
```javascript
// Want: all items from Tesco
?key="Tesco"  // Only works if you emit single key

// For array keys:
?startkey=["Tesco"]&endkey=["Tesco",{}]  // Correct
```

## Test Writing Tips:

1. ✅ **Always check for undefined fields** in map functions
2. ✅ **Handle rereduce case** in custom reduce functions
3. ✅ **Use array keys** for multi-level grouping
4. ✅ **Test your reduce function** with both reduce and rereduce
5. ✅ **Remember:** Map/reduce is for aggregation, not for transforming individual docs
6. ✅ **Know built-in reduces:** `_count`, `_sum`, `_stats`
7. ✅ **Understand query parameters:** `key`, `startkey`, `endkey`, `group`, `group_level`
