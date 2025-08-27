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

## Visual Guide to SQL JOINs

### 🔄 **JOIN Types with Venn Diagrams**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            SQL JOINS VISUAL GUIDE                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  🔵 INNER JOIN                    🟡 LEFT JOIN (LEFT OUTER JOIN)           │
│                                                                             │
│      ┌─────────┐                        ┌─────────┐                        │
│     ╱           ╲                      ╱███████████╲                       │
│    ╱      A      ╲                    ╱███████████  ╲                      │
│   ╱               ╲                  ╱███████████    ╲                     │
│  │        ████████ │ ╲              │███████████ ████ │ ╲                  │
│  │     ████████████ │  ╲             │███████████████│  ╲                 │
│  │  ████████████████│   │ B          │███████████████│   │ B              │
│  │     ████████████ │  ╱             │███████████████│  ╱                 │
│  │        ████████ │ ╱              │███████████ ████ │ ╱                  │
│   ╲               ╱                  ╲███████████    ╱                     │
│    ╲      A      ╱                    ╲███████████  ╱                      │
│     ╲           ╱                      ╲███████████╱                       │
│      └─────────┘                        └─────────┘                        │
│                                                                             │
│  Returns: Only matching records          Returns: All A + matching B       │
│                                                                             │
│                                                                             │
│  🟠 RIGHT JOIN (RIGHT OUTER JOIN)    🟢 FULL OUTER JOIN                    │
│                                                                             │
│      ┌─────────┐                        ┌─────────┐                        │
│     ╱           ╲                      ╱███████████╲                       │
│    ╱      A      ╲                    ╱███████████  ╲                      │
│   ╱               ╲                  ╱███████████    ╲                     │
│  │        ████    │ ╲              │███████████ ████ │ ╲                  │
│  │     ████████    │  ╲             │███████████████ │██╲                 │
│  │  ████████████   │   │ B          │███████████████ │███│ B              │
│  │     ████████    │  ╱             │███████████████ │██╱                 │
│  │        ████    │ ╱              │███████████ ████ │ ╱                  │
│   ╲               ╱                  ╲███████████    ╱                     │
│    ╲      A      ╱                    ╲███████████  ╱                      │
│     ╲           ╱                      ╲███████████╱                       │
│      └─────────┘                        └─────────┘                        │
│                                                                             │
│  Returns: All B + matching A            Returns: All A + All B             │
│                                                                             │
│                                                                             │
│  🔴 CROSS JOIN                       🟣 SELF JOIN                          │
│                                                                             │
│      ┌─────────┐                        ┌─────────┐                        │
│     ╱███████████╲                      ╱███████████╲                       │
│    ╱█████████████╲                    ╱█████████████╲                      │
│   ╱███████████████╲                  ╱███████████████╲                     │
│  │█████████████████│ ╲              │█████████████████│ ╲                  │
│  │█████████████████│██╲             │██ EMPLOYEE ██████│██╲                 │
│  │█████████████████│███│ B          │█████████████████│███│ EMPLOYEE        │
│  │█████████████████│██╱             │█████████████████│██╱                 │
│  │█████████████████│ ╱              │█████████████████│ ╱                  │
│   ╲███████████████╱                  ╲███████████████╱                     │
│    ╲█████████████╱                    ╲█████████████╱                      │
│     ╲███████████╱                      ╲███████████╱                       │
│      └─────────┘                        └─────────┘                        │
│          A                                                                  │
│                                                                             │
│  Returns: A × B (Cartesian Product)     Returns: Table joined with itself  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 📊 **Sample Tables for JOIN Examples**

```sql
-- EMPLOYEES Table
┌─────────┬─────────────┬─────────┬────────────┬────────────┐
│ emp_id  │   emp_name  │ dept_id │   salary   │ manager_id │
├─────────┼─────────────┼─────────┼────────────┼────────────┤
│   101   │ John Smith  │    1    │   85000    │    NULL    │
│   102   │ Sarah Wilson│    2    │   92000    │    101     │
│   103   │ Mike Johnson│    3    │   78000    │    101     │
│   104   │ Lisa Brown  │   NULL  │   65000    │    102     │ ← No Dept
│   105   │ Tom Davis   │    1    │   72000    │    101     │
│   106   │ Emma Garcia │    2    │   88000    │    102     │
└─────────┴─────────────┴─────────┴────────────┴────────────┘

-- DEPARTMENTS Table  
┌─────────┬──────────────┬─────────────┬────────────┐
│ dept_id │  dept_name   │  location   │ manager_id │
├─────────┼──────────────┼─────────────┼────────────┤
│    1    │ Engineering  │ Building A  │    101     │
│    2    │ Marketing    │ Building B  │    102     │
│    3    │ Finance      │ Building C  │    103     │
│    4    │ HR          │ Building D  │   NULL     │ ← No Manager
│    5    │ Research    │ Building E  │   NULL     │ ← No Employees
└─────────┴──────────────┴─────────────┴────────────┘

-- PROJECTS Table
┌────────────┬──────────────┬─────────┬──────────────┐
│ project_id │ project_name │ dept_id │    budget    │
├────────────┼──────────────┼─────────┼──────────────┤
│     P001   │ Web Platform │    1    │   500000     │
│     P002   │ Brand Campaign│   2    │   200000     │
│     P003   │ Budget System│   3    │   150000     │
│     P004   │ AI Research  │   5    │   800000     │
└────────────┴──────────────┴─────────┴──────────────┘
```

