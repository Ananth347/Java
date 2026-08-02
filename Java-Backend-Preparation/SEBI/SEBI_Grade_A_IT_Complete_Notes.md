# SEBI Grade A — Information Technology Stream
## Complete Study Notes (Phase I + Phase II)

> Prepared to cover the full official syllabus in depth, in plain explained English, with worked examples, code snippets, comparison tables, and exam traps. Read topic by topic — don't skim. Every topic in the syllabus is covered; nothing is skipped.

---

## HOW TO USE THESE NOTES

These notes are organized exactly along the two official syllabi:

- **Phase I Paper 2 (IT Stream)** — Database Concepts, SQL, Programming Concepts (Java/C/C++), Data Analytics Languages (Python/R), Algorithms, Networking, Cyber Security, Data Warehousing, Shell Programming.
- **Phase II Paper 2 (IT Stream)** — Algorithms, Data Structures, String Manipulation, OOP — but this time at a much deeper, more numerical/coding level, since Phase II questions are mostly "trace the code" or "compute the answer" style, exactly like the 69 questions you pulled from Reddit.

Each section has:
1. **Concept explanation** — in plain spoken English, not textbook jargon.
2. **Diagrams/structure described in words** where a picture would normally go.
3. **Worked numerical examples** — the kind SEBI actually asks.
4. **Code snippets** in Java, C, C++, and Python where relevant.
5. **Common traps** — the exact kind of "gotcha" that shows up in real papers.
6. **Quick-fire revision table** at the end of every section.

Target score: 90+/100. That means you cannot afford to lose easy DS/Algo/OOP marks (Phase II is 40% Data Structures alone), and in Phase I, SQL + Programming Concepts together are 40% of the paper. Prioritize accordingly, but this document still covers everything, including the "smaller" topics like shell scripting and data warehousing, because SEBI does ask 2-3 direct questions from even a 5% weightage topic.

---

# TABLE OF CONTENTS

**PHASE I**
1. Database Concepts (ER Model, Relational Model, Normalization, Indexing, Transactions)
2. SQL Queries
3. Programming Concepts (Java/C/C++)
4. Data Analytics Languages (Python/R)
5. Algorithms for Problem Solving
6. Networking Concepts
7. Information & Cyber Security
8. Data Warehousing
9. Shell Programming

**PHASE II**
10. Algorithms (deep dive, numerical)
11. Data Structures (deep dive, numerical)
12. String Manipulation
13. Object Oriented Programming (deep dive)

**APPENDIX**
- Master formula sheet
- Master trap sheet
- Practice question bank

---

# SECTION 1: DATABASE CONCEPTS

## 1.1 Why Databases, and What "Data Model" Means

A database is an organized collection of data that a computer can access, manage, and update efficiently. Before we touch SQL, SEBI wants you to understand the **design layer** — how you go from "the real world" (students, courses, banks, accounts) to a working relational database. This design layer has three classic stages:

1. **Conceptual design** — draw an **ER (Entity-Relationship) diagram**. This is a high-level, implementation-independent picture of what data exists and how it relates.
2. **Logical design** — convert the ER diagram into a **relational schema** (tables, columns, keys). This is where normalization happens.
3. **Physical design** — decide indexing, storage, partitioning — how data actually sits on disk.

SEBI questions test all three stages, but ER-to-relational mapping and normalization are the most heavily tested.

## 1.2 ER Model in Depth

### 1.2.1 Entities and Entity Sets

An **entity** is a real-world object that can be distinctly identified — e.g., a specific student "Ramesh Kumar, roll no 21." An **entity set** is a collection of similar entities — e.g., "all students." In a diagram, entity sets are drawn as **rectangles**.

An entity has **attributes** — properties that describe it. Attributes are drawn as **ovals** connected to the entity rectangle.

### 1.2.2 Types of Attributes

- **Simple (atomic) attribute** — cannot be divided further. E.g., Roll_No.
- **Composite attribute** — can be broken into sub-parts. E.g., Name → First_Name + Last_Name; Address → Street + City + Pincode.
- **Single-valued attribute** — has exactly one value per entity. E.g., Date_of_Birth.
- **Multi-valued attribute** — can have many values for one entity. E.g., Phone_Numbers (a person can have more than one). Drawn as a **double oval**.
- **Derived attribute** — computed from other attributes, not stored directly. E.g., Age derived from Date_of_Birth. Drawn as a **dashed oval**.
- **Key attribute** — uniquely identifies an entity. Underlined in diagrams.

**Exam trap:** SEBI often gives a scenario and asks "which attribute is derived / multivalued / composite" — read the scenario carefully; "Age" and "Total_Marks" (computed from subject marks) are the most common "derived attribute" answers.

### 1.2.3 Keys — This Is Heavily Tested

- **Super Key** — any set of attributes that can uniquely identify a tuple (row). A relation with attributes (A, B, C) where A alone is unique — then {A}, {A,B}, {A,C}, {A,B,C} are ALL super keys, because uniqueness is preserved even by adding extra columns.
- **Candidate Key** — a **minimal** super key, i.e., no attribute can be removed from it without losing the uniqueness property. There can be multiple candidate keys in a relation.
- **Primary Key** — the candidate key chosen by the database designer to be THE main identifier. Only one primary key per table. Cannot be NULL, must be unique.
- **Alternate Key** — candidate keys that were NOT chosen as the primary key.
- **Foreign Key** — an attribute in one relation that refers to the primary key of another (or the same) relation, used to enforce referential integrity between tables.
- **Composite Key** — a primary key made of more than one attribute together (neither alone is unique, but the combination is).

**Worked example:** Table STUDENT(Roll_No, Aadhar_No, Email, Name). Suppose Roll_No, Aadhar_No, and Email are all individually unique.
- Candidate keys: {Roll_No}, {Aadhar_No}, {Email} — three separate minimal unique sets.
- Suppose the designer picks Roll_No as Primary Key.
- Then Aadhar_No and Email become Alternate Keys.
- {Roll_No, Name} is a super key (not minimal, so not a candidate key).

### 1.2.4 Relationships and Degree

A **relationship** connects two or more entity sets — drawn as a **diamond**. The **degree** of a relationship = number of entity sets participating.
- **Unary/Recursive** (degree 1) — relates an entity set to itself. E.g., EMPLOYEE "manages" EMPLOYEE (a manager is also an employee).
- **Binary** (degree 2) — the most common. E.g., STUDENT "enrolls in" COURSE.
- **Ternary** (degree 3) — three entity sets together. E.g., SUPPLIER "supplies" PART to PROJECT — you genuinely need all three to define one fact; splitting it into three binary relationships would lose information.

### 1.2.5 Cardinality Ratios (Mapping Cardinalities)

This defines how many entity instances on one side can relate to how many on the other side.
- **One-to-One (1:1)** — E.g., a PERSON has exactly one PASSPORT, and a PASSPORT belongs to exactly one PERSON.
- **One-to-Many (1:N)** — E.g., one DEPARTMENT has many EMPLOYEES, but each EMPLOYEE belongs to only one DEPARTMENT.
- **Many-to-One (N:1)** — Same as above, viewed from the other side.
- **Many-to-Many (M:N)** — E.g., a STUDENT can enroll in many COURSES, and a COURSE can have many STUDENTS.

