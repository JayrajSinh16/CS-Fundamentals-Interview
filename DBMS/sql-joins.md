# SQL JOINs

JOINs combine data from multiple tables based on related columns. They are essential for retrieving meaningful information from normalized databases by establishing relationships between tables.

## Key Points

- **INNER JOIN**: Returns only matching records from both tables based on join condition. Most restrictive join type, filters out non-matching rows. Best performance for queries needing only related data. Most commonly used join in database applications.
- **LEFT JOIN**: Returns all records from left table + matching from right table. Non-matching right table columns show as NULL values. Useful for finding records that may or may not have related data. Preserves completeness of primary dataset.
- **RIGHT JOIN**: Returns all records from right table + matching from left table. Less commonly used than LEFT JOIN in practice. Non-matching left table columns show as NULL values. Can often be rewritten as LEFT JOIN by swapping table order.
- **FULL OUTER JOIN**: Returns all records when there's a match in either table. Combines results of both LEFT and RIGHT joins. Shows NULL values for non-matching columns from either side. Useful for comprehensive data analysis across related tables.
- **CROSS JOIN**: Cartesian product of both tables, every row combined with every row. Rarely used intentionally, often result of missing join conditions. Can produce very large result sets quickly. Useful for generating combinations or test data scenarios.
- **SELF JOIN**: Table joined with itself using aliases for comparison operations. Essential for hierarchical data like employee-manager relationships. Requires careful alias usage to distinguish between table instances. Common in organizational structures and recursive relationships.

## Example

```sql
-- Sample Tables
Employees: id, name, department_id
Departments: id, name, location

-- INNER JOIN - Only employees with departments
SELECT e.name, d.name as department, d.location
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;

-- LEFT JOIN - All employees, even without departments
SELECT e.name, d.name as department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;

-- RIGHT JOIN - All departments, even without employees
SELECT e.name, d.name as department
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;

-- SELF JOIN - Employee and their manager
SELECT e1.name as employee, e2.name as manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.id;
```

**Explanation**: Different JOINs serve different purposes - INNER for strict relationships, LEFT/RIGHT for optional relationships, and SELF for hierarchical data.

## Visual Representation

```
Table A (Employees)     Table B (Departments)
+----+-------+         +----+-----------+
| ID | Name  |         | ID | DeptName  |
+----+-------+         +----+-----------+
| 1  | John  |         | 1  | IT        |
| 2  | Mary  |         | 2  | HR        |
| 3  | Bob   |         | 3  | Finance   |
+----+-------+         +----+-----------+

INNER JOIN: Only matching records (John-IT, Mary-HR)
LEFT JOIN: All employees + matching departments
RIGHT JOIN: All departments + matching employees
FULL OUTER: All records from both tables
```

## Common Interview Questions

### Q1: What's the difference between INNER JOIN and LEFT JOIN?
**Answer**: 
- **INNER JOIN**: Returns only records that have matching values in both tables
- **LEFT JOIN**: Returns all records from the left table and matching records from right table, NULL for non-matching right table values

### Q2: When would you use a SELF JOIN?
**Answer**: SELF JOIN is used when data in a table has hierarchical relationships:
- Employee-Manager relationships
- Category-Subcategory structures  
- Friend relationships in social networks
- Organizational hierarchies

### Q3: What's the performance difference between different JOINs?
**Answer**:
- **INNER JOIN**: Usually fastest as it filters data
- **LEFT/RIGHT JOIN**: Slower due to NULL handling
- **FULL OUTER JOIN**: Slowest, combines LEFT and RIGHT
- **CROSS JOIN**: Potentially very slow, creates cartesian product

### Q4: How do you join more than two tables?
**Answer**:
```sql
SELECT o.order_id, c.customer_name, p.product_name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id;
```
Chain multiple JOINs, ensuring proper relationships between tables.

### Q5: What happens if you don't specify JOIN condition (ON clause)?
**Answer**: Without ON clause, you get a CROSS JOIN (Cartesian product) where every row from first table is combined with every row from second table. This usually produces unwanted results and poor performance.

---
[← Back to Main Guide](./README.md)
