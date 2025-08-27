# Subqueries

Subqueries are nested SQL queries where one query is embedded within another. They enable complex data retrieval by breaking down problems into smaller, manageable parts.

## Key Points

- **Subquery**: Query nested inside another query (also called inner query). Executes first to provide data for outer query evaluation. Can return single value, single row, or multiple rows/columns. Enables complex data retrieval by breaking problems into smaller parts.
- **Outer Query**: Main query that contains the subquery and uses its results. Depends on subquery results for proper execution. Can use subquery results in WHERE, SELECT, FROM, or HAVING clauses. Controls overall query logic and final result format.
- **Correlated**: Inner query references outer query columns, executes once per outer row. Performance can be slower due to repeated execution. Creates dynamic filtering based on current outer row values. Common for row-by-row comparisons and existence checks.
- **Non-correlated**: Inner query independent of outer query, executes once only. Better performance as subquery runs only once before outer query. Results are static and same for all outer query rows. Easier to optimize and understand for database engines.
- **Types**: Scalar (single value), Row (single row), Table (multiple rows/columns). Scalar subqueries can be used anywhere single value expected. Table subqueries useful in FROM clause as derived tables. Row subqueries enable multi-column comparisons.
- **Placement**: SELECT, FROM, WHERE, HAVING clauses all support subqueries. Each placement serves different purposes and has different restrictions. FROM clause subqueries create temporary result sets. WHERE clause subqueries provide dynamic filtering conditions.

## Example

```sql
-- Sample tables: employees, departments
-- Non-correlated subquery - find employees earning above average
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Correlated subquery - employees earning above their department average
SELECT e1.name, e1.salary, e1.department
FROM employees e1
WHERE salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department = e1.department
);

-- Subquery in FROM clause (derived table)
SELECT dept_stats.department, dept_stats.avg_salary
FROM (
    SELECT department, AVG(salary) as avg_salary
    FROM employees
    GROUP BY department
) as dept_stats
WHERE dept_stats.avg_salary > 50000;

-- EXISTS subquery - departments with employees
SELECT d.name
FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e WHERE e.department_id = d.id
);
```

**Explanation**: These demonstrate finding above-average earners, department-relative comparisons, derived table analysis, and existence checking.

## Common Interview Questions

### Q1: What's the difference between correlated and non-correlated subqueries?
**Answer**:
- **Non-correlated**: Inner query executes once, independent of outer query
- **Correlated**: Inner query executes for each row of outer query, references outer query columns

```sql
-- Non-correlated (executes once)
SELECT name FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);

-- Correlated (executes for each employee)
SELECT name FROM employees e1 
WHERE salary > (SELECT AVG(salary) FROM employees e2 WHERE e2.dept = e1.dept);
```

### Q2: When would you use EXISTS vs IN with subqueries?
**Answer**:
- **EXISTS**: Better for checking existence, short-circuits on first match, handles NULLs well
- **IN**: Better for exact value matching, but problematic with NULLs

```sql
-- EXISTS - checks if any matching record exists
SELECT * FROM customers c WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);

-- IN - checks for exact value matches
SELECT * FROM employees WHERE department_id IN (SELECT id FROM departments WHERE location = 'NYC');
```

### Q3: What are the performance implications of subqueries vs JOINs?
**Answer**:
- **JOINs**: Generally faster, optimized by query planner, can use indexes effectively
- **Subqueries**: Can be slower, especially correlated ones that execute repeatedly
- **Modern databases**: Often optimize subqueries to JOINs automatically
Choose JOINs for better performance, subqueries for better readability when appropriate.

### Q4: Can you use subqueries in UPDATE and DELETE statements?
**Answer**: Yes, subqueries can be used in UPDATE and DELETE:
```sql
-- UPDATE with subquery
UPDATE employees 
SET salary = salary * 1.1 
WHERE department_id IN (SELECT id FROM departments WHERE name = 'Engineering');

-- DELETE with subquery
DELETE FROM employees 
WHERE salary < (SELECT AVG(salary) FROM employees);
```

### Q5: What's a scalar subquery and when do you use it?
**Answer**: Scalar subquery returns exactly one value (single row, single column). Used when you need a single value for comparison or calculation:
```sql
-- Scalar subquery returning single value
SELECT name, salary, 
       salary - (SELECT AVG(salary) FROM employees) as salary_diff
FROM employees;
```
If scalar subquery returns multiple rows or no rows, it causes an error.

---
[← Back to Main Guide](./README.md)
