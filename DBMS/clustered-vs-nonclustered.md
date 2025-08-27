# Clustered vs Non-clustered Index

The fundamental difference lies in how they store data: clustered indexes determine the physical storage order of data, while non-clustered indexes create separate structures pointing to data locations.

## Key Points

- **Clustered Index**: Determines physical row storage order on disk, one per table maximum. Data pages are stored in order of clustered index key values. Leaf pages contain actual table data rather than pointers. Provides fastest access for range queries and sequential scans.
- **Non-clustered Index**: Separate structure with pointers to data rows, multiple allowed per table. Creates additional storage overhead but enables multiple access paths. Leaf pages contain row locators pointing to actual data. Better for point lookups and searches on non-clustered columns.
- **Clustered**: Faster range queries due to sequential I/O, slower inserts/updates due to potential page splits. Physical data organization matches index order for optimal performance. INSERT operations may require data movement to maintain sort order.
- **Non-clustered**: Faster point lookups with minimal impact on data modification operations. INSERT/UPDATE/DELETE operations don't affect physical data organization. May require key lookups to retrieve columns not included in index.
- **Leaf pages**: Clustered contains actual data rows, non-clustered contains row pointers or key values. This fundamental difference affects storage requirements and access patterns. Clustered eliminates additional lookup step for covered queries.
- **Primary key**: Usually becomes clustered index by default in most database systems. Can be changed to non-clustered if different clustering strategy needed. Choice affects overall table performance characteristics and storage layout.

## Example

```sql
-- Table creation with clustered index
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,        -- Clustered index (default)
    email VARCHAR(100),
    last_name VARCHAR(50),
    first_name VARCHAR(50),
    hire_date DATE,
    salary DECIMAL(10,2)
);

-- Non-clustered indexes
CREATE INDEX idx_email ON employees(email);           -- Non-clustered
CREATE INDEX idx_name ON employees(last_name, first_name); -- Non-clustered
CREATE INDEX idx_hire_date ON employees(hire_date);   -- Non-clustered

-- Query examples showing performance differences

-- Clustered index: Fast range scan (sequential I/O)
SELECT * FROM employees 
WHERE employee_id BETWEEN 1000 AND 2000;

-- Non-clustered index: Fast point lookup
SELECT * FROM employees 
WHERE email = 'john.doe@company.com';

-- Range query on non-clustered: May require key lookups
SELECT employee_id, first_name, last_name 
FROM employees 
WHERE hire_date BETWEEN '2020-01-01' AND '2020-12-31';
```

**Explanation**: Clustered index makes range queries on employee_id very fast due to sequential storage, while non-clustered indexes enable fast searches on other columns.

## Visual Representation

```
CLUSTERED INDEX (Physical Storage Order):
Page 1: [101,John,Smith] [102,Mary,Jones] [103,Bob,Wilson]
Page 2: [104,Lisa,Brown] [105,Tom,Davis] [106,Amy,Miller]
↑ Data stored in employee_id order

NON-CLUSTERED INDEX (Separate Structure):
Index on last_name:
Brown → Points to Page 2, Row 1
Davis → Points to Page 2, Row 2  
Jones → Points to Page 1, Row 2
Miller → Points to Page 2, Row 3
Smith → Points to Page 1, Row 1
Wilson → Points to Page 1, Row 3
```

## Common Interview Questions

### Q1: Can a table have multiple clustered indexes?
**Answer**: No, a table can have only one clustered index because it determines the physical storage order of data. You can't store the same data in multiple physical orders simultaneously. However, you can have multiple non-clustered indexes.

### Q2: What happens when you insert data into a table with clustered index?
**Answer**: The database finds the correct physical location based on the clustered index key and inserts the row there. This may require:
- **Page splits**: If the page is full, it's split into two pages
- **Data movement**: Existing rows may be moved to maintain sort order
- **Performance impact**: Inserts can be slower than non-clustered indexes

### Q3: When would you choose a non-clustered index over clustered?
**Answer**: Choose non-clustered when:
- Table already has a clustered index
- Need multiple indexes on different columns
- Frequent point lookups on specific columns
- Want to avoid data movement during inserts
- Table has frequent inserts and clustered index would cause page splits

### Q4: What's a key lookup and when does it occur?
**Answer**: Key lookup occurs when a non-clustered index doesn't contain all required columns:
```sql
-- Non-clustered index on last_name only
CREATE INDEX idx_lastname ON employees(last_name);

-- This query requires key lookup to get first_name and salary
SELECT first_name, salary FROM employees WHERE last_name = 'Smith';
```
Index finds matching rows, then looks up additional columns from clustered index.

### Q5: How do clustered and non-clustered indexes affect query performance differently?
**Answer**:
- **Clustered**: 
  - Range queries: Excellent (sequential I/O)
  - Point lookups: Good
  - Inserts: Can be slow (page splits)
- **Non-clustered**:
  - Point lookups: Excellent
  - Range queries: Good (may need key lookups)
  - Inserts: Minimal impact on existing data

---
[← Back to Main Guide](./README.md)
