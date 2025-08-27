# ACID Properties

ACID properties ensure database transactions are processed reliably and maintain data integrity. These four fundamental properties (Atomicity, Consistency, Isolation, Durability) guarantee that database transactions are completed successfully or rolled back entirely.

## Key Points

- **Atomicity**: Transaction is all-or-nothing (complete success or complete failure). Ensures that partial transactions don't leave database in inconsistent state. Implemented using transaction logs and rollback mechanisms.
- **Consistency**: Database remains in valid state before and after transaction execution. All integrity constraints, triggers, and business rules must be satisfied. Prevents violations of database constraints and maintains data validity.
- **Isolation**: Concurrent transactions don't interfere with each other during execution. Different isolation levels control degree of interference allowed. Prevents dirty reads, phantom reads, and non-repeatable reads based on isolation level chosen.
- **Durability**: Committed changes persist even after system failure or power outage. Changes are written to persistent storage (disk) before transaction completion. Ensures data survival through crashes, restarts, and hardware failures.

## Example

```sql
-- Bank transfer example demonstrating ACID
BEGIN TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
    UPDATE accounts SET balance = balance + 100 WHERE account_id = 'A002';
COMMIT;
```

**Explanation**: This transfer either completes entirely (both updates) or fails completely (no updates). The database maintains consistency (total money unchanged), transactions are isolated from others, and once committed, changes persist.

## Common Interview Questions

### Q1: What happens if a transaction violates ACID properties?
**Answer**: The transaction is rolled back to maintain database integrity. For example, if atomicity is violated during a bank transfer, both debit and credit operations are undone to prevent money creation/destruction.

### Q2: How does isolation prevent dirty reads?
**Answer**: Isolation uses locking mechanisms or MVCC (Multi-Version Concurrency Control) to ensure transactions see consistent data. This prevents reading uncommitted changes from other transactions.

### Q3: Explain durability with an example.
**Answer**: Once a transaction commits (like confirming an online purchase), the data must survive system crashes. This is achieved through write-ahead logging and flushing data to persistent storage.

### Q4: Can you have consistency without atomicity?
**Answer**: No. Without atomicity, partial transaction execution could leave the database in an inconsistent state. Atomicity ensures consistency by guaranteeing complete transaction execution.

### Q5: What's the difference between logical and physical consistency?
**Answer**: Logical consistency refers to business rules (e.g., account balance ≥ 0), while physical consistency refers to database constraints (e.g., foreign key relationships). Both must be maintained.

---
[← Back to Main Guide](./README.md)
