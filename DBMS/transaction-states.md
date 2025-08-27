# Transaction States

A transaction progresses through various states during its lifecycle. Understanding these states is crucial for managing database consistency and implementing proper error handling in applications.

## Key Points

- **Active**: Transaction is executing and making changes to database state. All read/write operations are performed during this state. Transaction can move to either partially committed or failed state. Locks are acquired and held during active state execution.
- **Partially Committed**: Transaction completed but changes not yet permanent on disk. All operations executed successfully but durability not yet guaranteed. Critical transition point where commit process begins. Can still fail during final commit phase due to system issues.
- **Committed**: Transaction completed successfully, changes are permanent and durable. All locks are released and changes are visible to other transactions. Recovery system guarantees durability of committed changes. Final successful state that cannot be reversed.
- **Failed**: Transaction cannot proceed due to error during execution or commit. Can occur during active state (execution error) or partially committed state (commit failure). Automatic transition to aborted state follows failure detection. System must undo any partial changes made.
- **Aborted**: Transaction rolled back, database restored to previous consistent state. All partial changes are undone using transaction log information. Resources and locks are released after rollback completion. Transaction can be restarted from beginning after abort.
- **State transitions are controlled by transaction manager**. Manager ensures ACID properties are maintained throughout state changes. Coordinates with lock manager, buffer manager, and recovery system. Handles concurrent transactions and system failures appropriately.

## Example

```sql
-- Transaction lifecycle example
BEGIN TRANSACTION;                    -- State: ACTIVE
    UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
    -- Still ACTIVE
    UPDATE accounts SET balance = balance + 100 WHERE account_id = 'A002';
    -- Transaction completed, State: PARTIALLY COMMITTED
COMMIT;                              -- State: COMMITTED

-- Failed transaction example
BEGIN TRANSACTION;                    -- State: ACTIVE
    UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
    -- Error occurs (e.g., constraint violation)
    UPDATE accounts SET balance = balance + 100 WHERE account_id = 'INVALID';
    -- State: FAILED
ROLLBACK;                            -- State: ABORTED

-- Savepoint example for partial rollback
BEGIN TRANSACTION;
    UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 101;
    SAVEPOINT sp1;
    UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 102;
    -- Error occurs, rollback to savepoint
    ROLLBACK TO sp1;                 -- Partial rollback
    -- Continue with transaction
COMMIT;
```

**Explanation**: Transactions move through states based on execution success/failure. Savepoints allow partial rollbacks within larger transactions.

## State Transition Diagram

```
    [BEGIN]
       ↓
   [ACTIVE] ────────────→ [FAILED]
       ↓                     ↓
[PARTIALLY COMMITTED]    [ABORTED]
       ↓
   [COMMITTED]

State Transitions:
1. BEGIN → ACTIVE: Transaction starts
2. ACTIVE → PARTIALLY COMMITTED: All operations complete
3. PARTIALLY COMMITTED → COMMITTED: Changes made permanent
4. ACTIVE → FAILED: Error occurs during execution
5. FAILED → ABORTED: Transaction rolled back
6. PARTIALLY COMMITTED → FAILED: Commit fails
```

## Common Interview Questions

### Q1: What's the difference between COMMIT and ROLLBACK?
**Answer**:
- **COMMIT**: Makes transaction changes permanent, moves to COMMITTED state
- **ROLLBACK**: Undoes all transaction changes, moves to ABORTED state
```sql
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
-- COMMIT makes changes permanent
-- ROLLBACK discards changes and restores original values
```

### Q2: Can a transaction move from COMMITTED state to any other state?
**Answer**: No, COMMITTED and ABORTED are final states. Once a transaction is committed, its changes are permanent and cannot be undone. To reverse committed changes, you need a new compensating transaction.

### Q3: What happens during the PARTIALLY COMMITTED state?
**Answer**: In PARTIALLY COMMITTED state:
- All transaction operations have completed successfully
- Changes are in memory/buffer but not yet written to disk
- Transaction can still fail during commit process (disk full, system crash)
- If commit succeeds → COMMITTED; if commit fails → FAILED → ABORTED

### Q4: How do savepoints affect transaction states?
**Answer**: Savepoints create intermediate checkpoints within a transaction:
```sql
BEGIN TRANSACTION;        -- ACTIVE
    UPDATE table1...;
    SAVEPOINT sp1;       -- Still ACTIVE, but checkpoint created
    UPDATE table2...;
    ROLLBACK TO sp1;     -- Partial rollback, still ACTIVE
    UPDATE table3...;
COMMIT;                  -- COMMITTED
```
Transaction remains ACTIVE throughout; savepoints only affect which changes are rolled back.

### Q5: What causes a transaction to enter FAILED state?
**Answer**: Common causes include:
- **Constraint violations**: Primary key, foreign key, check constraints
- **Deadlocks**: When transaction waits for locks held by other transactions
- **System errors**: Disk full, network failure, memory issues
- **Application errors**: Division by zero, data type mismatches
- **Timeouts**: Transaction exceeds maximum allowed time

---
[← Back to Main Guide](./README.md)