### 💫 **Detailed JOIN Examples with Results**

#### **🔵 INNER JOIN - Only Matching Records**
```sql
-- Find employees with their department information
SELECT 
    e.emp_name, 
    e.salary,
    d.dept_name, 
    d.location
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
```

**Result:**
```
┌─────────────┬─────────┬──────────────┬─────────────┐
│  emp_name   │ salary  │  dept_name   │  location   │
├─────────────┼─────────┼──────────────┼─────────────┤
│ John Smith  │  85000  │ Engineering  │ Building A  │
│ Sarah Wilson│  92000  │ Marketing    │ Building B  │
│ Mike Johnson│  78000  │ Finance      │ Building C  │
│ Tom Davis   │  72000  │ Engineering  │ Building A  │
│ Emma Garcia │  88000  │ Marketing    │ Building B  │
└─────────────┴─────────┴──────────────┴─────────────┘
Note: Lisa Brown (no dept) and HR/Research depts (no employees) excluded
```

#### **🟡 LEFT JOIN - All Records from Left Table**
```sql
-- Find all employees and their department info (even if no department)
SELECT 
    e.emp_name, 
    e.salary,
    COALESCE(d.dept_name, 'No Department') as dept_name,
    COALESCE(d.location, 'N/A') as location
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
ORDER BY e.emp_name;
```

**Result:**
```
┌─────────────┬─────────┬──────────────┬─────────────┐
│  emp_name   │ salary  │  dept_name   │  location   │
├─────────────┼─────────┼──────────────┼─────────────┤
│ Emma Garcia │  88000  │ Marketing    │ Building B  │
│ John Smith  │  85000  │ Engineering  │ Building A  │
│ Lisa Brown  │  65000  │ No Department│     N/A     │ ← NULL dept
│ Mike Johnson│  78000  │ Finance      │ Building C  │
│ Sarah Wilson│  92000  │ Marketing    │ Building B  │
│ Tom Davis   │  72000  │ Engineering  │ Building A  │
└─────────────┴─────────┴──────────────┴─────────────┘
Note: All employees included, even Lisa Brown without department
```

#### **🟠 RIGHT JOIN - All Records from Right Table**
```sql
-- Find all departments and their employees (even if no employees)
SELECT 
    COALESCE(e.emp_name, 'No Employees') as emp_name,
    e.salary,
    d.dept_name,
    d.location
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id
ORDER BY d.dept_name, e.emp_name;
```

**Result:**
```
┌─────────────┬─────────┬──────────────┬─────────────┐
│  emp_name   │ salary  │  dept_name   │  location   │
├─────────────┼─────────┼──────────────┼─────────────┤
│ John Smith  │  85000  │ Engineering  │ Building A  │
│ Tom Davis   │  72000  │ Engineering  │ Building A  │
│ Mike Johnson│  78000  │ Finance      │ Building C  │
│ No Employees│  NULL   │ HR          │ Building D  │ ← Empty dept
│ Emma Garcia │  88000  │ Marketing    │ Building B  │
│ Sarah Wilson│  92000  │ Marketing    │ Building B  │
│ No Employees│  NULL   │ Research    │ Building E  │ ← Empty dept
└─────────────┴─────────┴──────────────┴─────────────┘
Note: All departments included, even HR and Research with no employees
```

#### **🟢 FULL OUTER JOIN - All Records from Both Tables**
```sql
-- Find all employees and departments (complete picture)
SELECT 
    COALESCE(e.emp_name, 'No Employee') as emp_name,
    e.salary,
    COALESCE(d.dept_name, 'No Department') as dept_name,
    COALESCE(d.location, 'N/A') as location
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.dept_id
ORDER BY d.dept_name, e.emp_name;
```

**Result:**
```
┌─────────────┬─────────┬──────────────┬─────────────┐
│  emp_name   │ salary  │  dept_name   │  location   │
├─────────────┼─────────┼──────────────┼─────────────┤
│ John Smith  │  85000  │ Engineering  │ Building A  │
│ Tom Davis   │  72000  │ Engineering  │ Building A  │
│ Mike Johnson│  78000  │ Finance      │ Building C  │
│ No Employee │  NULL   │ HR          │ Building D  │
│ Emma Garcia │  88000  │ Marketing    │ Building B  │
│ Sarah Wilson│  92000  │ Marketing    │ Building B  │
│ Lisa Brown  │  65000  │ No Department│     N/A     │ ← Employee without dept
│ No Employee │  NULL   │ Research    │ Building E  │
└─────────────┴─────────┴──────────────┴─────────────┘
Note: Shows complete picture - all employees AND all departments
```

