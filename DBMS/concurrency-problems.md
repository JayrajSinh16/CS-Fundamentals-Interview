# Concurrency Problems

Concurrency problems occur when multiple transactions access the same data simultaneously without proper coordination. These issues can lead to data inconsistency and incorrect results in multi-user database environments.

## Key Points

- **Dirty Read**: Reading uncommitted changes from another transaction that may be rolled back. Creates dependency on potentially invalid data that could disappear. Most serious concurrency problem leading to incorrect business decisions. Prevented by READ COMMITTED isolation level and above.
- **Non-repeatable Read**: Getting different values when reading same data twice within transaction. Occurs when another transaction modifies and commits data between reads. Violates transaction isolation by showing inconsistent snapshots. Common in analytical queries requiring consistent data views.
- **Phantom Read**: New rows appearing between repeated range queries in same transaction. Happens when another transaction inserts rows matching query criteria. Affects aggregate functions and statistical calculations significantly. Different from non-repeatable read as it involves row count changes.
- **Lost Update**: One transaction's changes overwrite another's changes without detection. Classic problem when two transactions read-modify-write same data concurrently. Can result in significant business losses in financial applications. Requires explicit locking or optimistic concurrency control to prevent.
- **Isolation levels control which problems are prevented**. Each level provides different trade-offs between consistency and performance. Higher isolation prevents more problems but reduces concurrency. Applications must choose appropriate level based on business requirements.
- **Higher isolation = fewer problems but more locking**. Increased locking reduces concurrent access and may impact performance. Can lead to increased deadlock probability and longer wait times. Must balance data consistency needs with application performance requirements.

## Example

```sql
-- Dirty Read Example
-- Transaction T1:
BEGIN TRANSACTION;
UPDATE accounts SET balance = 1000 WHERE account_id = 'A001';
-- Don't commit yet

-- Transaction T2 (concurrent):
SELECT balance FROM accounts WHERE account_id = 'A001'; -- Reads 1000 (dirty)

-- T1 rolls back:
ROLLBACK; -- Balance is back to original value, T2 read wrong data

-- Non-repeatable Read Example
-- Transaction T1:
BEGIN TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 'A001'; -- Reads 500

-- Transaction T2 (concurrent):
UPDATE accounts SET balance = 1000 WHERE account_id = 'A001';
COMMIT;

-- T1 reads again:
SELECT balance FROM accounts WHERE account_id = 'A001'; -- Reads 1000 (different!)
COMMIT;

-- Phantom Read Example
-- Transaction T1:
BEGIN TRANSACTION;
SELECT COUNT(*) FROM employees WHERE salary > 50000; -- Returns 10

-- Transaction T2 (concurrent):
INSERT INTO employees VALUES (101, 'John', 60000);
COMMIT;

-- T1 reads again:
SELECT COUNT(*) FROM employees WHERE salary > 50000; -- Returns 11 (phantom!)
COMMIT;

-- Lost Update Example
-- Transaction T1:
BEGIN TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 'A001'; -- Reads 1000
-- Calculate new balance: 1000 + 100 = 1100

-- Transaction T2 (concurrent):
BEGIN TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 'A001'; -- Reads 1000
-- Calculate new balance: 1000 - 50 = 950
UPDATE accounts SET balance = 950 WHERE account_id = 'A001';
COMMIT;

-- T1 continues:
UPDATE accounts SET balance = 1100 WHERE account_id = 'A001'; -- Lost T2's update!
COMMIT;
```

**Explanation**: These examples show how concurrent transactions can interfere with each other, leading to incorrect data reads and lost changes.

## Common Interview Questions

### Q1: How do isolation levels prevent concurrency problems?
**Answer**:
- **READ UNCOMMITTED**: Allows all problems (dirty, non-repeatable, phantom reads)
- **READ COMMITTED**: Prevents dirty reads
- **REPEATABLE READ**: Prevents dirty and non-repeatable reads
- **SERIALIZABLE**: Prevents all problems (highest isolation)

Higher isolation levels use more locks, potentially reducing concurrency.

### Q2: What's the difference between dirty read and non-repeatable read?
**Answer**:
- **Dirty Read**: Reading data that was modified but not yet committed (may be rolled back)
- **Non-repeatable Read**: Reading committed data that changed between reads within same transaction

Dirty read involves uncommitted data; non-repeatable read involves committed data changes.

### Q3: How can you prevent lost update problems?
**Answer**: Several approaches:
```sql
-- 1. Optimistic locking with version numbers
UPDATE accounts SET balance = 1100, version = version + 1 
WHERE account_id = 'A001' AND version = 5;

-- 2. Pessimistic locking
SELECT balance FROM accounts WHERE account_id = 'A001' FOR UPDATE;

-- 3. Atomic operations
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'A001';
```

### Q4: When would you choose lower isolation levels despite concurrency problems?
**Answer**: Choose lower isolation when:
- **High concurrency requirements**: Many simultaneous users
- **Read-heavy workloads**: Data inconsistency risk is acceptable
- **Performance critical**: Locking overhead is too expensive
- **Analytical queries**: Approximate results are sufficient
Example: Web analytics where exact counts aren't critical.

### Q5: What's a phantom read and how is it different from non-repeatable read?
**Answer**:
- **Non-repeatable Read**: Same query returns different values for existing rows
- **Phantom Read**: Same query returns different number of rows (new/deleted rows)
```sql
-- Non-repeatable: Employee salary changes from 50K to 60K
-- Phantom: New employee with 55K salary appears in result set
```
Both affect query consistency but phantom specifically involves row count changes.

---
[← Back to Main Guide](./README.md)
