# Locking Mechanisms

Locking mechanisms control concurrent access to database resources to maintain data integrity. They prevent conflicting operations from executing simultaneously while allowing safe concurrent access where possible.

## Key Points

- **Shared Lock (S)**: Multiple transactions can read the same data simultaneously. Prevents other transactions from modifying data while allowing concurrent reads. Compatible with other shared locks but blocks exclusive locks. Essential for maintaining read consistency in multi-user environments.
- **Exclusive Lock (X)**: Only one transaction can modify data, blocking all other access. Prevents both reading and writing by other transactions during modification. Required for INSERT, UPDATE, DELETE operations to maintain data integrity. Ensures atomic updates without interference from concurrent operations.
- **Intent Locks**: Indicate locking intentions at higher levels (table/page) before acquiring row locks. Prevent conflicting operations without checking every individual row lock. Improve performance by avoiding expensive lock conflict detection. Essential for hierarchical locking in multi-level database systems.
- **Row-level vs Page-level vs Table-level**: Granularity of locking affects concurrency and overhead. Row-level provides maximum concurrency but highest overhead per lock. Table-level has minimal overhead but blocks entire table access. Page-level offers compromise between concurrency and overhead.
- **Two-Phase Locking (2PL)**: Growing phase (acquire locks) + Shrinking phase (release locks). Ensures conflict-serializable schedule execution for concurrent transactions. Once any lock is released, no new locks can be acquired. Fundamental protocol for maintaining transaction isolation and consistency.
- **Deadlock**: Circular wait condition between transactions holding locks each other needs. Requires detection and resolution by aborting one transaction. Can be prevented through consistent resource ordering or timeout mechanisms. Common problem in high-concurrency database applications.

## Example

```sql
-- Shared Lock Example (Read operations)
-- Transaction T1:
BEGIN TRANSACTION;
SELECT * FROM accounts WHERE account_id = 'A001'; -- Acquires S lock
-- Other transactions can also read (S lock compatible with S lock)

-- Transaction T2 (concurrent):
SELECT * FROM accounts WHERE account_id = 'A001'; -- Gets S lock (allowed)
-- But UPDATE would wait for T1's S lock to be released

-- Exclusive Lock Example (Write operations)
-- Transaction T1:
BEGIN TRANSACTION;
UPDATE accounts SET balance = 1000 WHERE account_id = 'A001'; -- Acquires X lock
-- No other transaction can read or write this row until T1 commits

-- Manual Locking Examples
-- Pessimistic locking with SELECT FOR UPDATE
BEGIN TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 'A001' FOR UPDATE; -- X lock
-- Prevents other transactions from reading or modifying
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'A001';
COMMIT;

-- Shared lock for consistent reads
BEGIN TRANSACTION;
SELECT * FROM accounts WHERE account_id = 'A001' LOCK IN SHARE MODE; -- S lock
-- Other transactions can read but not modify
COMMIT;

-- Lock timeout handling
SET LOCK_TIMEOUT 5000; -- 5 seconds
BEGIN TRANSACTION;
UPDATE accounts SET balance = 1000 WHERE account_id = 'A001';
-- If lock not acquired within 5 seconds, query fails
COMMIT;
```

**Explanation**: Different lock types enable safe concurrent access - shared locks allow multiple readers, exclusive locks ensure single writers, and manual locking provides explicit control.

## Lock Compatibility Matrix

```
        |  S  |  X  | 
   -----+-----+-----+
   S    | ✓   | ✗   |
   -----+-----+-----+
   X    | ✗   | ✗   |
   -----+-----+-----+

✓ = Compatible (allowed)
✗ = Incompatible (must wait)

S lock: Shared (read) lock
X lock: Exclusive (write) lock
```

## Common Interview Questions

### Q1: What's the difference between optimistic and pessimistic locking?
**Answer**:
- **Pessimistic Locking**: Acquires locks early, prevents conflicts proactively
  ```sql
  SELECT * FROM accounts WHERE id = 1 FOR UPDATE; -- Lock immediately
  ```
- **Optimistic Locking**: Assumes no conflicts, checks for changes at commit time
  ```sql
  -- Check version number before update
  UPDATE accounts SET balance = 1000, version = version + 1 
  WHERE id = 1 AND version = 5;
  ```

### Q2: What is Two-Phase Locking (2PL) protocol?
**Answer**: 2PL ensures serializability through two phases:
- **Growing Phase**: Transaction can only acquire locks, not release them
- **Shrinking Phase**: Transaction can only release locks, not acquire new ones

Once a transaction releases any lock, it cannot acquire new locks. This prevents cascading rollbacks and ensures conflict-serializable schedules.

### Q3: How do Intent Locks work and why are they needed?
**Answer**: Intent locks are placed at higher levels to indicate locking intentions:
- **IS (Intent Shared)**: Intention to place S locks at lower level
- **IX (Intent Exclusive)**: Intention to place X locks at lower level
- **SIX (Shared + Intent Exclusive)**: S lock + intention for X locks

They prevent conflicts without checking every lower-level lock, improving performance.

### Q4: What causes deadlocks and how can they be prevented?
**Answer**: Deadlocks occur when transactions wait for each other in a cycle:
```sql
-- Transaction T1:        Transaction T2:
UPDATE accounts SET     UPDATE accounts SET 
balance = 1000          balance = 2000
WHERE id = 1;          WHERE id = 2;

UPDATE accounts SET     UPDATE accounts SET
balance = 1500          balance = 2500  
WHERE id = 2;          WHERE id = 1;    -- Deadlock!
```

**Prevention methods**:
- Access resources in consistent order
- Use timeouts
- Reduce transaction length
- Lower isolation levels

### Q5: When would you use table-level locks instead of row-level locks?
**Answer**: Use table-level locks when:
- **Bulk operations**: Loading/updating large portions of table
- **DDL operations**: ALTER TABLE, CREATE INDEX
- **Low concurrency**: Single-user or batch processing
- **Simple systems**: When row-level complexity isn't justified

Row-level locks provide better concurrency but have higher overhead.

---
[← Back to Main Guide](./README.md)
