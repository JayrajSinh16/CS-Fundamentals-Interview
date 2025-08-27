# Views, Stored Procedures, Triggers

Views, stored procedures, and triggers are database objects that enhance functionality beyond basic tables. They provide data abstraction, business logic encapsulation, and automatic event handling within the database.

## Key Points

- **Views**: Virtual tables based on query results, provide data abstraction and security. Don't store data physically, executed each time view is accessed. Simplify complex queries and hide sensitive columns from users. Can be updatable or read-only depending on underlying query complexity.
- **Stored Procedures**: Pre-compiled SQL code blocks with parameters for business logic encapsulation. Executed on database server reducing network traffic and improving performance. Support input/output parameters, variables, and control flow statements. Enable code reuse and centralized business rule enforcement.
- **Triggers**: Automatic execution based on database events (INSERT/UPDATE/DELETE) without explicit calls. Fire automatically when specified events occur on target tables. Cannot be directly invoked by applications, only by database events. Essential for maintaining data integrity and implementing audit trails.
- **Benefits**: Security through controlled data access, performance through pre-compilation, code reusability across applications. Views hide complex JOINs and sensitive data from end users. Stored procedures reduce parsing overhead and enable complex business logic. Triggers ensure automatic enforcement of business rules and constraints.
- **Materialized Views**: Physically stored view results for performance, require periodic refresh. Trade storage space for query performance improvements. Useful for expensive aggregations and reporting queries. Must balance refresh frequency with data staleness tolerance.
- **Trigger Types**: BEFORE (validate/modify before operation), AFTER (log/update after operation), INSTEAD OF (replace operation). Each type serves different purposes in data processing workflow. BEFORE triggers can prevent operations, AFTER triggers can audit changes.

## Example

```sql
-- VIEWS Example
-- Create a view to hide sensitive data and simplify queries
CREATE VIEW employee_summary AS
SELECT 
    employee_id,
    CONCAT(first_name, ' ', last_name) as full_name,
    department,
    CASE 
        WHEN salary > 80000 THEN 'High'
        WHEN salary > 50000 THEN 'Medium'
        ELSE 'Low'
    END as salary_range
FROM employees
WHERE status = 'Active';

-- Use view like a regular table
SELECT * FROM employee_summary WHERE department = 'Engineering';

-- STORED PROCEDURES Example
-- Procedure for transferring money between accounts
DELIMITER //
CREATE PROCEDURE transfer_money(
    IN from_account INT,
    IN to_account INT,
    IN amount DECIMAL(10,2)
)
BEGIN
    DECLARE insufficient_funds CONDITION FOR SQLSTATE '45000';
    DECLARE current_balance DECIMAL(10,2);
    
    -- Start transaction
    START TRANSACTION;
    
    -- Check balance
    SELECT balance INTO current_balance 
    FROM accounts 
    WHERE account_id = from_account;
    
    IF current_balance < amount THEN
        SIGNAL insufficient_funds SET MESSAGE_TEXT = 'Insufficient funds';
    END IF;
    
    -- Perform transfer
    UPDATE accounts SET balance = balance - amount WHERE account_id = from_account;
    UPDATE accounts SET balance = balance + amount WHERE account_id = to_account;
    
    -- Log transaction
    INSERT INTO transaction_log (from_account, to_account, amount, timestamp)
    VALUES (from_account, to_account, amount, NOW());
    
    COMMIT;
END //
DELIMITER ;

-- Call stored procedure
CALL transfer_money(101, 102, 250.00);

-- TRIGGERS Example
-- Automatically update last_modified timestamp
CREATE TRIGGER update_employee_timestamp
    BEFORE UPDATE ON employees
    FOR EACH ROW
BEGIN
    SET NEW.last_modified = NOW();
END;

-- Audit trigger to log all changes
CREATE TRIGGER employee_audit
    AFTER UPDATE ON employees
    FOR EACH ROW
BEGIN
    INSERT INTO employee_audit_log (
        employee_id, 
        old_salary, 
        new_salary, 
        changed_by, 
        change_date
    )
    VALUES (
        NEW.employee_id,
        OLD.salary,
        NEW.salary,
        USER(),
        NOW()
    );
END;

-- Trigger to enforce business rules
CREATE TRIGGER check_salary_increase
    BEFORE UPDATE ON employees
    FOR EACH ROW
BEGIN
    IF NEW.salary > OLD.salary * 1.5 THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Salary increase cannot exceed 50%';
    END IF;
END;
```

**Explanation**: Views simplify data access, stored procedures encapsulate business logic, and triggers automatically enforce rules and maintain audit trails.

## Common Interview Questions

### Q1: What's the difference between a view and a materialized view?
**Answer**:
- **View**: Virtual table, query executed each time view is accessed
  - Pros: Always current data, no storage overhead
  - Cons: Performance overhead for complex queries
  
- **Materialized View**: Physical storage of query results
  - Pros: Fast query performance, pre-computed results
  - Cons: Storage overhead, data may be stale, requires refresh

```sql
-- Regular view (virtual)
CREATE VIEW sales_summary AS SELECT region, SUM(amount) FROM sales GROUP BY region;

-- Materialized view (physical storage)
CREATE MATERIALIZED VIEW sales_summary_mat AS SELECT region, SUM(amount) FROM sales GROUP BY region;
REFRESH MATERIALIZED VIEW sales_summary_mat; -- Update when needed
```

### Q2: When should you use stored procedures vs application code?
**Answer**: Use stored procedures when:
- **Database-intensive operations**: Multiple related queries
- **Performance critical**: Reduced network traffic
- **Data integrity**: Complex business rules enforcement
- **Security**: Controlled data access without direct table access

Use application code when:
- **Flexibility**: Easier to version and deploy
- **Testing**: Better unit testing capabilities
- **Portability**: Database-independent logic
- **Maintenance**: Easier debugging and monitoring

### Q3: What are the different types of triggers and when do you use each?
**Answer**:
- **BEFORE triggers**: Validate or modify data before operation
  ```sql
  -- Validate before insert
  CREATE TRIGGER validate_email BEFORE INSERT ON users
  FOR EACH ROW
  BEGIN
      IF NEW.email NOT LIKE '%@%.%' THEN
          SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Invalid email';
      END IF;
  END;
  ```
  
- **AFTER triggers**: Log changes or update related tables
  ```sql
  -- Update inventory after sale
  CREATE TRIGGER update_inventory AFTER INSERT ON sales
  FOR EACH ROW
  BEGIN
      UPDATE inventory SET quantity = quantity - NEW.quantity 
      WHERE product_id = NEW.product_id;
  END;
  ```

### Q4: What are the performance implications of views?
**Answer**: Performance considerations:
- **Simple views**: Minimal overhead, may use indexes effectively
- **Complex views**: Expensive operations (JOINs, aggregations) run each time
- **Nested views**: Views based on other views can be very slow
- **Optimization**: Database optimizers can sometimes push down predicates

Best practices:
- Keep views simple when possible
- Consider materialized views for complex aggregations
- Test performance with realistic data volumes

### Q5: Can triggers cause performance problems?
**Answer**: Yes, triggers can impact performance because:
- **Execution overhead**: Run automatically on every DML operation
- **Cascading effects**: Triggers can fire other triggers
- **Lock duration**: Extend transaction time and lock duration
- **Hidden complexity**: Application developers may not be aware of trigger logic

Mitigation strategies:
- Keep trigger logic simple and fast
- Avoid complex queries in triggers
- Consider asynchronous processing for heavy operations
- Document trigger behavior clearly

---
[← Back to Main Guide](./README.md)
