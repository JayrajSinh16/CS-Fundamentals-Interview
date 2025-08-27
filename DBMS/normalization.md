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

### 3NF - Remove Transitive Dependencies
```sql
Students:
| StudentID | StudentName | DeptID |
|-----------|-------------|--------|
| 1         | John        | D001   |
| 2         | Mary        | D002   |

Departments:
| DeptID | DeptName    | DeptHead  |
|--------|-------------|-----------|
| D001   | Engineering | Dr.Wilson |
| D002   | Science     | Dr.Adams  |

Enrollments:
| StudentID | CourseID | Grade |
|-----------|----------|-------|
| 1         | C001     | A     |
| 1         | C002     | B     |
| 2         | C003     | A     |

Courses:
| CourseID | CourseName | Credits | DeptID |
|----------|------------|---------|--------|
| C001     | Math       | 3       | D001   |
| C002     | Physics    | 4       | D001   |
| C003     | Chemistry  | 3       | D002   |
```

## Step-by-Step Normalization Process

### Converting from 0NF to 1NF
**Problem**: Repeating groups and non-atomic values
```sql
-- 0NF: Unnormalized table
Student_Info:
| StudentID | Name | Courses_Grades        | Phone              |
|-----------|------|-----------------------|--------------------|
| 1         | John | Math:A,Physics:B      | 123-456,789-012   |
| 2         | Mary | Chemistry:A           | 555-123           |

-- Step 1: Separate repeating phone numbers
| StudentID | Name | Courses_Grades   | Phone1   | Phone2   |
|-----------|------|------------------|----------|----------|
| 1         | John | Math:A,Physics:B | 123-456  | 789-012  |
| 2         | Mary | Chemistry:A      | 555-123  | NULL     |

-- Step 2: Make course-grade combinations atomic
1NF Result:
| StudentID | Name | Course    | Grade | Phone1   | Phone2   |
|-----------|------|-----------|-------|----------|----------|
| 1         | John | Math      | A     | 123-456  | 789-012  |
| 1         | John | Physics   | B     | 123-456  | 789-012  |
| 2         | Mary | Chemistry | A     | 555-123  | NULL     |
```

### Converting from 1NF to 2NF
**Problem**: Partial dependencies on composite primary key
```sql
-- 1NF table with composite key (StudentID, Course)
Student_Courses_1NF:
| StudentID | Course    | StudentName | Instructor | Grade |
|-----------|-----------|-------------|------------|-------|
| 1         | Math      | John        | Dr.Smith   | A     |
| 1         | Physics   | John        | Dr.Jones   | B     |
| 2         | Chemistry | Mary        | Dr.Brown   | A     |

-- Problem Analysis:
-- StudentName depends only on StudentID (partial dependency)
-- Instructor depends only on Course (partial dependency)
-- Grade depends on both StudentID and Course (full dependency)

-- Step 1: Create Students table (remove StudentName dependency)
Students:
| StudentID | StudentName |
|-----------|-------------|
| 1         | John        |
| 2         | Mary        |

-- Step 2: Create Courses table (remove Instructor dependency)
Courses:
| Course    | Instructor |
|-----------|------------|
| Math      | Dr.Smith   |
| Physics   | Dr.Jones   |
| Chemistry | Dr.Brown   |

-- Step 3: Keep only full dependencies
2NF Result - Enrollments:
| StudentID | Course    | Grade |
|-----------|-----------|-------|
| 1         | Math      | A     |
| 1         | Physics   | B     |
| 2         | Chemistry | A     |
```

### Converting from 2NF to 3NF
**Problem**: Transitive dependencies (A→B→C)
```sql
-- 2NF table with transitive dependency
Students_2NF:
| StudentID | StudentName | DeptName    | DeptHead  | Advisor |
|-----------|-------------|-------------|-----------|---------|
| 1         | John        | Engineering | Dr.Wilson | Dr.Lee  |
| 2         | Mary        | Science     | Dr.Adams  | Dr.Kim  |
| 3         | Bob         | Engineering | Dr.Wilson | Dr.Chen |

-- Problem Analysis:
-- StudentID → DeptName → DeptHead (transitive dependency)
-- DeptHead depends on DeptName, not directly on StudentID

-- Step 1: Remove transitive dependency by creating Departments table
Departments:
| DeptName    | DeptHead  |
|-------------|-----------|
| Engineering | Dr.Wilson |
| Science     | Dr.Adams  |

-- Step 2: Keep only direct dependencies
3NF Result - Students:
| StudentID | StudentName | DeptName    | Advisor |
|-----------|-------------|-------------|---------|
| 1         | John        | Engineering | Dr.Lee  |
| 2         | Mary        | Science     | Dr.Kim  |
| 3         | Bob         | Engineering | Dr.Chen |
```

