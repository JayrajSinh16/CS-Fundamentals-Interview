# Basic SQL Queries

SQL (Structured Query Language) provides CRUD operations to interact with relational databases. These fundamental operations form the backbone of database manipulation and data retrieval.

## Key Points

- **CREATE**: Insert new records into tables using INSERT statements. Can insert single or multiple rows at once. Supports default values and auto-increment columns. Essential for populating database with initial data.
- **READ**: Retrieve data using SELECT statements with various filtering options. Most commonly used operation in databases. Supports complex conditions, sorting, and column selection. Foundation for all data analysis and reporting.
- **UPDATE**: Modify existing records using UPDATE statements with WHERE clauses. Can update single or multiple columns simultaneously. Must use WHERE clause to avoid updating all rows. Critical for maintaining current and accurate data.
- **DELETE**: Remove records from tables using DELETE statements with conditions. Can delete single or multiple rows based on criteria. Use with caution as data removal is often irreversible. Should be combined with proper backup strategies.
- **WHERE**: Filter records based on conditions using comparison and logical operators. Supports complex expressions with AND, OR, NOT operators. Can use functions and subqueries in conditions. Essential for precise data retrieval and manipulation.
- **DISTINCT**: Remove duplicate values from results to show unique records only. Useful for data analysis and reporting unique values. Can be applied to single or multiple columns. Helps identify data patterns and eliminate redundancy in results.

## SQL Query Structure and Execution Flow

### 🔄 **SQL Query Execution Order (Internal Processing)**
```
┌─────────────────────────────────────────────────────────────────────────┐
│                     SQL QUERY EXECUTION FLOW                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  📝 Written Order:          🔄 Execution Order:                        │
│  ─────────────────          ──────────────────                         │
│                                                                         │
│  1️⃣ SELECT columns          1️⃣ FROM tables                             │
│  2️⃣ FROM tables             2️⃣ WHERE conditions                        │
│  3️⃣ WHERE conditions        3️⃣ GROUP BY columns                        │
│  4️⃣ GROUP BY columns        4️⃣ HAVING group_conditions                 │
│  5️⃣ HAVING group_cond       5️⃣ SELECT columns                          │
│  6️⃣ ORDER BY columns        6️⃣ ORDER BY columns                        │
│  7️⃣ LIMIT count             7️⃣ LIMIT count                             │
│                                                                         │
│  💡 Why this matters for query optimization and understanding!          │
│     • WHERE filters before grouping (faster)                           │
│     • HAVING filters after grouping (slower)                           │
│     • SELECT aliases not available in WHERE                            │
│     • Aggregate functions need GROUP BY or HAVING                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📊 **Sample Database for Examples**
```
🏢 COMPANY DATABASE SCHEMA:

┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   DEPARTMENTS   │    │    EMPLOYEES    │    │    PROJECTS     │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│ dept_id (PK)    │◄──┤ emp_id (PK)     │    │ project_id (PK) │
│ dept_name       │   │ emp_name        │    │ project_name    │
│ location        │   │ salary          │    │ budget          │
│ manager_id      │   │ dept_id (FK)    │    │ start_date      │
└─────────────────┘   │ hire_date       │    │ status          │
                      │ position        │    └─────────────────┘
                      └─────────────────┘             │
                               │                      │
                               └──────────┬───────────┘
                                         │
                               ┌─────────────────┐
                               │ EMP_PROJECTS    │
                               │ (Junction Table)│
                               ├─────────────────┤
                               │ emp_id (FK)     │
                               │ project_id (FK) │
                               │ role            │
                               │ hours_allocated │
                               └─────────────────┘

📋 SAMPLE DATA:

DEPARTMENTS:
| dept_id | dept_name    | location  | manager_id |
|---------|--------------|-----------|------------|
| 1       | Engineering  | Building A| 101        |
| 2       | Marketing    | Building B| 102        |
| 3       | Finance      | Building C| 103        |
| 4       | HR          | Building B| 104        |

