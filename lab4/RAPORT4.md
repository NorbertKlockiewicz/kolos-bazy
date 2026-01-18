# Advanced Database Systems - Lab 4 - Norbert Klockiewicz

## Transaction Isolation Levels

This lab demonstrates various concurrency phenomena that occur at different transaction isolation levels in PostgreSQL.

---

## Exercise 1: Non-Repeatable Read (READ COMMITTED)

### Scenario: Non-Repeatable Read

The scenario demonstrates how T1 reads the same row twice but gets different values because T2 updates the data between the two reads.

```sql
-- T1: BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
T1: BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- T1: First read of John Smith's vacation_days
T1: SELECT vacation_days FROM employees WHERE name = 'John Smith';
-- Result: 10

-- T2: Concurrent transaction starts
T2: BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- T2: Update John Smith's vacation days
T2: UPDATE employees SET vacation_days = 5 WHERE name = 'John Smith';

-- T2: Commit the update
T2: COMMIT;
-- At this point, the data is visible to other transactions (READ COMMITTED allows this)

-- T1: Read the same row again
T1: SELECT vacation_days FROM employees WHERE name = 'John Smith';
-- Result: 5 (DIFFERENT VALUE!)

-- T1: Commit the transaction
T1: COMMIT;
```

### What Happened

1. **T1 First Read**: T1 reads John Smith's vacation_days and gets **10**
2. **T2 Update**: T2 updates the vacation_days to **5** and commits
3. **T1 Second Read**: T1 reads the same row again and gets **5** (different value!)
4. **Non-Repeatable Read**: The same SELECT query within the same transaction returned different results

## Exercise 2: Explicit Locking (READ COMMITTED with Locks)

### Solution: Using Explicit Locks

By acquiring a lock on the table before reading, we prevent other transactions from modifying the data during our transaction.

```sql
-- T1: BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
T1: BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- T1: Acquire exclusive lock on employees table
-- This prevents T2 from updating while T1 is working
T1: LOCK TABLE employees IN EXCLUSIVE MODE;

-- T1: First read of John Smith's vacation_days
T1: SELECT vacation_days FROM employees WHERE name = 'John Smith';
-- Result: 10

-- T2: Try to begin a transaction
T2: BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- T2: Attempt to update John Smith's vacation days
-- This will BLOCK because T1 holds an exclusive lock
T2: UPDATE employees SET vacation_days = 5 WHERE name = 'John Smith';
-- WAITING FOR LOCK...

-- T1: Read the same row again
T1: SELECT vacation_days FROM employees WHERE name = 'John Smith';
-- Result: 10 (SAME VALUE - lock prevents changes)

-- T1: Commit the transaction and release the lock
T1: COMMIT;
-- Lock is now released

-- T2: Update now proceeds (unblocked)
T2: UPDATE employees SET vacation_days = 5 WHERE name = 'John Smith';

-- T2: Commit
T2: COMMIT;
```

### What Happened

1. **T1 Acquires Lock**: T1 acquires an EXCLUSIVE lock on the employees table
2. **T1 First Read**: T1 reads John Smith's vacation_days and gets **10**
3. **T2 Blocked**: T2's UPDATE is blocked waiting for the lock to be released
4. **T1 Second Read**: T1 reads the same row again and gets **10** (same value!)
5. **T1 Commits**: T1 commits and releases the lock
6. **T2 Proceeds**: T2's UPDATE now executes and commits

---

## Exercise 3: REPEATABLE READ Isolation Level

### Key Difference from Exercise 1

Exercise 1 (READ COMMITTED):
- T1's first read sees version at time T1 starts
- T2 modifies and commits
- T1's second read sees T2's changes (non-repeatable read)

Exercise 3 (REPEATABLE READ):
- T1's first read creates a snapshot of the database
- T2 modifies and commits
- T1's second read still sees the snapshot (consistent read)
- No non-repeatable read occurs

### Scenario: Repeatable Read Prevents Non-Repeatable Reads

