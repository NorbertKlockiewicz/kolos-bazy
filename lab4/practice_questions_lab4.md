# Lab 4: Transactions & Concurrency - Practice Questions

## Question 1: Non-Repeatable Reads

### Scenario Setup:

```sql
CREATE TABLE accounts (
    account_id SERIAL PRIMARY KEY,
    owner VARCHAR(100),
    balance NUMERIC(10,2)
);

INSERT INTO accounts (owner, balance)
VALUES ('Alice', 1000.00), ('Bob', 500.00);
```

### Task:

**Design a scenario** where two concurrent transactions at **READ COMMITTED** isolation level cause a non-repeatable read problem.

Your answer should include:
1. **Transaction 1 steps** (with BEGIN, SQL statements, COMMIT)
2. **Transaction 2 steps** (with BEGIN, SQL statements, COMMIT)
3. **Execution timeline** showing when each step happens
4. **Explanation** of what went wrong and why it's problematic

---

## Question 2: Preventing Non-Repeatable Reads with Locking

### Given the scenario from Question 1:

### Tasks:

1. **Solve the problem using explicit table-level locking** at READ COMMITTED level
   - Show the modified transactions with LOCK statements

2. **Solve the problem using REPEATABLE READ isolation level** (no explicit locks)
   - Show the modified transactions with SET TRANSACTION

3. **Compare both approaches** - which is better and why?

---

## Question 3: Serialization Errors

### Scenario:

```sql
CREATE TABLE inventory (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100),
    stock_quantity INTEGER
);

INSERT INTO inventory (product_name, stock_quantity)
VALUES ('Widget A', 10), ('Widget B', 5);
```

### Tasks:

1. **Create a scenario** that generates a serialization error at REPEATABLE READ isolation level

2. Include:
   - Transaction 1 steps
   - Transaction 2 steps
   - Execution timeline
   - The exact error message you would receive

3. **Explain why** the serialization error occurred

---

## Question 4: SELECT FOR UPDATE vs SELECT FOR SHARE

### Scenario:

```sql
CREATE TABLE seats (
    seat_id SERIAL PRIMARY KEY,
    row_number INTEGER,
    seat_number INTEGER,
    is_reserved BOOLEAN DEFAULT FALSE,
    reserved_by VARCHAR(100)
);
```

### Questions:

1. **Write two concurrent transactions** where:
   - Transaction 1 uses `SELECT FOR UPDATE`
   - Transaction 2 tries to read the same rows
   - Show what happens (does T2 wait? can T2 read?)

2. **Write two concurrent transactions** where:
   - Transaction 1 uses `SELECT FOR SHARE`
   - Transaction 2 tries to read the same rows
   - Transaction 3 tries to update the same rows
   - Show what happens to T2 and T3

3. **Explain the difference** between FOR UPDATE and FOR SHARE

4. **When would you use each one?**

---

## Question 5: Deadlock Detection

### Scenario:

```sql
CREATE TABLE accounts (
    account_id SERIAL PRIMARY KEY,
    balance NUMERIC(10,2)
);

INSERT INTO accounts VALUES (1, 1000), (2, 500);
```

### Task:

**Create a scenario that causes a deadlock:**

1. Show Transaction 1 acquiring locks on resources in order: A → B
2. Show Transaction 2 acquiring locks on resources in order: B → A
3. Draw or describe the wait-for graph
4. Explain how PostgreSQL detects and resolves the deadlock

---

## Question 6: Lost Updates Problem

### Scenario:

```sql
CREATE TABLE product_views (
    product_id INTEGER PRIMARY KEY,
    view_count INTEGER
);

INSERT INTO product_views VALUES (1, 100);
```

Two concurrent transactions both want to increment the view_count:

```sql
-- Transaction 1
BEGIN;
SELECT view_count FROM product_views WHERE product_id = 1; -- reads 100
-- ... some processing ...
UPDATE product_views SET view_count = 101 WHERE product_id = 1;
COMMIT;

-- Transaction 2
BEGIN;
SELECT view_count FROM product_views WHERE product_id = 1; -- reads 100
-- ... some processing ...
UPDATE product_views SET view_count = 101 WHERE product_id = 1;
COMMIT;
```