EMPLOYEES:
| emp_id | emp_name     | salary | dept_id | hire_date  | position        |
|--------|--------------|--------|---------|------------|-----------------|
| 101    | John Smith   | 85000  | 1       | 2022-01-15 | Sr. Engineer    |
| 102    | Sarah Wilson | 92000  | 2       | 2021-06-10 | Marketing Dir   |
| 103    | Mike Johnson | 78000  | 3       | 2023-02-20 | Financial Analyst|
| 104    | Lisa Brown   | 65000  | 4       | 2022-08-05 | HR Specialist   |
| 105    | Tom Davis    | 72000  | 1       | 2023-01-10 | Jr. Engineer    |
| 106    | Emma Garcia  | 88000  | 2       | 2021-12-01 | Marketing Mgr   |
```

## Comprehensive CRUD Examples

### 🟢 **CREATE (INSERT) Operations**

#### **Single Row Insert**
```sql
-- Basic INSERT
INSERT INTO employees (emp_id, emp_name, salary, dept_id, hire_date, position)
VALUES (107, 'Alex Thompson', 75000, 1, '2024-01-15', 'Software Engineer');

-- INSERT with default values
INSERT INTO employees (emp_name, salary, dept_id, position)
VALUES ('Maria Rodriguez', 70000, 2, 'Marketing Specialist');
-- Note: emp_id (auto-increment) and hire_date (DEFAULT CURRENT_DATE) automatically populated
```

#### **Multiple Row Insert**
```sql
-- Insert multiple employees at once
INSERT INTO employees (emp_name, salary, dept_id, hire_date, position) VALUES
    ('David Kim', 82000, 1, '2024-02-01', 'DevOps Engineer'),
    ('Rachel Green', 76000, 3, '2024-02-15', 'Budget Analyst'),
    ('James Wilson', 79000, 4, '2024-03-01', 'HR Manager'),
    ('Nina Patel', 85000, 2, '2024-03-10', 'Brand Manager');
```

#### **INSERT with Subquery**
```sql
-- Create backup table and populate with high-performers
CREATE TABLE high_performers AS 
SELECT emp_id, emp_name, salary, dept_id 
FROM employees 
WHERE salary > 80000;

-- Insert average salary by department into summary table
INSERT INTO dept_salary_summary (dept_id, avg_salary, employee_count)
SELECT 
    dept_id, 
    AVG(salary) as avg_salary,
    COUNT(*) as employee_count
FROM employees 
GROUP BY dept_id;
```

### 🔵 **READ (SELECT) Operations**

#### **Basic SELECT with Various Conditions**
```sql
-- Simple selection with multiple conditions
SELECT emp_name, salary, position
FROM employees
WHERE salary BETWEEN 70000 AND 90000
  AND dept_id IN (1, 2)
  AND hire_date >= '2022-01-01'
ORDER BY salary DESC, emp_name ASC;

-- Pattern matching and text functions
SELECT 
    emp_name,
    UPPER(position) as position_upper,
    CONCAT(emp_name, ' - ', position) as employee_title,
    LENGTH(emp_name) as name_length
FROM employees
WHERE emp_name LIKE '%son%'  -- Names containing 'son'
   OR position LIKE 'Sr.%'   -- Positions starting with 'Sr.'
ORDER BY name_length DESC;
```

#### **Advanced Grouping and Aggregation**
```sql
-- Department-wise statistics
SELECT 
    d.dept_name,
    COUNT(e.emp_id) as total_employees,
    AVG(e.salary) as avg_salary,
    MIN(e.salary) as min_salary,
    MAX(e.salary) as max_salary,
    SUM(e.salary) as total_payroll,
    ROUND(AVG(e.salary), 2) as avg_salary_rounded
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_id, d.dept_name
HAVING COUNT(e.emp_id) > 0  -- Only departments with employees
ORDER BY avg_salary DESC;
```

#### **Complex Filtering and Calculations**
```sql
-- Employees with salary above department average
SELECT 
    e.emp_name,
    e.salary,
    d.dept_name,
    dept_avg.avg_salary as dept_average,
    (e.salary - dept_avg.avg_salary) as salary_difference,
    CASE 
        WHEN e.salary > dept_avg.avg_salary * 1.2 THEN 'High Performer'
        WHEN e.salary > dept_avg.avg_salary THEN 'Above Average'
        ELSE 'Below Average'
    END as performance_category
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id
JOIN (
    SELECT dept_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY dept_id
) dept_avg ON e.dept_id = dept_avg.dept_id
WHERE e.salary > dept_avg.avg_salary
ORDER BY salary_difference DESC;
```

### 🟡 **UPDATE Operations**

#### **Simple Updates**
```sql
-- Single employee salary update
UPDATE employees 
SET salary = 88000, position = 'Senior Software Engineer'
WHERE emp_id = 105;