```sql
-- T1: BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
T1: BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- T1: First read of John Smith's vacation_days
-- A snapshot of the database is created at this moment
T1: SELECT vacation_days FROM employees WHERE name = 'John Smith';
-- Result: 10

-- T2: Concurrent transaction starts
T2: BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- T2: Update John Smith's vacation days
T2: UPDATE employees SET vacation_days = 5 WHERE name = 'John Smith';

-- T2: Commit the update
T2: COMMIT;
-- Update is committed and visible to new transactions

-- T1: Read the same row again
-- T1 is still using its original snapshot from when the transaction started
T1: SELECT vacation_days FROM employees WHERE name = 'John Smith';
-- Result: 10 (SAME VALUE - sees the snapshot, not T2's changes!)

-- T1: Commit the transaction
T1: COMMIT;
```

### What Happened

1. **T1 Starts Transaction**: REPEATABLE READ isolation level creates a snapshot
2. **T1 First Read**: T1 reads John Smith's vacation_days and gets **10** (from the snapshot)
3. **T2 Updates**: T2 updates the vacation_days to **5** and commits
4. **T1 Second Read**: T1 reads the same row and gets **10** (still sees the snapshot!)
5. **Non-Repeatable Read Prevented**: The isolation level prevents T1 from seeing T2's committed changes
---

## Exercise 4: Serialization Errors at REPEATABLE READ

### Serialization Conflict Scenario

A serialization conflict occurs when:
1. T1 reads a value and plans to modify it (based on its snapshot)
2. T2 reads the same value and commits a modification first
3. T1 tries to modify the value (based on its now-stale snapshot)
4. PostgreSQL detects this conflict and aborts T1 with a serialization error

### Scenario: Write-Write Conflict with REPEATABLE READ

```sql
-- Initial state: John Smith has 10 vacation days
-- Both transactions will try to modify the same row

-- T1: BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
T1: BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- T1: Read John Smith's current vacation days
T1: SELECT vacation_days FROM employees WHERE name = 'John Smith';
-- Result: 10

-- T1: Plans to give John 5 more vacation days
-- (reads current value, will compute: 10 + 5 = 15)

-- T2: BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
T2: BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- T2: Read John Smith's current vacation days
T2: SELECT vacation_days FROM employees WHERE name = 'John Smith';
-- Result: 10

-- T2: Update vacation days (T2 commits first)
-- T2: Updates vacation days to 8
T2: UPDATE employees SET vacation_days = 8 WHERE name = 'John Smith';

-- T2: Commit successfully
T2: COMMIT;

-- T1: Try to update vacation days (based on T1's snapshot which showed 10)
-- T1: Tries to set vacation days to 15
T1: UPDATE employees SET vacation_days = 15 WHERE name = 'John Smith';
-- ERROR: could not serialize access due to concurrent update

-- T1: Must ROLLBACK (transaction is aborted)
T1: ROLLBACK;
```

### What Happened

1. **T1 Reads**: T1 snapshots the data, seeing vacation_days = **10**
2. **T2 Reads**: T2 snapshots the same data, seeing vacation_days = **10**
3. **T2 Updates & Commits**: T2 modifies the row and commits successfully
4. **T1 Tries to Update**: T1 attempts to modify based on its snapshot
5. **Serialization Error**: PostgreSQL detects a write-write conflict and aborts T1
6. **Error Message**: `BŁĄD:  nie może serializować dostępu z powodu równoczesnej aktualizacji`

---

## Exercise 5: Record-Level Locks (SELECT FOR UPDATE / SELECT FOR SHARE)

### Scenario 1: SELECT FOR UPDATE with READ COMMITTED

By acquiring a lock on the row before reading, T1 prevents T2 from modifying it. Using READ COMMITTED isolation level, which doesn't create long-lived snapshots, allows proper coordination.