### Tasks:

1. **Explain what problem occurs** (what will the final count be?)

2. **Fix this using SELECT FOR UPDATE**

3. **Fix this using a single UPDATE statement** (atomic operation)

4. **Which solution is better and why?**

---

## Question 7: Isolation Levels Comparison

### Given these scenarios:

**Scenario A**: Bank account transfer (read balance, calculate, update balance)
**Scenario B**: Inventory check (read stock, if > 0 then decrement)
**Scenario C**: Reporting query (read-only, aggregate large dataset)
**Scenario D**: Ticket booking system (check availability, reserve seat)

### Tasks:

For each scenario, answer:

1. **What isolation level would you use?**
   - READ COMMITTED
   - REPEATABLE READ
   - SERIALIZABLE

2. **Why did you choose this level?**

3. **What concurrency problems could occur at lower levels?**

---

## Question 8: Phantom Reads

### Scenario:

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    total_amount NUMERIC(10,2),
    status VARCHAR(20)
);
```

### Task:

**Create a scenario** that demonstrates phantom reads at REPEATABLE READ level:

1. Transaction 1 reads all orders for customer 123 twice
2. Transaction 2 inserts a new order for customer 123 between the reads
3. Show what Transaction 1 sees in both reads
4. Explain if this is possible at REPEATABLE READ level in PostgreSQL

---

## Question 9: Write Skew

### Scenario:

```sql
CREATE TABLE shifts (
    shift_id SERIAL PRIMARY KEY,
    date DATE,
    employee_id INTEGER,
    shift_type VARCHAR(20) -- 'morning' or 'evening'
);

-- Rule: Each employee must work at least one shift per day
```

### Task:

**Create a scenario** where two concurrent transactions at REPEATABLE READ cause write skew:

1. Alice is scheduled for morning shift on 2024-01-15
2. Alice is scheduled for evening shift on 2024-01-15
3. Transaction 1 tries to delete Alice's morning shift (checks she still has evening)
4. Transaction 2 tries to delete Alice's evening shift (checks she still has morning)
5. Both succeed, violating the business rule!

Show:
- Complete transaction steps
- Why REPEATABLE READ doesn't prevent this
- How to fix it (SELECT FOR UPDATE or SERIALIZABLE)

---

## Question 10: Real-World Concurrency Scenario

### E-commerce Checkout System:

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    stock INTEGER,
    price NUMERIC(8,2)
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    total_amount NUMERIC(10,2),
    status VARCHAR(20)
);

CREATE TABLE order_items (
    item_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id),
    product_id INTEGER REFERENCES products(product_id),
    quantity INTEGER,
    price NUMERIC(8,2)
);
```

### Scenario:

Multiple customers are trying to buy the last 3 units of a popular product simultaneously.

### Tasks:

1. **Write a transaction** that correctly handles checkout with proper locking

2. **Explain what happens** when 5 customers try to buy 2 units each, but only 3 units are in stock

3. **What isolation level and locking strategy** would you use?

4. **What are the trade-offs** between correctness and performance?

---

# Tips for Lab 4 Questions:

## Isolation Levels:

### READ COMMITTED (PostgreSQL default):
- **Prevents:** Dirty reads
- **Allows:** Non-repeatable reads, phantom reads
- **Use when:** Most general-purpose queries
- **Performance:** Best

### REPEATABLE READ:
- **Prevents:** Dirty reads, non-repeatable reads, phantom reads (in PostgreSQL)
- **Allows:** Serialization errors, write skew
- **Use when:** Need consistent snapshot of data
- **Performance:** Good, but more serialization errors

### SERIALIZABLE:
- **Prevents:** All concurrency anomalies
- **Allows:** Nothing (truly serializable)
- **Use when:** Critical financial transactions, strict consistency needed
- **Performance:** More serialization errors, more retries needed

