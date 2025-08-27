# Normalization

Normalization is the process of organizing database schema to reduce redundancy and dependency. It divides larger tables into smaller, related tables and defines relationships between them to eliminate data anomalies.

## Key Points

- **1NF (First Normal Form)**: Eliminate repeating groups, atomic values only. Each cell contains single value, no arrays or lists. All entries in a column must be of same data type. Creates foundation for higher normal forms.
- **2NF (Second Normal Form)**: 1NF + eliminate partial dependencies on composite keys. Non-key attributes must depend on entire primary key, not just part of it. Reduces redundancy and prevents update anomalies in tables with composite keys.
- **3NF (Third Normal Form)**: 2NF + eliminate transitive dependencies between non-key attributes. Removes indirect relationships where A→B→C creates redundancy. Ensures non-key attributes depend only on primary key.
- **BCNF (Boyce-Codd Normal Form)**: Stricter version of 3NF, every determinant is a candidate key. Handles cases where 3NF still allows some anomalies. Eliminates all dependency-based redundancies in most practical scenarios.
- **Purpose**: Reduce redundancy, prevent update/insert/delete anomalies, improve data integrity. Makes database more maintainable and reduces storage space. Eliminates inconsistent data and makes schema more flexible.
- **Trade-off**: Higher normalization = less redundancy but more JOINs required for queries. May impact query performance due to multiple table joins. Balance between storage efficiency and query complexity.

## Example

### Unnormalized Table (0NF)
```sql
Student_Courses:
| StudentID | StudentName | Courses           | CourseInstructor    |
|-----------|-------------|-------------------|-------------------|
| 1         | John        | Math, Physics     | Dr.Smith, Dr.Jones|
| 2         | Mary        | Chemistry         | Dr.Brown          |
```

### 1NF - Atomic Values
```sql
Student_Courses:
| StudentID | StudentName | Course    | CourseInstructor |
|-----------|-------------|-----------|------------------|
| 1         | John        | Math      | Dr.Smith         |
| 1         | John        | Physics   | Dr.Jones         |
| 2         | Mary        | Chemistry | Dr.Brown         |
```

### 2NF - Remove Partial Dependencies
```sql
Students:                    Enrollments:
| StudentID | StudentName |  | StudentID | Course    |
|-----------|-------------|  |-----------|-----------|
| 1         | John        |  | 1         | Math      |
| 2         | Mary        |  | 1         | Physics   |
                             | 2         | Chemistry |

Courses:
| Course    | CourseInstructor |
|-----------|------------------|
| Math      | Dr.Smith         |
| Physics   | Dr.Jones         |
| Chemistry | Dr.Brown         |
```

## Common Interview Questions

### Q1: What are the benefits of normalization?
**Answer**: 
- Eliminates data redundancy and saves storage space
- Prevents update, insert, and delete anomalies
- Improves data integrity and consistency
- Makes schema more flexible for future changes

### Q2: When would you denormalize a database?
**Answer**: For performance optimization when:
- Read operations significantly outnumber writes
- Complex JOINs are causing performance bottlenecks
- Data warehouse/analytical systems need faster queries
- Trading storage space for query speed is acceptable

### Q3: What's the difference between 3NF and BCNF?
**Answer**: 
- **3NF**: No transitive dependencies (A→B→C where A is key)
- **BCNF**: Every determinant must be a candidate key (stricter)
BCNF eliminates anomalies that 3NF might still have when there are overlapping candidate keys.

### Q4: Explain with example: what is a transitive dependency?
**Answer**: A→B and B→C, so A→C (indirect dependency).
Example: StudentID → ZipCode → City
If we know StudentID, we can determine ZipCode, and from ZipCode we can determine City. This creates redundancy and update anomalies.

### Q5: Can a relation be in 2NF but not 1NF?
**Answer**: No, it's impossible. Normal forms are cumulative - to be in 2NF, a relation must first satisfy 1NF. You cannot skip levels in normalization.

---
[← Back to Main Guide](./README.md)
