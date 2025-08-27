# Aggregate Functions

Aggregate functions perform calculations on sets of rows and return single values. They are essential for data analysis and summarization in SQL queries.

## Key Points

- **COUNT()**: Counts number of rows or non-NULL values in specified column. COUNT(*) includes all rows including those with NULL values. COUNT(column) excludes NULL values from count. Essential for data validation and reporting statistics.
- **SUM()**: Calculates total of numeric values, ignoring NULL values automatically. Only works with numeric data types (INT, DECIMAL, FLOAT). Returns NULL if all values are NULL. Commonly used for financial calculations and totals.
- **AVG()**: Calculates average of numeric values, excluding NULL values from calculation. Divides sum by count of non-NULL values only. Can produce different results than manual SUM/COUNT when NULLs present. Important for statistical analysis and performance metrics.
- **MIN()/MAX()**: Finds minimum/maximum values in column, works with numeric, date, and string data. Ignores NULL values in comparison operations. Can be used with non-numeric data for alphabetical or chronological ordering. Useful for range analysis and data validation.
- **GROUP_CONCAT()**: Concatenates values from multiple rows into single string (MySQL specific). Useful for creating comma-separated lists from grouped data. Other databases have similar functions like STRING_AGG (PostgreSQL). Helps in reporting and data presentation.
- **NULL values are ignored** by most aggregate functions except COUNT(*). This behavior can affect results significantly in datasets with missing data. Understanding NULL handling is crucial for accurate calculations and avoiding unexpected results.

## Example

```sql
-- Sample: Employee table
CREATE TABLE employees (
    id INT, name VARCHAR(50), department VARCHAR(50),
    salary DECIMAL(10,2), hire_date DATE
);

-- Basic aggregate functions
SELECT 
    COUNT(*) as total_employees,
    COUNT(salary) as employees_with_salary,
    SUM(salary) as total_payroll,
    AVG(salary) as average_salary,
    MIN(salary) as lowest_salary,
    MAX(salary) as highest_salary
FROM employees;

-- Aggregate with GROUP BY
SELECT 
    department,
    COUNT(*) as employee_count,
    AVG(salary) as avg_dept_salary,
    MIN(hire_date) as first_hire,
    MAX(hire_date) as latest_hire
FROM employees
GROUP BY department
ORDER BY avg_dept_salary DESC;

-- Advanced usage with CASE
SELECT 
    department,
    COUNT(*) as total,
    COUNT(CASE WHEN salary > 50000 THEN 1 END) as high_earners,
    AVG(CASE WHEN hire_date > '2020-01-01' THEN salary END) as avg_new_hire_salary
FROM employees
GROUP BY department;
```

**Explanation**: These queries demonstrate counting employees, calculating payroll statistics, and analyzing departmental metrics with conditional aggregations.

## Common Interview Questions

### Q1: What's the difference between COUNT(*) and COUNT(column_name)?
**Answer**:
- **COUNT(*)**: Counts all rows including those with NULL values
- **COUNT(column_name)**: Counts only rows where column is NOT NULL
```sql
-- If table has 100 rows but 10 have NULL salary
SELECT COUNT(*) FROM employees;        -- Returns 100
SELECT COUNT(salary) FROM employees;   -- Returns 90
```

### Q2: How do aggregate functions handle NULL values?
**Answer**: Most aggregate functions ignore NULL values:
- **SUM(salary)**: Ignores NULL salaries, sums only valid numbers
- **AVG(salary)**: Calculates average of non-NULL values only
- **COUNT(*)**: Counts all rows including NULLs
- **COUNT(column)**: Counts only non-NULL values

### Q3: Can you use aggregate functions in WHERE clause?
**Answer**: No, you cannot use aggregate functions in WHERE clause because WHERE filters rows before grouping. Use HAVING instead:
```sql
-- Invalid
SELECT department FROM employees WHERE COUNT(*) > 5; -- Error
-- Valid
SELECT department FROM employees GROUP BY department HAVING COUNT(*) > 5;
```

### Q4: What's the difference between SUM and COUNT for calculating totals?
**Answer**:
- **SUM()**: Adds up numeric values (total amount)
- **COUNT()**: Counts number of occurrences (total transactions)
```sql
-- Different purposes
SELECT SUM(order_amount) as total_revenue FROM orders;    -- Total money
SELECT COUNT(*) as total_orders FROM orders;             -- Number of orders
```

### Q5: How do you find the second highest salary using aggregate functions?
**Answer**: Multiple approaches:
```sql
-- Method 1: Subquery with MAX
SELECT MAX(salary) as second_highest
FROM employees 
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Method 2: LIMIT with ORDER BY
SELECT salary as second_highest
FROM employees 
ORDER BY salary DESC 
LIMIT 1 OFFSET 1;
```

---
[← Back to Main Guide](./README.md)
