# Deadlock Handling

Deadlock occurs when two or more transactions wait indefinitely for each other to release locks, creating a circular wait condition. Database systems must detect and resolve deadlocks to maintain system availability.

## Key Points

- **Deadlock**: Circular wait condition where transactions block each other indefinitely. Occurs when each transaction holds locks needed by others in circular fashion. Natural consequence of concurrent database access with multiple resources. Requires active detection and resolution to maintain system availability.
- **Detection**: Database monitors wait-for graphs to identify cycles between transactions. Uses algorithms to find circular dependencies in lock wait relationships. Detection can be continuous or periodic depending on system configuration. More efficient than prevention but allows deadlocks to occur temporarily.
- **Resolution**: Abort one transaction (victim selection) to break the cycle and allow others to proceed. Victim selection considers factors like transaction age, work done, and rollback cost. Aborted transaction must be restarted, potentially causing delays. Critical for maintaining system liveness in concurrent environments.
- **Prevention**: Techniques to avoid deadlocks before they occur through resource ordering or timestamps. Includes strategies like consistent resource access ordering and timeout mechanisms. May reduce concurrency but eliminates deadlock occurrence entirely. Often combined with detection for comprehensive deadlock management.
- **Wait-Die/Wound-Wait**: Timestamp-based prevention schemes using transaction age for conflict resolution. Wait-Die allows older transactions to wait, kills younger ones. Wound-Wait allows older transactions to abort younger ones preemptively. Both prevent deadlocks but may cause unnecessary transaction aborts.
- **Deadlock victim**: Transaction chosen to be rolled back based on selection criteria. Selection considers transaction priority, resource usage, and completion progress. Goal is minimizing overall system impact and recovery cost. Victim selection algorithm significantly affects system performance and fairness.

## Example

```sql
-- Classic Deadlock Scenario
-- Transaction T1:
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001'; -- Locks A001
-- T1 waits here for A002 (held by T2)
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'A002';

-- Transaction T2 (concurrent):
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 50 WHERE account_id = 'A002';  -- Locks A002
-- T2 waits here for A001 (held by T1) → DEADLOCK!
UPDATE accounts SET balance = balance + 50 WHERE account_id = 'A001';

-- Deadlock Prevention - Consistent Ordering
-- All transactions access accounts in ID order
-- Transaction T1:
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001'; -- Lock A001 first
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'A002'; -- Then A002

-- Transaction T2:
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance + 50 WHERE account_id = 'A001';  -- Lock A001 first
UPDATE accounts SET balance = balance - 50 WHERE account_id = 'A002';  -- Then A002

-- Timeout-based Deadlock Resolution
SET LOCK_TIMEOUT 30000; -- 30 seconds
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
-- If next statement can't get lock within 30 seconds, transaction fails
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'A002';
COMMIT;
```

**Explanation**: The first example shows a classic deadlock, while prevention techniques include consistent resource ordering and timeout mechanisms.

## Deadlock Detection Algorithm

```
Wait-For Graph:
T1 → T2 (T1 waits for lock held by T2)
T2 → T1 (T2 waits for lock held by T1)

Cycle detected: T1 → T2 → T1

Resolution:
1. Choose victim (usually transaction with least work done)
2. Abort victim transaction
3. Restart aborted transaction
```

## Common Interview Questions

### Q1: How does a database detect deadlocks?
**Answer**: Databases use **wait-for graphs** where:
- Nodes represent transactions
- Edges represent wait relationships (T1 → T2 means T1 waits for T2)
- Cycles in the graph indicate deadlocks

Detection runs periodically or when waits exceed thresholds. When a cycle is found, one transaction is chosen as victim and aborted.

### Q2: What factors determine deadlock victim selection?
**Answer**: Common victim selection criteria:
- **Transaction age**: Newer transactions are often chosen
- **Work done**: Transaction with least work is preferred victim
- **Lock count**: Transaction holding fewer locks
- **Priority**: User-assigned priorities
- **Rollback cost**: Cheapest transaction to rollback

Goal is to minimize the cost of deadlock resolution.

### Q3: What's the difference between Wait-Die and Wound-Wait schemes?
**Answer**:
- **Wait-Die**: Older transaction waits, younger dies
  - If T1 (older) requests lock held by T2 (younger): T1 waits
  - If T2 (younger) requests lock held by T1 (older): T2 dies
  
- **Wound-Wait**: Older transaction wounds younger, younger waits
  - If T1 (older) requests lock held by T2 (younger): T2 is wounded (aborted)
  - If T2 (younger) requests lock held by T1 (older): T2 waits

Both prevent deadlocks but with different performance characteristics.

### Q4: How can application design prevent deadlocks?
**Answer**: Application-level prevention strategies:
```sql
-- 1. Consistent Resource Ordering
-- Always access tables/rows in same order (A before B)

-- 2. Minimize Transaction Length
BEGIN TRANSACTION;
-- Do minimal work
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT; -- Release locks quickly

-- 3. Use Lower Isolation Levels
SET TRANSACTION ISOLATION LEVEL READ COMMITTED; -- Reduces lock duration

-- 4. Retry Logic
try {
    executeTransaction();
} catch (DeadlockException e) {
    wait(randomDelay);
    retry();
}
```

### Q5: What's the trade-off between deadlock prevention and detection?
**Answer**:
- **Prevention**: 
  - Pros: No deadlocks occur, predictable performance
  - Cons: May abort transactions unnecessarily, lower concurrency
  
- **Detection**: 
  - Pros: Better concurrency, aborts only when deadlock occurs
  - Cons: Detection overhead, unpredictable transaction aborts

Most commercial databases use detection with timeouts as backup prevention.

---
[← Back to Main Guide](./README.md)
