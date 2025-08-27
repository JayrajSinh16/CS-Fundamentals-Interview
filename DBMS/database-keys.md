# Database Keys

Database keys are attributes or combinations of attributes that uniquely identify records and establish relationships between tables. They ensure data integrity and enable efficient data retrieval.

## Key Points

- **Primary Key**: Unique identifier for each record, cannot be NULL or duplicate. Automatically creates a clustered index for faster data retrieval. Only one primary key allowed per table, and it enforces entity integrity.
- **Foreign Key**: References primary key of another table, maintains referential integrity between tables. Can have multiple foreign keys in a table. Supports cascade operations (DELETE/UPDATE) and prevents orphaned records.
- **Candidate Key**: Minimal set of attributes that can uniquely identify records without redundancy. Multiple candidate keys can exist in a table. When one is chosen as primary key, others become alternate keys.
- **Super Key**: Any set of attributes that can uniquely identify records, may contain redundant attributes. Every candidate key is a super key, but not every super key is a candidate key.
- **Composite Key**: Primary key consisting of multiple attributes combined together. Useful when no single attribute can uniquely identify records. Common in junction tables for many-to-many relationships.
- **Alternate Key**: Candidate keys that are not chosen as primary key but still maintain uniqueness. Can be implemented using UNIQUE constraints. Useful for natural business identifiers like email addresses.

## Example

```sql
-- Students table
CREATE TABLE Students (
    student_id INT PRIMARY KEY,          -- Primary Key
    email VARCHAR(100) UNIQUE,          -- Candidate Key (Alternate Key)
    first_name VARCHAR(50),
    last_name VARCHAR(50)
);

-- Enrollments table
CREATE TABLE Enrollments (
    student_id INT,                     -- Foreign Key
    course_id INT,                      -- Foreign Key
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id), -- Composite Key
    FOREIGN KEY (student_id) REFERENCES Students(student_id)
);
```

**Explanation**: `student_id` uniquely identifies each student, while the combination of `student_id` and `course_id` uniquely identifies each enrollment record.

## Common Interview Questions

### Q1: What's the difference between Primary Key and Unique Key?
**Answer**: Primary Key cannot be NULL and there's only one per table, while Unique Key can have one NULL value and multiple unique keys are allowed per table. Primary Key automatically creates a clustered index.

### Q2: Can a table have multiple foreign keys?
**Answer**: Yes, a table can have multiple foreign keys referencing different tables or even the same table (self-referencing). Each foreign key maintains referential integrity with its referenced table.

### Q3: What happens when you try to delete a record referenced by a foreign key?
**Answer**: It depends on the referential action:
- **RESTRICT/NO ACTION**: Deletion is prevented
- **CASCADE**: Related records are also deleted
- **SET NULL**: Foreign key values are set to NULL
- **SET DEFAULT**: Foreign key values are set to default

### Q4: Why can't a Primary Key contain NULL values?
**Answer**: Primary Keys must uniquely identify each record. NULL represents unknown/missing data, so two NULL values can't be compared for uniqueness, violating the uniqueness constraint.

### Q5: What's a surrogate key vs natural key?
**Answer**: 
- **Natural Key**: Business-meaningful data (SSN, email)
- **Surrogate Key**: System-generated identifier (auto-increment ID)
Surrogate keys are preferred for primary keys as they're stable and don't change with business requirements.

---
[← Back to Main Guide](./README.md)