**How cardinality decides where the foreign key goes when converting ER → relational:**
- **1:1** — foreign key can go on either side (put it on the side that participates totally/mandatorily, to avoid NULLs).
- **1:N** — foreign key goes on the "many" side (the "N" side gets the FK pointing to the "1" side's primary key). E.g., EMPLOYEE table gets a Dept_ID column pointing to DEPARTMENT.
- **M:N** — you CANNOT just add a foreign key column; you must create a **separate junction/bridge table** containing the primary keys of both entities as a composite primary key (and any relationship attributes). E.g., STUDENT_COURSE(Roll_No, Course_ID, Enrollment_Date).

**This M:N → junction table rule is one of the most frequently tested ER concepts. Memorize it cold.**

### 1.2.6 Participation Constraints

- **Total participation** — every entity instance MUST participate in the relationship. Drawn as a double line.
- **Partial participation** — participation is optional. Drawn as a single line.

Example: Every EMPLOYEE must work in some DEPARTMENT (total participation from Employee side), but not every DEPARTMENT necessarily has employees assigned yet in a new company (partial participation from Department side).

### 1.2.7 Weak Entities

A **weak entity** does not have a primary key of its own — it depends on a "strong" (owner/identifying) entity for its identity. Its own key is called a **partial key/discriminator**, and the true unique identity comes from (partial key + owner's primary key).
- Drawn as a **double rectangle**; the identifying relationship is a **double diamond**.
- Classic example: DEPENDENT (a bank customer's dependents/children) — a Dependent_Name alone is not unique across the whole bank, but (Customer_ID, Dependent_Name) together is unique. DEPENDENT is a weak entity owned by CUSTOMER.

### 1.2.8 Generalization, Specialization, Aggregation (EER concepts)

- **Specialization** — top-down process: take a general entity (e.g., PERSON) and break it into specialized sub-entities (STUDENT, EMPLOYEE) based on distinguishing characteristics.
- **Generalization** — bottom-up process: combine several entities that share common features into one general super-entity.
- **Aggregation** — allows a relationship itself to participate in another relationship, treating a relationship-set as if it were an entity. Used when you need to relate a whole relationship to another entity. E.g., the relationship (EMPLOYEE works-on PROJECT) can itself be linked to MANAGER via "monitors."

## 1.3 Relational Model

### 1.3.1 Basic Terminology

- **Relation** = a table.
- **Tuple** = a row (one record).
- **Attribute** = a column.
- **Domain** = the set of allowed/legal values for an attribute (e.g., domain of Age might be positive integers 0-120).
- **Degree of a relation** = number of attributes (columns).
- **Cardinality of a relation** = number of tuples (rows) at a given time.

### 1.3.2 Integrity Constraints

- **Domain constraint** — every attribute value must be from its defined domain (correct data type/range).
- **Entity Integrity constraint** — no attribute that is part of the PRIMARY KEY can be NULL. (A primary key must fully, uniquely identify every row.)
- **Referential Integrity constraint** — a foreign key value must either match an existing primary key value in the referenced table, or be NULL (if allowed). You cannot insert a foreign key value that doesn't exist in the parent table, and you cannot delete a parent row that is still referenced (unless you use CASCADE rules).
- **Key constraint** — enforces that primary key values are unique.

**Referential actions on DELETE/UPDATE of a parent row (very commonly tested in SQL context too):**
- `CASCADE` — automatically delete/update the matching child rows too.
- `SET NULL` — set the foreign key in child rows to NULL.
- `RESTRICT` / `NO ACTION` — refuse the delete/update if child rows exist referencing it.
- `SET DEFAULT` — set the foreign key to a predefined default value.

### 1.3.3 Relational Algebra

Relational algebra is a **procedural query language** — a set of operations that take one or two relations as input and produce a new relation as output. It's the theoretical foundation SQL is built on.

**Basic (fundamental) operations:**
1. **Selection (σ)** — σ<sub>condition</sub>(R) — picks ROWS satisfying a condition. Example: σ<sub>Salary > 50000</sub>(EMPLOYEE).
2. **Projection (π)** — π<sub>col1, col2</sub>(R) — picks COLUMNS, and removes duplicate rows from the result. Example: π<sub>Name, Dept</sub>(EMPLOYEE).
3. **Union (∪)** — R ∪ S — combines tuples from both relations, removing duplicates. Requires R and S to be **union-compatible** (same number of attributes, same domains in order).
4. **Set Difference (−)** — R − S — tuples in R but NOT in S.
5. **Cartesian Product (×)** — R × S — every tuple of R combined with every tuple of S. If R has m tuples and S has n tuples, result has m×n tuples.
6. **Rename (ρ)** — ρ<sub>newname</sub>(R) — renames a relation or its attributes.

**Derived operations (built from the basic six):**
7. **Intersection (∩)** — R ∩ S = R − (R − S). Tuples common to both.
8. **Join (⋈)**:
   - **Theta Join (θ)** — combines tuples from two relations based on any given condition (θ can be =, <, >, etc.).
   - **Equijoin** — a theta join where the condition uses only equality (=).
   - **Natural Join (⋈)** — an equijoin on all attributes with the SAME NAME in both relations, and then the duplicate column is automatically removed from the result. This is the join used most in practice/exams.
   - **Outer Join** — preserves unmatched tuples too (Left, Right, Full) — filling missing side with NULLs.
9. **Division (÷)** — R ÷ S — used for "find X that are related to ALL of Y" type queries. Classic example: "find students who have enrolled in ALL courses that are offered" is naturally a division operation.

**Worked trap example:** If R has 5 tuples with 3 attributes, and S has 4 tuples with 3 attributes (same domains), then:
- R ∪ S → at most 9 tuples (fewer if duplicates), 3 attributes.
- R − S → at most 5 tuples, 3 attributes.
- R × S → exactly 20 tuples, 6 attributes.

### 1.3.4 Tuple Relational Calculus & Domain Relational Calculus

Unlike relational algebra (procedural — you specify HOW to get the result), **relational calculus is declarative** — you specify WHAT you want, not how.

- **Tuple Relational Calculus (TRC)**: queries are of the form `{ t | P(t) }` — "the set of all tuples t such that predicate P(t) is true." Variables represent tuples (rows).
  - Example: `{ t | t ∈ EMPLOYEE ∧ t.Salary > 50000 }` means "all employee tuples with salary greater than 50000."
- **Domain Relational Calculus (DRC)**: queries are of the form `{ <x1, x2, ..., xn> | P(x1, x2, ..., xn) }` — variables represent individual DOMAIN VALUES (individual attribute values), not whole tuples.

Both TRC and DRC are equivalent in expressive power to relational algebra (this equivalence is called **relational completeness** — a language is relationally complete if it can express everything relational algebra can express). SQL is also relationally complete (and more, since it has aggregate functions etc. that pure relational algebra lacks).

**Quantifiers used in calculus:**
- **∃ (existential)** — "there exists."
- **∀ (universal)** — "for all."

## 1.4 Normalization

Normalization is the process of organizing columns and tables of a relational database to **minimize data redundancy** and avoid **update, insertion, and deletion anomalies**.

### 1.4.1 The Three Anomalies (Why We Normalize)

Imagine one big unnormalized table: STUDENT_COURSE(Roll_No, Student_Name, Course_ID, Course_Name, Instructor).

- **Update anomaly** — if a course name changes, you must update it in EVERY row where that course appears; miss even one and your data becomes inconsistent.
- **Insertion anomaly** — you cannot add a new course to the database until at least one student enrolls in it (because Roll_No might be part of the key and can't be null), even though logically the course can exist independently.
- **Deletion anomaly** — if the last student enrolled in a course drops out and that row is deleted, you lose all information about that course entirely (its name, instructor) even though the course itself might still exist.

Normalization fixes these by splitting tables based on **functional dependencies**.

### 1.4.2 Functional Dependency (FD)

A functional dependency X → Y means: for any two tuples, if they agree on X, they must agree on Y. X is called the **determinant**. Read as "X functionally determines Y."

- **Trivial FD** — Y is a subset of X (e.g., {Roll_No, Name} → {Name}). Always true, not useful.
- **Non-trivial FD** — Y is NOT a subset of X.
- **Fully functional dependency** — Y depends on the WHOLE of X, not on any proper subset of X. Relevant only when X is a composite (multi-attribute) key.
- **Partial dependency** — Y depends on only PART of a composite key X (i.e., on a proper subset of X).
- **Transitive dependency** — X → Y and Y → Z, so X → Z indirectly (through Y), where Y is not a candidate key.

**Armstrong's Axioms** (rules for deriving all FDs from a given set — "closure"):
1. **Reflexivity**: if Y ⊆ X, then X → Y.
2. **Augmentation**: if X → Y, then XZ → YZ for any Z.
3. **Transitivity**: if X → Y and Y → Z, then X → Z.

Derived/secondary rules (provable from the above three, but useful shortcuts):
- **Union**: if X → Y and X → Z, then X → YZ.
- **Decomposition**: if X → YZ, then X → Y and X → Z.
- **Pseudotransitivity**: if X → Y and WY → Z, then WX → Z.

### 1.4.3 Normal Forms — The Core of This Topic

**1NF (First Normal Form)**: every attribute must contain only **atomic (indivisible) values** — no repeating groups, no multi-valued attributes, no arrays/lists inside a single cell. A table is in 1NF automatically if it's a valid relational table by definition — but exam questions test this by giving a table where one column has comma-separated values like "Phone: 9876543210, 9123456780" and asking you to identify the 1NF violation.

**2NF (Second Normal Form)**: must already be in 1NF, AND **no partial dependency** — every non-key attribute must depend on the WHOLE of the primary key, not just part of it. 2NF violations only matter when you have a **composite primary key**.
- Example: ORDER_ITEM(Order_ID, Product_ID, Product_Name, Quantity) with PK = (Order_ID, Product_ID). Here Product_Name depends only on Product_ID (a part of the key), NOT on the whole key → this is a partial dependency → violates 2NF. Fix: split into ORDER_ITEM(Order_ID, Product_ID, Quantity) and PRODUCT(Product_ID, Product_Name).

**3NF (Third Normal Form)**: must already be in 2NF, AND **no transitive dependency** of non-key attributes on the primary key — i.e., no non-key attribute depends on another non-key attribute.
- Example: EMPLOYEE(Emp_ID, Emp_Name, Dept_ID, Dept_Name). Here Emp_ID → Dept_ID → Dept_Name, and Dept_ID is not a key. This is a transitive dependency → violates 3NF. Fix: split into EMPLOYEE(Emp_ID, Emp_Name, Dept_ID) and DEPARTMENT(Dept_ID, Dept_Name).

**BCNF (Boyce-Codd Normal Form)**: a stronger version of 3NF. A relation is in BCNF if, for every non-trivial FD X → Y, X must be a **super key**. 
- The difference from 3NF: 3NF has an exception clause that allows Y to be a prime attribute (part of some candidate key) even if X isn't a super key; BCNF removes that exception, so BCNF is strictly stronger (every BCNF relation is in 3NF, but not vice versa).
- **Trap**: BCNF is not always achievable without losing a functional dependency (i.e., sometimes decomposing into BCNF causes you to lose the ability to enforce a certain FD via just primary-key constraints) — this is the classic "3NF vs BCNF" tradeoff question: 3NF decomposition is always **dependency preserving AND lossless**, but BCNF decomposition is always **lossless** but is NOT always **dependency preserving**.

**4NF (Fourth Normal Form)**: deals with **multi-valued dependencies (MVD)**. A relation is in 4NF if it's in BCNF and has no non-trivial multi-valued dependency. A multi-valued dependency X →→ Y means for a given X, there's a set of Y values independent of other attributes. Classic example: a table (Employee, Skill, Language) where Skills and Languages are independent of each other for a given employee — this causes redundant combinations and needs splitting into (Employee, Skill) and (Employee, Language).

**5NF (Fifth Normal Form / Project-Join Normal Form, PJNF)**: deals with **join dependencies** — a relation is decomposed into multiple relations such that joining them back together (via natural join) reconstructs the original relation exactly, with no spurious tuples. This is mostly theoretical for exam purposes — just remember the name and the concept "eliminates redundancy from join dependencies."

**Denormalization**: the reverse process — deliberately introducing redundancy into a normalized database (e.g., merging tables, adding derived/computed columns) to improve READ performance, at the cost of write complexity and redundancy. Used in reporting/data-warehouse systems (relevant to Section 8).

### 1.4.4 Decomposition Properties

When you split (decompose) a relation into smaller relations during normalization, two properties matter:

1. **Lossless (Lossless-Join) Decomposition** — you must be able to reconstruct the EXACT original relation by natural-joining the decomposed relations back together — no extra (spurious) rows, no missing rows. Test: a decomposition of R into R1 and R2 is lossless if and only if (R1 ∩ R2) → R1 OR (R1 ∩ R2) → R2 (the common attribute set must be a key of at least one of the two).
2. **Dependency Preservation** — all the original functional dependencies should still be enforceable by looking only at the individual decomposed relations, without needing to join them back together.

**Quick summary table:**

| Normal Form | Condition | Fixes |
|---|---|---|
| 1NF | Atomic values only | Repeating groups |
| 2NF | 1NF + no partial dependency | Redundancy from composite key parts |
| 3NF | 2NF + no transitive dependency | Redundancy via non-key attributes |
| BCNF | Every determinant is a super key | Stronger anomaly removal, may not preserve deps |
| 4NF | BCNF + no non-trivial MVD | Independent multi-valued facts |
| 5NF | No join dependency anomalies | Redundancy from complex joins |

## 1.5 File Organization

This is about how records are physically stored on disk.

- **Heap (unordered) file organization** — records are placed wherever there is space, in no particular order. Insertion is fast (O(1), just append), but searching requires a full linear scan O(n).
- **Sequential (ordered) file organization** — records are stored in sorted order of some field (usually the key). Searching can use binary search O(log n), but insertion is expensive because you may need to shift records to maintain order (unless you keep an overflow area).
- **Hash file organization** — a hash function maps the key to a specific storage location (bucket). Very fast direct access O(1) average, but poor for range queries and can suffer from collisions.
- **Clustered file organization** — records from different but related tables that are likely to be accessed together are stored physically close together on disk, to speed up joins.

## 1.6 Indexing

An **index** is an auxiliary data structure that allows faster retrieval of records, at the cost of extra storage and slower writes (every insert/update/delete must also update the index).

### 1.6.1 Types of Indexes

- **Primary Index** — built on the primary key of a file that IS sorted by that key. Contains one entry per DATA BLOCK (sparse), not per record.
- **Clustering Index** — built on a non-key field by which the file IS physically ordered (there can be duplicates). Only one clustering index possible per file (since a file can be physically sorted only one way).
- **Secondary Index** — built on a non-ordering field. Must have one entry per RECORD (dense), because the file is NOT sorted on this field, so you can't skip records. A file can have many secondary indexes.
- **Dense Index** — has an index entry for EVERY search key value (record) in the file.
- **Sparse Index** — has an index entry for only SOME of the search key values (typically one per block) — requires the data file to be sorted.
- **Multilevel Index** — an index on an index; used when the first-level index itself becomes too large to fit in memory. Reduces the number of disk I/Os needed.

### 1.6.2 B-Tree and B+ Tree — Extremely High Priority Topic

Both are **balanced, multi-way search trees** used almost universally for database indexing (and file systems) because they keep the tree height very small (logarithmic) even with millions of records, which minimizes disk I/O — and they self-balance on insert/delete so you never need to manually "re-sort."

**B-Tree:**
- Data (actual records or pointers to records) can be stored in BOTH internal (non-leaf) nodes AND leaf nodes.
- Each node can have multiple keys and multiple children — an order-m B-tree node has at most m children and at most m−1 keys.
- No duplicate storage of keys, so slightly more storage-efficient for point lookups.
- No linked list among leaves — makes range queries harder.

**B+ Tree (the one actually used in almost all real RDBMS — MySQL InnoDB, PostgreSQL, Oracle):**
- ALL actual data (or data pointers) are stored ONLY in the LEAF nodes. Internal nodes store only keys used for navigation/routing — they act purely as an index to guide the search to the correct leaf.
- Leaf nodes are linked together in a **linked list**, which makes **range queries and sequential/ordered scans very efficient** (once you find the start, just walk the linked list) — this is the single biggest practical reason B+ trees are preferred over B-trees in databases.
- Since internal nodes don't store actual data, they can hold MORE keys per node than a B-tree node of the same size → shorter tree → fewer disk accesses → faster.

**Height/search complexity:** both B-tree and B+ tree searches, inserts, and deletes run in O(log n) time, where the base of the log is the branching factor (order) of the tree — much shallower than a binary search tree for the same number of keys, because each node can hold many keys/children (not just 2).

**Exam trap seen in the sample paper (Q14)**: "Best data structure for database indexing?" → Answer is B-tree, but the more precise/modern answer that appears in most current RDBMS documentation is **B+ tree**, specifically because of range-query efficiency via the linked leaf list. If SEBI gives both as options, pick based on the exact wording — "used in database indexing generally" → B-tree (broader/older term used loosely); "best for range queries in RDBMS" → B+ tree.

- **Hashing-based indexing** (as an alternative to tree-based): **Static hashing** uses a fixed number of buckets — problems arise when data grows beyond the fixed capacity (overflow chaining needed). **Dynamic hashing** (extendible hashing, linear hashing) grows the bucket structure as data grows, avoiding a full rebuild.

## 1.7 Transactions and Concurrency Control

### 1.7.1 What Is a Transaction

A **transaction** is a single logical unit of work that accesses/modifies database contents, and must be executed as an all-or-nothing operation. Classic example: transferring money from Account A to Account B involves two updates (debit A, credit B) — both must succeed, or neither should take effect.

### 1.7.2 ACID Properties — Guaranteed to Be Asked

- **Atomicity** — "all or nothing." Either all operations of the transaction complete, or none do (rolled back on failure).
- **Consistency** — a transaction takes the database from one valid (consistent) state to another valid state, preserving all defined rules/constraints.
- **Isolation** — concurrently executing transactions should not interfere with each other; each transaction should appear to execute as if it were the only one running.
- **Durability** — once a transaction is committed, its changes are permanent, even if the system crashes immediately afterward (typically achieved via write-ahead logging).

### 1.7.3 Transaction States

`Active → Partially Committed → Committed` (success path), or `Active → Failed → Aborted` (failure path). A transaction can also go from Aborted back to Active if it's restarted.

### 1.7.4 Schedules

A **schedule** is the chronological order of execution of operations from multiple transactions.
- **Serial Schedule** — transactions execute one completely after another, no interleaving. Always consistent, but no concurrency (slow).
- **Concurrent (Non-serial) Schedule** — operations from different transactions are interleaved. Needed for performance, but can cause problems (see below) if not controlled.
- **Serializable Schedule** — a concurrent schedule whose EFFECT is equivalent to SOME serial schedule of the same transactions. This is the goal: get the performance of concurrency with the safety of serial execution.
  - **Conflict Serializability** — a schedule is conflict-serializable if it can be transformed into a serial schedule by swapping only NON-CONFLICTING adjacent operations (two operations conflict if they belong to different transactions, access the same data item, and at least one of them is a WRITE). Tested via a **precedence graph** — if the graph has NO cycle, the schedule is conflict-serializable.
  - **View Serializability** — a weaker/broader condition; a schedule is view-serializable if it's "view equivalent" to some serial schedule (same initial reads, same final writes, same read-from relationships). Every conflict-serializable schedule is view-serializable, but not vice versa.

### 1.7.5 Concurrency Problems (What Happens Without Control)

- **Dirty Read (WR conflict)** — a transaction reads data written by another transaction that has NOT yet committed (and might later roll back), leading to reading invalid/uncommitted data.
- **Lost Update (WW conflict)** — two transactions both read the same data, then both write back updated values; one transaction's update overwrites (and thus loses) the other's.
- **Unrepeatable Read (RW conflict)** — a transaction reads the same row twice within itself, and gets a DIFFERENT value the second time because another transaction updated and committed that row in between.
- **Phantom Read** — a transaction re-runs a query with a WHERE condition and gets a DIFFERENT SET OF ROWS the second time because another transaction inserted/deleted rows matching the condition in between.

### 1.7.6 Concurrency Control Techniques

- **Lock-Based Protocols**:
  - **Shared Lock (S / Read lock)** — multiple transactions can hold a shared lock on the same item simultaneously (for reading).
  - **Exclusive Lock (X / Write lock)** — only one transaction can hold an exclusive lock on an item; no other transaction can hold ANY lock (shared or exclusive) on it at the same time.
  - **Two-Phase Locking (2PL)** — every transaction has a **Growing Phase** (can only ACQUIRE locks, never release) followed by a **Shrinking Phase** (can only RELEASE locks, never acquire). Once a transaction releases its first lock, it cannot obtain any new lock. 2PL guarantees conflict serializability.
  - **Strict 2PL** — all exclusive (write) locks are held until the transaction commits/aborts (i.e., released only at the very end) — prevents dirty reads. This is what almost all real databases actually implement.
- **Deadlock** — occurs when two or more transactions are each waiting for a lock held by the other, forming a cycle, so none can proceed. Handled via:
  - **Deadlock Prevention** — ensure the system can never enter a deadlock state (e.g., Wait-Die and Wound-Wait schemes, which use transaction timestamps to decide whether an older transaction waits or the younger one is aborted).
  - **Deadlock Detection & Recovery** — allow deadlocks to occur, periodically build a **wait-for graph**, and if a cycle is found, abort one of the transactions in the cycle (the "victim") to break it.
- **Timestamp-Based Protocols** — every transaction gets a unique timestamp when it starts; conflicts are resolved by comparing timestamps to ensure the equivalent serial order matches timestamp order — no locks needed, no deadlocks possible (but can cause more aborts/restarts).
- **Optimistic Concurrency Control (Validation-based)** — assume conflicts are rare; let transactions execute freely, and only check for conflicts at commit time (validation phase); if a conflict is found, abort and retry. Good when conflicts are genuinely rare (low contention).
- **Multiversion Concurrency Control (MVCC)** — instead of locking, keep multiple versions of a data item (each write creates a new version); reads can access an older, consistent version without blocking writers, and vice versa. Used heavily in modern databases (PostgreSQL, MySQL InnoDB, Oracle) to give high read concurrency.

### 1.7.7 Isolation Levels (SQL Standard)

From weakest (fastest, least safe) to strongest (slowest, safest):

| Isolation Level | Dirty Read | Unrepeatable Read | Phantom Read |
|---|---|---|---|
| Read Uncommitted | Possible | Possible | Possible |
| Read Committed | Prevented | Possible | Possible |
| Repeatable Read | Prevented | Prevented | Possible |
| Serializable | Prevented | Prevented | Prevented |

**Exam trap**: people often think "Repeatable Read" prevents phantom reads too — it does NOT in the strict standard definition (though some databases like MySQL InnoDB's implementation of Repeatable Read does happen to also prevent phantoms via gap locking — but for the pure SQL-standard theory answer, use the table above).

### 1.7.8 Recovery Techniques

- **Log-based recovery** — every change is first written to a log (Write-Ahead Logging, WAL) before being applied to the actual database, so the system can recover to a consistent state after a crash by replaying/undoing logged operations.
  - **UNDO** — reverse the effects of uncommitted transactions.
  - **REDO** — re-apply the effects of committed transactions that might not have made it to disk before a crash.
- **Checkpointing** — periodically save the current consistent state so recovery doesn't need to replay the ENTIRE log from the beginning, only from the last checkpoint.
- **Shadow Paging** — an alternative to logging; maintains two page tables (current and shadow); on commit, the current table becomes the new shadow table.

---

# SECTION 1 QUICK REVISION TABLE

| Concept | One-line takeaway |
|---|---|
| Candidate key | Minimal super key |
| M:N relationship | Needs a separate junction table with a composite key |
| Weak entity | Has no PK of its own; needs owner's PK + partial key |
| 1NF | Atomic values |
| 2NF | No partial dependency (only matters with composite PK) |
| 3NF | No transitive dependency |
| BCNF | Every determinant is a super key |
| B+ Tree | Data only in leaves, leaves linked — best for range queries |
| ACID | Atomicity, Consistency, Isolation, Durability |
| 2PL | Growing phase (acquire only) then shrinking phase (release only) |
| Serializable isolation | Prevents dirty read, unrepeatable read, AND phantom read |


---

# SECTION 2: SQL QUERIES

## 2.1 SQL Command Categories

SQL commands are grouped into categories — SEBI likes to ask "which category does X belong to":

- **DDL (Data Definition Language)** — defines/modifies structure: `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME`. DDL commands are **auto-committed** in most databases (cannot be rolled back).
- **DML (Data Manipulation Language)** — manipulates data: `SELECT`, `INSERT`, `UPDATE`, `DELETE`. (Note: some textbooks classify SELECT separately as DQL — Data Query Language — SEBI may test either convention, know both.)
- **DCL (Data Control Language)** — controls access/permissions: `GRANT`, `REVOKE`.
- **TCL (Transaction Control Language)** — manages transactions: `COMMIT`, `ROLLBACK`, `SAVEPOINT`.

**Trap: DELETE vs TRUNCATE vs DROP**

| Command | Type | Removes | Rollback possible? | Resets identity/auto-increment? | Fires triggers? |
|---|---|---|---|---|---|
| `DELETE` | DML | Selected rows (WHERE optional) | Yes (before commit) | No | Yes |
| `TRUNCATE` | DDL | ALL rows, structure stays | No (auto-committed in most DBs) | Yes | No |
| `DROP` | DDL | Entire table (structure + data) | No | N/A (table gone) | No |

## 2.2 Basic SELECT Syntax and Clause Order

```sql
SELECT column1, column2, AGG_FUNC(column3)
FROM table1
JOIN table2 ON condition
WHERE row_condition
GROUP BY column1, column2
HAVING group_condition
ORDER BY column1 [ASC|DESC]
LIMIT n;
```

**Very important trap: the WRITTEN order and the LOGICAL EXECUTION order of clauses are different.** SEBI loves testing this.

Logical execution order (what actually happens first):
1. `FROM` (and `JOIN`s) — build the working table
2. `WHERE` — filter individual rows (row-level filter, BEFORE grouping)
3. `GROUP BY` — form groups
4. `HAVING` — filter GROUPS (not individual rows) — can use aggregate functions, WHERE cannot
5. `SELECT` — pick/compute the final columns
6. `DISTINCT` — remove duplicate result rows
7. `ORDER BY` — sort the final result
8. `LIMIT`/`OFFSET` — restrict number of rows returned

**Why this matters**: you CANNOT use a column alias defined in SELECT inside the WHERE clause (because WHERE executes before SELECT logically) — but you CAN use it in ORDER BY (which executes after SELECT). Similarly, you cannot use an aggregate function like `COUNT(*)` directly in WHERE — you must use HAVING, because WHERE runs before grouping happens.

## 2.3 WHERE vs HAVING — Guaranteed Exam Question

- `WHERE` filters individual ROWS, BEFORE any grouping. Cannot use aggregate functions (SUM, COUNT, AVG, etc.) in WHERE.
- `HAVING` filters GROUPS, AFTER grouping (GROUP BY) has happened. Can use aggregate functions.
- If there's no GROUP BY, HAVING treats the whole table as one group.

```sql
-- Find departments with more than 5 employees earning above 30000
SELECT dept_id, COUNT(*) as emp_count
FROM employee
WHERE salary > 30000        -- row filter first
GROUP BY dept_id
HAVING COUNT(*) > 5;        -- group filter after
```

## 2.4 Aggregate Functions

`COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`.

**Trap: `COUNT(*)` vs `COUNT(column)` vs `COUNT(DISTINCT column)`**
- `COUNT(*)` — counts ALL rows, including rows with NULLs in any column.
- `COUNT(column_name)` — counts only rows where that specific column is NOT NULL.
- `COUNT(DISTINCT column_name)` — counts only unique non-NULL values.

**Trap: NULL and aggregate functions** — all aggregate functions (SUM, AVG, MIN, MAX, COUNT(column)) IGNORE NULL values entirely — they do not treat NULL as zero. So `AVG(marks)` for rows (80, NULL, 60) = (80+60)/2 = 70, NOT (80+0+60)/3.

## 2.5 JOINS — Extremely High Priority

### 2.5.1 Inner Join

Returns only rows that have MATCHING values in BOTH tables.

```sql
SELECT e.name, d.dept_name
FROM employee e
INNER JOIN department d ON e.dept_id = d.dept_id;
```

### 2.5.2 Outer Joins

- **LEFT (OUTER) JOIN** — returns ALL rows from the LEFT table, plus matched rows from the right table; unmatched right-side columns become NULL.
- **RIGHT (OUTER) JOIN** — mirror of LEFT — all rows from the RIGHT table, unmatched left-side columns become NULL.
- **FULL (OUTER) JOIN** — returns all rows from BOTH tables; unmatched columns on either side become NULL. (Not natively supported in MySQL — must simulate with `LEFT JOIN UNION RIGHT JOIN`.)

**Trick to find "unmatched only" rows** (very common SEBI-style query): to find employees with NO department assigned:
```sql
SELECT e.name
FROM employee e
LEFT JOIN department d ON e.dept_id = d.dept_id
WHERE d.dept_id IS NULL;
```

### 2.5.3 Self Join

A table joined with itself, using aliases to distinguish the two "copies." Classic use: finding an employee's manager (who is also in the same employee table).
```sql
SELECT e.name AS employee, m.name AS manager
FROM employee e
LEFT JOIN employee m ON e.manager_id = m.emp_id;
```

### 2.5.4 Cross Join

Cartesian product — every row of table A combined with every row of table B, no join condition. Rarely intentional in exams but tests understanding of Cartesian product from relational algebra (Section 1.3.3).

### 2.5.5 Natural Join

Automatically joins on all columns with the SAME NAME in both tables, and includes that shared column only ONCE in the output (unlike a regular JOIN...ON which keeps both copies unless you specify columns).

## 2.6 Set Operations: UNION, UNION ALL, INTERSECT, EXCEPT/MINUS

All of these require the two SELECT queries to have the SAME NUMBER OF COLUMNS with COMPATIBLE (matching) DATA TYPES, in the same order.

- **UNION** — combines results of two queries and REMOVES DUPLICATES (internally does a sort/hash to dedupe — slightly slower).
- **UNION ALL** — combines results and KEEPS duplicates (faster, since no dedup step).
- **INTERSECT** — returns only rows that appear in BOTH result sets.
- **EXCEPT** (SQL Server/PostgreSQL) / **MINUS** (Oracle) — returns rows from the FIRST query that do NOT appear in the second query result.

```sql
SELECT city FROM customers
UNION
SELECT city FROM suppliers;
```

## 2.7 Subqueries (Nested Queries)

A query inside another query. Can appear in SELECT, FROM, WHERE, or HAVING clauses.

- **Scalar subquery** — returns exactly ONE value (one row, one column). Can be used anywhere a single value is expected.
- **Row subquery** — returns one row with multiple columns.
- **Table subquery** — returns multiple rows/columns; commonly used with `IN`, `EXISTS`, `ANY`, `ALL`, or in the `FROM` clause (called a "derived table" or "inline view").
- **Correlated subquery** — the inner query REFERENCES a column from the outer query, so it must be re-evaluated once for EACH row processed by the outer query (much slower than a non-correlated subquery, which is evaluated only ONCE).

```sql
-- Non-correlated subquery: employees earning more than average
SELECT name FROM employee
WHERE salary > (SELECT AVG(salary) FROM employee);

-- Correlated subquery: employees earning more than the average of their OWN department
SELECT name FROM employee e1
WHERE salary > (SELECT AVG(salary) FROM employee e2 WHERE e2.dept_id = e1.dept_id);
```

### 2.7.1 IN / NOT IN

Checks if a value matches ANY value in a list/subquery result.
```sql
SELECT name FROM employee WHERE dept_id IN (SELECT dept_id FROM department WHERE location = 'Mumbai');
```
**Trap**: `NOT IN` with a subquery that can return NULL is dangerous — if the subquery result contains even ONE NULL, the entire `NOT IN` comparison returns UNKNOWN for every row, and the outer query returns ZERO rows (because SQL's three-valued logic means `x <> NULL` is UNKNOWN, and ANDing UNKNOWN with anything in a NOT IN's implicit AND-chain across all list values kills the result). Always filter out NULLs in the subquery (`WHERE column IS NOT NULL`) when using `NOT IN`, or use `NOT EXISTS` instead (which does not have this problem).

### 2.7.2 EXISTS / NOT EXISTS

Checks whether the subquery returns ANY row at all (returns TRUE/FALSE, doesn't care about the actual values) — often more efficient than IN for large subqueries because the database can stop as soon as ONE matching row is found.
```sql
SELECT name FROM employee e
WHERE EXISTS (SELECT 1 FROM department d WHERE d.dept_id = e.dept_id AND d.location = 'Mumbai');
```

### 2.7.3 ANY / SOME / ALL

- `x > ANY (subquery)` — true if x is greater than AT LEAST ONE value returned by the subquery (equivalent to greater than the MINIMUM).
- `x > ALL (subquery)` — true if x is greater than EVERY value returned (equivalent to greater than the MAXIMUM).
- `SOME` is a synonym for `ANY` in standard SQL.

**This connects directly back to relational algebra's Division operator (Section 1.3.3)** — "find X related to ALL of Y" style queries can be written using `= ALL` or `NOT EXISTS` with a double-negative structure (find X such that there is no Y that X is NOT related to).

## 2.8 DDL Commands in Detail

```sql
CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    dept_id INT,
    salary DECIMAL(10,2) CHECK (salary > 0),
    FOREIGN KEY (dept_id) REFERENCES department(dept_id)
);

ALTER TABLE employee ADD COLUMN email VARCHAR(100);
ALTER TABLE employee DROP COLUMN email;
ALTER TABLE employee MODIFY COLUMN name VARCHAR(100);  -- MySQL syntax
ALTER TABLE employee RENAME COLUMN name TO full_name;

DROP TABLE employee;         -- removes table and data permanently
TRUNCATE TABLE employee;     -- removes all rows, keeps structure
```

### 2.8.1 Constraints

- `PRIMARY KEY` — unique + not null, one per table (can be composite).
- `FOREIGN KEY` — referential integrity link to another table's primary/unique key.
- `UNIQUE` — all values in the column must be distinct, but NULL is allowed (and multiple NULLs are typically allowed, unlike PRIMARY KEY).
- `NOT NULL` — column cannot store NULL.
- `CHECK` — enforces a custom boolean condition on the column's values.
- `DEFAULT` — provides a default value when none is specified during insert.

## 2.9 DML Commands in Detail

```sql
INSERT INTO employee (emp_id, name, dept_id, salary) VALUES (1, 'Ramesh', 10, 45000);

UPDATE employee SET salary = salary * 1.1 WHERE dept_id = 10;

DELETE FROM employee WHERE emp_id = 1;
```

## 2.10 Views

A **view** is a virtual table based on the result of a stored SQL query — it doesn't store data itself (in most cases; "materialized views" DO store data physically and need periodic refresh).

```sql
CREATE VIEW high_earners AS
SELECT name, salary FROM employee WHERE salary > 50000;

SELECT * FROM high_earners;   -- treated just like a table
```

- Views can simplify complex/repeated queries, and can be used to restrict access (show only certain columns/rows to certain users) — a security mechanism.
- **Updatable views**: a view is generally updatable (INSERT/UPDATE/DELETE through it affects the base table) only if it's based on a SINGLE table, has no aggregate functions, no GROUP BY/HAVING/DISTINCT, and includes all NOT NULL columns of the base table (for inserts).

## 2.11 GROUP BY In Depth

```sql
SELECT dept_id, AVG(salary), MAX(salary)
FROM employee
GROUP BY dept_id;
```

**Rule**: every column in the SELECT list that is NOT wrapped in an aggregate function MUST appear in the GROUP BY clause (strict SQL mode enforces this; MySQL's default mode used to be lenient about this but it's considered bad practice and modern MySQL with `ONLY_FULL_GROUP_BY` enforces it too).

`GROUP BY` can use multiple columns — groups are formed based on the combination of all listed columns.

## 2.12 ORDER BY

```sql
SELECT name, salary FROM employee ORDER BY salary DESC, name ASC;
```
Default is `ASC`. Multiple columns: sorts by the first column, and uses the second column only to break ties within equal values of the first.

**NULL ordering**: in most databases (PostgreSQL, Oracle), NULLs sort LAST in `ASC` order and FIRST in `DESC` order by default — but MySQL treats NULL as the SMALLEST value, so NULLs come FIRST in `ASC` and LAST in `DESC`. This is a genuinely tricky, database-specific trap.

## 2.13 String, Date, and NULL Handling in SQL

- `LIKE` — pattern matching. `%` = any sequence of characters (including zero), `_` = exactly one character.
  ```sql
  SELECT * FROM employee WHERE name LIKE 'A%';    -- starts with A
  SELECT * FROM employee WHERE name LIKE '_a%';   -- second letter is 'a'
  ```
- `IS NULL` / `IS NOT NULL` — the ONLY correct way to test for NULL. `column = NULL` is ALWAYS false/unknown (NULL is not "equal" to anything, not even itself) — this is one of the most repeated SQL traps in every exam.
- `COALESCE(a, b, c, ...)` — returns the first non-NULL value in the list. `IFNULL(a, b)` (MySQL) / `NVL(a, b)` (Oracle) do the same for exactly two arguments.
- `BETWEEN a AND b` — inclusive range check (a <= x <= b).

## 2.14 SQL Execution Order Trap — Worked Full Example

```sql
SELECT dept_id, COUNT(*) AS cnt
FROM employee
WHERE salary > 20000
GROUP BY dept_id
HAVING COUNT(*) > 2
ORDER BY cnt DESC;
```
Step by step logical execution:
1. `FROM employee` — get the full table.
2. `WHERE salary > 20000` — keep only rows where salary is above 20000 (row-level, before grouping).
3. `GROUP BY dept_id` — form groups by department.
4. `HAVING COUNT(*) > 2` — keep only groups (departments) that have MORE than 2 qualifying employees.
5. `SELECT dept_id, COUNT(*) AS cnt` — compute the final output columns; `cnt` alias is created here.
6. `ORDER BY cnt DESC` — since ORDER BY runs AFTER SELECT, it CAN use the `cnt` alias.

## 2.15 Indexes and Query Performance (SQL-level)

```sql
CREATE INDEX idx_salary ON employee(salary);
CREATE UNIQUE INDEX idx_email ON employee(email);
```
Indexes speed up `WHERE`, `JOIN`, and `ORDER BY` operations on the indexed column(s), at the cost of slower `INSERT`/`UPDATE`/`DELETE` (since the index must also be updated) and extra storage. Connects directly to Section 1.6 (B+ Trees).

---

# SECTION 2 QUICK REVISION TABLE

| Trap | Correct understanding |
|---|---|
| WHERE vs HAVING | WHERE filters rows before grouping (no aggregates); HAVING filters groups (aggregates OK) |
| `column = NULL` | Always false/unknown — must use `IS NULL` |
| `COUNT(*)` vs `COUNT(col)` | `COUNT(*)` counts all rows; `COUNT(col)` ignores NULLs in that column |
| `NOT IN` with NULLs in subquery | Returns zero rows — use `NOT EXISTS` instead |
| `UNION` vs `UNION ALL` | UNION removes duplicates (slower); UNION ALL keeps them (faster) |
| Alias usable in ORDER BY but not WHERE | Because SELECT executes before ORDER BY, but after WHERE |
| Correlated vs non-correlated subquery | Correlated re-runs per outer row; non-correlated runs once |
| DELETE vs TRUNCATE vs DROP | DELETE=DML/rollback-able; TRUNCATE=DDL/removes all rows; DROP=removes whole table |
| M:N in SQL | Needs a junction/bridge table |


---

# SECTION 3: PROGRAMMING CONCEPTS (Java / C / C++)

This is the **single highest-weightage topic in Phase I (30%)**. It overlaps heavily with Phase II's OOP section (Section 13), so mastering this pays double.

## 3.1 Program Control Structures

### 3.1.1 Iteration (Loops)

All three languages support `for`, `while`, `do-while`.

```c
// for loop
for (int i = 0; i < 5; i++) { printf("%d", i); }

// while loop - condition checked BEFORE each iteration
int i = 0;
while (i < 5) { printf("%d", i); i++; }

// do-while loop - body executes AT LEAST ONCE, condition checked AFTER
int i = 0;
do { printf("%d", i); i++; } while (i < 5);
```

**Trap**: `do-while` always executes the loop body at least once, even if the condition is false from the start. This is the #1 tested difference between `while` and `do-while`.

**Loop control statements:**
- `break` — exits the loop immediately (or switch statement).
- `continue` — skips the rest of the current iteration and jumps to the next one (condition check).
- In nested loops, `break`/`continue` (in C/C++/Java) affect only the INNERMOST loop they're written in — there's no native "labeled break to outer loop" in C/C++, but Java DOES support labeled break/continue:
```java
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) break outer;  // breaks the OUTER loop, Java-only feature
    }
}
```

### 3.1.2 Recursion

A function that calls itself, with a **base case** (stopping condition) and a **recursive case** that moves toward the base case.

```python
def factorial(n):
    if n == 0:          # base case
        return 1
    return n * factorial(n-1)   # recursive case
```

**Key concepts:**
- Every recursive call is pushed onto the **call stack** (this is WHY Q50 in the sample paper says "recursion internally uses a stack").
- **Stack overflow** occurs if recursion goes too deep (base case never reached, or too many levels for available memory).
- **Tail recursion** — the recursive call is the LAST operation in the function (nothing left to do after it returns). Many languages/compilers can optimize tail recursion into a loop internally (tail call optimization) to avoid stack growth — but note that standard Java and C (without special compiler flags) do NOT guarantee this optimization.
- **Types**: Direct recursion (function calls itself directly) vs Indirect/Mutual recursion (function A calls function B, which calls function A back).

**Worked trace — factorial(4):**
```
factorial(4) = 4 * factorial(3)
factorial(3) = 3 * factorial(2)
factorial(2) = 2 * factorial(1)
factorial(1) = 1 * factorial(0)
factorial(0) = 1   <- base case reached, stack starts unwinding
factorial(1) = 1*1 = 1
factorial(2) = 2*1 = 2
factorial(3) = 3*2 = 6
factorial(4) = 4*6 = 24
```

### 3.1.3 Functions

- **Function declaration/prototype** vs **definition** (C/C++ specific) — a prototype tells the compiler the function's signature (return type, name, parameter types) before it's actually defined, allowing calls before the definition appears in the file.
- **Function overloading** (C++/Java) — multiple functions with the SAME NAME but DIFFERENT parameter lists (different number and/or types of parameters). Resolved at COMPILE TIME (this is a form of **compile-time/static polymorphism**). Return type ALONE is not enough to overload — parameter list must differ.
- C does NOT support function overloading (no polymorphism support in C at all, since C is not object-oriented).

## 3.2 Scope of Variables

- **Local variable** — declared inside a function/block; only accessible within that function/block; created when the block is entered, destroyed when it exits.
- **Global variable** — declared outside all functions; accessible throughout the entire program (all functions can read/modify it, unless shadowed).
- **Block scope** — a variable declared inside `{ }` (e.g., inside an `if` or `for`) is only visible within that block.
- **Static local variable** (`static int x;` inside a function, in C/C++) — retains its value BETWEEN function calls (initialized only once, memory persists for the program's lifetime, but scope is still limited to that function).
- **Shadowing** — a local variable with the SAME NAME as a global variable "hides" the global one within that local scope; the global variable is still accessible via scope resolution (`::` in C++, or by not shadowing in the first place — Java has no true global variables, only class-level static fields).

```c
int x = 10;  // global
void foo() {
    int x = 20;  // local, shadows global x within foo()
    printf("%d", x);  // prints 20
}
```

## 3.3 Binding of Variables & Functions

- **Static (early) binding** — the compiler resolves WHICH function/variable to use at COMPILE TIME. Applies to: normal function calls, overloaded functions, static methods, private methods, and variables in general (variable resolution is always static/compile-time based on declared type in Java).
- **Dynamic (late) binding** — the decision of WHICH function to call is deferred to RUN TIME, based on the ACTUAL object type (not the declared/reference type). This is how **method overriding / runtime polymorphism** works — achieved via **virtual functions** in C++ (must be explicitly marked `virtual`) and is the DEFAULT behavior for all non-static, non-final, non-private instance methods in Java.

**This connects directly to sample paper Q15**: "Static parent method — what happens when called from child?" → static methods are resolved by STATIC/early binding based on the REFERENCE TYPE, not dynamic binding — so calling a static method through a child-class reference that has a same-named static method still depends on the compile-time type of the reference — this is called **method hiding**, NOT overriding, because overriding requires dynamic dispatch, which static methods don't participate in.

## 3.4 Parameter Passing

### 3.4.1 Pass by Value

A COPY of the argument's value is passed to the function. Changes made to the parameter INSIDE the function do NOT affect the original variable outside. This is the default in C, C++ (unless you use pointers/references), and Java (Java is ALWAYS pass-by-value — even for objects, what's passed by value is the reference/memory-address VALUE itself, which is why mutating an object's fields through that reference DOES affect the original object, but REASSIGNING the parameter to point to a new object does NOT affect the caller's reference).

```java
void modify(int x) { x = 100; }   // does NOT change the caller's variable
void modifyArr(int[] arr) { arr[0] = 100; }  // DOES change caller's array content
                                              // (because arr is a copy of the reference, but points to the same array object)
void reassign(int[] arr) { arr = new int[]{9,9,9}; } // does NOT change caller's array reference
```

### 3.4.2 Pass by Reference

The function receives a REFERENCE (alias/address) to the original variable, so changes inside the function DIRECTLY affect the caller's variable. 
- **C** — does not have true pass-by-reference; you simulate it by explicitly passing a POINTER (the address of a variable), and the function must dereference the pointer to modify the original.
- **C++** — has TRUE pass-by-reference using the `&` symbol in the parameter list: `void swap(int &a, int &b)`.
- **Java** — does NOT support pass-by-reference at all, in the true sense. (This is a very commonly misunderstood/tested point — "Java passes objects by reference" is a common WRONG belief; the correct statement is "Java passes object REFERENCES by VALUE.")

```cpp
void swap(int &a, int &b) {   // C++ true reference
    int temp = a; a = b; b = temp;
}
void swap(int *a, int *b) {   // C pointer simulation
    int temp = *a; *a = *b; *b = temp;
}
```

## 3.5 Functional and Logic Programming (Paradigm Awareness)

SEBI expects awareness of programming paradigms, not deep implementation:

- **Imperative programming** — describes HOW to do something, step by step, via statements that change program state (C, early Java code).
- **Declarative programming** — describes WHAT the result should be, not how to compute it (SQL is a classic declarative language).
- **Functional programming** — treats computation as evaluation of mathematical functions, avoids changing state/mutable data, favors pure functions (same input always gives same output, no side effects), first-class functions (can be passed as arguments, returned from other functions), and immutability. Examples of concepts: `map`, `filter`, `reduce`, lambda expressions. Languages: Haskell (pure), also supported in Python, Java 8+ (lambdas/streams), JavaScript.
- **Logic programming** — programs are expressed as a set of logical facts and rules; the system derives conclusions via logical inference (e.g., Prolog). You state relationships/facts, and QUERY the system to deduce answers, rather than specifying step-by-step instructions.

```python
# functional style in Python
nums = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x**2, nums))
evens = list(filter(lambda x: x % 2 == 0, nums))
total = 0
from functools import reduce
total = reduce(lambda a, b: a + b, nums)
```

## 3.6 OOP Concepts — Very High Priority (also see Section 13 for deeper Phase II treatment)

### 3.6.1 The Four Pillars

1. **Encapsulation** — bundling data (fields/attributes) and the methods that operate on that data into a single unit (a class), and RESTRICTING direct access to some of the object's internal state (typically via `private` fields + `public` getter/setter methods). Protects data integrity. (Sample paper Q29 directly tests this: "data and methods placed together" = Encapsulation.)
2. **Abstraction** — hiding complex implementation details and exposing only the essential/relevant features to the user. Achieved via abstract classes and interfaces. (Sample paper Q18 tests the negative: "data compilation" is NOT an OOP property — the four real ones are Encapsulation, Abstraction, Inheritance, Polymorphism.)
3. **Inheritance** — a mechanism where a new class (child/derived/subclass) acquires the properties and behaviors (fields and methods) of an existing class (parent/base/superclass), promoting code reuse.
4. **Polymorphism** — "many forms" — the ability of an object/method/operator to behave differently based on context. Two kinds:
   - **Compile-time (static) polymorphism** — method overloading, operator overloading (C++ only, Java doesn't support user-defined operator overloading).
   - **Runtime (dynamic) polymorphism** — method overriding, achieved via virtual functions (C++) / default dynamic dispatch (Java).

### 3.6.2 Class and Object

- **Class** — a blueprint/template that defines the structure (fields) and behavior (methods) that its objects will have. No memory is allocated just by defining a class.
- **Object** — a concrete INSTANCE of a class, created in memory (usually on the heap), with actual values for its fields.

```java
class Car {
    String brand;
    int speed;
    void accelerate() { speed += 10; }
}
Car myCar = new Car();   // 'myCar' is an object (instance) of class Car
```

### 3.6.3 Constructors

A **constructor** is a special method automatically invoked when an object is created, used to initialize the object's fields.
- Has the SAME NAME as the class (in Java/C++); has NO return type (not even `void`).
- **Default constructor** — provided automatically by the compiler ONLY if you don't define ANY constructor yourself; takes no arguments and typically initializes fields to default values (0, null, false, etc.).
- **Parameterized constructor** — explicitly accepts arguments to initialize fields with specific values.
- **Copy constructor** (C++ specific concept, though the term is used loosely elsewhere too) — creates a new object as a copy of an existing object of the same class.
- **Constructor overloading** — a class can have multiple constructors with different parameter lists (compile-time polymorphism).
- **Destructor** (C++ only concept as an explicit language feature: `~ClassName()`) — automatically called when an object is destroyed/goes out of scope, used for cleanup (freeing memory, closing files). Java does NOT have destructors — it has automatic **garbage collection**, and `finalize()` (deprecated/removed in newer Java versions) was never a guaranteed-timing equivalent.

**In Python specifically** (relevant to sample Q67): `__init__` is the initializer, automatically called right after object creation, to set up initial state — this is what most people call "the constructor" in Python, though technically `__new__` is what actually CREATES the object (allocates memory) and `__init__` just INITIALIZES the already-created object.

### 3.6.4 Inheritance Types

- **Single Inheritance** — one child class inherits from exactly one parent class.
- **Multiple Inheritance** — a child class inherits from MORE THAN ONE parent class SIMULTANEOUSLY. Supported by C++ (using multiple base classes in the class declaration) and Python. **NOT supported by Java or C#** for classes (to avoid the "Diamond Problem" — ambiguity when two parent classes have a method with the same signature). Java/C# instead allow a class to implement MULTIPLE INTERFACES, which gives similar flexibility without the ambiguity (since interfaces traditionally didn't have implementation, though Java 8's default methods reintroduced a limited diamond-problem risk, resolved by forcing the implementing class to explicitly override the conflicting method).
- **Multilevel Inheritance** — a chain: class C inherits from class B, which inherits from class A (grandparent → parent → child).
- **Hierarchical Inheritance** — ONE base/parent class is inherited by MULTIPLE child classes independently. (Sample paper Q47 tests this exact definition.)
- **Hybrid Inheritance** — a combination of two or more inheritance types (e.g., hierarchical + multilevel together).

**The Diamond Problem** (why Java disallows multiple class inheritance): if class B and class C both inherit from class A, and class D inherits from BOTH B and C, then D might inherit TWO different versions/copies of A's members through B and C — creating ambiguity about which version to use. C++ solves this with **virtual inheritance** (`class B : virtual public A`), which ensures only ONE shared copy of A exists in D.

### 3.6.5 Access Modifiers / Access Specifiers

| Modifier | Same class | Same package/file (no inheritance) | Subclass (different package) | Outside/anywhere |
|---|---|---|---|---|
| `private` | Yes | No | No | No |
| default/package-private (Java, no keyword) | Yes | Yes | No | No |
| `protected` | Yes | Yes | Yes | No |
| `public` | Yes | Yes | Yes | Yes |

C++ access specifiers work slightly differently in the context of inheritance — a `class` defaults all members to `private` and inheritance to `private` by default, while a `struct` defaults everything to `public`. This is a subtle but real difference between `class` and `struct` in C++ (in C, `struct` has no access control at all — it's just a plain data grouping).

### 3.6.6 Exception Handling

An **exception** is a runtime event that disrupts the normal flow of a program's execution (e.g., division by zero, array index out of bounds, null pointer access, file not found).

```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Cannot divide by zero: " + e.getMessage());
} finally {
    System.out.println("This always executes, exception or not");
}
```

**Key components:**
- `try` block — contains code that might throw an exception.
- `catch` block — handles a specific type of exception; you can have MULTIPLE catch blocks for different exception types, and they are checked in order (most specific exception type should come FIRST, before more general ones like the base `Exception` class — otherwise the general catch will "shadow" the specific ones, and in Java this is actually a COMPILE ERROR if an earlier catch block already covers a later, more specific exception type via inheritance).
- `finally` block — ALWAYS executes, whether an exception occurred or not, and whether it was caught or not (even if there's a `return` statement inside try/catch) — used for cleanup (closing files, releasing resources). The ONLY way `finally` doesn't execute is if the JVM itself exits abruptly (`System.exit()`) or the thread is killed.
- `throw` — used to explicitly THROW/raise an exception.
- `throws` (Java-specific, in method signature) — declares that a method MIGHT throw a certain checked exception, so callers are forced to handle it.

**Checked vs Unchecked exceptions (Java-specific distinction):**
- **Checked exceptions** — checked by the COMPILER at compile time; the code will not compile unless you either catch them or declare them with `throws`. Examples: `IOException`, `SQLException`. These extend `Exception` but NOT `RuntimeException`.
- **Unchecked exceptions (Runtime exceptions)** — NOT checked at compile time; the compiler doesn't force you to handle them (though you still can). Examples: `ArithmeticException`, `NullPointerException`, `ArrayIndexOutOfBoundsException`, `ClassCastException`. These extend `RuntimeException`.
- **Error** — represents serious problems that a reasonable application should NOT try to catch (e.g., `OutOfMemoryError`, `StackOverflowError`). Both `Exception` and `Error` extend the common base class `Throwable`.

**Exception hierarchy (simplified):**
```
Throwable
├── Exception
│   ├── RuntimeException (unchecked)
│   │     ├── ArithmeticException
│   │     ├── NullPointerException
│   │     ├── ArrayIndexOutOfBoundsException
│   │     └── ClassCastException
│   └── IOException, SQLException, ... (checked)
└── Error (Unchecked, should not be caught normally)
      ├── OutOfMemoryError
      └── StackOverflowError
```

C++ exception handling uses `try`/`catch`/`throw` too, but has no checked/unchecked distinction, and no `finally` keyword (though RAII — Resource Acquisition Is Initialization — via destructors achieves similar cleanup guarantees). C has NO built-in exception handling mechanism at all — errors are typically handled via return codes and the global `errno` variable, or `setjmp`/`longjmp` for a crude simulation.

## 3.7 Java-Specific Deep Dive

### 3.7.1 `final` keyword (three uses)

- `final` variable — value cannot be changed once assigned (a constant).
- `final` method — cannot be overridden by a subclass.
- `final` class — cannot be extended/subclassed (e.g., `String`, `Integer` are final classes in Java).

### 3.7.2 `static` keyword

- `static` variable — belongs to the CLASS, not any individual instance; shared across ALL objects of the class; only ONE copy exists regardless of how many objects are created.
- `static` method — belongs to the class; can be called WITHOUT creating an object (`ClassName.methodName()`); can only directly access other static members (cannot access instance/non-static fields or methods directly, since there's no implicit object context).
- `static` block — executed ONCE, when the class is first LOADED into memory (before any object is created or any static method is called) — typically used for one-time static initialization.

### 3.7.3 Abstract Classes vs Interfaces

| Feature | Abstract Class | Interface |
|---|---|---|
| Instantiation | Cannot be instantiated directly | Cannot be instantiated directly |
| Methods | Can have both abstract AND concrete (implemented) methods | Traditionally only abstract methods; Java 8+ allows `default` and `static` methods with a body |
| Fields | Can have instance variables of any access modifier | Fields are implicitly `public static final` (constants only) |
| Inheritance | A class can extend only ONE abstract class | A class can implement MULTIPLE interfaces |
| Constructors | Can have constructors | Cannot have constructors |
| Use case | "IS-A" relationship with shared partial implementation | Defines a "CAN-DO" contract/capability, often across unrelated classes |

**Sample paper Q16 connection**: Java 8 introduced **default methods** in interfaces specifically to preserve backward compatibility — this allowed the Java library designers to ADD new methods to existing interfaces (like adding `forEach()` to the `Collection` interface) WITHOUT breaking every single class that had already implemented that interface in the wild (they'd have been forced to implement the new method or fail to compile, otherwise).

### 3.7.4 `abstract` keyword

```java
abstract class Shape {
    abstract double area();     // no body - must be implemented by subclass
    void display() { System.out.println("This is a shape"); }  // concrete method, can be inherited as-is
}
class Circle extends Shape {
    double radius;
    double area() { return Math.PI * radius * radius; }  // MUST override, or Circle must also be abstract
}
```
A class with even ONE abstract method must itself be declared `abstract`. An abstract class CAN have zero abstract methods (just to prevent instantiation), but that's unusual style.

### 3.7.5 Pure Virtual Functions (C++)

```cpp
class Shape {
public:
    virtual double area() = 0;   // pure virtual function - makes Shape an ABSTRACT CLASS
    virtual void display() { cout << "Shape"; }  // regular virtual function - has a default body
};
class Circle : public Shape {
public:
    double radius;
    double area() override { return 3.14159 * radius * radius; }  // MUST override
};
```
A class containing at least one pure virtual function (`= 0`) becomes an **abstract class** in C++ and cannot be instantiated directly. Any concrete (non-abstract) derived class MUST provide an implementation (override) for every pure virtual function it inherits, or it too remains abstract. (This directly matches sample paper Q17.)

### 3.7.6 `virtual` keyword and Virtual Function Table (vtable)

In C++, a function is bound STATICALLY (compile-time, based on pointer/reference declared type) UNLESS it's marked `virtual` — then it's bound DYNAMICALLY (runtime, based on actual object type) via a mechanism called the **vtable (virtual table)** — a hidden table of function pointers maintained per class, that the compiler consults at runtime to find the correct overridden version to call.

```cpp
class Animal {
public:
    virtual void sound() { cout << "Some sound"; }
};
class Dog : public Animal {
public:
    void sound() override { cout << "Bark"; }
};
Animal *a = new Dog();
a->sound();  // prints "Bark" - dynamic dispatch via vtable, because sound() is virtual
```
If `sound()` were NOT marked `virtual`, then `a->sound()` would print "Some sound" instead — because without `virtual`, the call is resolved statically based on the POINTER's declared type (`Animal*`), not the actual object type (`Dog`).

### 3.7.7 Anonymous Inner Classes (Java)

A class with NO NAME, declared and instantiated in a SINGLE expression, typically used for one-time-use implementations (commonly for interfaces/abstract classes, especially in older-style event listener code before lambdas became common).

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running anonymously");
    }
};
```
(This directly matches sample paper Q51.)

### 3.7.8 Java 2D Arrays

```java
int[][] arr = new int[3][4];  // 3 rows, 4 columns
System.out.println(arr.length);      // 3 (number of rows)
System.out.println(arr[0].length);   // 4 (number of columns in row 0)
```
Java 2D arrays are technically "arrays of arrays" (jagged arrays are allowed — each row can have a DIFFERENT length, unlike a true rectangular matrix in C/C++). `arr.length` gives the outer array's size (number of rows); `arr[i].length` gives the size of row `i` specifically. (Matches sample paper Q19.)

### 3.7.9 Java Collections Framework — Queue Specifics

`Queue` is an INTERFACE in `java.util`, not a class — you cannot do `new Queue()`. It is implemented by:
- `LinkedList` — implements both `List` and `Deque`/`Queue`; commonly used as a general-purpose Queue implementation.
- `ArrayDeque` — a resizable-array implementation of `Deque` (double-ended queue); generally faster than `LinkedList` for queue/stack operations because it avoids per-node object overhead.
- `PriorityQueue` — orders elements according to their natural ordering or a supplied `Comparator`, NOT insertion order — the head of the queue is always the SMALLEST (or highest priority) element.
(Matches sample paper Q37.)

## 3.8 C-Specific Concepts

- **Pointers** — a variable that stores the MEMORY ADDRESS of another variable. `int *p = &x;` — `p` holds the address of `x`; `*p` (dereferencing) accesses the VALUE stored at that address.
- **Pointer arithmetic** — incrementing a pointer moves it forward by `sizeof(type)` bytes, not just 1 byte — e.g., `int *p; p++;` moves `p` forward by 4 bytes (on most systems where `int` is 4 bytes), because it's meant to point to the NEXT `int` in an array.
- **Arrays and pointers relationship** — an array name, when used in most expressions, DECAYS into a pointer to its first element. `arr[i]` is exactly equivalent to `*(arr + i)` in C.
- **Dynamic memory allocation** — `malloc()`, `calloc()`, `realloc()`, `free()` (from `<stdlib.h>`). `malloc(n)` allocates `n` bytes of UNINITIALIZED memory; `calloc(n, size)` allocates `n * size` bytes and initializes it all to ZERO. Forgetting `free()` causes a **memory leak**.
- **Structures (`struct`)** — a user-defined data type that groups together variables of DIFFERENT types under one name (unlike an array, which holds elements of the SAME type).

```c
struct Student {
    int roll_no;
    char name[50];
    float marks;
};
struct Student s1 = {1, "Ramesh", 85.5};
printf("%s", s1.name);
```

## 3.9 C++-Specific Concepts

- **References (`&`)** — an alias for an existing variable; MUST be initialized when declared, and cannot be reassigned to refer to a different variable afterward (unlike a pointer, which can be reassigned and can be NULL).
- **`new` and `delete`** — C++'s object-oriented equivalent of `malloc`/`free`, which also correctly call constructors/destructors.
- **Templates** — allow writing GENERIC functions/classes that work with ANY data type, without rewriting code for each type (similar in spirit to Java Generics).
```cpp
template <typename T>
T maxVal(T a, T b) { return (a > b) ? a : b; }
```
- **Operator Overloading** — C++ allows redefining the behavior of operators (`+`, `-`, `==`, etc.) for user-defined types (classes). Java does NOT support user-defined operator overloading (though it does have some BUILT-IN overloading, like `+` working for both numeric addition and String concatenation).

---

# SECTION 3 QUICK REVISION TABLE

| Concept | Key fact |
|---|---|
| Static method + child class | Method HIDING, not overriding — resolved by reference type (static/early binding) |
| Function overloading | Compile-time polymorphism; needs different parameter list, not just return type |
| Method overriding | Runtime polymorphism; needs `virtual` in C++, automatic in Java |
| Java pass by value | Even objects — the REFERENCE value is copied, not the object itself |
| Multiple inheritance | C++/Python: yes (classes); Java/C#: no (classes), yes via interfaces |
| Abstract class vs Interface | Abstract = partial implementation, single inheritance; Interface = pure contract, multiple implementation |
| `finally` block | Always runs except `System.exit()` or JVM crash |
| Checked vs Unchecked exception | Checked = compiler-enforced handling; Unchecked = RuntimeException subclasses |
| Java `Queue` | Interface, implemented by LinkedList, ArrayDeque, PriorityQueue |
| C++ pure virtual `=0` | Makes the class abstract; derived class must implement it |
| Encapsulation | Data + methods bound together, with access control |
| Java 2D array | `arr.length` = rows, `arr[0].length` = columns |


---

# SECTION 4: DATA ANALYTICS LANGUAGES (Python / R)

## 4.1 Python Basics Refresher (fast, since Phase II tests Python more numerically — see Section 10-13)

### 4.1.1 Regex (Regular Expressions)

Python's `re` module provides pattern matching over strings.

| Pattern | Meaning |
|---|---|
| `.` | any single character except newline |
| `*` | zero or more of the preceding element |
| `+` | one or more of the preceding element |
| `?` | zero or one (optional) of the preceding element |
| `^` | start of string |
| `$` | end of string |
| `[]` | character class, e.g. `[a-z]` any lowercase letter |
| `\d` | any digit `[0-9]` |
| `\w` | any word character `[a-zA-Z0-9_]` |
| `\s` | any whitespace |
| `{n,m}` | between n and m repetitions |
| `\|` | OR |
| `()` | grouping (also captures the match) |

```python
import re
re.match(pattern, string)     # checks match only at the BEGINNING of string
re.search(pattern, string)    # checks match ANYWHERE in string, returns first match
re.findall(pattern, string)   # returns ALL non-overlapping matches as a list
re.sub(pattern, repl, string) # replaces matches with repl
re.split(pattern, string)     # splits string by pattern matches

# Example
text = "My phone numbers are 9876543210 and 9123456780"
numbers = re.findall(r'\d{10}', text)   # ['9876543210', '9123456780']
```
**Trap**: `re.match` only checks the START of the string — `re.match('abc', 'xabc')` returns `None` even though 'abc' exists in the string, because it doesn't start there. `re.search` would find it.

### 4.1.2 Slicing

`sequence[start:stop:step]` — `start` inclusive, `stop` EXCLUSIVE, `step` optional (default 1).

```python
s = "Programming"
s[1:4]      # 'rog'  (indices 1,2,3 - index 4 excluded)
s[:-1]      # 'Programmin' (everything except last char) - matches sample Q27
s[::-1]     # 'gnimmargorP' (reversed string, step -1)
s[::2]      # every second character
s[2:]       # from index 2 to end
s[:5]       # from start to index 4 (5 excluded)
```
**Trap seen in sample paper Q28**: "substring from index 1 to 4th" — the natural-language phrase "1 to 4th" is ambiguous-sounding but SEBI means INCLUDING index 4, so you need `s[1:5]` (stop is exclusive, so to INCLUDE index 4 you must write 5). This exact trap is explicitly called out in the sample paper as a common wrong-answer trigger.

Negative indices count from the end: `s[-1]` = last character, `s[-2]` = second-last, etc.

### 4.1.3 Data Reshaping (pandas)

Reshaping means changing the LAYOUT/SHAPE of data without changing its actual values — critical for data analytics.

- **`pivot()` / `pivot_table()`** — converts data from LONG format (many rows, few columns) to WIDE format (fewer rows, more columns), by turning unique values of one column into new column headers. `pivot_table` additionally supports AGGREGATION (e.g., averaging duplicate entries), while `pivot` requires unique index/column combinations.
- **`melt()`** — the OPPOSITE of pivot — converts WIDE format to LONG format, "unpivoting" columns into rows.
- **`stack()` / `unstack()`** — pivot a level of column labels into the row index (`stack`) or vice versa (`unstack`), working with hierarchical (MultiIndex) data.
- **`merge()`** — combines two DataFrames based on common column(s)/keys, similar to SQL JOIN (supports `how='inner'`, `'left'`, `'right'`, `'outer'`).
- **`concat()`** — stacks DataFrames either vertically (`axis=0`, adding more rows) or horizontally (`axis=1`, adding more columns).
- **`groupby()`** — splits data into groups based on some criteria, applies a function to each group independently, and combines the results — directly analogous to SQL's `GROUP BY`.

```python
import pandas as pd
df = pd.DataFrame({'dept': ['A','A','B'], 'emp': ['x','y','z'], 'salary': [100,200,150]})
grouped = df.groupby('dept')['salary'].mean()
# dept A -> 150.0, dept B -> 150.0
```

### 4.1.4 Dataframes

A **DataFrame** (pandas) is a 2-dimensional, size-mutable, labeled data structure — conceptually like a spreadsheet/SQL table, with rows AND columns both having labels (index and column names respectively).

```python
import pandas as pd
df = pd.DataFrame({'Name': ['A','B','C'], 'Age': [25, 30, 35]})
df.head()          # first 5 rows
df.shape            # (rows, columns) tuple
df.columns          # column labels
df.dtypes           # data type of each column
df.describe()        # summary statistics (mean, std, min, max, quartiles) for numeric columns
df['Age']            # select a single column -> returns a Series
df[df['Age'] > 28]   # boolean filtering, similar to SQL WHERE
df.loc[0]            # label-based row selection
df.iloc[0]           # position/integer-based row selection
df.sort_values('Age')  # sort by column
df.isnull().sum()    # count missing values per column
df.dropna()           # remove rows with any NULL/NaN
df.fillna(0)           # replace NaN with a value
```

A **Series** is a 1-dimensional labeled array — essentially a single column of a DataFrame, or a labeled list.

### 4.1.5 Dictionaries and Sets

**Dictionary (`dict`)** — a mutable, UNORDERED (technically insertion-ordered since Python 3.7+) collection of KEY-VALUE pairs. Keys must be unique and HASHABLE (immutable types like strings, numbers, tuples — NOT lists or dicts, since those are mutable and thus unhashable).

```python
d = {'name': 'Ramesh', 'age': 25}
d['city'] = 'Mumbai'    # add a new key
d.get('name')             # 'Ramesh' - safe access, returns None (or default) if key missing, no error
d['unknown']              # raises KeyError if key doesn't exist
d.keys()                  # view of all keys
d.values()                # view of all values
d.items()                 # view of (key, value) pairs
del d['age']               # remove a key
```

**Set (`set`)** — an unordered collection of UNIQUE elements (no duplicates allowed), supports mathematical set operations.

```python
s1 = {1, 2, 3}
s2 = {2, 3, 4}
s1 | s2    # union -> {1,2,3,4}
s1 & s2    # intersection -> {2,3}
s1 - s2    # difference -> {1}
s1 ^ s2    # symmetric difference -> {1,4}
```

### 4.1.6 File Management

```python
f = open('data.txt', 'r')     # modes: 'r' read, 'w' write (overwrites), 'a' append, 'r+' read+write, 'rb'/'wb' binary
content = f.read()             # read entire file as one string
lines = f.readlines()          # read all lines into a list
f.close()                       # always close to release the file handle

# Preferred: context manager (auto-closes even on exception)
with open('data.txt', 'r') as f:
    for line in f:
        print(line.strip())
```
`with` statement is preferred because it guarantees the file is closed automatically, even if an exception occurs inside the block (similar concept to `try-finally` in Java, or RAII in C++).

### 4.1.7 Classes and Functions (Python OOP syntax specifics)

```python
class Employee:
    company = "SEBI"   # class variable (shared across all instances, like Java's static)

    def __init__(self, name, salary):   # constructor/initializer
        self.name = name       # instance variable
        self.salary = salary

    def display(self):          # instance method - 'self' refers to the calling object (like 'this' in Java)
        print(f"{self.name} earns {self.salary}")

    @staticmethod
    def company_info():          # static method - no 'self', can't access instance data
        return "SEBI is the regulator"

    @classmethod
    def from_string(cls, data_str):   # class method - receives the class itself ('cls'), not an instance
        name, salary = data_str.split(',')
        return cls(name, int(salary))

e = Employee("Ramesh", 50000)
e.display()
```
Python supports **inheritance**: `class Manager(Employee): ...`. Python DOES support multiple inheritance directly: `class C(A, B): ...`, resolved via **MRO (Method Resolution Order)**, computed using the **C3 linearization algorithm**.

### 4.1.8 Data Mining (Conceptual)

Data mining is the process of discovering patterns, correlations, and useful information from large datasets, using methods at the intersection of statistics, machine learning, and database systems. Key techniques (conceptual, no coding needed at this level):

- **Classification** — predicting a CATEGORICAL label for new data based on labeled training data (e.g., spam / not spam). Algorithms: Decision Trees, Naive Bayes, k-NN, SVM.
- **Clustering** — grouping similar data points together WITHOUT pre-labeled categories (unsupervised). Algorithms: k-Means, hierarchical clustering, DBSCAN.
- **Regression** — predicting a CONTINUOUS numeric value (e.g., predicting house price).
- **Association Rule Mining** — finding relationships between variables in large datasets (classic example: "Market Basket Analysis" — customers who buy bread also tend to buy butter). Key metrics: **Support** (how frequently the itemset appears), **Confidence** (likelihood of the consequent given the antecedent), **Lift** (how much more likely the consequent is, given the antecedent, compared to its baseline probability).
- **Anomaly/Outlier Detection** — identifying data points that deviate significantly from the norm.
- **Supervised vs Unsupervised learning**: Supervised = trained on LABELED data (input-output pairs known); Unsupervised = trained on UNLABELED data, finds hidden structure on its own.

### 4.1.9 Lists

A Python `list` is a mutable, ORDERED, INDEXED collection that can hold elements of MIXED types.

```python
lst = [1, 2, 3]
lst.append(4)         # adds to end, in-place, returns None -> matches sample Q64 trap
lst.insert(0, 0)       # insert at specific index
lst.pop()               # removes and returns LAST element - O(1)
lst.pop(0)              # removes and returns FIRST element - O(n), because all remaining elements must shift left
lst.remove(3)           # removes the FIRST occurrence of value 3 (by value, not index)
lst.sort()               # sorts in-place, returns None
sorted(lst)              # returns a NEW sorted list, original unchanged
lst.reverse()            # reverses in-place
len(lst)                  # O(1) - length is stored as an attribute, not computed by scanning
```
**Trap (sample Q64)**: `Y = X.append(4)` → `append()` MODIFIES the list in-place and returns `None`. So `Y` becomes `None`, NOT the new list. `X` itself now has the 4th element. This "mutating methods return None" trap applies to `append`, `sort`, `reverse`, `insert`, `remove`, `extend` — ALL of them return `None` in Python, unlike some other languages where mutator methods return `self`/the object for chaining.

**Trap (sample Q30)**: `list.pop(0)` is O(n) because ALL subsequent elements must be shifted one position to the left to fill the gap — Python lists are implemented as dynamic ARRAYS internally, not linked lists. For O(1) removal from the front, use `collections.deque` and its `popleft()` method instead.

### 4.1.10 Importing and Exporting Data

```python
import pandas as pd
df = pd.read_csv('file.csv')
df = pd.read_excel('file.xlsx')
df = pd.read_json('file.json')
df = pd.read_sql(query, connection)

df.to_csv('output.csv', index=False)
df.to_excel('output.xlsx', index=False)
df.to_json('output.json')
```

### 4.1.11 Charts and Graphs (matplotlib basics)

```python
import matplotlib.pyplot as plt
plt.plot(x, y)           # line chart
plt.bar(x, y)              # bar chart
plt.scatter(x, y)          # scatter plot
plt.hist(data, bins=10)    # histogram
plt.pie(sizes, labels=labels)   # pie chart
plt.xlabel('X'); plt.ylabel('Y'); plt.title('Title')
plt.legend()
plt.show()
```
Chart type selection logic (conceptual, often tested): **Line chart** for trends over time/continuous sequence; **Bar chart** for comparing categorical quantities; **Scatter plot** for relationship/correlation between two numeric variables; **Histogram** for distribution/frequency of a single numeric variable (note: histogram bars touch each other because the x-axis is continuous ranges/bins, unlike a bar chart where bars represent discrete unrelated categories and are drawn with gaps); **Pie chart** for proportion of a whole (parts of 100%); **Box plot** for showing median, quartiles, and outliers.

### 4.1.12 JSON in Python

```python
import json
data = {'name': 'Ramesh', 'age': 25}
json_str = json.dumps(data)      # Python dict -> JSON string
python_obj = json.loads(json_str) # JSON string -> Python dict

# 's' suffix = works with STRINGS. Without 's' = works with FILE OBJECTS
with open('data.json', 'w') as f:
    json.dump(data, f)             # write dict directly to a file
with open('data.json', 'r') as f:
    data2 = json.load(f)           # read dict directly from a file
```
**Trap (sample Q57)**: `json.dumps(data)` returns a NEW string; it does NOT modify `data` itself — `data` remains a `dict` (its `type()` is still `dict`), only the RETURNED value from `dumps()` is a string. **Trap (sample Q58)**: converting a JSON STRING to a Python object uses `json.loads()` (load-string), not `json.load()` (which is for file objects).

## 4.2 R Language Basics (lighter weight in the exam, but do cover fundamentals)

R is a language purpose-built for statistical computing and graphics.

```r
# Vectors - the fundamental R data structure (1-indexed, unlike Python's 0-indexing!)
v <- c(1, 2, 3, 4, 5)
v[1]          # 1 (R is 1-INDEXED, not 0-indexed - a major trap vs Python/Java/C)
length(v)      # 5

# Data frames - similar concept to pandas DataFrame
df <- data.frame(name = c("A","B"), age = c(25,30))
str(df)         # structure/summary of the data frame
summary(df)      # statistical summary

# Common functions
mean(v); median(v); sd(v); var(v)
sapply(v, function(x) x^2)    # apply a function over a vector, returns a vector/matrix
lapply(v, function(x) x^2)    # apply a function, returns a LIST
```
**Key R vs Python differences to remember:**
- R is 1-indexed; Python is 0-indexed.
- R uses `<-` as the conventional assignment operator (though `=` also works in most contexts).
- R's core data structure is the vector (everything is vectorized by default — operations apply element-wise automatically without explicit loops).
- `NULL` vs `NA` in R: `NULL` represents the absence of a value/object entirely; `NA` represents a MISSING value within an existing vector/structure (like SQL's NULL). This distinction is R-specific and sometimes tested.

---

# SECTION 4 QUICK REVISION TABLE

| Trap | Correct Answer |
|---|---|
| `s[1:5]` for "index 1 to 4th inclusive" | Stop index is exclusive, so use one more than the last index you want |
| `list.append(4)` return value | Returns `None`; modifies list in-place |
| `list.pop(0)` time complexity | O(n) — elements shift; use `deque.popleft()` for O(1) |
| `len()` time complexity | O(1) — stored as an attribute |
| `json.dumps()` | Returns new string, doesn't change original dict's type |
| `json.loads()` vs `json.load()` | `loads` = from string; `load` = from file object |
| R indexing | 1-indexed, not 0-indexed |
| Histogram vs Bar chart | Histogram = continuous bins (bars touch); Bar = discrete categories (bars separated) |
| `re.match` vs `re.search` | `match` only checks start of string; `search` checks anywhere |


---

# SECTION 5: ALGORITHMS FOR PROBLEM SOLVING (Phase I Overview)

> Note: This section gives Phase I level coverage. Section 10 (Phase II) goes much deeper into the SAME topics with numerical tracing, exactly the style of the 69 sample questions — read both.

## 5.1 Tree and Graph Traversals

### 5.1.1 Tree Traversals

Given a binary tree, there are three classic **Depth-First** traversal orders, defined by WHEN you visit the root relative to its children:

- **Inorder (Left → Root → Right)** — for a Binary SEARCH Tree specifically, inorder traversal always produces elements in SORTED (ascending) order. This is one of the most important facts in the entire syllabus.
- **Preorder (Root → Left → Right)** — root is visited FIRST. Useful for creating a COPY of the tree, or for prefix expression trees.
- **Postorder (Left → Right → Root)** — root is visited LAST. Useful for SAFELY DELETING a tree (delete children before the parent), and for postfix expression evaluation.
- **Level-order** — visits nodes level by level, left to right, using a QUEUE (this is essentially BFS applied to a tree).

**Sample Q12/Q13 connection**: "Left subtree is traversed before right subtree" is TRUE for ALL THREE of inorder, preorder, postorder — the only thing that changes between them is WHEN the root is visited (before both children = preorder, between them = inorder, after both = postorder).

### 5.1.2 Graph Traversals

- **BFS (Breadth-First Search)** — explores all neighbors at the current depth before moving to nodes at the next depth level. Implemented using a **QUEUE** (FIFO). Used for: shortest path in an UNWEIGHTED graph, level-order tree traversal, finding connected components.
- **DFS (Depth-First Search)** — explores as far as possible along each branch before backtracking. Implemented using a **STACK** (explicit stack, or implicitly via RECURSION, which uses the call stack). Used for: topological sorting, cycle detection, finding connected components, solving mazes.

**Trace example — BFS on graph with edges A-B, A-C, B-D, C-D, starting at A:**
```
Queue: [A] -> visit A, enqueue neighbors B, C -> Queue: [B, C]
Dequeue B, visit B, enqueue D (unvisited) -> Queue: [C, D]
Dequeue C, visit C, D already in queue/visited, skip -> Queue: [D]
Dequeue D, visit D, no new neighbors -> Queue: []
BFS order: A, B, C, D
```

Both BFS and DFS run in **O(V + E)** time (V = number of vertices, E = number of edges), when using an adjacency list representation.

## 5.2 Connected Components

A **connected component** of an undirected graph is a maximal set of vertices such that there is a PATH between every pair of vertices within that set. A graph can have one or more connected components. Found using either BFS or DFS: run a traversal from an unvisited node, mark everything reachable as one component, then repeat from the next unvisited node.

For DIRECTED graphs, the analogous (stronger) concept is a **Strongly Connected Component (SCC)** — a maximal set of vertices where every vertex is reachable from every other vertex WITHIN the set, following edge DIRECTIONS. Found using **Kosaraju's algorithm** or **Tarjan's algorithm**.

## 5.3 Spanning Trees

A **spanning tree** of a connected, undirected graph is a subgraph that includes ALL the vertices of the original graph, is CONNECTED, and has NO CYCLES (i.e., it's a tree) — it uses exactly **V − 1 edges** for V vertices (matches sample paper Q31: "number of edges in a tree of n vertices = n − 1", since a tree by definition is a connected acyclic graph).

A **Minimum Spanning Tree (MST)** is the spanning tree with the MINIMUM possible total edge weight, among all possible spanning trees of a weighted graph.

**Two classic MST algorithms — both GREEDY (not DP — matches sample Q32):**

- **Prim's Algorithm** — starts from an arbitrary vertex and GROWS the tree one edge at a time, always picking the MINIMUM weight edge that connects a vertex ALREADY in the tree to a vertex NOT YET in the tree (matches sample Q33's exact description: "pick minimum weight edge(i,j) where i is in tree and j is NOT in tree"). Efficient with a priority queue/min-heap: O(E log V).
- **Kruskal's Algorithm** — sorts ALL edges by weight ascending, then greedily picks the smallest edge that does NOT form a cycle with edges already selected (cycle detection typically done using the **Union-Find / Disjoint Set Union (DSU)** data structure). Runs in O(E log E) due to the initial sort.

**Key difference**: Prim's grows a SINGLE tree incrementally from a starting point; Kruskal's considers edges GLOBALLY across the whole graph regardless of connectivity at each step, using Union-Find to avoid cycles.

## 5.4 Shortest Path Algorithms

- **Dijkstra's Algorithm** — finds the shortest path from a SINGLE source vertex to ALL other vertices, in a graph with NON-NEGATIVE edge weights. Greedy approach: repeatedly picks the unvisited vertex with the smallest known distance, and RELAXES (updates) its neighbors' distances. Uses a min-priority queue for efficiency: O((V+E) log V). FAILS (gives wrong answers) if the graph has negative edge weights.
- **Bellman-Ford Algorithm** — also finds single-source shortest paths, but CAN handle NEGATIVE edge weights (though not negative CYCLES reachable from the source — it can DETECT such cycles, though). Works by relaxing ALL edges, V−1 times. Slower than Dijkstra: O(V·E).
- **Floyd-Warshall Algorithm** — finds shortest paths between ALL PAIRS of vertices simultaneously, using Dynamic Programming. Works with negative edge weights (but not negative cycles). Time complexity: O(V³).
- **A\* Search** — an extension of Dijkstra that uses a HEURISTIC function to guide the search toward the goal faster (used heavily in pathfinding for maps/games).

## 5.5 Hashing (Algorithmic View — see also Section 11 for the data-structure view)

**Hashing** maps data (keys) of arbitrary size to fixed-size values (hash codes) using a **hash function**, to enable near-constant-time O(1) average-case lookup, insertion, and deletion.

- **Collision** — when two different keys produce the SAME hash value (map to the same slot/bucket). Since the number of possible keys usually exceeds the number of slots, collisions are inevitable (Pigeonhole Principle) and MUST be handled.
- **Collision resolution techniques**:
  - **Separate Chaining** — each slot/bucket holds a LINKED LIST (or another structure) of all keys that hash to that slot (matches sample Q54). 
  - **Open Addressing** — on collision, probe for another EMPTY slot within the array itself, using a defined probing sequence:
    - **Linear Probing** — check the next slot sequentially (`(h(k) + i) mod m`). Prone to "primary clustering" (consecutive filled slots grow into large blocks, degrading performance).
    - **Quadratic Probing** — check slots at increasing quadratic distances (`(h(k) + i²) mod m`). Reduces primary clustering but can have secondary clustering.
    - **Double Hashing** — uses a SECOND hash function to determine the probe step size, spreading collisions more evenly; generally performs best among open-addressing schemes.
- **Load Factor (α)** — α = n/m, where n = number of stored keys, m = number of slots/buckets. Directly determines average performance — the higher the load factor, the more collisions and the slower the operations (matches sample Q22: "load factor means average number of keys in chains").

## 5.6 Sorting Algorithms — Overview (deep numerical dive in Section 10)

Quick conceptual pass here; Section 10 will trace through actual arrays.

- **Bubble Sort** — repeatedly compares ADJACENT elements and swaps if out of order; after each full pass, the largest unsorted element "bubbles up" to its correct final position at the end.
- **Selection Sort** — repeatedly finds the MINIMUM element from the unsorted portion and swaps it into the correct position at the FRONT of the unsorted portion (matches sample Q6: "every pass we get the minimum element").
- **Insertion Sort** — builds the sorted array one element at a time, by taking the next unsorted element and INSERTING it into its correct position within the already-sorted portion (matches sample Q5).
- **Merge Sort** — Divide and Conquer: recursively splits the array in half, sorts each half, then MERGES the two sorted halves back together. STABLE, always O(n log n) regardless of input (best/average/worst case are all the same, since it always does the same splitting/merging work).
- **Quick Sort** — Divide and Conquer: picks a PIVOT element, PARTITIONS the array so smaller elements go left and larger go right of the pivot, then recursively sorts each partition. NOT stable. Average case O(n log n), but WORST CASE O(n²) — occurs specifically when the pivot chosen is repeatedly the smallest or largest element, which classically happens when the array is ALREADY SORTED (or reverse sorted) and the pivot is naively chosen as the first or last element each time (matches sample Q9).
- **Heap Sort** — builds a MAX-HEAP from the array, then repeatedly extracts the maximum element (swap root with last, shrink heap, heapify down) to build the sorted array. O(n log n) always, but NOT stable (matches quick-reference table in the sample document).

**Stability definition (matches sample Q7/Q8)**: a sorting algorithm is STABLE if it preserves the RELATIVE ORDER of elements that compare as EQUAL (e.g., two employees with the same salary should remain in their original relative order after sorting by salary). Stable: Bubble, Insertion, Merge. Unstable: Quick, Selection, Heap.

## 5.7 Searching Algorithms

- **Linear Search** — checks each element one by one; O(n); works on ANY array (sorted or not).
- **Binary Search** — repeatedly halves the search range by comparing the target to the MIDDLE element; requires the array to be SORTED; O(log n).
- **Exponential Search** — first finds a RANGE where the target might exist by repeatedly doubling the index (1, 2, 4, 8, 16, ...) until the value at that index exceeds the target, then performs BINARY SEARCH within that identified range. Particularly efficient when the target is near the BEGINNING of a very large sorted array, or when the array size is unknown/unbounded (matches sample Q25). Overall O(log n).
- **Interpolation Search** — an improvement on binary search for UNIFORMLY DISTRIBUTED sorted data — instead of always checking the middle, it ESTIMATES the likely position of the target based on the target's VALUE (like how you'd open a phone book near "S" directly, rather than starting from the middle, if looking for "Sharma"). Average case O(log log n) for uniform data, but degrades to O(n) worst case for non-uniform/skewed data (matches sample Q26).

## 5.8 Algorithm Design Techniques

- **Greedy Algorithms** — make the LOCALLY optimal choice at each step, hoping it leads to a globally optimal solution. Works correctly only for problems that have the **"greedy choice property"** (a global optimum can be reached by making locally optimal choices) — e.g., Prim's, Kruskal's, Dijkstra's, Huffman Coding, Activity/Job Selection.
- **Dynamic Programming (DP)** — solves complex problems by breaking them into OVERLAPPING SUBPROBLEMS, solving each subproblem ONLY ONCE, and STORING the result (memoization/tabulation) to avoid redundant recomputation. Requires TWO properties (matches sample Q34):
  1. **Optimal Substructure** — an optimal solution to the problem can be constructed from optimal solutions of its subproblems.
  2. **Overlapping Subproblems** — the same subproblems are solved repeatedly if approached naively (recursively without memoization) — this is what DISTINGUISHES DP from plain Divide-and-Conquer.
  - Classic DP problems: 0/1 Knapsack, Longest Common Subsequence (LCS), Coin Change, Matrix Chain Multiplication, Fibonacci (with memoization), Edit Distance.
  - **Memoization (top-down)** — recursive approach, caching/storing results of subproblems as they're computed, checking the cache before recomputing.
  - **Tabulation (bottom-up)** — iterative approach, building up a table of subproblem solutions from the smallest subproblems upward.
- **Divide and Conquer** — breaks a problem into INDEPENDENT (non-overlapping) subproblems, solves each RECURSIVELY, then COMBINES the results. HAS optimal substructure, but does NOT have overlapping subproblems (this is exactly why Merge Sort/Quick Sort are Divide & Conquer, not DP — each recursive call works on a genuinely different/independent part of the array). Examples: Merge Sort, Quick Sort, Binary Search, Strassen's Matrix Multiplication.
- **Backtracking** — builds a solution incrementally, and ABANDONS ("backtracks" from) a partial solution as soon as it determines that solution cannot possibly lead to a valid/complete answer, trying a different path instead. Essentially a refined/pruned brute-force search via DFS on the "solution space tree." Examples: N-Queens (matches sample Q36 — no two queens on the same row, column, or diagonal), Sudoku solving (matches sample Q35), maze solving, generating permutations/subsets.

**Greedy vs DP vs Divide & Conquer — the ultimate comparison table:**

| Technique | Subproblems | Choice | Guarantees global optimum? |
|---|---|---|---|
| Greedy | Not necessarily solved/considered | Made once, never reconsidered | Only for problems with greedy-choice property |
| Divide & Conquer | Independent, non-overlapping | Combine results of independent subproblems | Yes, if subproblems are solved correctly |
| Dynamic Programming | Overlapping, solved once & cached | Considers all choices, picks the best using stored subproblem results | Yes, if optimal substructure holds |

## 5.9 Pattern Searching (String Matching Algorithms)

- **Naive Pattern Search** — slides the pattern over the text one position at a time, checking for a full match at EVERY position. Time complexity O(n·m), where n = text length, m = pattern length (matches sample Q43).
- **KMP (Knuth-Morris-Pratt) Algorithm** — preprocesses the PATTERN to build a "failure function" / **LPS array (Longest Proper Prefix which is also a Suffix)**, which tells the algorithm how far to "skip ahead" without re-checking characters it already knows will match, when a mismatch occurs — avoids redundant re-comparisons. Time complexity O(n + m).
- **Rabin-Karp Algorithm** — computes a HASH value for the pattern, and a ROLLING hash for each window of the text (updating the hash in O(1) as the window slides, rather than recomputing from scratch) — only does a full character-by-character comparison when hash values MATCH (to rule out hash collisions). Average case O(n + m), but worst case degrades to O(n·m) if there are many hash collisions (matches sample Q44).
- **Boyer-Moore Algorithm** (good to know conceptually) — scans the pattern from RIGHT to LEFT and can skip large sections of text on a mismatch, using "bad character" and "good suffix" heuristics; often the fastest in practice for large alphabets (like natural language text).

## 5.10 Divide and Conquer — Recurrence Relations and Master Theorem

A **recurrence relation** expresses the running time of a recursive algorithm in terms of the running time on smaller inputs.

**Merge Sort's recurrence (matches sample Q46):** T(n) = 2T(n/2) + O(n) — because merge sort splits the array into 2 halves (each of size n/2), recursively sorts each half, and then does O(n) work to MERGE them back together.

**Master Theorem** — a direct formula/shortcut for solving recurrences of the form T(n) = a·T(n/b) + f(n), where a ≥ 1, b > 1:
- Let f(n) be compared to n^(log_b a).
- **Case 1**: if f(n) = O(n^(log_b a − ε)) for some ε > 0 (f(n) grows POLYNOMIALLY SLOWER), then T(n) = Θ(n^(log_b a)).
- **Case 2**: if f(n) = Θ(n^(log_b a)) (f(n) grows at the SAME rate), then T(n) = Θ(n^(log_b a) · log n).
- **Case 3**: if f(n) = Ω(n^(log_b a + ε)) (f(n) grows POLYNOMIALLY FASTER) AND the regularity condition holds, then T(n) = Θ(f(n)).

**Applying to merge sort**: a=2, b=2, f(n)=n. n^(log_b a) = n^(log₂2) = n^1 = n. Since f(n) = n = Θ(n^1), this is CASE 2 → T(n) = Θ(n log n). This holds for merge sort's best, average, AND worst case identically — merge sort ALWAYS does the same amount of splitting/merging work regardless of input order, which is why it's the classic example of an algorithm with IDENTICAL best/average/worst case complexity.

**Strassen's Matrix Multiplication (matches sample Q69)**: standard matrix multiplication of two n×n matrices is O(n³) (three nested loops). Strassen's algorithm is a Divide & Conquer technique that cleverly reduces the number of RECURSIVE MULTIPLICATIONS needed per split from 8 down to 7 (using extra additions/subtractions to compensate), giving a recurrence T(n) = 7T(n/2) + O(n²), which by the Master Theorem (Case 1, since n² grows slower than n^(log₂7)) resolves to **T(n) = O(n^log₂7) ≈ O(n^2.807)**, better than the naive O(n³) for sufficiently large n.

---

# SECTION 5 QUICK REVISION TABLE

| Algorithm | Category | Time Complexity | Key Fact |
|---|---|---|---|
| BFS | Traversal | O(V+E) | Uses Queue |
| DFS | Traversal | O(V+E) | Uses Stack/recursion |
| Prim's | Greedy MST | O(E log V) | Grows tree from one vertex |
| Kruskal's | Greedy MST | O(E log E) | Picks global min edge, uses Union-Find |
| Dijkstra | Greedy shortest path | O((V+E) log V) | No negative weights |
| Bellman-Ford | DP shortest path | O(V·E) | Handles negative weights, detects negative cycles |
| Floyd-Warshall | DP shortest path | O(V³) | All-pairs |
| Naive pattern search | String matching | O(nm) | Brute force |
| KMP | String matching | O(n+m) | Uses LPS array |
| Rabin-Karp | String matching | O(n+m) avg | Uses rolling hash |
| Merge Sort | Divide & Conquer | O(n log n) always | Stable |
| Quick Sort | Divide & Conquer | O(n log n) avg, O(n²) worst | Worst case = sorted input |
| Strassen's | Divide & Conquer | O(n^2.81) | 7 multiplications instead of 8 |


---

# SECTION 6: NETWORKING CONCEPTS

## 6.1 OSI Model — 7 Layers (Guaranteed Question)

Mnemonic: "**A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing" (Application, Presentation, Session, Transport, Network, Data Link, Physical — top to bottom).

| Layer | Number | Function | Protocols/Devices |
|---|---|---|---|
| Application | 7 | Interface for end-user applications | HTTP, FTP, SMTP, DNS |
| Presentation | 6 | Data translation, encryption, compression | SSL/TLS, JPEG, ASCII |
| Session | 5 | Establishes, manages, terminates sessions between apps | NetBIOS, RPC |
| Transport | 4 | End-to-end delivery, reliability, flow control | TCP, UDP |
| Network | 3 | Logical addressing, routing between networks | IP, ICMP; Routers |
| Data Link | 2 | Physical addressing (MAC), error detection, framing | Ethernet; Switches, Bridges |
| Physical | 1 | Raw bit transmission over physical medium | Cables, Hubs, Repeaters |

**Key trap**: know WHICH device operates at WHICH layer.
- **Repeater/Hub** — Physical Layer (Layer 1) — just amplifies/regenerates signals, no intelligence, broadcasts to all ports.
- **Bridge/Switch** — Data Link Layer (Layer 2) — uses MAC addresses to forward frames only to the correct port (switch is essentially a multi-port bridge).
- **Router** — Network Layer (Layer 3) — uses IP addresses to forward packets between DIFFERENT networks.
- **Gateway** — can operate at any layer, typically the highest (Application layer conceptually) — connects networks using DIFFERENT protocols/architectures entirely (translates between them).

**TCP/IP Model (4-layer, practical alternative to OSI's 7-layer)**: Application, Transport, Internet, Network Access/Link. Maps roughly: TCP/IP's Application layer = OSI's Application+Presentation+Session; TCP/IP's Internet layer = OSI's Network layer; TCP/IP's Network Access layer = OSI's Data Link+Physical.

## 6.2 LAN Technologies

- **Ethernet** — the dominant wired LAN technology, based on the **CSMA/CD** (Carrier Sense Multiple Access with Collision Detection) access method historically (for shared/hub-based Ethernet) — a device LISTENS to the wire before transmitting (carrier sense), and if a collision is detected during transmission, it stops, waits a random backoff time, and retries. Modern SWITCHED Ethernet (full-duplex, one device per switch port) largely eliminates collisions, making CSMA/CD mostly historical/half-duplex-relevant now.
- **Token Ring** — an older LAN technology where a special data frame called a **token** circulates around the ring; a device can only transmit data when it POSSESSES the token, which guarantees no collisions (deterministic access) but is generally slower/less flexible than modern switched Ethernet, and is now largely obsolete.
- **Wi-Fi (Wireless LAN, 802.11)** — uses **CSMA/CA** (Collision AVOIDANCE, not detection) since wireless devices generally cannot reliably detect collisions while transmitting — instead, a device waits a random time and senses the channel is clear before transmitting, sometimes using RTS/CTS (Request to Send/Clear to Send) handshaking to reduce collision chances.

## 6.3 TCP vs UDP — Very Frequently Tested

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Reliable (acknowledgments, retransmission) | Unreliable (no guarantee of delivery) |
| Ordering | Guarantees in-order delivery | No ordering guarantee |
| Speed | Slower (overhead of reliability) | Faster (minimal overhead) |
| Header size | Larger (20 bytes minimum) | Smaller (8 bytes) |
| Use cases | Web browsing (HTTP), email, file transfer | Video/audio streaming, DNS, VoIP, online gaming |
| Flow/Congestion control | Yes | No |

**TCP 3-way handshake (connection establishment):**
1. Client sends `SYN` (synchronize) to server.
2. Server responds with `SYN-ACK` (synchronize-acknowledge).
3. Client sends `ACK` (acknowledge) — connection established.

**TCP connection termination** typically uses a 4-way handshake (`FIN`, `ACK`, `FIN`, `ACK`), since TCP is full-duplex and each direction must be closed independently.

## 6.4 IP Addressing

- **IPv4** — 32-bit address, written as four decimal octets (e.g., 192.168.1.1), giving ~4.3 billion possible addresses (a number increasingly insufficient for the modern internet — this is why IPv6 was created).
- **IPv6** — 128-bit address, written in 8 groups of hexadecimal digits (e.g., 2001:0db8::1), providing a vastly larger address space, designed to solve IPv4 exhaustion.
- **Classful IP addressing (historical)**:
  - Class A: 1st octet 1–126, default subnet mask 255.0.0.0 (/8), huge number of hosts per network, few networks — for very large organizations.
  - Class B: 1st octet 128–191, default mask 255.255.0.0 (/16) — for medium organizations.
  - Class C: 1st octet 192–223, default mask 255.255.255.0 (/24) — for small networks, most common.
  - Class D: 224–239 — reserved for MULTICAST.
  - Class E: 240–255 — reserved for experimental/research use.
  - (127.x.x.x is reserved for LOOPBACK — testing on the local machine itself, e.g., `127.0.0.1` = "localhost.")
- **Subnetting** — dividing a large network into smaller sub-networks using a **subnet mask** — improves address utilization, reduces broadcast domain size, and improves security/manageability. **CIDR (Classless Inter-Domain Routing)** notation (e.g., `192.168.1.0/24`) specifies the number of bits used for the NETWORK portion of the address, replacing the old rigid classful system.
- **Public vs Private IP** — Private IP ranges (not routable on the public internet, used within LANs) are: `10.0.0.0 – 10.255.255.255` (Class A private block), `172.16.0.0 – 172.31.255.255` (Class B private block), `192.168.0.0 – 192.168.255.255` (Class C private block). **NAT (Network Address Translation)** allows multiple devices with private IPs to share a single public IP for internet access.
- **DHCP (Dynamic Host Configuration Protocol)** — automatically assigns IP addresses (and other network configuration like subnet mask, default gateway, DNS server) to devices on a network, avoiding manual configuration.

## 6.5 Switches, Gateways, and Routers (recap with more depth)

- **Switch** — a Layer 2 device that connects multiple devices within the SAME network/LAN, and intelligently forwards Ethernet frames only to the specific port where the destination MAC address is located (learned via a MAC address table), rather than broadcasting to every port (unlike a hub).
- **Router** — a Layer 3 device that connects DIFFERENT networks together (e.g., your home LAN to the internet/ISP network), and makes forwarding decisions based on IP addresses, using ROUTING TABLES and routing protocols (like OSPF, BGP, RIP) to determine the best path.
- **Gateway** — a broader term for any device/node that serves as an entry/exit point between two networks that may use DIFFERENT protocols or architectures — a home router is often ALSO functioning as the "default gateway" for the LAN.

## 6.6 Application Layer Protocols

| Protocol | Full Form | Port | Purpose |
|---|---|---|---|
| HTTP | HyperText Transfer Protocol | 80 | Web page transfer (stateless) |
| HTTPS | HTTP Secure | 443 | HTTP + TLS/SSL encryption |
| FTP | File Transfer Protocol | 20 (data), 21 (control) | File transfer between client and server |
| SMTP | Simple Mail Transfer Protocol | 25 | SENDING email |
| POP3 | Post Office Protocol v3 | 110 | RETRIEVING email (downloads and typically removes from server) |
| IMAP | Internet Message Access Protocol | 143 | RETRIEVING email (keeps mail synced on server, supports multiple devices) |
| DNS | Domain Name System | 53 | Translates domain names (e.g., google.com) to IP addresses |
| DHCP | Dynamic Host Configuration Protocol | 67 (server), 68 (client) | Automatic IP address assignment |
| Telnet | — | 23 | Remote login, UNENCRYPTED (insecure, largely replaced) |
| SSH | Secure Shell | 22 | Remote login, ENCRYPTED (secure replacement for Telnet) |

**DNS resolution process (conceptual)**: when you type a domain name, your device queries a DNS resolver, which (if not cached) queries the ROOT server → TLD (Top-Level Domain, e.g., ".com") server → AUTHORITATIVE server for that specific domain, to progressively resolve the full IP address — this hierarchical lookup process is called **recursive/iterative DNS resolution**.

## 6.7 Firewalls

A **firewall** is a network security device/software that MONITORS and CONTROLS incoming and outgoing network traffic based on predetermined security RULES, acting as a barrier between a trusted internal network and an untrusted external network (like the internet).

- **Packet-filtering firewall** — inspects individual packets against rules based on source/destination IP, port, protocol — simple and fast, but has no awareness of connection STATE or application-level content.
- **Stateful Inspection firewall** — tracks the STATE of active connections (e.g., recognizes that a response packet belongs to an established outgoing request) and makes filtering decisions based on the context of the traffic flow, not just individual packets in isolation.
- **Proxy Firewall (Application-level gateway)** — operates at the Application layer, acting as an intermediary that fully terminates and re-establishes connections on behalf of clients, allowing deep inspection of the actual application data/content.
- **Next-Generation Firewall (NGFW)** — combines traditional firewall capabilities with additional features like intrusion prevention, deep packet inspection, and application awareness.

---

# SECTION 7: INFORMATION & CYBER SECURITY CONCEPTS

## 7.1 CIA Triad — The Foundational Concept, Guaranteed Question

- **Confidentiality** — ensuring information is accessible ONLY to those AUTHORIZED to access it. Achieved via encryption, access controls, authentication.
- **Integrity** — ensuring information is accurate and has NOT been tampered with/modified by unauthorized parties. Achieved via hashing, checksums, digital signatures, version control.
- **Availability** — ensuring authorized users can access information/systems WHEN NEEDED. Threatened by DoS attacks, hardware failures. Achieved via redundancy, backups, disaster recovery planning.

Extended models add **Non-repudiation** (a party cannot deny having performed an action — achieved via digital signatures) and **Authenticity** (verifying the genuine identity/source of data).

## 7.2 Types of Cyber Attacks

- **Malware** — umbrella term for malicious software:
  - **Virus** — attaches itself to a legitimate program/file and REQUIRES human action (running the infected file) to spread; can replicate and corrupt/delete data.
  - **Worm** — SELF-REPLICATING malware that spreads AUTOMATICALLY across networks WITHOUT needing to attach to a file or require human action — this is the key distinguishing feature from a virus.
  - **Trojan Horse** — disguises itself as legitimate/useful software to trick users into installing it; does NOT self-replicate (unlike virus/worm); typically creates a backdoor for attackers.
  - **Ransomware** — encrypts the victim's data and demands payment (ransom) for the decryption key.
  - **Spyware** — secretly monitors/collects user information without consent.
  - **Rootkit** — designed to gain and maintain privileged (root/admin) access to a system while actively hiding its own presence.
  - **Adware** — automatically displays/downloads unwanted advertisements.
  - **Keylogger** — records keystrokes to capture sensitive information like passwords.
- **Phishing** — fraudulent attempt to obtain sensitive information (passwords, credit card details) by disguising as a trustworthy entity, typically via deceptive emails/messages. **Spear phishing** = targeted at a SPECIFIC individual/organization (more personalized, higher success rate). **Whaling** = phishing targeted specifically at HIGH-PROFILE individuals (executives).
- **Denial of Service (DoS)** — floods a system/server with excessive traffic/requests to exhaust its resources, making it UNAVAILABLE to legitimate users (attacks Availability from the CIA triad).
- **Distributed Denial of Service (DDoS)** — a DoS attack launched from MULTIPLE compromised systems (a "botnet") simultaneously, making it harder to block (since traffic comes from many different sources) and typically more powerful.
- **Man-in-the-Middle (MITM)** — an attacker secretly intercepts (and possibly alters) communication between two parties who believe they're communicating directly with each other.
- **SQL Injection** — inserting malicious SQL code into an input field (e.g., a login form) to manipulate/bypass the backend database query — e.g., entering `' OR '1'='1` into a login field to bypass authentication by making the WHERE clause always evaluate to true. Prevention: parameterized queries/prepared statements, input validation/sanitization.
- **Cross-Site Scripting (XSS)** — injecting malicious CLIENT-SIDE scripts (usually JavaScript) into web pages viewed by OTHER users, exploiting a site's failure to properly sanitize user input before displaying it.
- **Cross-Site Request Forgery (CSRF)** — tricks an authenticated user's browser into unknowingly sending a malicious request to a web application they're currently logged into, performing an unwanted action on their behalf.
- **Brute Force Attack** — systematically trying every possible password/key combination until the correct one is found.
- **Zero-Day Attack** — exploits a software vulnerability that is UNKNOWN to the vendor/developer (no patch/fix exists yet at the time of the attack).
- **Social Engineering** — manipulating people (rather than exploiting technical vulnerabilities) into divulging confidential information or performing security-compromising actions.

## 7.3 Authentication

The process of VERIFYING the identity of a user/system.

- **Single-Factor Authentication (SFA)** — uses only ONE method (typically just a password).
- **Two-Factor Authentication (2FA) / Multi-Factor Authentication (MFA)** — requires TWO or more INDEPENDENT categories of verification:
  1. **Something you KNOW** — password, PIN.
  2. **Something you HAVE** — OTP on phone, hardware token, smart card.
  3. **Something you ARE** — biometrics (fingerprint, face recognition, iris scan).
- **Single Sign-On (SSO)** — allows a user to authenticate once and gain access to MULTIPLE independent systems/applications, without re-entering credentials for each.

## 7.4 Encryption / Cryptography Basics

- **Symmetric Key Encryption** — the SAME key is used for both encryption and decryption. Faster, but the key must be securely shared between parties beforehand (key distribution problem). Examples: AES (Advanced Encryption Standard), DES, 3DES.
- **Asymmetric Key Encryption (Public Key Cryptography)** — uses a MATHEMATICALLY LINKED PAIR of keys: a PUBLIC key (shared openly, used to encrypt or verify) and a PRIVATE key (kept secret, used to decrypt or sign). Slower than symmetric, but solves the key distribution problem. Examples: RSA, ECC (Elliptic Curve Cryptography). Used for: digital signatures, SSL/TLS handshake (key exchange), secure email.
- **Hashing** — a ONE-WAY function that converts input data of any size into a FIXED-SIZE output (hash/digest); NOT REVERSIBLE (you cannot get the original data back from the hash), and even a tiny change in input produces a drastically different output (avalanche effect). Used for: password storage (store the hash, not the plaintext password), data integrity verification (checksums). Examples: MD5 (now considered weak/broken for security purposes), SHA-1 (also weakened), SHA-256 (currently widely used and considered secure).
- **Digital Signature** — uses asymmetric cryptography to prove the AUTHENTICITY and INTEGRITY of a message — the sender signs a HASH of the message with their PRIVATE key; anyone can verify it using the sender's PUBLIC key, confirming both that the message came from the claimed sender (authenticity) and that it hasn't been altered (integrity) — also provides non-repudiation.
- **SSL/TLS** — protocols that provide encrypted communication over a network (the padlock icon/HTTPS in browsers); TLS is the modern successor to SSL. Uses asymmetric cryptography during the initial handshake to securely exchange a SYMMETRIC session key, then switches to (faster) symmetric encryption for the actual data transfer — a hybrid approach combining the strengths of both.

## 7.5 Software Development Security

- **Secure SDLC (Software Development Life Cycle)** — integrating security practices at EVERY phase of development (requirements, design, coding, testing, deployment, maintenance), rather than treating security as an afterthought bolted on at the end.
- **Input Validation** — checking that all user-supplied data conforms to expected format/type/range BEFORE processing it — the primary defense against SQL Injection, XSS, buffer overflows, etc.
- **Principle of Least Privilege** — every user/process/system component should have ONLY the minimum level of access/permissions necessary to perform its function, nothing more — limits the damage if that component is compromised.
- **Defense in Depth** — layering MULTIPLE independent security controls, so that if one layer fails/is bypassed, other layers still provide protection.
- **Code Review / Static Application Security Testing (SAST)** — analyzing source code (without executing it) to find security vulnerabilities early.
- **Penetration Testing** — authorized simulated cyberattacks against a system to evaluate its security and find exploitable vulnerabilities BEFORE real attackers do.

## 7.6 Network Security

- **VPN (Virtual Private Network)** — creates an ENCRYPTED tunnel over a public network (like the internet), allowing secure remote access to a private network as if directly connected to it.
- **IDS (Intrusion Detection System)** — MONITORS network/system traffic for suspicious activity/policy violations and ALERTS administrators — passive, detection only, does not block traffic itself.
- **IPS (Intrusion Prevention System)** — like IDS, but also ACTIVELY BLOCKS/PREVENTS detected threats in real time.
- **DMZ (Demilitarized Zone)** — a separate network segment that sits between an internal trusted network and the untrusted external internet, hosting public-facing services (web servers, mail servers) — isolates them so that if compromised, the attacker still doesn't have direct access to the fully internal/trusted network.

## 7.7 Network Audit and Systems Audit

- **Network Audit** — a systematic evaluation of a network's infrastructure, security policies, configurations, and performance, to identify vulnerabilities, ensure compliance with security policies, and optimize the network.
- **Systems Audit (IT/Information Systems Audit)** — an examination and evaluation of an organization's IT systems, infrastructure, operations, and controls, to assess whether they safeguard assets, maintain data integrity, and operate effectively/efficiently in alignment with organizational goals and regulatory requirements (highly relevant to SEBI's own regulatory/compliance role over stock exchanges, depositories, and market infrastructure institutions — this connects directly to SEBI's real-world mandate, so expect scenario-based questions here).
- **Compliance frameworks** commonly referenced (good general awareness): ISO/IEC 27001 (Information Security Management Systems standard), COBIT (Control Objectives for Information and Related Technologies), NIST Cybersecurity Framework.

---

# SECTION 8: DATA WAREHOUSING

## 8.1 What Is a Data Warehouse

A **Data Warehouse (DW)** is a large, centralized repository of INTEGRATED data collected from MULTIPLE SOURCES, specifically designed to support ANALYSIS and REPORTING (business intelligence/decision-making), rather than day-to-day transactional operations.

**Key characteristics (Bill Inmon's classic definition)**:
- **Subject-Oriented** — organized around major business subjects/entities (e.g., "Sales", "Customer"), not around individual operational applications.
- **Integrated** — data pulled from disparate sources is made CONSISTENT (uniform naming, units, formats) before storage.
- **Time-Variant** — data is stored WITH a time dimension, allowing analysis of trends/changes over time (unlike operational systems, which typically only hold current data).
- **Non-Volatile** — once data enters the warehouse, it is NOT updated or deleted in the normal course of operations (only appended/refreshed on a schedule) — it's a stable historical record.

**Data Warehouse vs Operational Database (OLTP):**

| Feature | OLTP (operational DB) | OLAP/Data Warehouse |
|---|---|---|
| Purpose | Day-to-day transaction processing | Analysis, reporting, decision support |
| Data | Current, detailed | Historical, summarized/aggregated |
| Design | Normalized (3NF, for write efficiency) | Denormalized (star/snowflake schema, for read/query efficiency) |
| Operations | Frequent small reads/writes (INSERT/UPDATE) | Complex, large-scale read queries (SELECT/aggregate) |
| Users | Clerks, front-line staff | Analysts, managers, executives |

## 8.2 ETL Process — Extract, Transform, Load

This is the core PIPELINE by which raw operational data becomes usable warehouse data.

1. **Data Extraction** — pulling raw data OUT of various heterogeneous source systems (different databases, flat files, APIs, legacy systems).
2. **Data Cleaning** — identifying and CORRECTING (or removing) inaccurate, incomplete, duplicate, or inconsistent data — e.g., handling missing values, fixing typos, removing duplicate records, standardizing formats (e.g., date formats).
3. **Data Transformation** — converting cleaned data into the FORMAT/STRUCTURE required by the target warehouse — includes aggregation, normalization/standardization of units, joining data from different sources, applying business rules, encoding categorical values.
4. **Data Loading** — writing the transformed data INTO the data warehouse. Can be a **Full Load** (entire dataset loaded, typically only done once initially) or an **Incremental Load** (only new/changed data is loaded periodically, far more common in ongoing operations).

(Note: some modern architectures use **ELT** — Extract, Load, Transform — loading raw data first and transforming it later, typically within a powerful cloud data warehouse — but the classical/exam-tested term is ETL.)

## 8.3 Metadata

**Metadata** is "data about data" — it describes the structure, source, meaning, and usage of the actual data stored in the warehouse (e.g., table/column definitions, data types, source system of origin, last-updated timestamp, business definitions of terms). Critical for data governance, lineage tracking, and helping users understand what the data actually means.

## 8.4 Data Cube (OLAP Cube)

A **data cube** is a multi-dimensional structure that allows data to be modeled and viewed across MULTIPLE DIMENSIONS simultaneously (e.g., Sales data viewed by Product × Time × Region — three dimensions). Enables fast, flexible analysis via OLAP operations:

- **Roll-up (Drill-up)** — aggregating data by CLIMBING UP a concept hierarchy (e.g., summarizing daily sales into monthly, then yearly sales) or by reducing a dimension.
- **Drill-down** — the OPPOSITE of roll-up — breaking aggregated data down into more DETAILED/granular levels (e.g., yearly sales broken down into monthly, then daily).
- **Slice** — selecting a single value for ONE dimension, reducing the cube's dimensionality by one (e.g., viewing sales data for only "2024").
- **Dice** — selecting a SUB-CUBE by specifying ranges/values for TWO OR MORE dimensions simultaneously (e.g., viewing sales for "2024" AND "Region=North").
- **Pivot (Rotate)** — reorienting the multidimensional view of the data to see it from a different perspective (e.g., swapping rows and columns in a cross-tabulation).

## 8.5 Data Mart

A **Data Mart** is a SUBSET of a data warehouse, focused on a SPECIFIC business area/department/subject (e.g., a "Sales Data Mart" or "Finance Data Mart"), rather than the entire organization. Data marts are typically smaller, faster to implement, and more targeted for a specific group of users compared to a full enterprise data warehouse.
- **Dependent Data Mart** — sourced FROM an existing central data warehouse (top-down approach — build the warehouse first, then carve out marts from it).
- **Independent Data Mart** — built directly from operational source systems WITHOUT going through a central warehouse first (bottom-up approach — Ralph Kimball's methodology, sometimes leading to the warehouse being built as a UNION of conformed data marts).

## 8.6 Data Warehouse Schema Models

- **Star Schema** — a central **FACT table** (containing measurable, quantitative business data — e.g., Sales_Amount, Quantity_Sold) is directly connected to multiple **DIMENSION tables** (containing descriptive attributes — e.g., Product, Customer, Time, Store), forming a shape resembling a star. Dimension tables in a pure star schema are DENORMALIZED (not further broken down) — simpler, faster to query, but with some data redundancy.
- **Snowflake Schema** — an EXTENSION of the star schema where dimension tables are further NORMALIZED into multiple related sub-dimension tables (e.g., a Product dimension might be split into Product, Category, and Sub-Category tables) — reduces redundancy/storage, but requires MORE JOINS for queries, making it comparatively slower/more complex than a star schema.
- **Fact Constellation Schema (Galaxy Schema)** — MULTIPLE fact tables that SHARE some common dimension tables between them — used for more complex data warehouses covering multiple related business processes.

**Fact table types:**
- **Additive facts** — can be meaningfully summed across ALL dimensions (e.g., Sales_Amount).
- **Semi-additive facts** — can be summed across SOME dimensions but not others (e.g., Account_Balance can be summed across accounts, but summing it across TIME doesn't make sense — you'd want the balance AT a point in time, not a sum over months).
- **Non-additive facts** — cannot be meaningfully summed across ANY dimension (e.g., a Ratio or Percentage value).

---

# SECTION 9: SHELL PROGRAMMING

## 9.1 Shell Scripting Basics

A **shell script** is a text file containing a sequence of commands for a UNIX/Linux shell (like `bash`) to execute, used for automating repetitive tasks.

```bash
#!/bin/bash
# The "shebang" line above tells the OS which interpreter to use to run this script

echo "Hello, World!"
```

Making a script executable and running it:
```bash
chmod +x script.sh     # grant execute permission
./script.sh             # run the script
bash script.sh           # alternative way to run without needing execute permission
```

## 9.2 Shell Variables

```bash
name="Ramesh"          # NO SPACES around '=' — a very common syntax trap in shell scripting
echo "$name"            # access variable using $ prefix
echo "Hello, $name!"     # variable interpolation inside double-quoted strings

# Variables are UNTYPED (treated as strings by default) unless declared otherwise
count=10
count=$((count + 1))     # arithmetic expansion for numeric operations
echo $count               # 11

readonly PI=3.14           # creates a read-only/constant variable
unset name                  # deletes/unsets a variable
```

**Special/built-in variables:**
- `$0` — name of the script itself.
- `$1, $2, $3, ...` — positional parameters (command-line arguments passed to the script).
- `$#` — number of arguments passed.
- `$@` — all arguments as SEPARATE quoted strings.
- `$*` — all arguments as a SINGLE string.
- `$?` — exit status of the LAST executed command (0 = success, non-zero = some kind of failure/error).
- `$$` — process ID (PID) of the current script.

## 9.3 Shell Script Arguments

```bash
#!/bin/bash
echo "Script name: $0"
echo "First argument: $1"
echo "Total arguments: $#"
echo "All arguments: $@"
```
Run as: `./script.sh apple banana` → `$1` = "apple", `$2` = "banana", `$#` = 2.

## 9.4 If Statement (Conditionals)

```bash
#!/bin/bash
num=10
if [ $num -gt 5 ]; then
    echo "Greater than 5"
elif [ $num -eq 5 ]; then
    echo "Equal to 5"
else
    echo "Less than 5"
fi
```

**Comparison operators in `[ ]` (test command):**

| Numeric | Meaning | String | Meaning |
|---|---|---|---|
| `-eq` | equal | `=` or `==` | equal |
| `-ne` | not equal | `!=` | not equal |
| `-gt` | greater than | `-z` | string is empty |
| `-lt` | less than | `-n` | string is not empty |
| `-ge` | greater or equal | | |
| `-le` | less or equal | | |

File test operators: `-f file` (is a regular file), `-d file` (is a directory), `-e file` (exists), `-r/-w/-x` (readable/writable/executable).

**Trap**: shell `[ ]` syntax REQUIRES spaces around the brackets and operators (`[ $num -gt 5 ]`, not `[$num -gt 5]` or `[$num-gt5]`) — this whitespace-sensitivity is a classic beginner mistake and a fair exam trap.

## 9.5 Loops

```bash
# for loop
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

# for loop with a range (C-style, bash-specific)
for ((i=0; i<5; i++)); do
    echo $i
done

# while loop
count=1
while [ $count -le 5 ]; do
    echo $count
    count=$((count + 1))
done

# until loop (opposite of while - runs UNTIL condition becomes true)
count=1
until [ $count -gt 5 ]; do
    echo $count
    count=$((count + 1))
done
```

## 9.6 Functions and Return

```bash
#!/bin/bash
greet() {
    echo "Hello, $1!"
    return 0        # return an EXIT STATUS (0-255 only, NOT arbitrary values/strings)
}
greet "Ramesh"
echo "Exit status: $?"
```
**Trap**: unlike most programming languages, shell functions' `return` value is restricted to an integer EXIT STATUS between 0 and 255 — it is NOT meant for returning actual computed DATA (like a string or a large number). To "return" actual data, the convention is to `echo` the value and CAPTURE it using command substitution: `result=$(greet "Ramesh")`.

## 9.7 Basic UNIX/Linux Commands (High-Yield List)

| Command | Purpose |
|---|---|
| `ls` | list directory contents (`-l` long format, `-a` show hidden files) |
| `cd` | change directory |
| `pwd` | print current/working directory |
| `mkdir` | create a directory |
| `rmdir` | remove an EMPTY directory |
| `rm` | remove files (`-r` recursive for directories, `-f` force) |
| `cp` | copy files/directories |
| `mv` | move or rename files |
| `cat` | display file content (also used to concatenate files) |
| `grep` | search for a pattern within file(s) — extremely commonly tested |
| `find` | search for files/directories matching criteria |
| `chmod` | change file permissions |
| `chown` | change file owner |
| `ps` | display currently running processes |
| `kill` | terminate a process by PID |
| `top` | real-time view of running processes/system resource usage |
| `man` | display the manual/help page for a command |
| `head` / `tail` | display the first/last N lines of a file (`tail -f` follows a live-growing file, e.g., a log) |
| `sort` | sort lines of a file |
| `uniq` | remove/report adjacent DUPLICATE lines (typically used after `sort`) |
| `wc` | word/line/character count (`wc -l` counts lines) |
| `awk` | powerful pattern-scanning and text-processing tool/language |
| `sed` | stream editor for filtering/transforming text (commonly used for find-and-replace) |
| `chmod 755 file` | owner: read+write+execute (7), group: read+execute (5), others: read+execute (5) — permission numbers: read=4, write=2, execute=1, summed per category |

**File permission trap**: `chmod 755` means owner=rwx(4+2+1=7), group=r-x(4+0+1=5), others=r-x(4+0+1=5). Know how to READ a permission number, since this is asked directly sometimes ("what does chmod 644 mean?" → owner: read+write(6), group: read-only(4), others: read-only(4)).

---

# SECTIONS 6-9 QUICK REVISION TABLE

| Concept | Key Fact |
|---|---|
| OSI Layer for Router | Network Layer (3), uses IP |
| OSI Layer for Switch | Data Link Layer (2), uses MAC |
| TCP vs UDP | TCP=reliable/connection-oriented; UDP=fast/connectionless |
| CIA Triad | Confidentiality, Integrity, Availability |
| Virus vs Worm | Virus needs a host file + human action; Worm self-replicates automatically over network |
| Symmetric vs Asymmetric encryption | Symmetric=same key, faster; Asymmetric=key pair, solves distribution problem |
| Star vs Snowflake schema | Star=denormalized dims, faster query; Snowflake=normalized dims, less redundancy, more joins |
| ETL | Extract, Transform (clean+convert), Load |
| Shell `$?` | Exit status of last command |
| Shell `$#` | Number of arguments |
| `chmod 755` | rwx for owner, r-x for group and others |


---
---

# PHASE II — DEEP DIVE (NUMERICAL / CODE-TRACING STYLE)

> Phase II questions are NOT conceptual definitions — they give you an array, a piece of code, or a tree and ask you to COMPUTE the exact answer. This is exactly the style of all 69 sample questions you provided. From here on, every subtopic includes at least one full worked numerical trace, because that is what actually gets tested.

# SECTION 10: ALGORITHMS (Phase II Deep Dive) — 30% Weightage

## 10.1 Sorting — Full Worked Traces

### 10.1.1 Bubble Sort — Full Trace

Array: `[5, 1, 4, 2, 8]`

**Pass 1** (compare adjacent pairs, swap if left > right):
- (5,1) → swap → [1,5,4,2,8]
- (5,4) → swap → [1,4,5,2,8]
- (5,2) → swap → [1,4,2,5,8]
- (5,8) → no swap → [1,4,2,5,8]
End of Pass 1: `[1,4,2,5,8]` (largest element 8 correctly bubbled to the end)

**Pass 2:**
- (1,4) no swap
- (4,2) swap → [1,2,4,5,8]
- (4,5) no swap
- (5,8) no swap (this comparison is often skipped in an OPTIMIZED version, since the last element is already sorted)
End of Pass 2: `[1,2,4,5,8]`

**Pass 3:** (1,2) no swap, (2,4) no swap, (4,5) no swap → already sorted, no swaps at all.

**Number of comparisons (unoptimized, worst case)**: for n elements, total comparisons = n(n-1)/2. For n=5: 5×4/2 = 10 comparisons total across all passes (this directly explains sample Q4's style of question — always compute using n(n-1)/2 for worst case, but note the ACTUAL number depends on early-exit optimization if the array becomes sorted early).

**Optimized bubble sort**: add a flag — if NO swaps occur during a full pass, the array is already sorted, so you can STOP early (best case becomes O(n) for an already-sorted array, since only one pass with zero swaps is needed to detect this and exit).

### 10.1.2 Selection Sort — Full Trace

Array: `[29, 10, 14, 37, 13]`

**Pass 1**: find minimum in [29,10,14,37,13] → 10 (index 1). Swap with index 0 → `[10, 29, 14, 37, 13]`
**Pass 2**: find minimum in [29,14,37,13] (indices 1-4) → 13 (index 4). Swap with index 1 → `[10, 13, 14, 37, 29]`
**Pass 3**: find minimum in [14,37,29] (indices 2-4) → 14 already at index 2, no swap needed → `[10, 13, 14, 37, 29]`
**Pass 4**: find minimum in [37,29] (indices 3-4) → 29 (index 4). Swap with index 3 → `[10, 13, 14, 29, 37]`
Sorted: `[10, 13, 14, 29, 37]`

Always does exactly n(n-1)/2 COMPARISONS regardless of input (must scan the remaining unsorted portion every pass to find the min), but the number of SWAPS is at most n-1 (much fewer than bubble sort in the worst case) — this is why selection sort is preferred when swap/write operations are expensive (e.g., writing to flash memory), even though its comparison count is the same order as bubble sort.

### 10.1.3 Insertion Sort — Full Trace

Array: `[12, 11, 13, 5, 6]`

Start: sorted portion = [12]. Take 11 → compare with 12, 11<12 so shift 12 right, insert 11 at front → `[11,12,13,5,6]` (wait — recompute carefully)

Let's redo cleanly, one element inserted at a time into the growing sorted prefix:
- i=1, key=11: compare with 12 (sorted[0]). 11<12 → shift 12 right → insert 11 at position 0. Array: `[11, 12, 13, 5, 6]`
- i=2, key=13: compare with 12. 13>12 → no shift needed, 13 stays. Array: `[11, 12, 13, 5, 6]`
- i=3, key=5: compare with 13 (shift right), 12 (shift right), 11 (shift right) — 5 is smaller than all of them → insert at position 0. Array: `[5, 11, 12, 13, 6]`
- i=4, key=6: compare with 13 (shift), 12 (shift), 11 (shift), 5 (5<6, stop) → insert 6 right after 5. Array: `[5, 6, 11, 12, 13]`

Final sorted: `[5, 6, 11, 12, 13]`. Best case (already sorted array): O(n) — only one comparison per element needed, no shifting. Worst case (reverse sorted): O(n²).

### 10.1.4 Merge Sort — Full Trace

Array: `[38, 27, 43, 3, 9, 82, 10]`

**Divide** (recursively split in half):
```
[38,27,43,3,9,82,10]
  ->  [38,27,43,3]        [9,82,10]
  -> [38,27] [43,3]      [9,82] [10]
  -> [38][27] [43][3]    [9][82] [10]
```
**Conquer/Merge** (merge sorted halves back together, comparing front elements):
```
[38][27] -> merge -> [27,38]
[43][3]  -> merge -> [3,43]
[27,38][3,43] -> merge -> [3,27,38,43]

[9][82] -> merge -> [9,82]
[10] stays -> [10]
[9,82][10] -> merge -> [9,10,82]

[3,27,38,43][9,10,82] -> merge -> [3,9,10,27,38,43,82]
```
Final sorted array: `[3, 9, 10, 27, 38, 43, 82]`

**Merge step mechanics** (how two sorted arrays are merged): keep two pointers, one at the start of each sub-array; compare the elements they point to; copy the SMALLER one into the result and advance that pointer; repeat until one sub-array is exhausted, then copy the remainder of the other directly.

### 10.1.5 Quick Sort — Full Trace (pivot = last element, Lomuto partition scheme)

Array: `[10, 80, 30, 90, 40, 50, 70]`, pivot = 70 (last element)

Partition process: maintain index `i` for the "boundary of elements smaller than pivot"; scan `j` from start to second-last element:
- j=0, arr[j]=10 < 70 → i=0, swap arr[i] and arr[j] (no-op, same position) → `[10,80,30,90,40,50,70]`
- j=1, arr[j]=80, not < 70 → skip
- j=2, arr[j]=30 < 70 → i=1, swap arr[1] and arr[2] → `[10,30,80,90,40,50,70]`
- j=3, arr[j]=90, not <70 → skip
- j=4, arr[j]=40<70 → i=2, swap arr[2] and arr[4] → `[10,30,40,90,80,50,70]`
- j=5, arr[j]=50<70 → i=3, swap arr[3] and arr[5] → `[10,30,40,50,80,90,70]`
- End of scan. Swap pivot (arr[last]) with arr[i+1] = arr[4] → `[10,30,40,50,70,90,80]`
- Pivot 70 is now at its correct final sorted position (index 4). Everything left of it (10,30,40,50) is smaller, everything right (90,80) is larger.

Recursively quicksort the left part `[10,30,40,50]` and right part `[90,80]` the same way, until fully sorted: `[10,30,40,50,70,80,90]`.

**Worst case scenario, concretely**: if the array is `[1,2,3,4,5]` (already sorted) and you always pick the LAST element as pivot, then every partition step produces one side completely EMPTY and the other side with n-1 elements — this degrades to O(n²), exactly like the sample paper's Q9 answer.

### 10.1.6 Heap Sort — Concept + Trace

**Step 1: Build a Max-Heap** from the array (see Section 11.7 for heap mechanics). For `[4, 10, 3, 5, 1]`:
- Array as a complete binary tree: root=4, children=10,3; 10's children=5,1.
- Heapify from the last non-leaf node upward: index for last non-leaf = (n/2)-1 = 1 (0-indexed), which is value 10. 10's children are 5,1 — 10 is already bigger than both, no change.
- Move to index 0 (value 4): children are 10 and 3. Largest child is 10 → swap 4 and 10 → `[10, 4, 3, 5, 1]`. Now heapify down from the new position of 4 (index 1): children are 5,1. 5>4 → swap → `[10, 5, 3, 4, 1]`.
- Max-Heap built: `[10, 5, 3, 4, 1]`

**Step 2: Repeatedly extract max** — swap root with last element, shrink heap size by 1, heapify down the new root:
- Swap arr[0]=10 with arr[4]=1 → `[1,5,3,4,10]`, heap size=4. Heapify `[1,5,3,4]`: children of 1 are 5,3, largest=5, swap → `[5,1,3,4]`, then check 1's new position(index1): child is 4(index3), 4>1, swap → `[5,4,3,1]`.
- Swap arr[0]=5 with arr[3]=1 (last active) → `[1,4,3,5,10]`, heap size=3. Heapify `[1,4,3]`: children of 1 are 4,3, largest=4, swap → `[4,1,3]`.
- Swap arr[0]=4 with arr[2]=3 → `[3,1,4,5,10]`, heap size=2. Heapify `[3,1]`: child of 3 is 1, no swap needed.
- Swap arr[0]=3 with arr[1]=1 → `[1,3,4,5,10]`, heap size=1. Done.

Final sorted array: `[1, 3, 4, 5, 10]`

## 10.2 Searching — Full Worked Traces

### 10.2.1 Binary Search — Trace

Sorted array: `[2, 5, 8, 12, 16, 23, 38, 45, 56, 72, 91]` (indices 0-10). Find target = 23.

- low=0, high=10, mid=5 → arr[5]=23 → **FOUND at index 5** (this example resolves in one step; let's also trace a multi-step search)

Find target = 91:
- low=0, high=10, mid=5 → arr[5]=23. 91>23 → search right half → low=6
- low=6, high=10, mid=8 → arr[8]=56. 91>56 → search right → low=9
- low=9, high=10, mid=9 → arr[9]=72. 91>72 → search right → low=10
- low=10, high=10, mid=10 → arr[10]=91 → **FOUND at index 10**

Total comparisons for a sorted array of n elements: at most ⌈log₂(n+1)⌉.

### 10.2.2 Exponential Search — Trace

Sorted array (large, e.g., 20 elements), target near the start, say target is at index 3.
- Start with bound=1: arr[1] — if target > arr[1], double bound → bound=2
- arr[2] — if target > arr[2], double → bound=4
- arr[4] — if target <= arr[4], we now know target lies within range [bound/2, bound] = [2,4]
- Perform BINARY SEARCH within just that small range [2,4] to pinpoint the exact index (3).
This confirms why exponential search is efficient for targets near the beginning — it finds the bounding range in O(log(position)) time rather than needing the full O(log n) of a standard binary search starting from the middle of the whole array.

## 10.3 Graph Algorithms — Numerical Traces

### 10.3.1 Dijkstra's Algorithm — Full Trace

Graph (adjacency, weighted, undirected): A-B(4), A-C(1), C-B(2), B-D(5), C-D(8), D-E(3). Find shortest paths from source A.

Initialize: dist[A]=0, dist[B]=∞, dist[C]=∞, dist[D]=∞, dist[E]=∞. Visited = {}

**Step 1**: pick unvisited with min dist → A(0). Relax neighbors: B: 0+4=4 (update dist[B]=4). C: 0+1=1 (update dist[C]=1). Visited={A}.

**Step 2**: pick min unvisited → C(1). Relax neighbors: B: 1+2=3 < current 4 → update dist[B]=3. D: 1+8=9 (update dist[D]=9). Visited={A,C}.

**Step 3**: pick min unvisited → B(3). Relax neighbors: D: 3+5=8 < current 9 → update dist[D]=8. Visited={A,C,B}.

**Step 4**: pick min unvisited → D(8). Relax neighbors: E: 8+3=11 (update dist[E]=11). Visited={A,C,B,D}.

**Step 5**: pick min unvisited → E(11). No unvisited neighbors left. Visited={A,C,B,D,E}.

**Final shortest distances from A**: A=0, B=3 (via A→C→B), C=1 (via A→C), D=8 (via A→C→B→D), E=11 (via A→C→B→D→E).

### 10.3.2 Prim's Algorithm — Full MST Trace

Same graph as above. Start MST from A.

- MST = {A}. Candidate edges from A: A-B(4), A-C(1). Pick minimum → A-C(1). MST={A,C}, MST edges={A-C}.
- Candidate edges from {A,C} to outside: A-B(4), C-B(2), C-D(8). Pick minimum → C-B(2). MST={A,C,B}, edges={A-C, C-B}.
- Candidate edges from {A,C,B}: B-D(5), C-D(8) [A-B(4) is now internal, ignore]. Pick minimum → B-D(5). MST={A,C,B,D}, edges={A-C,C-B,B-D}.
- Candidate edges from {A,C,B,D}: D-E(3). Pick → D-E(3). MST={A,C,B,D,E}, edges={A-C,C-B,B-D,D-E}.

**Total MST weight**: 1+2+5+3 = 11. MST edges: A-C, C-B, B-D, D-E.

### 10.3.3 Kruskal's Algorithm — Same Graph, Trace

Sort all edges ascending by weight: A-C(1), C-B(2), D-E(3), A-B(4), B-D(5), C-D(8).

Process each edge, add if it does NOT form a cycle (using Union-Find):
- A-C(1): A and C in different components → add. MST edges={A-C}. Components: {A,C}, {B}, {D}, {E}.
- C-B(2): C and B in different components → add. MST edges={A-C, C-B}. Components: {A,C,B}, {D}, {E}.
- D-E(3): different components → add. MST edges={A-C,C-B,D-E}. Components: {A,C,B}, {D,E}.
- A-B(4): A and B are ALREADY in the SAME component {A,C,B} → would form a cycle → SKIP.
- B-D(5): B in {A,C,B}, D in {D,E} → different components → add. MST edges={A-C,C-B,D-E,B-D}. Components merge: {A,C,B,D,E}. All vertices connected with V-1=4 edges → STOP.

**Result**: same MST as Prim's (A-C, C-B, D-E, B-D), total weight = 1+2+3+5 = 11. (For a graph with UNIQUE edge weights, the MST is always unique regardless of algorithm used — this is a good fact to remember for exam traps.)

## 10.4 Dynamic Programming — Worked Examples

### 10.4.1 0/1 Knapsack Problem

Items: weights=[1,3,4,5], values=[1,4,5,7]. Capacity W=7.

Build a DP table `dp[i][w]` = max value using first `i` items with capacity `w`. Recurrence:
`dp[i][w] = max(dp[i-1][w], value[i] + dp[i-1][w-weight[i]])` if weight[i] <= w, else `dp[i][w] = dp[i-1][w]`.

Working through it: with items (1,1),(3,4),(4,5),(5,7) [weight,value] and capacity 7, the optimal solution takes items with weight 3 (value 4) and weight 4 (value 5), total weight=7, total value=9 — this beats other combinations like weight 1+5=6 (value 1+7=8) or weight 1+3=4(value 5, leaves 3 capacity unused productively). **Maximum value = 9.**

### 10.4.2 Fibonacci with Memoization (illustrates DP's core idea — avoiding recomputation)

```python
def fib(n, memo={}):
    if n in memo: return memo[n]         # overlapping subproblem already solved - reuse it
    if n <= 1: return n
    memo[n] = fib(n-1, memo) + fib(n-2, memo)
    return memo[n]
```
Without memoization, naive recursive `fib(n) = fib(n-1) + fib(n-2)` has exponential time O(2^n), because `fib(3)`, `fib(2)`, etc. get recomputed many times independently (classic OVERLAPPING SUBPROBLEMS). With memoization, each subproblem `fib(k)` is computed exactly ONCE and cached, bringing time down to O(n).

### 10.4.3 Longest Common Subsequence (LCS) — Trace

Strings: X = "ABCBDAB", Y = "BDCABA". 

Build a DP table where `dp[i][j]` = length of LCS of X[0..i-1] and Y[0..j-1]. Recurrence: if X[i-1]==Y[j-1], `dp[i][j] = dp[i-1][j-1] + 1`; else `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`.

The LCS of "ABCBDAB" and "BDCABA" turns out to be "BCBA" or "BDAB" (length 4) — **LCS length = 4**. (For the exam, focus on knowing the RECURRENCE and being able to fill a small DP table by hand more than memorizing this specific answer.)

## 10.5 Backtracking — N-Queens Trace (4-Queens)

Place 4 queens on a 4×4 board so no two attack each other (same row, column, or diagonal).

- Try Q1 at (row0, col0). Try Q2 at (row1, col0) → same column, invalid. Try (row1,col1) → diagonal, invalid. Try (row1,col2) → valid (not same row/col/diag as Q1). Place Q2 at (1,2).
- Try Q3 at (row2, col0) → check against Q1(0,0): same column, invalid. Try (2,1) → diagonal with Q2(1,2)? diff row=1,diff col=1 → diagonal, invalid. Try (2,3) → check Q1(0,0): ok. check Q2(1,2): diff row=1, diff col=1 → diagonal, invalid. NO valid position for Q3 in row 2 → BACKTRACK, move Q2.
- Try Q2 at (row1, col3) instead. Try Q3 at (row2,col0): check Q1(0,0) same col, invalid. Try (2,1): check Q1 ok, check Q2(1,3): diff row1,diff col2, not diagonal, OK → place Q3 at (2,1).
- Try Q4 at (row3,col*): col0: same col as Q1, invalid. col1: same col as Q3, invalid. col2: check Q1(0,0)ok, Q2(1,3): diff row2,diff col1,not diag,ok, Q3(2,1): diff row1 diffcol1 → diagonal, invalid. col3: same col as Q2, invalid. NO valid position → BACKTRACK further...

(Eventually, this systematic try-fail-backtrack process finds the two valid solutions for 4-Queens: Q at (0,1),(1,3),(2,0),(3,2) and its mirror (0,2),(1,0),(2,3),(3,1).) The key exam takeaway is the MECHANISM: try a placement → check constraints → if invalid, try next option in that position → if ALL options in a position are exhausted, BACKTRACK to the previous queen and try ITS next option.

## 10.6 Complexity Analysis — Big-O Cheat Sheet

| Notation | Meaning |
|---|---|
| O (Big-O) | Upper bound — worst case growth rate |
| Ω (Omega) | Lower bound — best case growth rate |
| Θ (Theta) | Tight bound — when upper and lower bounds match |

**Common complexity classes, from fastest to slowest growth:**
O(1) constant < O(log n) logarithmic < O(n) linear < O(n log n) linearithmic < O(n²) quadratic < O(n³) cubic < O(2^n) exponential < O(n!) factorial

**Quick complexity lookup table for common operations:**

| Operation | Complexity |
|---|---|
| Array access by index | O(1) |
| Array search (unsorted) | O(n) |
| Binary search (sorted array) | O(log n) |
| Hash table average lookup | O(1) |
| Hash table worst-case lookup | O(n) (all collisions) |
| BST search (balanced) | O(log n) |
| BST search (unbalanced/skewed) | O(n) |
| Sorting (comparison-based, best possible) | O(n log n) |

---

# SECTION 10 QUICK REVISION TABLE

| Algorithm | Trace Takeaway |
|---|---|
| Bubble sort | Adjacent swaps, largest bubbles to end each pass |
| Selection sort | Find min, swap to front, every pass |
| Insertion sort | Insert into sorted prefix, shifting larger elements right |
| Merge sort | Split fully, then merge pairs comparing fronts |
| Quick sort | Partition around pivot; worst case = already sorted with naive pivot choice |
| Dijkstra | Greedy, pick min unvisited distance, relax neighbors |
| Prim's | Grow tree, always pick min edge crossing the boundary |
| Kruskal's | Sort all edges, add if no cycle (Union-Find) |
| DP | Optimal substructure + overlapping subproblems; build table bottom-up or memoize top-down |
| Backtracking | Try, check constraint, backtrack on failure |


---

# SECTION 11: DATA STRUCTURES (Phase II Deep Dive) — 40% Weightage, HIGHEST PRIORITY SECTION

## 11.1 Arrays

An array stores elements of the SAME type in CONTIGUOUS memory locations (matches sample Q59), which is exactly what enables O(1) random access — the address of element `i` can be computed directly: `base_address + i × size_of_element`, no traversal needed.

| Operation | Time Complexity | Why |
|---|---|---|
| Access by index | O(1) | Direct address computation |
| Search (unsorted) | O(n) | Must check each element |
| Search (sorted, binary search) | O(log n) | Halving strategy |
| Insertion at end (with space) | O(1) | Just place it |
| Insertion at beginning/middle | O(n) | Must shift all subsequent elements right |
| Deletion at end | O(1) | Just remove it |
| Deletion at beginning/middle | O(n) | Must shift all subsequent elements left |

**2D Arrays / Matrix storage**: stored in either **Row-Major Order** (all of row 0, then all of row 1, etc. — used by C, C++, Java, Python) or **Column-Major Order** (all of column 0, then column 1, etc. — used by Fortran, MATLAB, R). 

**Address calculation formula (frequently tested numerically)**: for a 2D array `A[m][n]` with base address `B` and element size `w`, in Row-Major order: `Address(A[i][j]) = B + w × (i × n + j)`. In Column-Major order: `Address(A[i][j]) = B + w × (j × m + i)`.

**Worked example**: Array A[10][20], base address 1000, each element takes 4 bytes, ROW-MAJOR order. Find address of A[3][5]:
`Address = 1000 + 4 × (3×20 + 5) = 1000 + 4×65 = 1000+260 = 1260`

## 11.2 Linked List

A linked list is a linear data structure where elements (**nodes**) are NOT stored contiguously — each node contains DATA plus a POINTER/REFERENCE to the next node. This trades away O(1) random access for O(1) insertion/deletion at known positions (no shifting needed).

### 11.2.1 Types

- **Singly Linked List** — each node points only to the NEXT node; traversal is one-directional (forward only).
- **Doubly Linked List** — each node has pointers to BOTH the next AND previous node — allows bidirectional traversal, but uses more memory (extra pointer per node).
- **Circular Linked List** — the LAST node points back to the FIRST node (instead of NULL), forming a loop — useful for round-robin scheduling type applications.

### 11.2.2 Complexity Table

| Operation | Singly Linked List | Array |
|---|---|---|
| Access by index | O(n) — must traverse from head | O(1) |
| Insert at beginning | O(1) — matches sample Q62 | O(n) |
| Insert at end (no tail pointer) | O(n) — must traverse to find last node | O(1) amortized (with space) |
| Insert at end (with tail pointer) | O(1) | O(1) amortized |
| Delete first node | O(1) | O(n) |
| Delete LAST node (singly linked, no tail-prev tracking) | O(n) — matches sample Q63; must traverse to the SECOND-LAST node to update its `next` pointer to NULL | O(1) |
| Search | O(n) | O(n) unsorted, O(log n) if sorted |

**Why deleting the last node of a SINGLY linked list is O(n)** (sample Q63): you need to set the second-to-last node's `next` pointer to NULL, but in a singly linked list you can only move FORWARD, so you must start from the head and walk all the way to the second-last node — there's no way to jump backward. (In a DOUBLY linked list with a tail pointer, this same operation becomes O(1), since you can go directly to the tail and then step back ONE using the `prev` pointer.)

### 11.2.3 Node Structure and Basic Operations (Code)

```c
struct Node {
    int data;
    struct Node* next;
};

// Insert at beginning - O(1)
struct Node* insertAtBeginning(struct Node* head, int val) {
    struct Node* newNode = (struct Node*)malloc(sizeof(struct Node));
    newNode->data = val;
    newNode->next = head;   // new node points to old head
    return newNode;          // new node becomes the new head
}

// Reverse a singly linked list - classic exam question
struct Node* reverse(struct Node* head) {
    struct Node *prev = NULL, *curr = head, *next = NULL;
    while (curr != NULL) {
        next = curr->next;   // save next before overwriting
        curr->next = prev;    // reverse the pointer
        prev = curr;           // advance prev
        curr = next;            // advance curr
    }
    return prev;   // prev is now the new head
}
```

**Trace of reverse on list 1→2→3→NULL:**
- Initial: prev=NULL, curr=1, next=NULL
- Iter1: next=2, 1->next=NULL (prev), prev=1, curr=2. List so far: 1→NULL
- Iter2: next=3, 2->next=1, prev=2, curr=3. List so far: 2→1→NULL
- Iter3: next=NULL, 3->next=2, prev=3, curr=NULL. List so far: 3→2→1→NULL
- Loop ends (curr=NULL). Return prev=3. New head is 3, list is 3→2→1→NULL. ✓ Correctly reversed.

**Detecting a cycle in a linked list — Floyd's Cycle Detection (Tortoise and Hare)**: use two pointers, `slow` (moves 1 step at a time) and `fast` (moves 2 steps at a time). If there's a cycle, `fast` will eventually "lap" `slow` and they will meet at the same node; if there's no cycle, `fast` will reach NULL first. This is one of the most classic linked-list interview/exam questions.

## 11.3 Stack

A **LIFO (Last In, First Out)** data structure — the last element added is the first one removed. Think of a stack of plates.

**Core operations (all O(1)):**
- `push(x)` — add element x to the top.
- `pop()` — remove and return the top element.
- `peek()`/`top()` — view the top element without removing it.
- `isEmpty()` — check if the stack has no elements.

**Applications** (heavily tested — know these by heart): 
- Function call management (the **call stack** — matches sample Q50).
- Expression evaluation and conversion (infix ↔ postfix ↔ prefix).
- Undo/Redo functionality in editors.
- Balanced parentheses/bracket matching.
- Backtracking algorithms (DFS uses an explicit or implicit/recursive stack).
- Browser back button history.

### 11.3.1 Infix, Prefix, Postfix Conversion — Frequently Tested

- **Infix**: operator BETWEEN operands. `A + B`
- **Prefix (Polish notation)**: operator BEFORE operands. `+ A B`
- **Postfix (Reverse Polish notation)**: operator AFTER operands. `A B +`

**Prefix to Postfix conversion (matches sample Q66 exactly)**: 
Given prefix: `+ p q - s t * ` — wait, let's use the EXACT sample question: `+ p q - s t *`. Let's parse right to left is wrong for prefix; for PREFIX we scan LEFT TO RIGHT is also not standard — the standard technique is to scan the prefix expression from RIGHT TO LEFT, and use a stack:
- If the symbol is an OPERAND, push it onto the stack.
- If the symbol is an OPERATOR, POP two operands (call them op1 = first pop, op2 = second pop), form the string `(op1 op2 operator)`, and push this combined string back.

Actually the cleanest way to verify the sample answer is to first recover the INFIX meaning: prefix `+ p q - s t` reads as `(+ p q)` combined with... let's carefully parse: `+ p q - s t` — reading left to right: `+` needs two operands: first is `p`... but next token is `q` which could be the second operand of `+`, giving `(+ p q)`. Then remaining `- s t` is a separate prefix expression `(- s t)`. But then there's a trailing `*` which needs two operands — those two operands are exactly the two expressions we just built: `(+pq)` and `(-st)`. So the full expression is: `* (+ p q) (- s t)`, meaning **infix = (p + q) * (s − t)**.

**Postfix of this**: postfix of `(p+q)` is `p q +`. Postfix of `(s-t)` is `s t -`. Combined with `*` LAST (since multiplication is the outermost/last operation): **postfix = p q + s t - \*** — which exactly matches the sample paper's answer: "p q + s t - \*".

**General conversion algorithm using a stack (Infix to Postfix — Shunting Yard, simplified)**:
1. Scan infix expression left to right.
2. If operand → add directly to output.
3. If `(` → push onto stack.
4. If `)` → pop from stack to output until `(` is found, discard the `(`.
5. If operator → pop from stack to output all operators with GREATER OR EQUAL precedence, then push the current operator.
6. At the end, pop all remaining operators from stack to output.

### 11.3.2 Balanced Parentheses Checking (Classic Code)

```python
def is_balanced(expr):
    stack = []
    pairs = {')': '(', ']': '[', '}': '{'}
    for char in expr:
        if char in '([{':
            stack.append(char)
        elif char in ')]}':
            if not stack or stack.pop() != pairs[char]:
                return False
    return len(stack) == 0
```

## 11.4 Queue

A **FIFO (First In, First Out)** data structure — the first element added is the first one removed. Think of a queue/line at a ticket counter.

**Core operations (all O(1) with proper implementation):**
- `enqueue(x)` — add element x to the REAR.
- `dequeue()` — remove and return the element from the FRONT.
- `peek()`/`front()` — view the front element without removing it.

**Types of Queues:**
- **Simple Queue** — basic FIFO as described.
- **Circular Queue** — the rear wraps around to reuse empty space at the front (freed by earlier dequeues) instead of leaving it permanently unused — solves the "false full" problem of a naive array-based simple queue.
- **Priority Queue** — elements are dequeued based on PRIORITY, not strictly insertion order — typically implemented using a HEAP (see Section 11.7) for O(log n) insert/extract.
- **Double-Ended Queue (Deque)** — allows insertion and deletion from BOTH ends (front and rear).

**Queue using two stacks (matches sample Q10 exactly)**: use an "input stack" (s1) for enqueue operations, and an "output stack" (s2) for dequeue operations.
- `enqueue(x)`: simply `s1.push(x)`.
- `dequeue()`: if `s2` is EMPTY, first transfer ALL elements from `s1` to `s2` (this REVERSES their order, which is exactly what converts LIFO behavior into FIFO behavior), THEN pop from `s2`. If `s2` is NOT empty, just pop directly from `s2` (no need to re-transfer).

**Trace**: enqueue 1,2,3 → s1=[1,2,3] (3 on top), s2=[]. Now dequeue(): s2 is empty, so transfer all of s1 to s2 → pop 3, push to s2; pop 2, push to s2; pop 1, push to s2 → s1=[], s2=[3,2,1] (1 on top now). Pop from s2 → returns 1 (correct FIFO order — 1 was enqueued first). Now enqueue 4 → s1=[4]. Dequeue() again: s2=[3,2] (not empty, 2 on top) → pop directly → returns 2 (correct, no need to touch s1 yet).

## 11.5 Binary Trees

### 11.5.1 Terminology

- **Root** — topmost node (no parent).
- **Leaf** — a node with NO children.
- **Height of a tree** — the number of edges on the LONGEST path from root to a leaf (some definitions count nodes instead of edges — be alert to which convention a question uses; SEBI's AVL height formula in the sample paper uses height where a single node = height 0).
- **Depth of a node** — number of edges from the root to that node.
- **Degree of a node** — number of children it has.

### 11.5.2 Types of Binary Trees

- **Full/Strict Binary Tree** — every node has EITHER 0 or 2 children (never exactly 1). Matches sample Q55: for a full binary tree, if L = number of leaves, total NODES = 2L − 1, and internal (non-leaf) nodes = L − 1.
- **Complete Binary Tree** — all levels are COMPLETELY filled except possibly the last, which is filled from LEFT to RIGHT with no gaps. (This is the structure used to represent HEAPS efficiently as an array.)
- **Perfect Binary Tree** — ALL internal nodes have exactly 2 children AND all leaves are at the SAME level/depth. A perfect binary tree of height h has exactly 2^(h+1) − 1 total nodes, and 2^h leaves.
- **Balanced Binary Tree** — the height difference between the left and right subtrees of EVERY node is bounded (e.g., AVL trees bound this difference to at most 1) — ensures O(log n) operations.
- **Degenerate/Skewed Binary Tree** — each parent has only ONE child, making the tree essentially a linked list — worst case, O(n) operations instead of O(log n).

### 11.5.3 Binary Search Tree (BST)

A binary tree where, for EVERY node: all values in its LEFT subtree are SMALLER, and all values in its RIGHT subtree are LARGER (assuming no duplicates).

**BST property → sorted inorder traversal** (extremely important, appears repeatedly): traversing a BST INORDER (Left→Root→Right) ALWAYS produces elements in SORTED ASCENDING order — this is one of the single most tested facts about BSTs in the entire syllabus.

**BST operations complexity:**

| Operation | Balanced BST | Skewed/Unbalanced BST |
|---|---|---|
| Search | O(log n) | O(n) |
| Insert | O(log n) | O(n) |
| Delete | O(log n) | O(n) |

**Building a BST from a Preorder sequence — worked trace (matches sample Q56 exactly)**

Given preorder: `16, 11, 13, 12, 17, 20, 21`. Preorder is Root→Left→Right, so the FIRST element is always the root, and we insert each subsequent element following standard BST insertion rules (smaller goes left, larger goes right):
- Insert 16 → root.
- Insert 11: 11<16 → goes left of 16.
- Insert 13: 13<16 → go left to 11; 13>11 → goes right of 11.
- Insert 12: 12<16 → go left to 11; 12>11 → go right to 13; 12<13 → goes left of 13.
- Insert 17: 17>16 → goes right of 16.
- Insert 20: 20>16 → go right to 17; 20>17 → goes right of 17.
- Insert 21: 21>16 → go right to 17; 21>17 → go right to 20; 21>20 → goes right of 20.

**Resulting tree structure:**
```
                16
              /    \
            11       17
              \         \
               13        20
              /            \
            12               21
```

**Postorder traversal (Left→Right→Root) of this tree**: 
- Start at 16: go left to 11's subtree first.
  - 11's subtree: 11 has only a right child, 13. 13 has only a left child, 12.
    - Visit 12's subtree (leaf) → 12.
    - Visit 13's right subtree → none.
    - Visit 13 itself → 13.
    - Visit 11's left → none. Visit 11 itself → 11.
  - So left subtree postorder: 12, 13, 11.
- Then go right to 17's subtree.
  - 17 has only a right child, 20. 20 has only a right child, 21.
    - Visit 21 (leaf) → 21.
    - Visit 20's right done, visit 20 itself → 20.
    - Visit 17's left → none, visit 17 itself → 17.
  - Right subtree postorder: 21, 20, 17.
- Finally visit root 16.

**Full postorder = 12, 13, 11, 21, 20, 17, 16** — exactly matching the sample paper's given answer.

### 11.5.4 AVL Tree — Self-Balancing BST

An AVL tree is a BST where, for every node, the **Balance Factor** = height(left subtree) − height(right subtree) is restricted to be **−1, 0, or +1**. If an insertion/deletion violates this, the tree performs ROTATIONS to rebalance.

**Rotation types**:
- **LL Rotation (single right rotation)** — used when a node is inserted into the LEFT subtree of the LEFT child (imbalance caused on the left-left side).
- **RR Rotation (single left rotation)** — mirror case, right-right imbalance.
- **LR Rotation (left-right, double rotation)** — insertion into the RIGHT subtree of the LEFT child; requires a LEFT rotation on the child first, THEN a RIGHT rotation on the node.
- **RL Rotation (right-left, double rotation)** — mirror of LR.

**Minimum number of nodes formula (matches sample Q3 exactly)**: Let N(h) = minimum number of nodes in an AVL tree of height h.
- N(0) = 1 (a single node has height 0)
- N(1) = 2
- N(h) = N(h-1) + N(h-2) + 1 (the "+1" accounts for the root itself; the tree is built by taking the minimum-node tree of height h-1 as one subtree and height h-2 as the other, since that's the "worst" — sparsest — valid AVL configuration that still meets the balance-factor constraint)

**Computing N(4) step by step:**
- N(0) = 1
- N(1) = 2
- N(2) = N(1) + N(0) + 1 = 2 + 1 + 1 = 4
- N(3) = N(2) + N(1) + 1 = 4 + 2 + 1 = 7
- N(4) = N(3) + N(2) + 1 = 7 + 4 + 1 = **12** — exactly matches the sample paper's answer.

This is essentially the same recurrence pattern as the Fibonacci sequence (shifted), and it's what gives AVL trees their guaranteed O(log n) height bound — the number of nodes grows at LEAST exponentially with height, so height grows at MOST logarithmically with the number of nodes.

## 11.6 Graph Representation

- **Adjacency Matrix** — a V×V 2D array where `matrix[i][j] = 1` (or the edge weight) if an edge exists between vertex i and j, else 0 (or infinity). Space complexity: O(V²), regardless of how many edges actually exist. Edge lookup (does edge i-j exist?): O(1) — very fast. BEST for **DENSE graphs** (matches sample Q49) where E is close to V².
- **Adjacency List** — for each vertex, maintain a LIST of its neighboring vertices (and edge weights, if applicable). Space complexity: O(V+E) — much more efficient for SPARSE graphs (where E is much less than V²). Edge lookup: O(degree of vertex), slower than adjacency matrix in the worst case, but traversal (visiting all neighbors) is faster overall since you don't waste time scanning non-existent edges.

## 11.7 Heap

A **heap** is a COMPLETE binary tree satisfying the **heap property**:
- **Max-Heap** — every parent node's value is ≥ its children's values (root = maximum element).
- **Min-Heap** — every parent node's value is ≤ its children's values (root = minimum element).

**Array representation of a heap** (since a heap is always a COMPLETE binary tree, it can be stored efficiently as a plain array, with NO pointers needed): for a node at index `i` (0-indexed array):
- Left child index = `2i + 1`
- Right child index = `2i + 2`
- Parent index = `(i - 1) / 2` (integer division)

**Heap operations:**
- `insert(x)` — add x at the END of the array (next available complete-tree position), then **"bubble up" / "sift up"** — repeatedly compare with its parent and swap if it violates the heap property, until it doesn't (or it reaches the root). O(log n).
- `extractMax()`/`extractMin()` (delete root) — save the root value to return, move the LAST element in the array to the root position, shrink the array size by one, then **"heapify down" / "sift down"** — compare the new root with its children, swap with the LARGER child (for max-heap) if it violates the heap property, and repeat down the tree until it doesn't (matches sample Q21's exact description: "parent is compared with left and right child, then swapped appropriately"). O(log n).
- `buildHeap()` from an arbitrary unsorted array — surprisingly, this takes only O(n) time overall (not O(n log n)), achieved by calling "heapify down" starting from the LAST NON-LEAF node and working backward up to the root (this is what Section 10.1.6's heap sort trace demonstrated).

**Heap Sort connection**: build a max-heap O(n), then repeatedly extract the max (swap to end, shrink, heapify) n times, each O(log n) → total O(n log n).

## 11.8 Hashing (Data Structure View)

See also Section 5.5 for the algorithmic view. As a data structure, a **Hash Table** maps KEYS to VALUES using a hash function to compute an array index, giving average O(1) insert/search/delete.

**Java's `HashMap`** — implemented internally using an array of "buckets," where each bucket is (in modern Java 8+) either a linked list OR, if a bucket gets too many entries (default threshold 8), a self-balancing RED-BLACK TREE for that specific bucket (an optimization to guard against worst-case O(n) degradation from poor/malicious hash distributions — this is a good "did you know" fact if it comes up).

**Hash function quality**: a good hash function should distribute keys as UNIFORMLY as possible across all buckets, to minimize collisions, and should be fast to compute.

## 11.9 Matrix

A matrix is essentially a 2D array (see 11.1 for address calculation). Special matrix types worth knowing conceptually:
- **Sparse Matrix** — a matrix where MOST elements are zero. Storing it as a full 2D array wastes memory; instead, only NON-ZERO elements are stored, typically as a list/array of (row, column, value) triples, saving significant space when the matrix is genuinely sparse.
- **Identity Matrix** — a square matrix with 1s on the main diagonal and 0s elsewhere; acts as the multiplicative identity (A × I = A).
- **Transpose of a Matrix** — flipping a matrix over its diagonal, so `A_transpose[i][j] = A[j][i]`; rows become columns and vice versa.
- **Matrix Multiplication** — standard algorithm is O(n³) for two n×n matrices (three nested loops); Strassen's algorithm improves this to O(n^2.81) (see Section 5.10/10 for the recurrence-based derivation).

## 11.10 JSON Objects (as a data structure concept)

**JSON (JavaScript Object Notation)** is a lightweight, text-based, language-independent data-interchange format, structurally very similar to a Python dictionary or a Java `Map` — a collection of KEY-VALUE pairs, where keys are always STRINGS, and values can be strings, numbers, booleans, `null`, arrays (`[...]`), or NESTED JSON objects (`{...}`).

```json
{
  "name": "Ramesh",
  "age": 25,
  "skills": ["Python", "SQL", "Java"],
  "address": {
    "city": "Mumbai",
    "pincode": "400001"
  },
  "is_active": true
}
```

JSON maps naturally onto common data structures across languages:
- JSON object `{}` ↔ Python `dict` / Java `HashMap` / a hash table generally.
- JSON array `[]` ↔ Python `list` / Java `ArrayList` / a dynamic array generally.

See Section 4.1.12 for the Python `json` module's `dumps`/`loads`/`dump`/`load` functions and their traps.

---

# SECTION 11 QUICK REVISION TABLE

| Structure | Key Fact |
|---|---|
| Array | Contiguous memory, O(1) access, O(n) insert/delete at arbitrary position |
| Singly Linked List | O(1) insert at head, O(n) delete at tail (must find second-last node) |
| Stack | LIFO; used for recursion/call stack, expression conversion, balanced parens |
| Queue | FIFO; two-stack implementation transfers only when output stack is empty |
| Full Binary Tree | Nodes = 2L − 1 |
| BST inorder | Always gives sorted output |
| AVL Balance Factor | Must be -1, 0, or +1; N(h) = N(h-1)+N(h-2)+1 |
| Adjacency Matrix | O(V²) space, O(1) lookup, best for dense graphs |
| Adjacency List | O(V+E) space, best for sparse graphs |
| Heap | Complete binary tree; array-based; parent=(i-1)/2, children=2i+1,2i+2 |
| Row-major address | B + w×(i×n + j) |


---

# SECTION 12: STRING MANIPULATION (Phase II) — 10% Weightage

## 12.1 Length

| Language | Function | Notes |
|---|---|---|
| C | `strlen(s)` | from `<string.h>`; counts characters up to (not including) the null terminator `\0` |
| C++ | `s.size()` or `s.length()` | both identical for `std::string`, fully interchangeable |
| Java | `s.length()` | note: it's a METHOD `length()` on String, but a FIELD `.length` (no parens) on arrays — a very commonly confused syntax trap |
| Python | `len(s)` | O(1), stored as an attribute internally |

## 12.2 Substring Extraction

| Language | Syntax | Parameter meaning |
|---|---|---|
| C++ | `s.substr(pos, len)` | starting position, LENGTH of substring (not end index!) |
| Java | `s.substring(start, end)` | start INCLUSIVE, end EXCLUSIVE (an index, not a length) |
| Python | `s[start:end]` | start INCLUSIVE, end EXCLUSIVE (slicing) |
| C | no built-in function | typically done manually with `strncpy()` or a loop |

**The single most repeated trap in this entire topic (matches sample paper's explicit trap table)**: C++'s `substr(pos, length)` takes a LENGTH as the second argument, while Java's `substring(start, end)` takes an END INDEX as the second argument. Mixing these up (assuming Java-style behavior in C++ or vice versa) is a guaranteed wrong answer if you're not careful.

**Worked examples:**
```cpp
string s = "Hello World";
s.substr(6, 5);     // "World" - start at index 6, take 5 characters
```
```java
String s = "Hello World";
s.substring(6, 11);  // "World" - from index 6 up to (not including) index 11
s.substring(6);       // "World" - single-argument version goes to the end of the string
```
```python
s = "Hello World"
s[6:11]    # "World"
s[6:]      # "World" - omitting the end goes to the end of the string
```

## 12.3 Finding a Character or Substring

| Language | Find first occurrence of a CHARACTER | Find first occurrence of a SUBSTRING |
|---|---|---|
| C | `strchr(str, char)` — returns a pointer to the first occurrence, or `NULL` if not found | `strstr(str1, str2)` |
| C++ | `s.find(c)` — returns an index, or `string::npos` if not found | `s.find(substring)` |
| Java | `s.indexOf(char)` | `s.indexOf(substring)` |
| Python | `s.find(char)` or `s.index(char)` | `s.find(sub)` — `find` returns -1 if not found; `index` raises an exception if not found |

- `strrchr()` (C) / `s.rfind()` (C++/Python) / `s.lastIndexOf()` (Java) — find the LAST occurrence instead of the first.

## 12.4 Comparison

| Language | Function |
|---|---|
| C | `strcmp(s1, s2)` — returns 0 if equal, negative if s1<s2 (lexicographically), positive if s1>s2 |
| C++ | `s1.compare(s2)` — same convention as `strcmp` |
| Java | `s1.equals(s2)` for VALUE equality; `s1 == s2` checks REFERENCE equality (whether they're the exact same object in memory) — this is a hugely important Java-specific trap |
| Python | `s1 == s2` — checks value equality directly (Python's `==` for strings compares content, unlike Java) |

**Java `==` vs `.equals()` trap, explained fully**: in Java, `String` is an OBJECT type (a reference type), not a primitive. `==` for objects compares whether two references point to the EXACT SAME object in memory, NOT whether their content is equal. `.equals()` is specifically OVERRIDDEN by the `String` class to compare actual character content instead. Because Java uses a "String pool" (an internal cache of string literals) for optimization, two string literals with the same value (`"abc"` and `"abc"`) OFTEN end up `==` true due to pooling — but a string created explicitly with `new String("abc")` will NOT be `==` to a literal `"abc"`, even though `.equals()` would still return true for both. **Always use `.equals()` for content comparison in Java, never `==`.**

## 12.5 Concatenation

| Language | Function |
|---|---|
| C | `strcat(s1, s2)` — appends s2 onto the END of s1 (s1's buffer must be large enough) |
| C++ | `s1 + s2` (operator overloading works for `std::string`) |
| Java | `s1.concat(s2)` OR simply `s1 + s2` (Java's `+` is overloaded specifically for String, one of Java's rare built-in operator overloads) |
| Python | `s1 + s2` |

**Important performance note (Java-specific, sometimes tested conceptually)**: Java's `String` is IMMUTABLE — every concatenation with `+` or `.concat()` actually creates a BRAND NEW String object rather than modifying the original. Doing this repeatedly inside a loop is inefficient (O(n²) for building a string of length n character by character) — the recommended solution is `StringBuilder` (mutable, efficient `.append()`), which is NOT thread-safe but fast, or `StringBuffer` (mutable, thread-safe via synchronization, slightly slower).

## 12.6 Immutability of Strings

- **Java**: `String` is immutable (any "modification" method actually returns a NEW string). `StringBuilder`/`StringBuffer` ARE mutable.
- **Python**: strings are immutable (`s[0] = 'x'` raises a `TypeError`) — to "modify," you must create a new string, or convert to a `list` of characters, modify that, then `''.join()` back.
- **C++**: `std::string` IS mutable (individual characters can be assigned directly: `s[0] = 'X';`).
- **C**: character arrays (`char[]`) are mutable; string literals (`char *s = "hello";`) are technically NOT safely mutable (attempting to modify a string literal is undefined behavior in C).

## 12.7 Regex in String Manipulation (cross-reference to Section 4.1.1)

Regex is the most powerful general-purpose tool for SEARCH within strings, especially for PATTERN-based searches (not just exact substrings) — e.g., validating an email format, extracting all phone numbers, checking if a string is purely numeric.

```python
import re
bool(re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', email))  # basic email validation pattern
```

## 12.8 String Manipulation — Worked Numerical Trace (matches sample Q65 style)

`s = "Programming"`. Compute `len(s) + s.index("m")`.
- `len("Programming")` — count the characters: P-r-o-g-r-a-m-m-i-n-g = 11 characters. `len(s) = 11`.
- `s.index("m")` — find the position of the FIRST occurrence of 'm'. Index positions: P(0) r(1) o(2) g(3) r(4) a(5) m(6) m(7) i(8) n(9) g(10). First 'm' is at index **6**.
- `len(s) + s.index("m") = 11 + 6 = 17`.

## 12.9 Common String Manipulation Problems (Practice Patterns)

- **Reverse a string**: `s[::-1]` (Python); manual two-pointer swap in C/C++/Java (`char[]` array, swap from both ends moving inward).
- **Check palindrome**: compare string with its reverse, OR use two pointers (left, right) moving toward the center, comparing characters at each step, stopping/failing on first mismatch.
- **Count vowels/character frequency**: iterate through the string, use a hashmap/dictionary/array (size 26 for lowercase English letters) to count occurrences.
- **Check anagram**: two strings are anagrams if they contain the EXACT SAME characters with the SAME frequency counts (just rearranged) — check by sorting both strings and comparing (O(n log n)), or by comparing character-frequency counts (O(n), typically faster).
- **Remove duplicate characters**: use a `set`/hash-based structure to track seen characters while building the result.
- **String to integer / integer to string conversions**: `int(s)` / `str(x)` in Python; `Integer.parseInt(s)` / `String.valueOf(x)` in Java; `atoi(s)` / `sprintf` in C; `stoi(s)` / `to_string(x)` in C++.

---

# SECTION 12 QUICK REVISION TABLE

| Trap | Correct Fact |
|---|---|
| C++ `substr(pos, len)` | Second parameter is LENGTH |
| Java `substring(start, end)` | Second parameter is an END INDEX (exclusive) |
| Java `==` on Strings | Compares REFERENCE, not content — use `.equals()` |
| Python string mutation | Strings are immutable; must build a new string |
| Java string concatenation in a loop | Inefficient — use `StringBuilder` |
| `strcmp` return convention | 0=equal, negative=s1<s2, positive=s1>s2 |

---

# SECTION 13: OBJECT ORIENTED PROGRAMMING (Phase II Deep Dive) — 20% Weightage

> Phase II tests OOP more through CODE-TRACING than definitions. Section 3.6-3.7 already covered the conceptual depth — this section adds numerical/output-tracing practice specifically.

## 13.1 Encapsulation — Code-Level View

```java
class BankAccount {
    private double balance;   // hidden from direct outside access

    public BankAccount(double initial) { this.balance = initial; }

    public double getBalance() { return balance; }   // controlled READ access
    public void deposit(double amt) {
        if (amt > 0) balance += amt;   // controlled WRITE access, with validation logic
    }
}
```
Direct access like `account.balance = -500;` from outside the class is BLOCKED by `private` — the only way to modify `balance` is through the controlled `deposit()` method, which can enforce business rules (like rejecting negative amounts) — this validation-on-write is the PRACTICAL benefit of encapsulation, beyond just "hiding data."

## 13.2 Abstraction — Code-Level View

```java
abstract class PaymentMethod {
    abstract void pay(double amount);   // WHAT needs to happen, not HOW
}
class CreditCardPayment extends PaymentMethod {
    void pay(double amount) { System.out.println("Paying " + amount + " via Credit Card"); }
}
class UPIPayment extends PaymentMethod {
    void pay(double amount) { System.out.println("Paying " + amount + " via UPI"); }
}
// Client code doesn't need to know HOW each payment method works internally:
PaymentMethod pm = new UPIPayment();
pm.pay(500);   // "Paying 500 via UPI" - the caller only knows the abstract contract, not the implementation detail
```

## 13.3 Polymorphism — Full Output-Tracing Examples

### 13.3.1 Method Overloading (Compile-time) — Trace

```java
class Calculator {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }
}
Calculator c = new Calculator();
System.out.println(c.add(2, 3));         // calls add(int,int) -> 5
System.out.println(c.add(2.5, 3.5));      // calls add(double,double) -> 6.0
System.out.println(c.add(1, 2, 3));        // calls add(int,int,int) -> 6
```
The COMPILER decides which `add` to call based on the NUMBER and TYPES of arguments at the call site — this decision is made at COMPILE TIME (static binding), which is why it's called compile-time/static polymorphism.

### 13.3.2 Method Overriding (Runtime) — Trace

```java
class Animal {
    void sound() { System.out.println("Animal makes a sound"); }
}
class Dog extends Animal {
    @Override
    void sound() { System.out.println("Dog barks"); }
}
class Cat extends Animal {
    @Override
    void sound() { System.out.println("Cat meows"); }
}

Animal[] animals = { new Dog(), new Cat(), new Animal() };
for (Animal a : animals) {
    a.sound();   
}
// Output:
// Dog barks
// Cat meows
// Animal makes a sound
```
Even though the DECLARED type of each array element is `Animal`, the ACTUAL method that executes depends on the REAL/RUNTIME object type (`Dog`, `Cat`, or plain `Animal`) — this is dynamic/runtime binding, achieved via the JVM's virtual method dispatch mechanism (conceptually similar to C++'s vtable, described in Section 3.7.6).

## 13.4 Inheritance — Constructor Chaining Trace

```java
class Vehicle {
    Vehicle() { System.out.println("Vehicle constructor"); }
}
class Car extends Vehicle {
    Car() {
        super();   // explicit call to parent constructor (this happens IMPLICITLY even if you omit it, as long as there's no other explicit constructor call)
        System.out.println("Car constructor");
    }
}
Car myCar = new Car();
// Output:
// Vehicle constructor
// Car constructor
```
**Key rule**: the PARENT class constructor ALWAYS runs BEFORE the child class constructor's own body executes — either via an explicit `super(...)` call (which, if used, MUST be the very first statement in the child constructor) or an IMPLICIT call to the parent's no-argument constructor if you don't write `super()` yourself.

## 13.5 The `this` and `super` Keywords

- `this` — refers to the CURRENT object instance; commonly used to disambiguate between a field and a parameter/local variable with the SAME NAME (`this.name = name;`), or to call another constructor in the SAME class (`this(...)`, constructor chaining within one class).
- `super` — refers to the PARENT class; used to call the parent's constructor (`super(...)`), or to explicitly call a parent's method that has been overridden (`super.methodName()`), or to access a parent's field that's shadowed by a child's field of the same name.

## 13.6 Interfaces — Multiple Implementation Trace

```java
interface Flyable { void fly(); }
interface Swimmable { void swim(); }

class Duck implements Flyable, Swimmable {
    public void fly() { System.out.println("Duck flies"); }
    public void swim() { System.out.println("Duck swims"); }
}
```
A class CAN implement multiple interfaces (unlike extending multiple classes) — this is precisely how Java achieves interface-based multiple inheritance while avoiding the Diamond Problem, because (pre-Java-8) interfaces had no implementation to conflict over. With Java 8+ `default` methods, IF two interfaces provide CONFLICTING default implementations for the same method signature, the implementing class is FORCED by the compiler to explicitly override that method itself (resolving the ambiguity manually) — the compiler will NOT guess for you.

## 13.7 Static vs Instance Members — Output Trace

```java
class Counter {
    static int count = 0;      // shared across ALL instances
    int id;                       // unique per instance

    Counter() {
        count++;                 // increments the SHARED counter
        id = count;                // this instance's own copy, set to the current shared count
    }
}
Counter c1 = new Counter();   // count becomes 1, c1.id = 1
Counter c2 = new Counter();   // count becomes 2, c2.id = 2
Counter c3 = new Counter();   // count becomes 3, c3.id = 3
System.out.println(Counter.count);   // 3 - accessed via CLASS name, shared value
System.out.println(c1.id);             // 1 - each instance's OWN value, frozen at creation time
System.out.println(c2.id);             // 2
```
This exact pattern (a `static` counter incremented in the constructor to auto-generate unique IDs) is a very common exam trace question — make sure you can track the shared `static` variable separately from each object's own instance variable.

## 13.8 Constructor Overloading Trace

```java
class Rectangle {
    int length, width;
    Rectangle() { this(1, 1); }               // calls the 2-arg constructor below via 'this(...)'
    Rectangle(int side) { this(side, side); }  // square - calls 2-arg constructor
    Rectangle(int l, int w) { length = l; width = w; }
}
Rectangle r1 = new Rectangle();          // length=1, width=1 (chained through both constructors)
Rectangle r2 = new Rectangle(5);          // length=5, width=5
Rectangle r3 = new Rectangle(4, 6);        // length=4, width=6
```

## 13.9 Exception Handling — Output Tracing

```java
public class Test {
    public static void main(String[] args) {
        try {
            int[] arr = {1, 2, 3};
            System.out.println(arr[5]);         // throws ArrayIndexOutOfBoundsException
        } catch (ArithmeticException e) {
            System.out.println("Arithmetic error");
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Array index error");     // THIS block executes
        } finally {
            System.out.println("Finally block always runs");
        }
        System.out.println("Program continues after try-catch");
    }
}
// Output:
// Array index error
// Finally block always runs
// Program continues after try-catch
```
Note that even though an exception occurred, the PROGRAM DOES NOT CRASH — because the exception was CAUGHT — execution continues normally with the statement right after the `try-catch-finally` block, once the appropriate `catch` block (and then `finally`) have run.

## 13.10 Object Cloning and Equality (Java specifics, good to know)

- `==` between two objects checks REFERENCE equality (same memory location) by default.
- `.equals()` — by default (inherited from `Object`), also just does reference equality (`==`), UNLESS the class overrides it (as `String`, `Integer`, and most wrapper classes do, to compare actual VALUE/content).
- `hashCode()` — should be overridden CONSISTENTLY with `equals()`: if two objects are `.equals()`, they MUST have the same `hashCode()` (required for correct behavior in hash-based collections like `HashMap`/`HashSet`) — though the reverse isn't required (different objects CAN share a hash code — that's just a collision, which is allowed).

---

# SECTION 13 QUICK REVISION TABLE

| Concept | Key Fact |
|---|---|
| Overloading | Compile-time, resolved by parameter list |
| Overriding | Runtime, resolved by actual object type |
| Constructor order | Parent constructor always runs before child's own body |
| `this` | Current instance; also used for same-class constructor chaining |
| `super` | Parent class access — constructor, method, or field |
| Interface + default methods | Conflicting defaults force explicit override in implementing class |
| `static` field | Shared across all instances, one copy total |
| `equals()` vs `==` | Override `equals()` for value comparison; keep `hashCode()` consistent with it |


---
---

# APPENDIX A: MASTER FORMULA SHEET

Use this as your final-day revision sheet — every numeric formula referenced anywhere in these notes, collected in one place.

## A.1 Graph & Tree Formulas

| Formula | Meaning |
|---|---|
| edges = (n × degree) / 2 | Handshaking lemma for a k-regular graph with n vertices |
| edges = n − 1 | Number of edges in any tree with n vertices |
| Full binary tree: nodes = 2L − 1 | L = number of leaves |
| Full binary tree: internal nodes = L − 1 | L = number of leaves |
| Perfect binary tree: nodes = 2^(h+1) − 1 | h = height (edge-counted) |
| Perfect binary tree: leaves = 2^h | h = height |
| AVL min nodes: N(h) = N(h−1) + N(h−2) + 1 | N(0)=1, N(1)=2 |
| Complete graph edges = n(n−1)/2 | Every pair of n vertices connected |
| BFS/DFS complexity = O(V + E) | Adjacency list representation |
| Binary search comparisons ≤ ⌈log₂(n+1)⌉ | Worst case |

## A.2 Complexity Formulas

| Algorithm | Best | Average | Worst |
|---|---|---|---|
| Bubble Sort | O(n) optimized | O(n²) | O(n²) |
| Selection Sort | O(n²) | O(n²) | O(n²) |
| Insertion Sort | O(n) | O(n²) | O(n²) |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) |
| Quick Sort | O(n log n) | O(n log n) | O(n²) |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) |
| Binary Search | O(1) | O(log n) | O(log n) |
| Naive String Match | O(n) | O(nm) | O(nm) |
| KMP | O(n+m) | O(n+m) | O(n+m) |
| Rabin-Karp | O(n+m) | O(n+m) | O(nm) |
| Dijkstra | — | O((V+E) log V) | — |
| Prim's | — | O(E log V) | — |
| Kruskal's | — | O(E log E) | — |
| Bellman-Ford | — | — | O(V·E) |
| Floyd-Warshall | — | O(V³) | O(V³) |
| Strassen's Multiplication | — | — | O(n^2.81) |

## A.3 Address Calculation

- Row-major: `Address(A[i][j]) = Base + w × (i × n + j)`, n = number of columns
- Column-major: `Address(A[i][j]) = Base + w × (j × m + i)`, m = number of rows

## A.4 Master Theorem (Divide & Conquer)

For T(n) = a·T(n/b) + f(n):
- Compare f(n) with n^(log_b a).
- f(n) grows SLOWER → T(n) = Θ(n^(log_b a))
- f(n) grows at the SAME rate → T(n) = Θ(n^(log_b a) · log n)
- f(n) grows FASTER (+ regularity condition) → T(n) = Θ(f(n))

## A.5 Normalization Quick Check

- 1NF: atomic values only
- 2NF: 1NF + no PARTIAL dependency (non-key attr depends on part of a composite key)
- 3NF: 2NF + no TRANSITIVE dependency (non-key attr depends on another non-key attr)
- BCNF: every determinant (LHS of every non-trivial FD) is a super key

## A.6 Isolation Level vs Anomaly Table

| Level | Dirty Read | Unrepeatable Read | Phantom Read |
|---|---|---|---|
| Read Uncommitted | Possible | Possible | Possible |
| Read Committed | Prevented | Possible | Possible |
| Repeatable Read | Prevented | Prevented | Possible |
| Serializable | Prevented | Prevented | Prevented |

## A.7 CIDR / Subnetting Quick Reference

| CIDR | Subnet Mask | Usable Hosts (approx) |
|---|---|---|
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /27 | 255.255.255.224 | 30 |
| /28 | 255.255.255.240 | 14 |

Formula: usable hosts = 2^(32 − CIDR) − 2 (subtract 2 for network address and broadcast address).

## A.8 chmod / Permission Numbers

read = 4, write = 2, execute = 1. Sum digits per category (owner, group, others).
- `755` = rwx r-x r-x
- `644` = rw- r-- r--
- `700` = rwx --- ---
- `777` = rwx rwx rwx (full permissions for everyone — generally a security risk if used carelessly)

---

# APPENDIX B: MASTER TRAP SHEET

The exact "gotchas" that repeatedly cost marks. Read this the night before the exam.

1. **Static method + inheritance** → method HIDING, not overriding. Resolved by reference type at compile time.
2. **`column = NULL` in SQL** → always false/unknown. Must use `IS NULL`.
3. **`NOT IN` with NULLs in the subquery** → returns zero rows. Use `NOT EXISTS` instead.
4. **Java `==` on Strings/objects** → reference comparison, not value. Use `.equals()`.
5. **`list.pop(0)` in Python** → O(n), not O(1) — array-shifting under the hood.
6. **`X.append(4)` returns `None`** → the list itself is modified in-place; don't assign the return value expecting the new list.
7. **Python slicing "1 to 4th inclusive"** → needs `s[1:5]`, not `s[1:4]`, since stop index is exclusive.
8. **C++ `substr(pos, len)` vs Java `substring(start, end)`** → C++'s second argument is a LENGTH, Java's is an END INDEX.
9. **Quick sort worst case** → occurs on an ALREADY SORTED (or reverse-sorted) array with naive first/last-element pivot selection, NOT on random/shuffled input.
10. **Prim's/Kruskal's classification** → GREEDY algorithms, NOT Dynamic Programming.
11. **DP requirement** → needs BOTH optimal substructure AND overlapping subproblems; missing overlapping subproblems makes it Divide & Conquer instead.
12. **`json.dumps()`** → returns a new string; does NOT change the type of the original dict.
13. **Full binary tree node-leaf formula** → nodes = 2L − 1, not 2L or L+1.
14. **AVL minimum nodes** → N(h) = N(h−1) + N(h−2) + 1, mirrors Fibonacci-style recursion, NOT a simple doubling formula.
15. **Multiple inheritance** → C++/Python support it for classes; Java/C# do NOT (classes), but DO allow multiple interface implementation.
16. **`len()` in Python** → O(1), stored as an attribute, NOT computed by scanning the string/list each time.
17. **R language indexing** → 1-indexed, unlike Python/Java/C/C++ which are all 0-indexed.
18. **WHERE vs HAVING** → WHERE filters rows before grouping (no aggregate functions allowed); HAVING filters groups after grouping (aggregate functions allowed).
19. **DELETE vs TRUNCATE vs DROP** → DELETE is DML (rollback-able, can use WHERE); TRUNCATE is DDL (removes all rows, resets identity, usually not rollback-able); DROP removes the entire table structure.
20. **Virus vs Worm** → Virus needs a host file and human action to spread; Worm self-replicates automatically across a network without needing either.
21. **Adjacency Matrix vs List** → Matrix is better for DENSE graphs (O(V²) space, O(1) lookup); List is better for SPARSE graphs (O(V+E) space).
22. **BST inorder traversal** → ALWAYS produces sorted (ascending) output — this single fact underlies many BST-related questions.
23. **Stable vs unstable sorts** → Stable: Bubble, Insertion, Merge. Unstable: Selection, Quick, Heap. ("SIMple sorts are stable, Some Quirky Heaps aren't" — mnemonic.)
24. **Deleting the last node of a SINGLY linked list** → O(n), because you must traverse from the head to find the second-last node (no backward pointer exists).
25. **Java pass-by-value, always** → even for objects, what's copied is the REFERENCE value, not the object itself; reassigning the parameter inside the method does NOT affect the caller's reference, but mutating the object THROUGH the reference DOES.
26. **`finally` block** → always executes except for `System.exit()` or a JVM crash — even if there's a `return` inside `try` or `catch`.
27. **Checked vs Unchecked exceptions (Java)** → Checked = compiler-enforced (`IOException`, `SQLException`); Unchecked = `RuntimeException` subclasses (`ArithmeticException`, `NullPointerException`, `ArrayIndexOutOfBoundsException`).
28. **`re.match()` vs `re.search()`** → `match` only checks the START of the string; `search` checks the entire string.
29. **Histogram vs Bar Chart** → Histogram bars TOUCH (continuous numeric bins); Bar chart bars are SEPARATED (discrete categories).
30. **B-Tree vs B+ Tree** → B+ Tree stores data ONLY in leaves, with leaves linked together — this is what makes B+ Trees superior for range queries, and is why virtually all real RDBMS indexes use B+ Trees specifically.

---

# APPENDIX C: PRACTICE QUESTION BANK

Work through these without looking at the answers first. Answers are given immediately after each question for self-checking, but COVER them while attempting.

**C.1** A graph has 8 vertices, each of degree 4. How many edges does it have?
> Answer: (8×4)/2 = 16 edges.

**C.2** A full binary tree has 15 leaf nodes. How many total nodes does it have?
> Answer: 2×15 − 1 = 29 nodes.

**C.3** What is N(5) using the AVL minimum-node recurrence?
> Answer: N(0)=1, N(1)=2, N(2)=4, N(3)=7, N(4)=12, N(5)=N(4)+N(3)+1=12+7+1=20.

**C.4** Array `[8, 4, 23, 42, 16, 15]` — trace ONE pass of bubble sort.
> Answer: (8,4)→swap→[4,8,23,42,16,15]; (8,23)→no swap; (23,42)→no swap; (42,16)→swap→[4,8,23,16,42,15]; (42,15)→swap→[4,8,23,16,15,42]. End of pass 1: `[4,8,23,16,15,42]`.

**C.5** In SQL, write a query to find employees who earn MORE than the average salary of their OWN department (correlated subquery).
> Answer: `SELECT name FROM employee e1 WHERE salary > (SELECT AVG(salary) FROM employee e2 WHERE e2.dept_id = e1.dept_id);`

**C.6** Is `SELECT dept, COUNT(*) FROM emp WHERE COUNT(*) > 5 GROUP BY dept;` valid SQL? Why or why not?
> Answer: NOT valid — `COUNT(*)` (an aggregate function) cannot be used inside `WHERE`. It must be moved to a `HAVING` clause instead: `... GROUP BY dept HAVING COUNT(*) > 5;`

**C.7** Java: what does `s1 == s2` check for two `String` objects, and what should you use instead for content comparison?
> Answer: `==` checks reference (memory address) equality; use `.equals()` for content comparison.

**C.8** Python: `x = [1,2,3]; y = x; y.append(4); print(len(x))` — what does this print, and why?
> Answer: prints `4`. `y = x` does NOT copy the list — both `x` and `y` reference the SAME list object, so modifying `y` also affects what `x` sees.

**C.9** What is the time complexity of building a heap from an unsorted array of n elements — and why is it NOT O(n log n)?
> Answer: O(n). Although each individual heapify-down call is O(log n) and you might naively expect O(n log n) for n calls, most nodes in a complete binary tree are near the BOTTOM (leaves and near-leaves), where heapify-down does very little work (short distance to sift down) — the sum across all levels works out to O(n) overall, not O(n log n).

**C.10** Convert infix `(A + B) * (C − D)` to postfix.
> Answer: `A B + C D − *`

**C.11** What's the output?
```java
class A {
    static void greet() { System.out.println("A"); }
}
class B extends A {
    static void greet() { System.out.println("B"); }
}
A obj = new B();
obj.greet();
```
> Answer: prints `A`. Static methods are resolved by the REFERENCE TYPE (`A`), not the actual object type (`B`) — this is method hiding, not overriding, since static methods don't participate in dynamic dispatch.

**C.12** Given preorder traversal `50, 30, 20, 40, 70, 60, 80` of a BST, what is the inorder traversal (without drawing the tree)?
> Answer: Since inorder traversal of ANY BST is always the SORTED order of its elements: `20, 30, 40, 50, 60, 70, 80`.

**C.13** What's wrong with this shell script line: `if [$x -gt 5]; then`?
> Answer: missing spaces around the brackets — must be `if [ $x -gt 5 ]; then` (space after `[` and before `]`).

**C.14** In relational algebra, if R has 4 tuples and 3 attributes, and S has 6 tuples and 3 attributes (union-compatible), what is the maximum possible cardinality of R ∪ S? What about R × S?
> Answer: R ∪ S → maximum 10 tuples (fewer if there are duplicates removed). R × S → exactly 4×6 = 24 tuples, with 3+3=6 attributes.

**C.15** Why does Quick Sort degrade to O(n²) specifically on an already-sorted array (with naive pivot selection)?
> Answer: If the pivot is always chosen as, say, the LAST element, and the array is already sorted ascending, the pivot will always be the LARGEST remaining element — so every partition splits the array into one side with n−1 elements and one side with 0 elements, giving a completely unbalanced recursion tree of depth n, with O(n) work at each level → O(n²) total.

**C.16** What does `SELECT COUNT(*) , COUNT(email) FROM users;` tell you if the two numbers are DIFFERENT?
> Answer: `COUNT(*)` counts all rows including those with NULL email; `COUNT(email)` counts only rows where email is NOT NULL — a difference between the two numbers directly tells you how many rows have a NULL email.

**C.17** Explain why `dp[i][w] = max(dp[i-1][w], value[i] + dp[i-1][w-weight[i]])` is the correct recurrence for 0/1 Knapsack.
> Answer: at each item `i` and capacity `w`, you have exactly two choices: EXCLUDE item i (value stays as whatever the best was using only the first i−1 items at the same capacity w — `dp[i-1][w]`), or INCLUDE item i (only possible if weight[i] ≤ w — you gain value[i], but now only have w−weight[i] capacity left for the remaining first i−1 items — `value[i] + dp[i-1][w-weight[i]]`). Since it's a 0/1 knapsack (each item used at most once), you take whichever of these two choices gives the higher value.

**C.18** In a hash table using separate chaining, if there are 20 keys and 8 buckets, what is the load factor, and what does it represent?
> Answer: load factor α = 20/8 = 2.5 — on average, each bucket's chain contains 2.5 keys; a higher load factor means longer chains and thus SLOWER average search/insert/delete time.

**C.19** What is printed?
```python
def f(lst):
    lst.append(100)
    lst = [1,2,3]   # reassignment - creates a NEW local list, doesn't affect the caller

x = [5, 6]
f(x)
print(x)
```
> Answer: `[5, 6, 100]`. The `append(100)` mutates the ORIGINAL list object (visible to the caller, since `lst` and `x` point to the same object at that moment). But the SUBSEQUENT reassignment `lst = [1,2,3]` only makes the LOCAL variable `lst` point to a brand-new list — it does NOT affect what `x` points to outside the function. This mirrors the Java pass-by-value-of-reference trap exactly (Section 3.4.1/13).

**C.20** Why is B+ Tree preferred over B-Tree specifically for RANGE QUERIES in a database?
> Answer: In a B+ Tree, all actual data resides in the LEAF nodes, and the leaf nodes are LINKED TOGETHER in a linked list — so once you locate the starting point of a range via a single tree descent, you can simply walk the leaf-level linked list to retrieve the rest of the range sequentially, without needing to re-traverse the tree. A B-Tree stores data in internal nodes too and has no such leaf-linking, making range scans require more scattered tree traversal.

**C.21** What SQL clause would you use to find departments where the AVERAGE salary exceeds 50000, and what's the full query?
> Answer: `SELECT dept_id, AVG(salary) FROM employee GROUP BY dept_id HAVING AVG(salary) > 50000;`

**C.22** True or False: "In C++, if a base class method is not marked `virtual`, calling it through a base-class pointer to a derived object will still invoke the derived class's overridden version." Explain.
> Answer: FALSE. Without `virtual`, the call is resolved STATICALLY based on the pointer's DECLARED type (the base class), not the actual object type — so the BASE class's version executes, not the derived class's override. This directly parallels Java's static-method-hiding trap.

**C.23** What is the output of this Java code and why?
```java
public class Test {
    public static void main(String[] args) {
        try {
            return;
        } finally {
            System.out.println("Finally executed");
        }
    }
}
```
> Answer: prints `Finally executed`. Even though `return` is encountered inside `try`, the `finally` block STILL executes before the method actually returns control to the caller — this is one of the most important guarantees of `finally`.

**C.24** Why is Merge Sort's time complexity IDENTICAL (Θ(n log n)) for best, average, AND worst case, unlike Quick Sort?
> Answer: Merge Sort ALWAYS splits the array into exactly two equal(ish) halves regardless of the input's initial order, and ALWAYS does O(n) work to merge them back — this behavior is completely independent of how the input data happens to be arranged, so there's no "unlucky" input that can degrade its performance, unlike Quick Sort where the CHOICE of pivot relative to the data's order directly determines whether partitions are balanced or not.

**C.25** Design question: Why does a Data Warehouse typically use a DENORMALIZED (star schema) design, when Section 1 taught you that normalization is generally "good practice"?
> Answer: Normalization (3NF/BCNF) is optimized to minimize redundancy and prevent anomalies during FREQUENT WRITES (as in an OLTP transactional system) — but a data warehouse is READ-HEAVY (complex analytical queries, aggregations, reporting) and relatively WRITE-LIGHT (periodic batch loads via ETL). A denormalized star schema reduces the NUMBER OF JOINS needed for typical analytical queries (since dimension tables aren't further split), making read/query performance much faster — the redundancy tradeoff is acceptable because the data isn't being constantly updated in place, so update anomalies are far less of a practical concern in this context.

---

# APPENDIX D: FINAL EXAM-DAY CHECKLIST

Before you walk in, make sure you can, without hesitation:

- [ ] Derive the AVL minimum-node recurrence and compute N(h) for any h up to 6.
- [ ] Trace bubble, selection, insertion, merge, and quick sort on a 5-7 element array by hand.
- [ ] Build a BST from a given preorder sequence and produce its postorder (and vice versa).
- [ ] Convert infix ↔ prefix ↔ postfix in both directions.
- [ ] Trace Dijkstra's, Prim's, and Kruskal's on a small weighted graph.
- [ ] Write a correlated subquery AND explain why it's slower than a non-correlated one.
- [ ] Explain WHERE vs HAVING with a working example query.
- [ ] Distinguish 1NF/2NF/3NF/BCNF given a table and its functional dependencies.
- [ ] Explain method overloading vs overriding, with Java code for both.
- [ ] Explain Java's static-method-hiding trap and C++'s non-virtual-function trap — these are THE SAME underlying concept (early binding) tested in two languages.
- [ ] Recite the OSI 7 layers top to bottom and bottom to top, with one device/protocol example per layer.
- [ ] Recite the CIA triad and give one attack example that specifically threatens each property.
- [ ] Explain the difference between B-Tree and B+ Tree, and why B+ Tree wins for range queries.
- [ ] Trace Python list/dict mutation vs reassignment inside a function.
- [ ] Compute a 2D array memory address given base address, dimensions, and element size, in both row-major and column-major order.
- [ ] Explain DP's two required properties and give one example problem that has ONLY optimal substructure (Divide & Conquer) vs one that has BOTH (true DP).

---

*End of notes. Revise Appendix B (Master Trap Sheet) and Appendix D (Checklist) one final time the morning of the exam — these two sections alone cover the highest-frequency wrong-answer patterns across all 69 sample questions and the full syllabus. All the best for scoring 90+.*