```sql
-- Initial state: John Smith has 10 vacation days

-- T1: BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
T1: BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- T1: Read with exclusive lock (SELECT FOR UPDATE)
-- This locks the row for John Smith, preventing others from updating it
T1: SELECT vacation_days FROM employees
    WHERE name = 'John Smith' FOR UPDATE;
-- Result: 10
-- Lock acquired: T1 now holds exclusive lock on this row

-- T1: Plans to give John 5 more vacation days

-- T2: BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
T2: BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- T2: Try to read with exclusive lock (SELECT FOR UPDATE)
-- This will BLOCK because T1 holds the lock
T2: SELECT vacation_days FROM employees
    WHERE name = 'John Smith' FOR UPDATE;
-- WAITING FOR LOCK...

-- Back to T1: Read the same row again
T1: SELECT vacation_days FROM employees WHERE name = 'John Smith';
-- Result: 10 (still locked by T1)

-- T1: Update vacation days
T1: UPDATE employees SET vacation_days = 15 WHERE name = 'John Smith';
-- Success: Update proceeds because T1 has the lock

-- T1: Commit and release the lock
T1: COMMIT;
-- Lock is released

-- T2: SELECT FOR UPDATE now proceeds (unblocked)
T2: SELECT vacation_days FROM employees
    WHERE name = 'John Smith' FOR UPDATE;
-- Result: 15 (sees T1's committed update)

-- T2: Update
T2: UPDATE employees SET vacation_days = 12 WHERE name = 'John Smith';

-- T2: Commit
T2: COMMIT;
```

### What Happened in Scenario 1

1. **T1 Locks**: T1 acquires exclusive lock with `SELECT FOR UPDATE`
2. **T1 First Read**: T1 reads vacation_days = **10**
3. **T2 Blocks**: T2's `SELECT FOR UPDATE` blocks waiting for the lock
4. **T1 Updates**: T1 successfully updates to **15**
5. **T1 Commits**: T1 commits and releases the lock
6. **T2 Unblocks**: T2 now reads the new value (15) and proceeds
7. **T2 Updates**: T2 updates to **12** based on the current value
8. **No Serialization Error**: The lock + READ COMMITTED prevents the conflict entirely

### Scenario 2: SELECT FOR SHARE Allows Multiple Readers

Multiple transactions can hold shared locks simultaneously, but `SELECT FOR UPDATE` is blocked.

```sql
-- Initial state: John Smith has 15 vacation days

-- T1: BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
T1: BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- T1: Read with shared lock (SELECT FOR SHARE)
T1: SELECT vacation_days FROM employees
    WHERE name = 'John Smith' FOR SHARE;
-- Result: 15
-- Lock acquired: T1 holds shared lock

-- T3: BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
T3: BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- T3: Try to read with shared lock (SELECT FOR SHARE)
-- This succeeds because multiple shared locks are compatible
T3: SELECT vacation_days FROM employees
    WHERE name = 'John Smith' FOR SHARE;
-- Result: 15
-- Lock acquired: T3 also holds shared lock (coexists with T1)

-- T2: BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
T2: BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- T2: Try to update the locked row (implicitly uses SELECT FOR UPDATE)
T2: UPDATE employees SET vacation_days = 10 WHERE name = 'John Smith';
-- WAITING FOR LOCK...
-- (Cannot proceed because T1 and T3 hold shared locks)

-- T1: Commit and release shared lock
T1: COMMIT;

-- T3: Commit and release shared lock
T3: COMMIT;

-- T2: UPDATE now proceeds (all shared locks released)
T2: UPDATE employees SET vacation_days = 10 WHERE name = 'John Smith';
-- Success: 1 row updated

-- T2: Commit
T2: COMMIT;
```

### What Happened in Scenario 2

1. **T1 Acquires Shared Lock**: `SELECT FOR SHARE` locks the row
2. **T3 Acquires Shared Lock**: Another `SELECT FOR SHARE` succeeds (compatible locks)
3. **T2 Blocks**: UPDATE (which needs exclusive lock) blocks
4. **T1 & T3 Commit**: Release their shared locks
5. **T2 Proceeds**: Now can acquire exclusive lock and update
6. **Coordination**: Multiple readers can coordinate while blocking writers

