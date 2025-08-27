# Types of Indexes

Indexes are database objects that improve query performance by creating shortcuts to data. They work like book indexes, providing fast access paths to table data without scanning entire tables.

## Key Points

- **Primary Index**: Automatically created for primary key, enforces uniqueness and fast access. Determines physical storage order of data in clustered systems. Only one primary index per table possible. Essential for maintaining entity integrity and optimal performance.
- **Secondary Index**: Created on non-key columns for faster searches and queries. Multiple secondary indexes allowed per table for different access patterns. Points to primary key or row location rather than storing data. Critical for optimizing WHERE clause performance.
- **Unique Index**: Ensures uniqueness while providing fast lookups like primary index. Can be created on any column or combination of columns. Allows one NULL value unless specified otherwise. Useful for business keys like email addresses or SSN.
- **Composite Index**: Built on multiple columns together in specific order. Column order matters significantly for query optimization effectiveness. Most effective when queries filter on leading columns of index. Reduces need for multiple single-column indexes.
- **Partial Index**: Index on subset of rows meeting certain conditions. Smaller size improves performance and reduces storage overhead. Useful for indexing frequently queried data subsets only. Particularly effective for large tables with skewed data distribution.
- **Functional Index**: Index on expressions or function results rather than raw column values. Enables optimization of queries using functions in WHERE clauses. Requires exact function match in queries to be utilized. Useful for case-insensitive searches or calculated fields.

## Example

```sql
-- Create different types of indexes
CREATE TABLE employees (
    id INT PRIMARY KEY,           -- Automatic primary index
    email VARCHAR(100),
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE
);

-- Secondary index on frequently searched column
CREATE INDEX idx_email ON employees(email);

-- Unique index to enforce uniqueness
CREATE UNIQUE INDEX idx_unique_email ON employees(email);

-- Composite index for multi-column searches
CREATE INDEX idx_name_dept ON employees(last_name, first_name, department);

-- Partial index for specific conditions
CREATE INDEX idx_high_salary ON employees(salary) WHERE salary > 50000;

-- Functional index on expression
CREATE INDEX idx_full_name ON employees(CONCAT(first_name, ' ', last_name));

-- Index usage examples
SELECT * FROM employees WHERE email = 'john@company.com';     -- Uses idx_email
SELECT * FROM employees WHERE last_name = 'Smith';            -- Uses idx_name_dept
SELECT * FROM employees WHERE salary > 75000;                 -- May use idx_high_salary
```

**Explanation**: Different index types serve specific purposes - primary for uniqueness, secondary for searches, composite for multi-column queries, and partial for conditional filtering.

## Common Interview Questions

### Q1: What's the difference between Primary and Secondary indexes?
**Answer**:
- **Primary Index**: Automatically created for primary key, determines physical storage order, one per table
- **Secondary Index**: Manually created on other columns, points to primary key/row location, multiple allowed per table

### Q2: When should you use a composite index vs multiple single-column indexes?
**Answer**: Use composite index when:
- Queries frequently filter on multiple columns together
- Query uses WHERE clause with multiple columns in AND condition
- Need to optimize ORDER BY on multiple columns

Single-column indexes are better when columns are searched independently.

### Q3: What are the disadvantages of having too many indexes?
**Answer**:
- **Storage overhead**: Each index requires additional disk space
- **Insert/Update/Delete slowdown**: All relevant indexes must be updated
- **Maintenance cost**: Indexes need to be rebuilt/reorganized periodically
- **Memory usage**: Index pages compete for buffer pool space

### Q4: How does a partial index improve performance?
**Answer**: Partial indexes are smaller and faster because they only include rows meeting specific conditions:
```sql
-- Instead of indexing all employees
CREATE INDEX idx_all_salary ON employees(salary);

-- Index only high earners (smaller, faster)
CREATE INDEX idx_high_salary ON employees(salary) WHERE salary > 50000;
```
Queries filtering for high salaries use the smaller, more efficient partial index.

### Q5: What's a covering index and why is it beneficial?
**Answer**: Covering index includes all columns needed by a query, eliminating the need to access the actual table:
```sql
-- Query needs: last_name, first_name, salary
CREATE INDEX idx_covering ON employees(last_name, first_name, salary);

-- This query can be satisfied entirely from the index
SELECT first_name, salary FROM employees WHERE last_name = 'Smith';
```
Benefits: Faster queries, reduced I/O, no table access needed.

---
[← Back to Main Guide](./README.md)
