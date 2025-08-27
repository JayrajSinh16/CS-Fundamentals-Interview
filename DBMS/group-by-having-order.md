# GROUP BY, HAVING, ORDER BY

These SQL clauses organize and filter query results. GROUP BY groups rows with same values, HAVING filters groups, and ORDER BY sorts the final result set.

## Key Points

- **GROUP BY**: Groups rows sharing common values for aggregate functions like COUNT, SUM, AVG. Creates separate groups for each unique combination of grouping columns. Essential for data summarization and analysis. All non-aggregate columns in SELECT must appear in GROUP BY.
- **HAVING**: Filters groups after GROUP BY operation, like WHERE for groups. Can use aggregate functions in conditions unlike WHERE clause. Applied after grouping is complete but before ORDER BY. Essential for filtering summarized data based on aggregate calculations.
- **ORDER BY**: Sorts final result set in ascending (ASC) or descending (DESC) order. Can sort by multiple columns with different sort directions. Applied last in query execution after all filtering and grouping. Can sort by column names, positions, or expressions.
- **Execution Order**: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT. Understanding this order helps write correct queries and debug issues. WHERE filters before grouping, HAVING filters after grouping. SELECT determines final columns shown.
- **Must include non-aggregate columns in GROUP BY when using aggregate functions**. This prevents ambiguous results when grouping data. Database engines enforce this rule to ensure consistent results. Violating this rule causes syntax errors in most SQL databases.

## Example

```sql
-- Sample: Sales table with salesperson, region, amount
CREATE TABLE sales (
    id INT, salesperson VARCHAR(50), region VARCHAR(50), 
    amount DECIMAL(10,2), sale_date DATE
);

-- GROUP BY with aggregate functions
SELECT region, COUNT(*) as total_sales, SUM(amount) as total_revenue
FROM sales
WHERE sale_date >= '2024-01-01'
GROUP BY region
ORDER BY total_revenue DESC;

-- HAVING to filter groups
SELECT salesperson, AVG(amount) as avg_sale
FROM sales
GROUP BY salesperson
HAVING AVG(amount) > 5000
ORDER BY avg_sale DESC;

-- Multiple grouping columns
SELECT region, salesperson, COUNT(*) as sales_count, MAX(amount) as highest_sale
FROM sales
WHERE amount > 1000
GROUP BY region, salesperson
HAVING COUNT(*) >= 3
ORDER BY region, sales_count DESC;
```

**Explanation**: These queries demonstrate grouping data by region/salesperson, filtering high-performing groups, and sorting results for analysis.

## Common Interview Questions

### Q1: What's the difference between WHERE and HAVING?
**Answer**:
- **WHERE**: Filters individual rows before grouping, cannot use aggregate functions
- **HAVING**: Filters groups after GROUP BY, can use aggregate functions
```sql
-- WHERE filters rows first
SELECT region, COUNT(*) FROM sales WHERE amount > 1000 GROUP BY region;
-- HAVING filters groups after grouping
SELECT region, COUNT(*) FROM sales GROUP BY region HAVING COUNT(*) > 10;
```

### Q2: Can you use ORDER BY with columns not in SELECT?
**Answer**: Yes, but with restrictions:
- **Without GROUP BY**: You can order by any column from the table
- **With GROUP BY**: You can only order by columns in SELECT list or aggregate functions
```sql
-- Valid: ordering by non-selected column
SELECT name FROM employees ORDER BY salary DESC;
-- Invalid with GROUP BY: can't order by salary if not in SELECT
SELECT department FROM employees GROUP BY department ORDER BY salary; -- Error
```

### Q3: What happens if you use SELECT without GROUP BY but with aggregate functions?
**Answer**: The entire result set is treated as one group. You cannot mix aggregate and non-aggregate columns without GROUP BY:
```sql
-- Valid: all aggregate functions
SELECT COUNT(*), AVG(salary) FROM employees;
-- Invalid: mixing aggregate with non-aggregate
SELECT name, COUNT(*) FROM employees; -- Error
```

### Q4: Can you use multiple columns in GROUP BY?
**Answer**: Yes, GROUP BY can use multiple columns. Rows are grouped only when all specified columns have the same values:
```sql
SELECT department, job_title, COUNT(*), AVG(salary)
FROM employees
GROUP BY department, job_title;
```
This groups employees by both department AND job title combinations.

### Q5: What's the difference between ORDER BY column_name and ORDER BY column_position?
**Answer**:
- **Column name**: `ORDER BY salary DESC` - More readable, preferred
- **Column position**: `ORDER BY 2 DESC` - Orders by 2nd column in SELECT list
Position-based ordering is less maintainable and discouraged in modern SQL practices.

---
[← Back to Main Guide](./README.md)
