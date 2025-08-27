# Basic SQL Queries

SQL (Structured Query Language) provides CRUD operations to interact with relational databases. These fundamental operations form the backbone of database manipulation and data retrieval.

## Key Points

- **CREATE**: Insert new records into tables using INSERT statements. Can insert single or multiple rows at once. Supports default values and auto-increment columns. Essential for populating database with initial data.
- **READ**: Retrieve data using SELECT statements with various filtering options. Most commonly used operation in databases. Supports complex conditions, sorting, and column selection. Foundation for all data analysis and reporting.
- **UPDATE**: Modify existing records using UPDATE statements with WHERE clauses. Can update single or multiple columns simultaneously. Must use WHERE clause to avoid updating all rows. Critical for maintaining current and accurate data.
- **DELETE**: Remove records from tables using DELETE statements with conditions. Can delete single or multiple rows based on criteria. Use with caution as data removal is often irreversible. Should be combined with proper backup strategies.
- **WHERE**: Filter records based on conditions using comparison and logical operators. Supports complex expressions with AND, OR, NOT operators. Can use functions and subqueries in conditions. Essential for precise data retrieval and manipulation.
- **DISTINCT**: Remove duplicate values from results to show unique records only. Useful for data analysis and reporting unique values. Can be applied to single or multiple columns. Helps identify data patterns and eliminate redundancy in results.

## Example

```sql
-- CREATE (INSERT)
INSERT INTO employees (id, name, department, salary)
VALUES (101, 'John Doe', 'Engineering', 75000);

-- READ (SELECT)
SELECT name, salary 
FROM employees 
WHERE department = 'Engineering' 
ORDER BY salary DESC;

-- UPDATE
UPDATE employees 
SET salary = 80000 
WHERE id = 101;

-- DELETE
DELETE FROM employees 
WHERE department = 'Marketing' AND salary < 50000;

-- Complex SELECT with conditions
SELECT DISTINCT department, COUNT(*) as employee_count
FROM employees 
WHERE salary BETWEEN 50000 AND 100000
GROUP BY department
HAVING COUNT(*) > 5;
```

**Explanation**: These operations demonstrate complete data lifecycle - inserting new employee, querying engineering staff by salary, updating John's salary, removing low-paid marketing staff, and analyzing department sizes.

## Common Interview Questions

### Q1: What's the difference between WHERE and HAVING clauses?
**Answer**: 
- **WHERE**: Filters individual rows before grouping, cannot use aggregate functions
- **HAVING**: Filters groups after GROUP BY, can use aggregate functions
```sql
-- WHERE filters before grouping
SELECT department, AVG(salary) FROM employees WHERE salary > 50000 GROUP BY department;
-- HAVING filters after grouping  
SELECT department, AVG(salary) FROM employees GROUP BY department HAVING AVG(salary) > 60000;
```

### Q2: Explain the order of execution in a SELECT statement.
**Answer**: 
1. **FROM** - Identify tables
2. **WHERE** - Filter rows
3. **GROUP BY** - Group rows
4. **HAVING** - Filter groups
5. **SELECT** - Choose columns
6. **ORDER BY** - Sort results
7. **LIMIT** - Limit results

### Q3: What's the difference between DELETE, DROP, and TRUNCATE?
**Answer**:
- **DELETE**: Removes specific rows, can use WHERE, slower, can be rolled back
- **TRUNCATE**: Removes all rows, faster, cannot be rolled back, resets auto-increment
- **DROP**: Removes entire table structure and data

### Q4: How do you prevent SQL injection in queries?
**Answer**: 
- Use parameterized queries/prepared statements
- Input validation and sanitization
- Escape special characters
- Use stored procedures
- Apply principle of least privilege

### Q5: What's the difference between UNION and UNION ALL?
**Answer**:
- **UNION**: Combines results and removes duplicates (slower)
- **UNION ALL**: Combines results keeping duplicates (faster)
```sql
SELECT name FROM employees WHERE department = 'IT'
UNION ALL
SELECT name FROM employees WHERE salary > 70000;
```

---
[← Back to Main Guide](./README.md)
