# ER Diagrams

Entity-Relationship (ER) diagrams are visual representations of database structure showing entities, attributes, and relationships. They help in database design by modeling real-world scenarios before implementing the physical database schema.

## Key Points

- **Entity**: Real-world object (Rectangle) - Student, Course, Department. Represents distinct objects that have independent existence. Can be physical (Person, Car) or conceptual (Course, Order). Each entity has unique identity and set of attributes.
- **Attribute**: Properties of entities (Oval) - Name, Age, ID. Describe characteristics or features of entities. Can be simple, composite, derived, or multivalued. Help define the structure and properties of each entity.
- **Relationship**: Associations between entities (Diamond) - Enrolls, Teaches, Works_for. Shows how entities interact or relate to each other. Can have attributes of their own and represent business rules or constraints.
- **Cardinality**: 1:1, 1:M, M:N relationships defining participation constraints. Specifies how many instances of one entity relate to instances of another. Critical for determining database table structure and foreign key relationships.
- **Primary Key**: Underlined attributes that uniquely identify entity instances. Cannot be null and must be unique within entity set. May be simple (single attribute) or composite (multiple attributes combined).
- **Weak Entity**: Entity dependent on strong entity (Double rectangle) for identification. Cannot exist without relationship to strong entity. Uses partial key combined with strong entity's key for unique identification.

## ER Diagram Components and Notation

### 🔷 **Basic Components**
```
┌─────────────────────────────────────────────────────────────────────────┐
│                           ER DIAGRAM SYMBOLS                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  📦 ENTITY (Rectangle)                🔗 RELATIONSHIP (Diamond)         │
│  ┌─────────────────┐                  ◊─────────────────◊             │
│  │     STUDENT     │                 ╱                   ╲            │
│  │                 │                ╱     ENROLLS_IN      ╲           │
│  └─────────────────┘               ╱                       ╲          │
│                                   ◊─────────────────────────◊         │
│                                                                         │
│  ⭕ ATTRIBUTE (Oval)               🔑 PRIMARY KEY (Underlined)         │
│     ╭─────────────╮                   StudentID                       │
│    ╱    Name      ╲                   ─────────                       │
│   ╱               ╲                                                    │
│  ╱                 ╲                                                   │
│ ╲                   ╱                                                   │
│  ╲                 ╱                🔸 WEAK ENTITY (Double Rectangle)  │
│   ╲               ╱                 ╔═════════════════╗                │
│    ╲_____________╱                  ║    DEPENDENT    ║                │
│                                     ║                 ║                │
│                                     ╚═════════════════╝                │
│                                                                         │
│  📊 CARDINALITY NOTATION:                                              │
│  ──── 1 (One)        ────< M (Many)        ────|| Total Participation │
│  ────○ 0 (Zero)      ────┤ Exactly One     ──── Partial Participation │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 🏫 **University System - Complete ER Diagram**

```
                    🎓 UNIVERSITY DATABASE SYSTEM 🎓

      ╔══════════════════╗                    ╔══════════════════╗
      ║     STUDENT      ║                    ║    INSTRUCTOR    ║
      ║                  ║                    ║                  ║
      ║ 🔑 StudentID     ║                    ║ 🔑 InstructorID  ║
      ║    Name          ║                    ║    Name          ║
      ║    Email         ║                    ║    Department    ║
      ║    Phone         ║                    ║    Salary        ║
      ║    DOB           ║                    ║    Office        ║
      ╚══════════════════╝                    ╚══════════════════╝
               │                                       │
               │ 1                                     │ M
               │                                       │
        ◊─────────────◊                        ◊─────────────◊
       ╱               ╲                      ╱               ╲
      ╱   HAS_ADVISOR   ╲                    ╱     TEACHES     ╲
     ╱                   ╲                  ╱                   ╲
    ◊─────────────────────◊                ◊─────────────────────◊
               │                                       │
               │ M                                     │ 1
               │                                       │
      ╔══════════════════╗                    ╔══════════════════╗
      ║     ADVISOR      ║                    ║      COURSE      ║
      ║                  ║                    ║                  ║
      ║ 🔑 AdvisorID     ║                    ║ 🔑 CourseID      ║
      ║    Name          ║                    ║    CourseName    ║
      ║    Department    ║                    ║    Credits       ║
      ║    Office        ║                    ║    Department    ║
      ║    Phone         ║                    ║    Prerequisites ║
      ╚══════════════════╝                    ╚══════════════════╝
                                                       │
                                                       │ M
                                                       │
                             ╔═══════════════════════════════════════╗
                             ║            ENROLLMENT                 ║
                             ║        (M:N Relationship)            ║
                             ║                                       ║
                             ║  🔑 StudentID + CourseID             ║
                             ║     Semester                          ║
                             ║     Grade                             ║
                             ║     EnrollmentDate                    ║
                             ╚═══════════════════════════════════════╝
                                                       │
                                                       │ M
                                                       │
                                              ╔══════════════════╗
                                              ║     STUDENT      ║
                                              ║   (Same as above) ║
                                              ╚══════════════════╝

         🏢 DEPARTMENT HIERARCHY

      ╔══════════════════╗
      ║    DEPARTMENT    ║                    ╔═══════════════════╗
      ║                  ║ 1            M    ║     EMPLOYEE      ║
      ║ 🔑 DeptID        ║────────────────────║                   ║
      ║    DeptName      ║   WORKS_FOR        ║ 🔑 EmployeeID     ║
      ║    Location      ║                    ║    Name           ║
      ║    Budget        ║                    ║    Position       ║
      ║ 🔗 ManagerID     ║                    ║    Salary         ║
      ╚══════════════════╝                    ║    HireDate       ║
               │                              ╚═══════════════════╝
               │ ISA                                   │
               │ (Inheritance)                        │ M
               ▼                                      │
        ╔══════════════════╗                         │ 1
        ║     MANAGER      ║                  ◊─────────────◊
        ║                  ║                 ╱               ╲
        ║ (Inherits from   ║                ╱   SUPERVISES    ╲
        ║  Employee)       ║               ╱                   ╲
        ║                  ║              ◊─────────────────────◊
        ║ + Bonus          ║                         │
        ║ + ManagedDept    ║                         │ M
        ╚══════════════════╝                         │
                                              ╔══════════════════╗
                                              ║   SUBORDINATE    ║
                                              ║                  ║
                                              ║ (Same Employee   ║
                                              ║  Entity)         ║
                                              ╚══════════════════╝