### Converting from 3NF to BCNF
**Problem**: Determinant that is not a candidate key
```sql
-- 3NF table that violates BCNF
Course_Instructor_3NF:
| CourseID | StudentID | Instructor | Room |
|----------|-----------|------------|------|
| C001     | 1         | Dr.Smith   | R101 |
| C001     | 2         | Dr.Smith   | R101 |
| C001     | 3         | Dr.Jones   | R102 |
| C002     | 1         | Dr.Brown   | R103 |

-- Problem Analysis:
-- Instructor → Room (Dr.Smith always teaches in R101)
-- But Instructor is not a candidate key for the table
-- This violates BCNF rule: every determinant must be a candidate key

-- BCNF Solution: Separate the problematic dependency
Course_Assignments:
| CourseID | StudentID | Instructor |
|----------|-----------|------------|
| C001     | 1         | Dr.Smith   |
| C001     | 2         | Dr.Smith   |
| C001     | 3         | Dr.Jones   |
| C002     | 1         | Dr.Brown   |

Instructor_Rooms:
| Instructor | Room |
|------------|------|
| Dr.Smith   | R101 |
| Dr.Jones   | R102 |
| Dr.Brown   | R103 |
```

## Normalization Decision Tree

```
Start with Unnormalized Table
         ↓
    Has repeating groups? → YES → Separate into rows (1NF)
         ↓ NO
    Has partial dependencies? → YES → Create separate tables (2NF)
         ↓ NO
    Has transitive dependencies? → YES → Remove indirect dependencies (3NF)
         ↓ NO
    Non-candidate key determinants? → YES → Separate determinants (BCNF)
         ↓ NO
    Fully Normalized ✓
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
**Answer**: A transitive dependency occurs when A→B and B→C, so A→C (indirect dependency).
Example: StudentID → ZipCode → City
If we know StudentID, we can determine ZipCode, and from ZipCode we can determine City. This creates redundancy and update anomalies.

**Step-by-step removal**:
```sql
-- Before (2NF with transitive dependency):
Students:
| StudentID | Name | ZipCode | City      |
|-----------|------|---------|-----------|
| 1         | John | 10001   | New York  |
| 2         | Mary | 10001   | New York  |

-- After (3NF - transitive dependency removed):
Students:
| StudentID | Name | ZipCode |
|-----------|------|---------|
| 1         | John | 10001   |
| 2         | Mary | 10001   |

ZipCodes:
| ZipCode | City     |
|---------|----------|
| 10001   | New York |
```

### Q5: Can a relation be in 2NF but not 1NF?
**Answer**: No, it's impossible. Normal forms are cumulative - to be in 2NF, a relation must first satisfy 1NF. You cannot skip levels in normalization.

## Complete Normalization Example: Library System

### Original Unnormalized Table (0NF)
```sql
Library_Records:
| MemberID | MemberName | Books_Issued                    | Authors              | DueDate              | Fine |
|----------|------------|---------------------------------|----------------------|----------------------|------|
| M001     | John Doe   | "Database Systems, SQL Guide"   | "Elmasri, Ullman"   | "2024-01-15,2024-01-20" | 5.00 |
| M002     | Jane Smith | "Data Mining"                   | "Han"               | "2024-01-18"         | 0.00 |
```

### Step 1: Convert to 1NF (Atomic Values)
```sql
Library_Records_1NF:
| MemberID | MemberName | BookTitle        | Author   | DueDate    | Fine |
|----------|------------|------------------|----------|------------|------|
| M001     | John Doe   | Database Systems | Elmasri  | 2024-01-15 | 5.00 |
| M001     | John Doe   | SQL Guide        | Ullman   | 2024-01-20 | 5.00 |
| M002     | Jane Smith | Data Mining      | Han      | 2024-01-18 | 0.00 |
```

### Step 2: Convert to 2NF (Remove Partial Dependencies)
Primary Key: (MemberID, BookTitle)
- MemberName depends only on MemberID (partial dependency)
- Fine depends only on MemberID (partial dependency)

```sql
Members:
| MemberID | MemberName | Fine |
|----------|------------|------|
| M001     | John Doe   | 5.00 |
| M002     | Jane Smith | 0.00 |

Books:
| BookTitle        | Author  |
|------------------|---------|
| Database Systems | Elmasri |
| SQL Guide        | Ullman  |
| Data Mining      | Han     |

Issues:
| MemberID | BookTitle        | DueDate    |
|----------|------------------|------------|
| M001     | Database Systems | 2024-01-15 |
| M001     | SQL Guide        | 2024-01-20 |
| M002     | Data Mining      | 2024-01-18 |
```

### Step 3: Convert to 3NF (Remove Transitive Dependencies)
If we had: BookTitle → Category → Section
This would be a transitive dependency requiring further normalization.

### Final Normalized Schema:
```sql
-- Members table
CREATE TABLE Members (
    MemberID VARCHAR(10) PRIMARY KEY,
    MemberName VARCHAR(100),
    Fine DECIMAL(5,2)
);

-- Books table  
CREATE TABLE Books (
    BookID VARCHAR(10) PRIMARY KEY,
    BookTitle VARCHAR(200),
    Author VARCHAR(100),
    Category VARCHAR(50)
);

-- Issues table
CREATE TABLE Issues (
    IssueID INT PRIMARY KEY,
    MemberID VARCHAR(10),
    BookID VARCHAR(10),
    IssueDate DATE,
    DueDate DATE,
    ReturnDate DATE,
    FOREIGN KEY (MemberID) REFERENCES Members(MemberID),
    FOREIGN KEY (BookID) REFERENCES Books(BookID)
);
```

---
[← Back to Main Guide](./README.md)