## PostgreSQL Isolation Level Behavior:

**Important:** PostgreSQL's REPEATABLE READ is stronger than SQL standard!
- PostgreSQL REPEATABLE READ prevents phantom reads (uses MVCC snapshot)
- Other databases' REPEATABLE READ may allow phantom reads

## Lock Types:

### Table-Level Locks:

```sql
BEGIN;
LOCK TABLE table_name IN lock_mode MODE;
-- ... operations ...
COMMIT;
```

**Lock modes:**
- `ACCESS SHARE` - Read lock (SELECT)
- `ROW EXCLUSIVE` - Write lock (INSERT, UPDATE, DELETE)
- `EXCLUSIVE` - Blocks all concurrent access
- `ACCESS EXCLUSIVE` - Strongest lock (ALTER TABLE, DROP TABLE)

### Row-Level Locks:

```sql
-- Exclusive lock (blocks other writes)
SELECT * FROM table WHERE ... FOR UPDATE;

-- Shared lock (allows other reads, blocks writes)
SELECT * FROM table WHERE ... FOR SHARE;
```

## Common Concurrency Problems:

### 1. Lost Updates
**Problem:** Two transactions read same value, both update, one overwrites the other
**Solution:** SELECT FOR UPDATE, or atomic UPDATE

### 2. Non-Repeatable Reads
**Problem:** Same SELECT returns different results within same transaction
**Solution:** REPEATABLE READ isolation level or explicit locking

### 3. Phantom Reads
**Problem:** Same query returns different rows within same transaction
**Solution:** REPEATABLE READ (in PostgreSQL) or SERIALIZABLE

### 4. Write Skew
**Problem:** Two transactions read overlapping data, make decisions, both commit
**Solution:** SELECT FOR UPDATE or SERIALIZABLE

### 5. Dirty Reads
**Problem:** Reading uncommitted data from other transaction
**Solution:** Never happens in PostgreSQL (minimum is READ COMMITTED)

## Transaction Syntax:

```sql
-- Set isolation level
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- or
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Explicit locking
LOCK TABLE accounts IN EXCLUSIVE MODE;

-- Row-level locking
SELECT * FROM accounts WHERE account_id = 1 FOR UPDATE;

-- Read-only transaction (optimization)
BEGIN TRANSACTION READ ONLY;

-- Commit or rollback
COMMIT;
ROLLBACK;
```

## Common Errors:

### Serialization Error:
```
ERROR: could not serialize access due to concurrent update
```
**Solution:** Retry transaction

### Deadlock Error:
```
ERROR: deadlock detected
DETAIL: Process 1234 waits for ShareLock on transaction 5678;
blocked by process 5679.
```
**Solution:** One transaction automatically aborted, retry it

## Writing Scenario Answers:

**Good format:**
```
-- Transaction 1
BEGIN;                                          -- T1: Start
SELECT balance FROM accounts WHERE id = 1;      -- T1: reads 1000
-- (Transaction 2 executes here)
UPDATE accounts SET balance = 900 WHERE id = 1; -- T1: writes 900
COMMIT;                                         -- T1: Commits

-- Transaction 2
                                                -- T2: Start
BEGIN;                                          -- (after T1's SELECT)
UPDATE accounts SET balance = 800 WHERE id = 1; -- T2: writes 800
COMMIT;                                         -- T2: Commits

Result: T1's update is lost! Final balance is 800, not 900.
```

## Key Concepts to Remember:

1. **MVCC (Multi-Version Concurrency Control)** - PostgreSQL uses snapshots
2. **Locks are released at COMMIT** - not before
3. **FOR UPDATE blocks other FOR UPDATE** and UPDATEs
4. **FOR SHARE blocks UPDATEs** but not other FOR SHARE or plain SELECTs
5. **Higher isolation = more safety but more errors** - need retry logic
6. **Always handle serialization errors** in application code