-- Bulk update with conditions
UPDATE employees 
SET salary = salary * 1.05  -- 5% raise
WHERE dept_id = 1 
  AND hire_date < '2023-01-01'
  AND performance_rating >= 4;
```

#### **Advanced Updates with Subqueries**
```sql
-- Update salaries based on department averages
UPDATE employees 
SET salary = (
    SELECT AVG(salary) * 1.1  -- 10% above department average
    FROM (SELECT * FROM employees) as emp_copy  -- Avoid self-reference
    WHERE emp_copy.dept_id = employees.dept_id
)
WHERE position LIKE '%Manager%';

-- Update based on complex conditions
UPDATE employees e1
SET salary = (
    SELECT MAX(e2.salary) * 0.95  -- 95% of highest salary in department
    FROM employees e2
    WHERE e2.dept_id = e1.dept_id
    AND e2.emp_id != e1.emp_id
)
WHERE EXISTS (
    SELECT 1 FROM employees e3
    WHERE e3.dept_id = e1.dept_id
    AND e3.salary > e1.salary
    GROUP BY e3.dept_id
    HAVING COUNT(*) >= 2  -- At least 2 employees earning more
);
```

### 🔴 **DELETE Operations**

#### **Conditional Deletes**
```sql
-- Delete employees with specific criteria
DELETE FROM employees 
WHERE salary < 50000 
  AND hire_date < '2020-01-01'
  AND dept_id IN (
      SELECT dept_id FROM departments 
      WHERE location = 'Building D'
  );

-- Delete with complex conditions
DELETE e1 FROM employees e1
INNER JOIN employees e2 ON e1.dept_id = e2.dept_id
WHERE e1.salary < (
    SELECT AVG(salary) * 0.8  -- Below 80% of department average
    FROM (SELECT * FROM employees) emp_avg
    WHERE emp_avg.dept_id = e1.dept_id
);
```

### 📈 **Advanced Query Patterns**

#### **Window Functions and Analytics**
```sql
-- Ranking and analytics
SELECT 
    emp_name,
    salary,
    dept_id,
    ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) as rank_in_dept,
    DENSE_RANK() OVER (ORDER BY salary DESC) as overall_rank,
    LAG(salary, 1) OVER (PARTITION BY dept_id ORDER BY salary) as prev_salary,
    salary - LAG(salary, 1) OVER (PARTITION BY dept_id ORDER BY salary) as salary_gap,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) OVER (PARTITION BY dept_id) as median_salary
FROM employees
ORDER BY dept_id, rank_in_dept;
```

#### **Common Table Expressions (CTEs)**
```sql
-- Recursive CTE for organizational hierarchy
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: Top-level managers
    SELECT emp_id, emp_name, manager_id, 1 as level, emp_name as path
    FROM employees 
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: Employees reporting to managers
    SELECT e.emp_id, e.emp_name, e.manager_id, eh.level + 1, 
           CONCAT(eh.path, ' -> ', e.emp_name)
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.emp_id
)
SELECT * FROM employee_hierarchy
ORDER BY level, emp_name;

-- Complex analysis with multiple CTEs
WITH dept_stats AS (
    SELECT 
        dept_id,
        COUNT(*) as emp_count,
        AVG(salary) as avg_salary,
        STDDEV(salary) as salary_stddev
    FROM employees
    GROUP BY dept_id
),
high_performers AS (
    SELECT emp_id, emp_name, salary, dept_id
    FROM employees
    WHERE salary > (SELECT AVG(salary) FROM employees)
)
SELECT 
    d.dept_name,
    ds.emp_count,
    ds.avg_salary,
    COUNT(hp.emp_id) as high_performer_count,
    ROUND(COUNT(hp.emp_id) * 100.0 / ds.emp_count, 2) as high_performer_percentage
FROM departments d
JOIN dept_stats ds ON d.dept_id = ds.dept_id
LEFT JOIN high_performers hp ON d.dept_id = hp.dept_id
GROUP BY d.dept_id, d.dept_name, ds.emp_count, ds.avg_salary
ORDER BY high_performer_percentage DESC;
```

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
