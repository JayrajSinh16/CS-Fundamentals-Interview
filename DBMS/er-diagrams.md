# ER Diagrams

Entity-Relationship (ER) diagrams are visual representations of database structure showing entities, attributes, and relationships. They help in database design by modeling real-world scenarios before implementing the physical database schema.

## Key Points

- **Entity**: Real-world object (Rectangle) - Student, Course, Department. Represents distinct objects that have independent existence. Can be physical (Person, Car) or conceptual (Course, Order). Each entity has unique identity and set of attributes.
- **Attribute**: Properties of entities (Oval) - Name, Age, ID. Describe characteristics or features of entities. Can be simple, composite, derived, or multivalued. Help define the structure and properties of each entity.
- **Relationship**: Associations between entities (Diamond) - Enrolls, Teaches, Works_for. Shows how entities interact or relate to each other. Can have attributes of their own and represent business rules or constraints.
- **Cardinality**: 1:1, 1:M, M:N relationships defining participation constraints. Specifies how many instances of one entity relate to instances of another. Critical for determining database table structure and foreign key relationships.
- **Primary Key**: Underlined attributes that uniquely identify entity instances. Cannot be null and must be unique within entity set. May be simple (single attribute) or composite (multiple attributes combined).
- **Weak Entity**: Entity dependent on strong entity (Double rectangle) for identification. Cannot exist without relationship to strong entity. Uses partial key combined with strong entity's key for unique identification.

## Example

```
University ER Diagram:

STUDENT                    ENROLLMENT                    COURSE
+----------+              +------------+               +----------+
|StudentID |◄────────────►|            |◄────────────►|CourseID  |
|Name      |              | Grade      |               |CourseName|
|Email     |              | Semester   |               |Credits   |
|Phone     |              +------------+               |Department|
+----------+                    M:N                    +----------+
     |                                                      |
     | 1:M                                                  | M:1
     ▼                                                      ▼
ADVISOR                                               INSTRUCTOR
+----------+                                         +----------+
|AdvisorID |                                         |InstrID   |
|Name      |                                         |Name      |
|Office    |                                         |Department|
+----------+                                         +----------+
```

**Explanation**: Students can enroll in multiple courses (M:N), each student has one advisor (1:M), and each course has one instructor (M:1).

## Common Interview Questions

### Q1: What's the difference between Strong and Weak Entity?
**Answer**: 
- **Strong Entity**: Has its own primary key and exists independently (Student, Course)
- **Weak Entity**: Depends on strong entity for identification (Dependent of Employee)
Weak entities are represented with double rectangles and have partial keys that combine with the strong entity's key.

### Q2: How do you resolve M:N relationships in database implementation?
**Answer**: Create a junction/bridge table with foreign keys from both entities:
```sql
-- M:N: Student-Course becomes:
Students(StudentID, Name)
Courses(CourseID, CourseName)
Enrollments(StudentID, CourseID, Grade) -- Junction table
```

### Q3: What are the types of attributes in ER diagrams?
**Answer**:
- **Simple**: Cannot be divided (Age, Name)
- **Composite**: Can be subdivided (Address = Street + City + ZIP)
- **Derived**: Calculated from other attributes (Age from DOB)
- **Multivalued**: Multiple values allowed (Phone numbers)

### Q4: How do you represent inheritance in ER diagrams?
**Answer**: Use ISA (is-a) relationships with a triangle symbol. Superclass attributes are inherited by subclasses:
```
PERSON (PersonID, Name, DOB)
   |
   ▼ ISA
STUDENT (StudentID)    EMPLOYEE (EmployeeID, Salary)
```

### Q5: What's the difference between total and partial participation?
**Answer**:
- **Total Participation** (double line): Every entity must participate (Every student must be enrolled)
- **Partial Participation** (single line): Not all entities need to participate (Not all students need to be library members)

---
[← Back to Main Guide](./README.md)