#### **🔴 CROSS JOIN - Cartesian Product**
```sql
-- Generate all possible employee-project combinations (usually unintentional!)
SELECT 
    e.emp_name,
    p.project_name,
    CONCAT(e.emp_name, ' could work on ', p.project_name) as possibility
FROM employees e
CROSS JOIN projects p
WHERE e.dept_id IS NOT NULL  -- Filter to reduce output
LIMIT 10;  -- Limit to see pattern
```

**Result:**
```
┌─────────────┬──────────────┬─────────────────────────────────────────────┐
│  emp_name   │ project_name │                possibility                  │
├─────────────┼──────────────┼─────────────────────────────────────────────┤
│ John Smith  │ Web Platform │ John Smith could work on Web Platform       │
│ John Smith  │ Brand Campaign│ John Smith could work on Brand Campaign    │
│ John Smith  │ Budget System│ John Smith could work on Budget System      │
│ John Smith  │ AI Research  │ John Smith could work on AI Research        │
│ Sarah Wilson│ Web Platform │ Sarah Wilson could work on Web Platform     │
│ Sarah Wilson│ Brand Campaign│ Sarah Wilson could work on Brand Campaign  │
│ Sarah Wilson│ Budget System│ Sarah Wilson could work on Budget System    │
│ Sarah Wilson│ AI Research  │ Sarah Wilson could work on AI Research      │
│ Mike Johnson│ Web Platform │ Mike Johnson could work on Web Platform     │
│ Mike Johnson│ Brand Campaign│ Mike Johnson could work on Brand Campaign  │
└─────────────┴──────────────┴─────────────────────────────────────────────┘
Note: Every employee paired with every project (5 employees × 4 projects = 20 rows)
```

#### **🟣 SELF JOIN - Table Joined with Itself**
```sql
-- Find employees and their managers (hierarchical relationship)
SELECT 
    emp.emp_name as employee,
    emp.salary as emp_salary,
    COALESCE(mgr.emp_name, 'No Manager') as manager,
    mgr.salary as mgr_salary,
    CASE 
        WHEN mgr.salary IS NOT NULL THEN (mgr.salary - emp.salary)
        ELSE NULL 
    END as salary_difference
FROM employees emp
LEFT JOIN employees mgr ON emp.manager_id = mgr.emp_id
ORDER BY mgr.emp_name, emp.emp_name;
```

**Result:**
```
┌─────────────┬────────────┬─────────────┬────────────┬──────────────────┐
│  employee   │ emp_salary │   manager   │ mgr_salary │ salary_difference│
├─────────────┼────────────┼─────────────┼────────────┼──────────────────┤
│ John Smith  │   85000    │ No Manager  │    NULL    │      NULL        │
│ Mike Johnson│   78000    │ John Smith  │   85000    │      7000        │
│ Tom Davis   │   72000    │ John Smith  │   85000    │     13000        │
│ Emma Garcia │   88000    │ Sarah Wilson│   92000    │      4000        │
│ Lisa Brown  │   65000    │ Sarah Wilson│   92000    │     27000        │
│ Sarah Wilson│   92000    │ John Smith  │   85000    │     -7000        │ ← Manager earns less!
└─────────────┴────────────┴─────────────┴────────────┴──────────────────┘
Note: Shows employee-manager relationships and salary comparisons
```

### 🔗 **Advanced JOIN Patterns**

#### **Multiple Table JOINs**
```sql
-- Complex query joining employees, departments, and projects
SELECT 
    e.emp_name,
    d.dept_name,
    p.project_name,
    p.budget,
    CASE 
        WHEN p.budget > 300000 THEN 'High Budget'
        WHEN p.budget > 150000 THEN 'Medium Budget'
        ELSE 'Low Budget'
    END as budget_category
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
INNER JOIN projects p ON d.dept_id = p.dept_id
WHERE e.salary > 75000
ORDER BY p.budget DESC, e.emp_name;
```

#### **JOIN with Aggregations**
```sql
-- Department statistics with employee counts and average salaries
SELECT 
    d.dept_name,
    d.location,
    COUNT(e.emp_id) as employee_count,
    COALESCE(AVG(e.salary), 0) as avg_salary,
    COALESCE(SUM(e.salary), 0) as total_payroll,
    MAX(e.salary) as highest_salary,
    MIN(e.salary) as lowest_salary
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_id, d.dept_name, d.location
ORDER BY employee_count DESC, avg_salary DESC;
```

#### **Conditional JOINs**
```sql
-- Find employees who earn more than their department's average
SELECT 
    e.emp_name,
    e.salary,
    d.dept_name,
    dept_avg.avg_salary,
    (e.salary - dept_avg.avg_salary) as above_average_by
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
INNER JOIN (
    SELECT 
        dept_id, 
        AVG(salary) as avg_salary
    FROM employees 
    WHERE dept_id IS NOT NULL
    GROUP BY dept_id
) dept_avg ON e.dept_id = dept_avg.dept_id
WHERE e.salary > dept_avg.avg_salary
ORDER BY above_average_by DESC;
```

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