```

### 🔄 **Cardinality Examples with Visual Representation**

#### **1:1 (One-to-One) Relationship**
```
    PERSON                    PASSPORT
 ┌──────────┐              ┌──────────┐
 │PersonID  │ 1        1   │PassportNo│
 │Name      │──────────────│IssueDate │
 │Address   │   HAS_A      │ExpiryDate│
 └──────────┘              └──────────┘

 💡 Each person has exactly one passport
    Each passport belongs to exactly one person
```

#### **1:M (One-to-Many) Relationship**
```
    CUSTOMER                   ORDER
 ┌──────────┐              ┌──────────┐
 │CustomerID│ 1        M   │OrderID   │
 │Name      │──────────────│OrderDate │
 │Email     │   PLACES     │Amount    │
 └──────────┘              └──────────┘

 💡 One customer can place many orders
    Each order belongs to exactly one customer
```

#### **M:N (Many-to-Many) Relationship**
```
    STUDENT                    COURSE
 ┌──────────┐              ┌──────────┐
 │StudentID │ M        M   │CourseID  │
 │Name      │──────────────│CourseName│
 │Major     │  ENROLLS_IN  │Credits   │
 └──────────┘              └──────────┘
       │                         │
       └─────────────┬───────────┘
                     │
              ┌─────────────┐
              │ ENROLLMENT  │ (Junction Table)
              │             │
              │ StudentID   │ (FK)
              │ CourseID    │ (FK)
              │ Grade       │
              │ Semester    │
              └─────────────┘

 💡 One student can enroll in many courses
    One course can have many students enrolled
```

### 🔐 **Participation Constraints**

```
📋 PARTICIPATION TYPES:

🔸 TOTAL PARTICIPATION (Double Line ══)
   Every entity MUST participate in the relationship

    EMPLOYEE                   DEPARTMENT
 ┌──────────┐              ┌──────────┐
 │EmployeeID│              │ DeptID   │
 │Name      │══════════════│DeptName  │
 │Position  │  WORKS_FOR   │Location  │
 └──────────┘              └──────────┘

 💡 Every employee MUST work for a department
    No employee can exist without a department


🔹 PARTIAL PARTICIPATION (Single Line ──)
   Not all entities need to participate

    EMPLOYEE                   PROJECT
 ┌──────────┐              ┌──────────┐
 │EmployeeID│              │ProjectID │
 │Name      │──────────────│ProjectName│
 │Position  │  WORKS_ON    │Budget    │
 └──────────┘              └──────────┘

 💡 Some employees may not work on any project
    This is optional participation
```

### 🔗 **Weak Entity Example**

```
      STRONG ENTITY                    WEAK ENTITY
   ┌─────────────────┐             ╔═══════════════════╗
   │    EMPLOYEE     │ 1       M   ║     DEPENDENT     ║
   │                 │─────────────║                   ║
   │ 🔑 EmployeeID   │   HAS_A     ║ 🔸 Name          ║ (Partial Key)
   │    Name         │             ║    Age            ║
   │    Department   │             ║    Relationship   ║
   └─────────────────┘             ╚═══════════════════╝

 📝 WEAK ENTITY CHARACTERISTICS:
 • Cannot exist without strong entity (Employee)
 • Partial key (Name) + Strong entity key (EmployeeID) = Unique identification
 • Shown with double rectangle
 • Identifying relationship shown with double diamond
 • Example: Employee "John Smith" has dependents "Mary" and "Tom"
   - Dependents identified as: (John_Smith_ID + "Mary"), (John_Smith_ID + "Tom")
```

### 🏗️ **ISA Hierarchy (Inheritance)**

```
                        PERSON
                   ┌─────────────┐
                   │ 🔑 PersonID │
                   │    Name     │
                   │    Address  │
                   │    Phone    │
                   └─────────────┘
                          │
                          │ ISA
                          ▼
                    ╱─────────────╲
                   ╱               ╲
                  ╱                 ╲
                 ▼                   ▼
         ┌─────────────┐      ┌─────────────┐
         │   STUDENT   │      │  EMPLOYEE   │
         │             │      │             │
         │ + StudentID │      │ + EmployeeID│
         │ + Major     │      │ + Salary    │
         │ + GPA       │      │ + Department│
         └─────────────┘      └─────────────┘
                                     │
                                     │ ISA
                                     ▼
                              ╱─────────────╲
                             ╱               ╲
                            ▼                 ▼
                    ┌─────────────┐   ┌─────────────┐
                    │   FACULTY   │   │    STAFF    │
                    │             │   │             │
                    │ + Rank      │   │ + Position  │
                    │ + ResearchArea│ │ + ShiftTime │
                    └─────────────┘   └─────────────┘

 💡 INHERITANCE RULES:
 • Child entities inherit all attributes from parent
 • Child entities can have additional specific attributes
 • ISA relationship is always 1:1
 • Can be TOTAL (every Person must be Student OR Employee)
 • Can be PARTIAL (some Persons may be neither)
 • Can be OVERLAPPING (Person can be both Student AND Employee)
```

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
