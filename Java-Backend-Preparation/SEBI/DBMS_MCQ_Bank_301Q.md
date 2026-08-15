# DBMS MCQ Bank — ER Model, Relational Model, Normalization, File Organization, Indexing, Transactions & Concurrency

**Coverage:** ER Model · Relational Model (Relational Algebra & Tuple Calculus) · Integrity Constraints · Normal Forms · File Organization · Indexing (B/B+ Trees) · Transactions & Concurrency Control

**Format:** Each question has 4 options. Answer key with a one-line rationale is given immediately after each question (useful for self-study). For timed practice, cover the answer line before attempting.

---

## Section 1: ER Model (Entity-Relationship Model)

**Q1.** In the ER model, an entity is:
A) A relationship between two tables
B) A "thing" or "object" in the real world that is distinguishable from other objects
C) An attribute of a table
D) A constraint on a relation
**Answer: B** — An entity is a real-world object with independent existence, distinguishable from all other objects.

**Q2.** A key attribute in an ER diagram is represented by:
A) An ellipse with dashed border
B) An underlined attribute name
C) A double ellipse
D) A diamond
**Answer: B** — Key/candidate attributes are shown underlined.

**Q3.** Which of the following is a multivalued attribute example?
A) Age
B) Phone_Numbers (a person can have several)
C) Date_of_Birth
D) SSN
**Answer: B** — Multivalued attributes can hold more than one value for an entity, shown as a double ellipse.

**Q4.** A derived attribute is one that:
A) Cannot be computed from other attributes
B) Is computed from other stored attributes (e.g., Age from Date_of_Birth)
C) Is always a key
D) Is always multivalued
**Answer: B** — Derived attributes (dashed ellipse) are calculated, not stored directly.

**Q5.** Composite attributes:
A) Cannot be divided further
B) Can be divided into smaller sub-parts representing more basic attributes (e.g., Name → First, Middle, Last)
C) Are always derived
D) Are represented by rectangles
**Answer: B**

**Q6.** A weak entity set is one that:
A) Has no attributes
B) Does not have sufficient attributes to form a primary key on its own and depends on a strong (owner) entity
C) Cannot participate in relationships
D) Is always in 1NF
**Answer: B** — Weak entities are identified using a discriminating (partial) key plus the primary key of the owner entity.

**Q7.** The partial key of a weak entity set is also called:
A) Primary key
B) Discriminator
C) Foreign key
D) Super key
**Answer: B**

**Q8.** In ER diagrams, a weak entity set is represented by:
A) A single rectangle
B) A double rectangle
C) A diamond
D) An ellipse
**Answer: B** — And its identifying relationship is shown with a double diamond.

**Q9.** The relationship connecting a weak entity to its owner (strong) entity is called:
A) Recursive relationship
B) Identifying relationship
C) Ternary relationship
D) Aggregated relationship
**Answer: B**

**Q10.** Degree of a relationship refers to:
A) Number of attributes in the relationship
B) Number of entity sets participating in the relationship
C) Number of tuples
D) Number of keys
**Answer: B** — E.g., binary (2), ternary (3), n-ary relationships.

**Q11.** A relationship set involving three entity sets is called:
A) Binary
B) Unary/Recursive
C) Ternary
D) Quaternary
**Answer: C**

**Q12.** A recursive relationship is one where:
A) Two different entity sets participate
B) The same entity set participates more than once in the same relationship
C) No entity participates
D) Only weak entities participate
**Answer: B** — E.g., "Employee supervises Employee."

**Q13.** Cardinality ratio in a relationship describes:
A) The number of attributes
B) The number of entities to which another entity can be associated via the relationship (1:1, 1:N, M:N)
C) The number of relationship sets
D) The strength of an entity
**Answer: B**

**Q14.** In a 1:N relationship between Department and Employee (one department has many employees), the foreign key should be placed:
A) In the Department relation
B) In the Employee relation (referencing Department)
C) In a separate bridge/junction table
D) In neither relation
**Answer: B** — For 1:N, FK goes on the "many" side.

**Q15.** In an M:N (many-to-many) relationship, implementation in the relational model requires:
A) Merging both entities into one table
B) A separate relationship/junction table containing the primary keys of both participating entities
C) Placing FK in either entity table
D) It cannot be implemented
**Answer: B**

**Q16.** Total participation (existence dependency) of an entity set E in relationship R means:
A) Every entity in E must participate in at least one relationship instance in R
B) No entity in E participates in R
C) Only some entities in E participate
D) E must be a weak entity
**Answer: A** — Shown by a double line connecting E to R.

**Q17.** Partial participation is represented in an ER diagram by:
A) Double line
B) Single line
C) Dashed line
D) Bold diamond
**Answer: B**

**Q18.** Which notation shows minimum and maximum number of relationship instances an entity can participate in?
A) Chen notation cardinality only
B) (min, max) structural constraint notation
C) Crow's foot only
D) None
**Answer: B**

**Q19.** Generalization in EER (Enhanced ER) model is the process of:
A) Splitting one entity into several sub-entities
B) Combining several entity sets that share common features into a higher-level (superclass) entity set
C) Removing redundant attributes
D) Creating weak entities
**Answer: B** — Bottom-up approach.

**Q20.** Specialization is:
A) A top-down process of defining subclasses of an entity set based on distinguishing characteristics
B) Same as generalization
C) Only applicable to weak entities
D) Merging entities
**Answer: A**

**Q21.** In EER, "disjoint" constraint on a specialization means:
A) An entity can belong to multiple subclasses simultaneously
B) An entity can belong to at most one subclass
C) All entities must belong to some subclass
D) None of the entities belong to any subclass
**Answer: B** — Opposite is "overlapping."

**Q22.** "Total" specialization/generalization constraint means:
A) Every entity in the superclass must belong to at least one subclass
B) No entity needs to belong to a subclass
C) Only weak entities are involved
D) Subclasses must be disjoint
**Answer: A** — Opposite is "partial," where superclass entities need not belong to any subclass.

**Q23.** Aggregation in ER modeling is used to:
A) Delete redundant relationships
B) Treat a relationship set (along with its entity sets) as a higher-level entity so it can participate in other relationships
C) Combine two weak entities
D) Convert M:N to 1:N
**Answer: B**

**Q24.** Which of these is NOT a component of the basic ER model?
A) Entity
B) Attribute
C) Relationship
D) Normal Form
**Answer: D** — Normal forms belong to the relational model/normalization theory, not ER modeling.

**Q25.** A candidate key at the ER level that uniquely identifies entity instances is chosen as the:
A) Foreign key
B) Primary key
C) Super key only
D) Composite attribute
**Answer: B**

**Q26.** Which symbol represents a relationship set in Chen's ER notation?
A) Rectangle
B) Diamond
C) Ellipse
D) Triangle
**Answer: B**

**Q27.** An attribute that is a set of attributes, some derived and some composite, is called:
A) Simple attribute
B) Complex attribute
C) Key attribute
D) Weak attribute
**Answer: B**

**Q28.** Converting an M:N relationship with descriptive attributes (e.g., "Works_On" with Hours) into relational tables requires:
A) Ignoring the descriptive attribute
B) Creating a new table with FKs from both entities plus the descriptive attribute(s)
C) Storing the attribute in both entity tables
D) Storing it as a derived attribute
**Answer: B**

**Q29.** In mapping a weak entity set to a relation, the primary key of the resulting table consists of:
A) Only the discriminator
B) The discriminator combined with the primary key of the owner (strong) entity
C) No primary key needed
D) Only the owner's primary key
**Answer: B**

**Q30.** Which of the following best differentiates a strong entity from a weak entity?
A) Strong entities have no attributes
B) A strong entity has a primary key of its own; a weak entity depends on another entity for its primary key
C) Weak entities cannot have relationships
D) Strong entities are always in relationships
**Answer: B**

**Q31.** ISA (is-a) relationship in EER models represents:
A) Aggregation
B) Superclass/subclass (inheritance) relationship
C) Weak entity dependency
D) Ternary relationship
**Answer: B**

**Q32.** A subclass in specialization inherits:
A) Nothing from the superclass
B) All attributes and relationships of the superclass, plus its own specific attributes
C) Only the primary key
D) Only relationships, not attributes
**Answer: B**

**Q33.** Union type / category in EER model is used to represent:
A) A subclass that is a subset of the union of distinct superclasses (multiple inheritance style, e.g., Owner = Person ∪ Bank ∪ Company)
B) A weak entity
C) A ternary relationship
D) A derived attribute
**Answer: A**

**Q34.** Self-referencing (recursive) relationships require role names because:
A) The entity set participates twice in the relationship, and role names distinguish the two participations
B) It is mandatory for all relationships
C) It converts the relationship into a weak entity
D) It removes ambiguity between entity sets
**Answer: A**

**Q35.** Which cardinality best fits: "A Student can enroll in many Courses, and a Course can have many Students"?
A) 1:1
B) 1:N
C) M:N
D) N:1
**Answer: C**

**Q36.** An attribute that uniquely identifies a weak entity within the scope of its owner entity is:
A) Primary key
B) Partial key (discriminator)
C) Foreign key
D) Candidate key
**Answer: B**

**Q37.** In ER-to-relational mapping, a 1:1 relationship is typically implemented by:
A) Creating a separate table always
B) Placing the FK of either entity in the other entity's table (often on the side with total participation), or merging both tables
C) It cannot be mapped
D) Using two separate FKs in a third table only
**Answer: B**

**Q38.** Enhanced ER (EER) model extends the basic ER model primarily to support:
A) SQL syntax
B) Superclass/subclass relationships, specialization/generalization, categories, and aggregation
C) Only weak entities
D) Only relational algebra operations
**Answer: B**

**Q39.** Which of the following is TRUE about attributes of a relationship set (e.g., "grade" in Student-Course enrollment)?
A) They must be stored in the entity tables
B) They belong to the relationship itself and, when mapped, typically go into the relationship's own table (especially for M:N)
C) They cannot exist
D) They are always derived
**Answer: B**

**Q40.** Which of these is an example of a good practice while converting ER diagrams to relational schema?
A) Ignore all constraints
B) Map each strong entity to a table, weak entity with combined key, and handle relationships based on cardinality (merge for 1:1/1:N where appropriate, separate table for M:N)
C) Always create a separate table for every attribute
D) Never use foreign keys
**Answer: B**

## Section 2: Relational Model & Relational Algebra

**Q41.** A relation in the relational model is formally a:
A) Ordered list of tuples
B) Subset of the Cartesian product of domains (a set of tuples)
C) A tree structure
D) A graph
**Answer: B**

**Q42.** The degree of a relation is:
A) The number of tuples
B) The number of attributes (columns)
C) The number of keys
D) The number of relations in the database
**Answer: B**

**Q43.** The cardinality of a relation is:
A) Number of attributes
B) Number of tuples (rows)
C) Number of domains
D) Number of foreign keys
**Answer: B**

**Q44.** Which of the following relational algebra operations is NOT a basic/fundamental operator?
A) Selection (σ)
B) Projection (π)
C) Join (⋈)
D) Union (∪)
**Answer: C** — The fundamental operators are σ, π, ∪, −, ×, and ρ (rename); Join is a derived operator (= σ over ×).

**Q45.** The Selection operation (σ) in relational algebra:
A) Selects a subset of columns
B) Selects a subset of tuples (rows) satisfying a given predicate
C) Combines two relations
D) Renames a relation
**Answer: B**

**Q46.** The Projection operation (π) in relational algebra:
A) Selects rows
B) Selects a subset of columns, and by definition removes duplicate tuples from the result
C) Combines relations using a condition
D) Computes the Cartesian product
**Answer: B**

**Q47.** For Union (R ∪ S) and Set Difference (R − S) to be valid in relational algebra, R and S must be:
A) Of the same degree with compatible (union-compatible) domains for corresponding attributes
B) Of any degree
C) Sorted identically
D) Indexed
**Answer: A**

**Q48.** The Cartesian Product (R × S) of relations with degree m and n and cardinalities p and q produces a relation with:
A) Degree m+n, cardinality p+q
B) Degree m+n, cardinality p×q
C) Degree m×n, cardinality p×q
D) Degree m, cardinality p
**Answer: B**

**Q49.** The natural join (R ⋈ S) combines tuples from R and S:
A) With no condition, like Cartesian product
B) Based on equality of all common attribute names, and keeps only one copy of each common attribute
C) Only for attributes explicitly listed with theta condition
D) Only when relations have no common attributes
**Answer: B**

**Q50.** A theta join (R ⋈θ S) is:
A) A join with no condition
B) A join based on any general condition θ (not necessarily equality)
C) Only defined for equality
D) Same as natural join
**Answer: B**

**Q51.** An equijoin is a theta join where:
A) All comparisons in θ use the "=" operator
B) All comparisons use "<"
C) It removes duplicate columns automatically like natural join
D) It's the same as Cartesian product
**Answer: A** — Note: unlike natural join, equijoin does NOT automatically remove the duplicate (redundant) join column.

**Q52.** Which join includes tuples from the left relation even if there is no matching tuple in the right relation (filling with NULLs)?
A) Inner join
B) Left outer join
C) Right outer join
D) Natural join
**Answer: B**

**Q53.** The full outer join preserves:
A) Only unmatched tuples from the left relation
B) Only unmatched tuples from the right relation
C) Unmatched tuples from BOTH relations (padded with NULLs)
D) Only matched tuples
**Answer: C**

**Q54.** The Rename operator (ρ) is used to:
A) Delete a relation
B) Rename a relation and/or its attributes, useful in self-joins and to avoid ambiguity
C) Perform selection
D) Compute intersection
**Answer: B**

**Q55.** The Intersection operation (R ∩ S) can be expressed in terms of set difference as:
A) R ∩ S = R − (R − S)
B) R ∩ S = R ∪ S
C) R ∩ S = R × S
D) R ∩ S = S − R
**Answer: A** — Intersection is a derived operator, not fundamental.

**Q56.** The Division operator (R ÷ S) in relational algebra is typically used for queries of the type:
A) "Find tuples in R related to ALL tuples in S" (e.g., "students who have taken ALL courses")
B) Simple filtering
C) Cartesian product
D) Union of two relations
**Answer: A**

**Q57.** If R has attributes (A,B) and S has attribute (B) with S ⊆ π_B(R), then R ÷ S returns:
A) All A values in R
B) All A values such that (A,b) ∈ R for every b in S
C) All B values in S
D) An empty relation always
**Answer: B**

**Q58.** Which relational algebra expression represents "employees who work in either Department 1 OR Department 2"?
A) σ(dept=1)(Employee) ∩ σ(dept=2)(Employee)
B) σ(dept=1)(Employee) ∪ σ(dept=2)(Employee)
C) σ(dept=1)(Employee) − σ(dept=2)(Employee)
D) σ(dept=1)(Employee) × σ(dept=2)(Employee)
**Answer: B**

**Q59.** Relational algebra is considered:
A) A declarative, non-procedural query language
B) A procedural query language where operations are applied in a specified sequence
C) Not a formal language
D) Only used for updates
**Answer: B** — It specifies a sequence of operations to produce the result, unlike tuple calculus.

**Q60.** Which of the following is TRUE regarding relational algebra and SQL?
A) Relational algebra operators have no equivalent SQL clauses
B) Relational algebra is the theoretical procedural foundation on which SQL query evaluation/optimization is based
C) SQL cannot express joins
D) They are unrelated
**Answer: B**

**Q61.** The aggregate function operator in extended relational algebra is denoted:
A) σ
B) π
C) ℱ (or 𝔊, generalized projection with grouping)
D) ρ
**Answer: C**

**Q62.** Which of these is an "extended" relational algebra operator (not part of the original/basic set)?
A) Selection
B) Outer Join
C) Union
D) Cartesian Product
**Answer: B** — Outer joins, generalized projection, and aggregate operators are extensions.

**Q63.** In relational algebra, σ_(salary>50000)(Employee) selects:
A) Columns where salary > 50000
B) Rows (tuples) where the salary attribute value is greater than 50000
C) All employees
D) Distinct salaries only
**Answer: B**

**Q64.** π_(name,salary)(Employee) returns:
A) All rows and all columns
B) Only the name and salary columns for all rows (duplicates removed)
C) Only rows with a specific salary
D) A Cartesian product
**Answer: B**

**Q65.** Semi-join (R ⋉ S) returns:
A) All tuples of R that have at least one matching tuple in S, projected on R's attributes only
B) All tuples of S
C) The Cartesian product of R and S
D) Only unmatched tuples
**Answer: A**

**Q66.** Anti-join (R ▷ S) returns:
A) Tuples of R that have NO matching tuple in S
B) Tuples of S with no match in R
C) Cartesian product minus join
D) All tuples of both relations
**Answer: A**

**Q67.** Which operator would you use to find employees who are NOT managers, given Employee and Manager (subset of employee IDs) relations?
A) Union
B) Set Difference
C) Cartesian Product
D) Natural Join
**Answer: B**

**Q68.** A query to find "names of employees who earn more than their manager" primarily requires:
A) Only Projection
B) A self-join (join Employee with itself) using rename, followed by selection
C) Only Union
D) Division
**Answer: B**

**Q69.** In relational algebra, closure property means:
A) The output of every operation is again a relation, allowing operations to be composed/nested
B) Operations cannot be repeated
C) Relations must be closed sets
D) Only selection has closure
**Answer: A**

**Q70.** Which of the following correctly orders relational algebra operator precedence (highest to lowest) in typical evaluation?
A) Union, Selection, Projection
B) Selection/Projection/Rename (unary) generally evaluated before binary operators like Join, then Union/Intersection/Difference
C) All operators have equal precedence and order doesn't matter
D) Division always first
**Answer: B**

**Q71.** Outer union (used when relations are not union-compatible but share some attributes) does what?
A) Produces NULLs for missing attribute values in tuples from relations with differing schemas, while keeping the union semantics
B) Only works on identical schemas
C) Same as Cartesian product
D) Removes all non-common attributes
**Answer: A**

**Q72.** The expression π_A(σ_p(R)) vs σ_p(π_A(R)) — when are these equivalent?
A) Always
B) Only when the selection condition p involves only attributes present in A (the projected list)
C) Never
D) Only for Cartesian products
**Answer: B** — This is a classic query optimization equivalence rule (pushing selection).

**Q73.** Which relational algebra expression finds customers who have accounts at ALL branches, given Depositor(cust, acc) and Account(acc, branch)?
A) A Union operation
B) Uses the Division operator: π_(cust,branch)(Depositor⋈Account) ÷ π_branch(Branch)
C) A simple selection
D) Cartesian product only
**Answer: B**

**Q74.** Which of the following is TRUE about natural join vs theta join with equality on the same attribute?
A) They always produce identical results including duplicate columns
B) Natural join automatically eliminates the duplicate (redundant) common column(s); equijoin does not
C) Theta join always has fewer columns
D) They are unrelated operations
**Answer: B**

**Q75.** π (projection) followed immediately by another π on a subset of already-projected attributes is equivalent to:
A) A single π on the final (innermost) attribute list directly on the original relation
B) It cannot be simplified
C) A join
D) A selection
**Answer: A** — Projection cascading/idempotence rule used in query optimization.

**Q76.** Given R(A,B,C) and S(C,D), the natural join R⋈S is computed by combining tuples where:
A) R.A = S.D
B) R.C = S.C, keeping one copy of C in the result
C) Cartesian product with no condition
D) R.B = S.D
**Answer: B**

**Q77.** Which of these correctly describes a "condition/theta join" written as R ⋈_(R.a < S.b) S?
A) Same as natural join
B) It's a Cartesian product of R and S followed by selection on the condition R.a < S.b
C) It only works with "="
D) It removes duplicate attributes automatically
**Answer: B**

**Q78.** Which relational algebra property allows σ_p1(σ_p2(R)) = σ_p2(σ_p1(R)) = σ_(p1∧p2)(R)?
A) Commutativity and combination of selections (cascade of selection)
B) Associativity of join
C) Distributivity of projection over union
D) Idempotence of union
**Answer: A**

**Q79.** In extended relational algebra, the assignment operator (←) is used to:
A) Compare two relations
B) Store the result of an expression into a temporary relation variable, useful for breaking complex queries into steps
C) Perform selection
D) Delete tuples
**Answer: B**

**Q80.** Relational Algebra is said to be "procedural" because:
A) It only allows procedures, not queries
B) A query specifies a sequence/order of operations describing HOW to obtain the result
C) It cannot be optimized
D) It has no set operators
**Answer: B**

**Q81.** π_(A,B)(R) − π_(A,B)(S) with same-degree union-compatible relations R and S is used to find:
A) Tuples with (A,B) values present in R but not in S
B) Tuples common to both
C) Cartesian product
D) All (A,B) pairs in S
**Answer: A**

**Q82.** Which is the correct relational algebra for "names of students enrolled in course 'DB101'" given Student(sid,name) and Enroll(sid,cid)?
A) π_name(σ_(cid='DB101')(Student ⋈ Enroll))
B) σ_name(π_(cid='DB101')(Student))
C) π_cid(Student)
D) σ_sid(Enroll)
**Answer: A**

**Q83.** Which of the following expresses "count of employees in each department" using extended relational algebra?
A) π_dept(Employee)
B) dept 𝔊 count(*) (Employee) — a generalized-projection/aggregation grouped by dept
C) σ_dept(Employee)
D) Employee ÷ dept
**Answer: B**

**Q84.** The Cartesian product operator is:
A) Commutative but not associative
B) Both commutative (up to attribute reordering) and associative
C) Neither commutative nor associative
D) Only defined for identical schemas
**Answer: B**

**Q85.** Which relational algebra operator combination is equivalent to natural join when there are NO common attribute names between R and S?
A) Cartesian product (R × S), since there's nothing to equate
B) Union
C) Set difference
D) It's undefined
**Answer: A**

## Section 3: Tuple Relational Calculus & Domain Relational Calculus

**Q86.** Relational calculus is a:
A) Procedural language specifying how to compute the result
B) Declarative (non-procedural) language specifying WHAT is required, not how to compute it
C) Same as relational algebra in every respect
D) Used only for updates
**Answer: B**

**Q87.** In Tuple Relational Calculus (TRC), a query has the general form:
A) { t | P(t) }, meaning the set of all tuples t such that predicate P(t) is true
B) { R × S }
C) σ_p(R)
D) π_A(R)
**Answer: A**

**Q88.** In TRC, the notation t[A] refers to:
A) The whole tuple t
B) The value of attribute A in tuple t
C) A relation named A
D) A predicate
**Answer: B**

**Q89.** The existential quantifier (∃) in TRC means:
A) "For all"
B) "There exists at least one"
C) "None"
D) "Exactly one"
**Answer: B**

**Q90.** The universal quantifier (∀) in TRC means:
A) "There exists"
B) "For all" tuples in the domain, the predicate must hold
C) "For none"
D) "For exactly one"
**Answer: B**

**Q91.** A TRC formula is "safe" if:
A) It always returns an infinite result
B) It guarantees a finite result set, typically by restricting values to those appearing in the relations/domains referenced
C) It contains no quantifiers
D) It uses only AND operators
**Answer: B** — Unsafe expressions could yield infinite relations (e.g., using negation without domain restriction).

**Q92.** Domain Relational Calculus (DRC) differs from TRC mainly in that:
A) DRC uses domain variables that range over attribute domains (single values), while TRC uses tuple variables
B) DRC has no quantifiers
C) DRC is procedural
D) There is no difference
**Answer: A**

**Q93.** ∀x (P(x)) is logically equivalent to:
A) ∃x (¬P(x))
B) ¬∃x (¬P(x))
C) ∃x (P(x))
D) P(x) ∧ Q(x)
**Answer: B** — De Morgan's law equivalence used to convert between quantifiers.

**Q94.** The expression { t | t ∈ Employee ∧ t[salary] > 50000 } represents:
A) All employees with salary > 50000 (equivalent to σ_(salary>50000)(Employee) in algebra)
B) All employees regardless of salary
C) Only distinct salary values
D) A join operation
**Answer: A**

**Q95.** TRC and DRC are both known to be equivalent in expressive power to:
A) Tuple calculus alone
B) Basic (core) relational algebra — this is called relational completeness
C) SQL DDL only
D) B+ trees
**Answer: B** — A language at least as expressive as relational algebra is "relationally complete."

**Q96.** In DRC, a query is written as: {⟨x1,x2,...,xn⟩ | P(x1,x2,...,xn)}. Here x1...xn are:
A) Relation names
B) Domain variables, each representing a value from an attribute's domain
C) Tuples
D) Predicates only
**Answer: B**

**Q97.** Which of the following is a key difference between relational algebra and relational calculus in terms of use?
A) Algebra tells "how" to compute (procedural); calculus tells "what" to compute (declarative/non-procedural)
B) Calculus is procedural, algebra is declarative
C) They cannot express the same set of queries
D) Calculus does not support conditions
**Answer: A**

**Q98.** The formula ∃t ∈ Employee (t[dept]='Sales' ∧ t[salary] > 60000) is used to check:
A) That all employees are in Sales
B) Whether there exists at least one employee in the Sales department earning more than 60000
C) The count of employees
D) A join between two relations
**Answer: B**

**Q99.** In TRC, "range of a tuple variable" refers to:
A) The set of tuples over which the variable can take values, typically defined by a relation membership condition (t ∈ R)
B) A numeric range only
C) The degree of the relation
D) The primary key range
**Answer: A**

**Q100.** Query Processing engines (like SQL optimizers) internally often translate declarative SQL queries into:
A) Tuple calculus expressions directly executed
B) An equivalent relational algebra expression (or extended algebra/query tree) for evaluation and optimization
C) Machine code directly with no intermediate representation
D) ER diagrams
**Answer: B**

---

## Section 4: Integrity Constraints

**Q101.** Domain constraint ensures that:
A) Every attribute value must be from its specified domain (atomic, valid data type/range)
B) No two tuples can be identical
C) Foreign keys must always be NOT NULL
D) All relations must have the same degree
**Answer: A**

**Q102.** Entity integrity constraint states that:
A) No attribute of a primary key can be NULL, and each relation must have a primary key
B) Foreign keys must reference existing values
C) All attributes must be unique
D) Domains must be identical across relations
**Answer: A**

**Q103.** Referential integrity constraint states that:
A) Primary key must not be NULL
B) A foreign key value must either match a primary/candidate key value in the referenced relation or be NULL (if allowed)
C) All attributes must have unique values
D) Domains must match exactly across all tables
**Answer: B**

**Q104.** A candidate key is:
A) Any attribute in a table
B) A minimal super key — a set of attributes that uniquely identifies a tuple, with no proper subset also doing so
C) Always a foreign key
D) Only applicable to weak entities
**Answer: B**

**Q105.** A super key is:
A) A minimal set of attributes uniquely identifying a tuple
B) Any set of attributes (possibly with extra/redundant attributes) that uniquely identifies each tuple
C) Only the primary key
D) A foreign key referencing itself
**Answer: B** — Every candidate key is a super key, but not vice versa.

**Q106.** When there are multiple candidate keys in a relation, the one chosen for implementation as the main identifier is called the:
A) Alternate key
B) Primary key
C) Foreign key
D) Composite key
**Answer: B** — The unchosen candidate keys become "alternate keys."

**Q107.** A foreign key is:
A) An attribute that must always be NULL
B) An attribute (or set) in one relation that references the primary/candidate key of another (or the same) relation, enforcing referential integrity
C) Always the same as a primary key
D) Only used in weak entities
**Answer: B**

**Q108.** ON DELETE CASCADE, as a referential action, means:
A) Deleting a referenced (parent) row is disallowed if child rows exist
B) Deleting a referenced row automatically deletes all dependent (child) rows referencing it
C) The child's foreign key is set to NULL
D) Nothing happens
**Answer: B**

**Q109.** ON DELETE SET NULL means:
A) Deleting the parent row is blocked
B) The parent row cannot exist
C) When a parent row is deleted, the foreign key value(s) in dependent child rows are automatically set to NULL
D) The child rows are deleted too
**Answer: C**

**Q110.** ON DELETE RESTRICT (or NO ACTION) means:
A) The delete cascades automatically
B) The delete operation on the parent is rejected/prevented if there exist matching (dependent) rows in the child table
C) Foreign key becomes NULL
D) Child rows are deleted
**Answer: B**

**Q111.** A NOT NULL constraint ensures:
A) An attribute can never be updated
B) An attribute must always have a value; NULL is disallowed
C) An attribute must be unique
D) An attribute is a foreign key
**Answer: B**

**Q112.** A CHECK constraint in SQL is used to:
A) Enforce referential integrity
B) Specify a condition/predicate that every value (or tuple) in a column must satisfy (a form of domain/tuple constraint)
C) Create indexes
D) Define primary keys only
**Answer: B**

**Q113.** Which of these constraints can span multiple relations (inter-relational constraints), unlike simple domain/key constraints?
A) Domain constraint
B) Referential integrity / general assertions
C) NOT NULL
D) UNIQUE
**Answer: B**

**Q114.** An "assertion" in SQL is:
A) A predicate expressing a condition the database must always satisfy, checked automatically (a general-purpose integrity constraint, not tied to one table)
B) A type of trigger only
C) A stored index
D) A type of join
**Answer: A**

**Q115.** Which of the following is TRUE about NULL values in relational integrity?
A) NULL is treated the same as zero or empty string
B) NULL represents an unknown, missing, or inapplicable value and follows special three-valued logic in comparisons
C) NULL is not allowed anywhere in a database
D) Two NULLs are always considered equal in comparisons
**Answer: B**

**Q116.** In three-valued logic, NULL = NULL evaluates to:
A) TRUE
B) FALSE
C) UNKNOWN
D) Error
**Answer: C**

**Q117.** The UNIQUE constraint differs from PRIMARY KEY in that:
A) UNIQUE columns can accept multiple NULL values (typically), while PRIMARY KEY columns cannot accept any NULL
B) UNIQUE always disallows NULLs entirely
C) A table can have only one UNIQUE constraint
D) There is no difference
**Answer: A**

**Q118.** Which integrity constraint would prevent inserting an Employee row with a department ID that doesn't exist in the Department table?
A) Entity integrity
B) Referential integrity
C) Domain constraint
D) Key constraint alone
**Answer: B**

**Q119.** A composite key is:
A) A key formed from a single attribute only
B) A candidate/primary key composed of two or more attributes together
C) Always a foreign key
D) A derived attribute
**Answer: B**

**Q120.** Which of the following best distinguishes a "trigger" from a "constraint" for enforcing integrity?
A) They are identical
B) A constraint is a declarative rule checked automatically; a trigger is a procedural block of code executed automatically in response to specified events (INSERT/UPDATE/DELETE), often used for complex business rules beyond declarative constraints
C) Triggers only apply to SELECT statements
D) Constraints require explicit invocation
**Answer: B**

## Section 5: Functional Dependencies & Normal Forms

**Q121.** A functional dependency X → Y holds in relation R if:
A) For any two tuples with the same Y value, X values must also match
B) For any two tuples agreeing on X values, they must also agree on Y values
C) X and Y must always be candidate keys
D) X must be a subset of Y
**Answer: B**

**Q122.** X → Y is a trivial functional dependency if:
A) Y is a subset of X
B) X is empty
C) X and Y are disjoint
D) Y determines X
**Answer: A** — Trivial FDs always hold and provide no real constraint info.

**Q123.** Armstrong's Axioms consist of which three basic inference rules?
A) Union, Difference, Intersection
B) Reflexivity, Augmentation, Transitivity
C) Selection, Projection, Join
D) Insertion, Deletion, Update
**Answer: B**

**Q124.** The Reflexivity axiom states:
A) If X → Y, then XZ → YZ
B) If Y ⊆ X, then X → Y
C) If X → Y and Y → Z, then X → Z
D) If X → Y, then Y → X
**Answer: B**

**Q125.** The Augmentation axiom states:
A) If X → Y, then XZ → YZ for any Z
B) If Y ⊆ X, then X → Y
C) If X → Y and Y → Z then X → Z
D) X → X always
**Answer: A**

**Q126.** The Transitivity axiom states:
A) If X → Y and Y → Z, then X → Z
B) If X → Y, then Y → X
C) If Y ⊆ X, then X → Y
D) X → Y implies XZ → Y
**Answer: A**

**Q127.** The "closure of a set of attributes X" (denoted X+) with respect to F is:
A) The set of all attributes functionally determined by X, given F
B) The power set of X
C) The set of candidate keys only
D) Always equal to all attributes of R
**Answer: A**

**Q128.** X is a super key of R if and only if:
A) X+ equals all attributes of R
B) X is empty
C) X+ = X
D) X contains no attributes from R
**Answer: A**

**Q129.** Two sets of functional dependencies F and G are equivalent (F ≡ G) if:
A) F = G exactly, symbol for symbol
B) F+ = G+ (their closures are equal), i.e., each set logically implies the other
C) They have the same number of FDs
D) F is a subset of G only
**Answer: B**

**Q130.** A canonical (minimal) cover of a set of FDs F is:
A) A set with maximum redundant FDs
B) An equivalent set of FDs where every FD has a single attribute on the RHS, no extraneous attributes on the LHS, and no FD is redundant
C) The same as the primary key
D) A cover containing only trivial FDs
**Answer: B**

**Q131.** First Normal Form (1NF) requires that:
A) There should be no partial dependency
B) All attribute values must be atomic (indivisible) — no repeating groups or multivalued/composite attributes within a single cell
C) There should be no transitive dependency
D) Every determinant must be a candidate key
**Answer: B**

**Q132.** A relation is in Second Normal Form (2NF) if it is in 1NF and:
A) Has no functional dependencies
B) Every non-prime attribute is fully functionally dependent on every candidate key (no partial dependency of a non-prime attribute on a proper subset of a candidate key)
C) Every determinant is a candidate key
D) Has no multivalued dependencies
**Answer: B**

**Q133.** Partial dependency occurs when:
A) A non-prime attribute depends on the whole candidate key
B) A non-prime attribute depends on a proper subset of a composite candidate key (rather than the whole key)
C) Two attributes are mutually dependent
D) An attribute depends on a non-key attribute
**Answer: B** — This violates 2NF; only relevant for composite keys.

**Q134.** A relation is in Third Normal Form (3NF) if, for every non-trivial FD X → A, at least one of these holds:
A) X is a super key, OR A is a prime attribute (part of some candidate key)
B) X must always be the primary key
C) A must be a foreign key
D) X must be a single attribute
**Answer: A**

**Q135.** Transitive dependency means:
A) X → Y directly, with no intermediate attribute
B) X → Y and Y → Z (where Y is not a super key and Z is a non-prime attribute), implying X → Z indirectly through Y
C) X → X
D) Y → X and X → Y simultaneously
**Answer: B** — 3NF eliminates transitive dependencies of non-prime attributes on keys (with the 3NF "prime attribute exception," unlike BCNF).

**Q136.** Boyce-Codd Normal Form (BCNF) requires that for every non-trivial FD X → Y:
A) X must be a super key (no exception for prime attributes, unlike 3NF)
B) Y must be a prime attribute
C) X must be a candidate key exactly, not a super key
D) There should be no functional dependencies at all
**Answer: A**

**Q137.** Which of the following is TRUE about the relationship between 3NF and BCNF?
A) BCNF is strictly weaker than 3NF
B) Every relation in BCNF is also in 3NF, but the converse is not always true (BCNF is stricter)
C) They are exactly equivalent in all cases
D) 3NF does not allow any functional dependency
**Answer: B**

**Q138.** A classic example where a relation is in 3NF but NOT in BCNF involves:
A) A relation with only one candidate key
B) A relation with overlapping composite candidate keys where a non-key attribute determines part of a key attribute (prime attribute) — e.g., R(A,B,C) with candidate keys AB and CB, and C→A
C) A relation with no functional dependencies
D) Any relation already in 1NF
**Answer: B**

**Q139.** BCNF decomposition guarantees:
A) Both lossless-join and dependency-preservation always
B) Lossless-join decomposition, but dependency preservation is NOT always guaranteed
C) Neither property
D) Only dependency preservation
**Answer: B** — This is a key trade-off: BCNF prioritizes eliminating redundancy over preserving all FDs.

**Q140.** 3NF decomposition (via the synthesis algorithm using a minimal cover) guarantees:
A) Neither lossless join nor dependency preservation
B) Both lossless-join decomposition AND dependency preservation
C) Only lossless join
D) Only dependency preservation
**Answer: B** — This is why 3NF is often used as a practical compromise when BCNF can't preserve dependencies.

**Q141.** A lossless-join decomposition of R into R1 and R2 requires that:
A) R1 ∩ R2 must be empty
B) The common attributes (R1 ∩ R2) must functionally determine either R1 or R2 (i.e., (R1∩R2) → R1 or (R1∩R2) → R2)
C) R1 and R2 must have the same attributes
D) No condition is needed; all decompositions are lossless
**Answer: B**

**Q142.** A "lossy" (lossless-join violating) decomposition, when rejoined via natural join, results in:
A) Exactly the original relation
B) A relation with the original tuples plus extra spurious tuples not present originally
C) An empty relation
D) Fewer tuples than the original
**Answer: B** — Spurious tuple generation is the key danger of a bad decomposition.

**Q143.** Multivalued dependency (MVD) X →→ Y in relation R means:
A) X functionally determines Y
B) For each value of X, the set of associated Y values is independent of the other attributes (Z = R−X−Y), leading to redundant combinations
C) Y determines X
D) X and Y are both keys
**Answer: B**

**Q144.** Fourth Normal Form (4NF) requires that:
A) All FDs are trivial
B) For every non-trivial multivalued dependency X →→ Y, X must be a super key
C) There are no candidate keys
D) It's the same requirement as 2NF
**Answer: B**

**Q145.** A relation R(A,B,C) with two independent multivalued dependencies A→→B and A→→C but no functional dependency, when NOT in 4NF, typically causes:
A) No redundancy at all
B) Redundant storage of combinations of B and C values for each A value (needs decomposition into R1(A,B) and R2(A,C))
C) Loss of the primary key
D) Automatic normalization
**Answer: B**

**Q146.** Fifth Normal Form (5NF) / Project-Join Normal Form (PJNF) deals with:
A) Functional dependencies only
B) Join dependencies — a relation is decomposed into multiple relations such that rejoining them via natural join reconstructs the original with no loss, and this decomposition cannot be simplified further
C) Multivalued dependencies exclusively
D) Domain constraints
**Answer: B**

**Q147.** Denormalization is the process of:
A) Converting a relation from 1NF to BCNF
B) Intentionally introducing redundancy into a normalized schema (merging tables) typically to improve read/query performance at the cost of update anomalies
C) Removing all functional dependencies
D) A mandatory step before normalization
**Answer: B**

**Q148.** An insertion anomaly occurs when:
A) You cannot insert certain information into the database without also having other unrelated information available (due to poor design), e.g., can't add a new course unless a student is enrolled
B) A row is duplicated
C) An update is delayed
D) A row cannot be deleted
**Answer: A**

**Q149.** A deletion anomaly occurs when:
A) Deleting a row unintentionally causes loss of other, unrelated information stored in the same row
B) A row cannot be updated
C) There is no key
D) Data is inserted twice
**Answer: A**

**Q150.** An update anomaly occurs when:
A) A piece of information is duplicated across multiple rows, requiring multiple updates to keep data consistent (risk of inconsistency if not all are updated)
B) Data can never be updated
C) A row is deleted incorrectly
D) The schema has too few attributes
**Answer: A**

**Q151.** Which normal form specifically addresses/eliminates all three classic anomalies (insertion, deletion, update) caused by partial and transitive dependencies for most practical schemas?
A) 1NF
B) 2NF only
C) 3NF/BCNF (by removing partial and transitive dependencies on keys)
D) None do
**Answer: C**

**Q152.** Attribute closure algorithm is primarily used to:
A) Compute candidate keys and verify functional dependency implications (whether X → Y is implied by F)
B) Compute joins
C) Draw ER diagrams
D) Compute B+ tree height
**Answer: A**

**Q153.** Which of these is an "extraneous attribute" in an FD's left-hand side (LHS)?
A) An attribute that, if removed from the LHS, the FD still holds (implied by the remaining attributes) — its presence is redundant
B) An attribute required for the FD to hold
C) The right-hand side attribute
D) A prime attribute always
**Answer: A**

**Q154.** In finding candidate keys of a relation using FDs, an attribute that appears only on the LHS of every FD and never on the RHS:
A) Can never be part of any candidate key
B) Must be part of EVERY candidate key
C) Is always a non-prime attribute
D) Is irrelevant to key computation
**Answer: B**

**Q155.** An attribute appearing on the RHS only (never on LHS) of any FD:
A) Must be part of every candidate key
B) Cannot be part of any candidate key (unless it is one of the attributes not covered by any FD)
C) Is always the primary key
D) Doesn't matter for normalization
**Answer: B**

**Q156.** Which of the following relations, given R(A,B,C,D) with FD set {A→B, B→C}, is a prime attribute?
A) A only
B) A and D (assuming AD is the only candidate key since D is not determined by anything)
C) B and C only
D) All attributes
**Answer: B** — Since A→B→C, A+ = {A,B,C}; D is needed to reach all attributes, so candidate key = AD; prime attributes are A and D.

**Q157.** Dependency preservation in decomposition means:
A) All original FDs (or their logical equivalent) can be checked/enforced using only the FDs local to each decomposed relation, without needing to join relations back together
B) The decomposition must be lossy
C) All attributes must appear in every decomposed relation
D) No FDs need to be preserved ever
**Answer: A**

**Q158.** Given FD A → BC, this is equivalent (by decomposition rule) to:
A) A → B and A → C separately
B) B → A and C → A
C) BC → A
D) A → B only
**Answer: A** — Decomposition/union rules of Armstrong's axioms (derived).

**Q159.** The Union rule of FDs states:
A) If X → Y and X → Z, then X → YZ
B) If X → YZ then X→Y and X→Z
C) If X→Y then Y→X
D) If Y⊆X then X→Y
**Answer: A**

**Q160.** The Pseudo-transitivity rule states:
A) If X → Y and WY → Z, then WX → Z
B) If X → Y then Y → X
C) Reflexivity implies transitivity
D) None of the above
**Answer: A**

**Q161.** In practice, which normal form is generally considered the recommended minimum target for most transactional (OLTP) database designs to avoid anomalies while remaining practical?
A) 1NF
B) 2NF
C) 3NF (or BCNF where dependency preservation isn't a concern)
D) 5NF always
**Answer: C**

**Q162.** A relation with a single candidate key that is also a single attribute (simple key) is automatically in:
A) 1NF only
B) At least 2NF (since partial dependency requires a composite key, which doesn't exist here)
C) Always in 5NF
D) Never in 3NF
**Answer: B**

**Q163.** Which statement about BCNF decomposition algorithm is correct?
A) It finds an FD X→Y violating BCNF (X not a superkey) and decomposes R into (X⁺) and (R − (X⁺ − X)), recursively repeating until all relations are in BCNF
B) It never terminates
C) It only works on relations with no FDs
D) It ignores functional dependencies
**Answer: A**

**Q164.** If a relation R has only ONE candidate key and is in 3NF, then R is:
A) Not necessarily in BCNF
B) Automatically in BCNF (a known theorem: 3NF with a single candidate key implies BCNF, since prime-attribute exception becomes moot practically in most cases)
C) Never in 1NF
D) Automatically in 5NF
**Answer: B**

**Q165.** Which of the following best explains WHY normalization is performed?
A) To increase data redundancy for faster reads
B) To reduce data redundancy and eliminate insertion/deletion/update anomalies by organizing attributes based on functional dependencies
C) To eliminate the need for primary keys
D) To remove all relationships between tables
**Answer: B**

## Section 6: File Organization

**Q166.** In a heap (unordered) file organization, new records are:
A) Inserted in sorted order based on a key
B) Inserted wherever there is space, typically at the end of the file (no particular order)
C) Always inserted using hashing
D) Not allowed after initial load
**Answer: B**

**Q167.** Searching for a record by a non-key attribute in a heap file with n blocks requires, on average:
A) O(log n) block accesses
B) n/2 block accesses on average (linear search), n in worst case
C) O(1) block access always
D) O(n log n)
**Answer: B**

**Q168.** In a sequential (sorted) file organization, records are physically stored:
A) Randomly
B) In sorted (ascending/descending) order based on a search key
C) Using a hash function only
D) In insertion order regardless of key
**Answer: B**

**Q169.** Searching a sorted sequential file of n blocks for a key value using binary search takes:
A) O(n)
B) O(log₂ n) block accesses
C) O(n²)
D) O(1) always
**Answer: B**

**Q170.** A major drawback of sequential file organization is:
A) Slow binary search
B) Insertions and deletions require reorganizing (shifting) the file to maintain sort order, which is costly; overflow blocks are often used to mitigate this
C) It cannot support range queries
D) It requires no maintenance
**Answer: B**

**Q171.** In hashing-based (hash) file organization, the address of a record's block is computed by:
A) Sequential scanning
B) Applying a hash function to the record's search/hash key value
C) Binary search on sorted keys
D) Using B+ tree traversal
**Answer: B**

**Q172.** Static hashing suffers from which major problem as the file grows or shrinks significantly?
A) No problems; it adapts automatically
B) Bucket overflow (too many collisions when file grows) or wasted space (when file shrinks), since the number of buckets is fixed
C) It cannot support any insertions
D) It requires B+ trees internally
**Answer: B**

**Q173.** Dynamic hashing (e.g., extendible hashing) addresses static hashing's limitations by:
A) Keeping the number of buckets always fixed
B) Allowing the hash structure (directory/bucket count) to grow or shrink dynamically as the database grows or shrinks, avoiding overflow chains
C) Removing the need for a hash function
D) Using only sequential search
**Answer: B**

**Q174.** In Extendible Hashing, the "global depth" refers to:
A) The number of buckets only
B) The number of bits of the hash value used by the directory to determine bucket pointers
C) The height of a B+ tree
D) The number of records per bucket
**Answer: B**

**Q175.** In Extendible Hashing, "local depth" of a bucket refers to:
A) The number of bits actually used to determine which records belong in that specific bucket (≤ global depth)
B) Always equal to global depth
C) The total number of buckets
D) The number of directory entries
**Answer: A**

**Q176.** When a bucket overflows in Extendible Hashing and its local depth equals the global depth, what happens?
A) The bucket is simply left overflowing
B) The directory doubles in size (global depth increases by 1) and the bucket is split
C) The entire file is rehashed from scratch
D) A new hash function is created randomly
**Answer: B**

**Q177.** Linear Hashing differs from Extendible Hashing in that:
A) It requires no directory structure; buckets are split in a predetermined linear order as the file grows, regardless of which bucket actually overflowed (using overflow chaining meanwhile)
B) It always doubles the directory upon overflow
C) It's identical to static hashing
D) It doesn't use a hash function
**Answer: A**

**Q178.** Clustering file organization (or clustered index) stores records:
A) Randomly ignoring the value of any attribute
B) Physically grouped together on disk based on the value of a particular (often non-key) attribute, useful for range/equality queries on that clustering attribute across possibly multiple relations
C) Only using hashing
D) Always in a single relation only
**Answer: B**

**Q179.** A "clustering index" differs from a table's primary organization in that:
A) It's built on a non-key attribute used to group/order records by that attribute's value; a table can have only ONE clustering order (since data can be physically sorted only one way) but multiple secondary/non-clustered indexes
B) There can be many clustering indexes per table
C) It never affects physical storage order
D) It only works with hashing
**Answer: A**

**Q180.** Which file organization is best suited for a workload dominated by range queries (e.g., "find all records with salary between 40000 and 60000")?
A) Heap file
B) Sequential (sorted) file organization or B+ tree index
C) Static hashing on salary
D) Random organization
**Answer: B** — Hashing is efficient for equality lookups, not range queries.

**Q181.** Which file organization is generally best for equality-only lookups (e.g., "find record with ID = 1023") with minimal I/O?
A) Sequential file with binary search
B) Hashing (ideally O(1) average-case block access)
C) Heap file linear scan
D) B+ tree (though also good, hashing can be faster for pure equality)
**Answer: B**

**Q182.** Overflow chaining in hash file organization is used to handle:
A) Directory doubling
B) Collisions — when a bucket is full, additional records are stored in linked overflow blocks attached to that bucket
C) Sorting of records
D) B+ tree splitting
**Answer: B**

**Q183.** A "record" in file organization terminology refers to:
A) An entire relation
B) A single row/tuple's collection of field (attribute) values, typically stored contiguously in a block
C) A single attribute value
D) An index entry only
**Answer: B**

**Q184.** Fixed-length records simplify file organization primarily because:
A) They save more space than variable-length records always
B) The starting address of the i-th record can be computed directly via simple arithmetic (base + i × record_size), enabling direct access
C) They cannot be deleted
D) They require hashing
**Answer: B**

**Q185.** Variable-length records commonly require additional structures such as:
A) Nothing extra is needed
B) Length indicators/separators, or a slot/offset directory within the block (e.g., a page/slot directory design) to locate each record
C) Fixed block sizes only
D) B+ trees mandatorily
**Answer: B**

**Q186.** In file organization, a "block" (or page) is:
A) A single byte
B) The unit of data transfer between disk and memory — typically holds multiple records, and I/O cost is measured in block accesses
C) Always exactly one record
D) A type of index
**Answer: B**

**Q187.** Which of these best explains why minimizing the number of block (disk I/O) accesses is central to file organization and index design?
A) CPU operations are the primary bottleneck
B) Disk I/O is orders of magnitude slower than memory access, so database performance is generally dominated by the number of block transfers, not CPU cycles
C) Memory is always the bottleneck
D) It has no real impact on performance
**Answer: B**

**Q188.** Tombstoning (using deletion markers) in file organization is used to:
A) Physically remove and compact space immediately
B) Mark a record as deleted without immediately reclaiming/reorganizing space, deferring actual space reclamation
C) Create a new index
D) Sort the file
**Answer: B**

**Q189.** Bucket-based hashing organizations typically define a "bucket" as:
A) A single record
B) A unit of storage (which may be one or more blocks) that can hold one or more records, addressed by a hash value
C) The entire file
D) A type of B+ tree node
**Answer: B**

**Q190.** Which file organization strategy is most vulnerable to poor performance from a bad/non-uniform hash function causing many collisions?
A) Heap file
B) Hash file organization
C) Sequential file
D) None are affected by hash functions
**Answer: B**

## Section 7: Indexing — Single-level, Multilevel, B-Trees and B+ Trees

**Q191.** A primary index is built on:
A) Any non-key attribute of an unordered file
B) The ordering key attribute of a sequentially ordered (sorted) file
C) Only secondary storage
D) A hash key exclusively
**Answer: B**

**Q192.** A dense index contains:
A) An index entry for only some of the search key values
B) An index entry for EVERY search key value (or every record) in the data file
C) No entries at all
D) Only entries for the first record in each block
**Answer: B**

**Q193.** A sparse (non-dense) index contains:
A) An index entry for every record
B) An index entry for only SOME search key values, typically one per block (e.g., the first key of each block), requiring the data file to be sorted on that key
C) No relation to sorting
D) Entries for deleted records only
**Answer: B**

**Q194.** A primary index built on a candidate key of a sorted file is typically:
A) Dense only
B) Sparse (one entry per block is sufficient since the file is sorted on that key)
C) Neither dense nor sparse
D) Impossible to build
**Answer: B** — Though a dense primary index is also possible; sparse is the classic space-efficient choice.

**Q195.** A clustering index is built on:
A) A non-key ordering field of a sorted file (where multiple records can share the same value), unlike a primary index which is on a key
B) A key attribute exclusively
C) An unsorted file only
D) Only hash-based files
**Answer: A**

**Q196.** A secondary index is built on:
A) A field that does NOT determine the physical ordering of the file, so it must generally be DENSE (an entry for every record, since records with that key value can be scattered anywhere)
B) Always the same as a primary index
C) A sorted field, sparse always
D) It cannot support non-key fields
**Answer: A**

**Q197.** A multilevel index is created primarily to:
A) Slow down search
B) Reduce the number of block accesses in a search by treating the first-level index itself as a data file and building a second-level (index-on-index) structure, and so on, until the top level fits in one block
C) Remove the need for a first-level index
D) Only works with hash files
**Answer: B**

**Q198.** The search cost for a multilevel index with t levels is approximately:
A) O(n) block accesses
B) O(t) block accesses — one per level, generally much smaller than a single-level index's O(log₂(index blocks))
C) O(n²)
D) Constant regardless of levels
**Answer: B**

**Q199.** A B-Tree of order p (or degree) has each internal node containing:
A) Exactly p keys always
B) At most p−1 keys and at most p tree pointers (children), with search key values AND associated data pointers stored at every node (including internal nodes)
C) Only leaf-level data
D) No pointers at all
**Answer: B**

**Q200.** The key structural difference between a B-Tree and a B+ Tree is:
A) B-trees store data (record pointers) at every node (internal and leaf); B+ trees store data pointers ONLY at leaf nodes, with internal nodes used purely for indexing/navigation
B) B+ trees are unbalanced while B-trees are balanced
C) B-trees don't support range queries at all
D) They are exactly identical structures
**Answer: A**

**Q201.** In a B+ tree, leaf nodes are additionally linked together via:
A) No linkage at all
B) A linked list (sibling pointers) enabling efficient sequential/range-query traversal without going back up the tree
C) Only parent pointers
D) A hash table
**Answer: B**

**Q202.** Why are B+ trees generally preferred over B-trees in most commercial DBMS implementations?
A) B+ trees are simpler to implement in every case
B) B+ trees pack more keys per internal node (since they don't store data pointers there), reducing tree height/fan-out needs, and support efficient sequential range scans via linked leaves — both improving I/O performance
C) B-trees cannot be balanced
D) B+ trees use less total disk space always
**Answer: B**

**Q203.** In a B+ tree of order p, an internal (non-leaf) node has:
A) Exactly p pointers always, no exceptions
B) At most p tree pointers and at most (p−1) search key values, with at least ⌈p/2⌉ pointers (except the root)
C) Only 2 pointers always
D) No key values
**Answer: B**

**Q204.** In a B+ tree, the leaf node order p_leaf typically holds:
A) At most p_leaf − 1 search values and p_leaf − 1 record pointers, plus one extra pointer to the next leaf node
B) Exactly p_leaf keys with no next-pointer
C) Only internal keys
D) Nothing — leaves are empty
**Answer: A**

**Q205.** B-trees and B+ trees remain "balanced" because:
A) They are never restructured
B) Insertions/deletions use split and merge (or redistribution) operations that keep all leaf nodes at the same depth from the root
C) Only the root ever changes
D) Balance is not actually guaranteed
**Answer: B**

**Q206.** When a B+ tree leaf node overflows during insertion, the typical action is:
A) Delete the tree and rebuild
B) Split the leaf node into two, redistribute keys, and insert (copy up) the middle/first key of the new right leaf into the parent
C) Ignore the overflow
D) Convert to a hash table
**Answer: B**

**Q207.** When an internal (non-leaf) node overflows in a B+ tree, splitting differs from leaf split in that:
A) It's identical to leaf split in every way
B) The middle key is pushed UP into the parent (not copied — it's removed from the node being split, unlike leaf splits which copy it up)
C) No key moves up at all
D) The entire tree is rebuilt
**Answer: B**

**Q208.** If deletion from a B+ tree leaf causes the node to have fewer than the minimum required entries, the typical fix-up strategy is:
A) Leave it as is always
B) Try to redistribute entries from an adjacent sibling; if not possible, merge with a sibling and adjust the parent (which may recursively underflow)
C) Delete the entire tree
D) Convert the node into a root
**Answer: B**

**Q209.** The height of a B+ tree indexing N records, with each node having a maximum fan-out of p, is approximately:
A) O(N)
B) O(log_p N) — since each level reduces the search space by a factor of p (roughly), giving logarithmic height
C) O(p)
D) Always exactly 1
**Answer: B**

**Q210.** In terms of worst-case search, insertion, and deletion cost, B+ trees provide:
A) O(n) linear time
B) O(log n) time (proportional to tree height), making them efficient for large, dynamically changing datasets
C) O(n²)
D) O(1) always
**Answer: B**

**Q211.** A B+ tree is especially well suited for disk-based DBMS indexing because:
A) It requires very deep trees
B) Its high fan-out (many keys per node, sized to match the disk block size) keeps the tree height low, minimizing disk I/O per search
C) It cannot handle range queries
D) It requires no rebalancing
**Answer: B**

**Q212.** Range queries (e.g., "find all records with key between 10 and 50") are handled efficiently in a B+ tree by:
A) Full linear scan of the data file only
B) Locating the starting key via a top-down search to the leaf level, then following the leaf-level linked list sequentially until the end of range is reached
C) Rebuilding the tree for every query
D) B+ trees cannot support range queries
**Answer: B**

**Q213.** Which statement about B-tree node occupancy (minimum fill factor) is generally true (excluding the root)?
A) Nodes can be completely empty
B) Every non-root node must be at least half full (contain at least ⌈p/2⌉ pointers/keys, depending on convention), ensuring reasonable space utilization
C) Nodes must always be 100% full
D) There's no minimum requirement
**Answer: B**

**Q214.** A "sparse" secondary index is generally NOT feasible because:
A) Secondary index keys don't correspond to the physical sort order of the data file, so without an entry for every value, some records could not be located directly — hence secondary indexes must typically be dense
B) It's actually always feasible
C) Sparse indexes don't exist
D) Secondary indexes never need dense entries
**Answer: A**

**Q215.** Which of these correctly compares dense vs sparse indexes in terms of trade-offs?
A) Dense indexes use less storage than sparse but are slower
B) Sparse indexes use less storage (fewer entries) but require the file to be sorted; dense indexes use more storage but allow faster/direct lookup without depending on file order and support existence checks without accessing the data file
C) They have identical storage requirements
D) Sparse indexes work on unsorted files
**Answer: B**

**Q216.** A composite (multi-attribute) index built on (A, B) is most useful for queries that:
A) Filter only on B
B) Filter on A alone, or on A and B together (leftmost prefix), due to the ordering of the index primarily by A then B
C) Cannot be used for any queries
D) Only work with hash indexes
**Answer: B**

**Q217.** Bitmap indexes are particularly efficient for:
A) High-cardinality (many distinct values) columns like a primary key
B) Low-cardinality columns (few distinct values, e.g., gender, status flags), enabling fast bitwise operations (AND/OR) for combining multiple conditions
C) Only numeric range queries
D) B+ tree replacement in all cases
**Answer: B**

**Q218.** A "covering index" is one that:
A) Contains only the primary key
B) Contains all the columns needed to satisfy a query directly from the index itself, without needing to access the actual data rows (avoiding extra I/O)
C) Covers the entire table physically
D) Is the same as a clustering index always
**Answer: B**

**Q219.** In terms of ordering of keys within nodes, a defining property of both B-trees and B+ trees is:
A) Keys within a node are unordered
B) Keys within each node are maintained in sorted order, and subtree pointers partition the key space accordingly (all keys in left subtree < key, all in right subtree ≥ key, etc.)
C) Only leaf nodes are sorted
D) Sorting is irrelevant to tree operations
**Answer: B**

**Q220.** Compared to a binary search tree (BST) used for the same purpose, a B+ tree is preferred for disk-resident indexes primarily because:
A) BSTs are always faster on disk
B) A B+ tree's high fan-out drastically reduces tree height (and thus disk I/O) compared to a BST, which has fan-out of only 2 per node and would require far more disk block accesses for the same number of keys
C) BSTs cannot store any keys
D) There is no meaningful difference
**Answer: B**

**Q225.** Hash indexes generally outperform B+ tree indexes for:
A) Range queries
B) Exact-match (equality) queries, offering close to O(1) average lookup time versus O(log n) for B+ trees
C) Sorted output requirements
D) Multi-attribute composite queries
**Answer: B**

**Q221.** The "fan-out" of a B+ tree node refers to:
A) The number of keys stored in a leaf only
B) The number of child pointers an internal node can have — higher fan-out means a shallower tree for the same number of keys
C) The number of levels in the tree
D) The number of deleted entries
**Answer: B**

**Q222.** Which of the following is TRUE when comparing a B+ tree index to a full table (heap) scan for a highly selective equality query (returning very few rows out of millions)?
A) A full table scan is always faster
B) The B+ tree index is typically far more efficient, since it needs only O(log n) block accesses versus scanning the entire table
C) They perform identically
D) B+ trees cannot be used for equality queries
**Answer: B**

**Q223.** A non-clustered (secondary) B+ tree index's leaf nodes typically store:
A) The full data record itself
B) The search key value along with a pointer (record ID/RID) to the actual data record located elsewhere (e.g., in the heap or clustered structure)
C) No information at all
D) Only the primary key of a different table
**Answer: B**

**Q224.** A clustered B+ tree index's leaf level typically:
A) Stores only pointers, never actual data
B) IS the actual data file itself (or very close to it) — the table's rows are physically stored in the order of the index key at the leaf level
C) Is unrelated to physical row order
D) Requires a completely separate secondary structure always
**Answer: B**

## Section 8: Transactions — ACID Properties & Transaction States

**Q226.** A transaction is:
A) A single SQL SELECT statement only
B) A logical unit of work comprising one or more operations (reads/writes) that must be executed as an atomic, indivisible whole
C) A physical disk block
D) A type of index
**Answer: B**

**Q227.** The 'A' in ACID stands for Atomicity, which means:
A) A transaction can be partially completed and left that way permanently
B) Either ALL operations of a transaction are completed successfully (committed), or NONE are (fully rolled back) — "all or nothing"
C) Transactions run one at a time only
D) Data must always be numeric
**Answer: B**

**Q228.** Consistency (the 'C' in ACID) guarantees that:
A) A transaction takes the database from one consistent state to another, preserving all defined integrity constraints and invariants
B) Two transactions always produce the same result
C) Data is never modified
D) Transactions run in isolation always
**Answer: A**

**Q229.** Isolation (the 'I' in ACID) ensures that:
A) Transactions must run one after another with no concurrency ever
B) Concurrently executing transactions appear (from each transaction's perspective) as if they were executed serially/independently — intermediate states of one transaction are not visible to others
C) Data changes are never made permanent
D) All transactions share the same variables
**Answer: B**

**Q230.** Durability (the 'D' in ACID) guarantees that:
A) Once a transaction commits, its changes persist permanently in the database even in the event of subsequent system failures (e.g., via write-ahead logging)
B) Data is deleted after use
C) Transactions are always fast
D) Isolation is maintained
**Answer: A**

**Q231.** The transaction states, in typical order, are:
A) Active → Partially Committed → Committed (or Failed → Aborted)
B) Committed → Active → Failed
C) Aborted → Active → Committed
D) There is only one state: Active
**Answer: A**

**Q232.** The "Active" state of a transaction means:
A) The transaction has finished and committed
B) The transaction is currently executing (initial state, most operations happen here)
C) The transaction has failed
D) The transaction is waiting to be rolled back
**Answer: B**

**Q233.** "Partially Committed" state occurs:
A) Before the transaction starts
B) After the final statement has executed but before the changes are confirmed as permanently committed to the database (some checks/writes to stable storage still pending)
C) After a full rollback
D) It's the same as Active
**Answer: B**

**Q234.** A transaction enters the "Failed" state when:
A) It has committed successfully
B) The normal execution cannot proceed further due to a hardware/software error or an internal condition (e.g., constraint violation), and the transaction cannot continue
C) It is merely reading data
D) It is waiting for a lock only, with no error
**Answer: B**

**Q235.** The "Aborted" state means:
A) The transaction has committed
B) The transaction has been rolled back and the database restored to the state prior to the transaction's start (after a failure); the transaction may be restarted or killed
C) The transaction is still active
D) It refers only to read-only transactions
**Answer: B**

**Q236.** Once a transaction reaches the "Committed" state:
A) It can still be rolled back easily
B) Its effects are permanent (durable) and cannot be undone by aborting; a "compensating transaction" would be needed to reverse effects if required
C) It has failed
D) It has no effect on the database at all
**Answer: B**

**Q237.** A "schedule" in transaction processing refers to:
A) A single transaction's internal operations only
B) A sequence (chronological order) of the interleaved operations from one or more transactions as they are executed by the DBMS
C) A type of index
D) The physical layout of files on disk
**Answer: B**

**Q238.** A "serial schedule" is one where:
A) Operations of different transactions are interleaved arbitrarily
B) Transactions are executed one after another completely (no interleaving) — one transaction finishes entirely before the next begins
C) No transaction ever completes
D) Only read operations are allowed
**Answer: B**

**Q239.** A schedule is "serializable" if:
A) It must literally be a serial schedule
B) Its effect (final database state / outcome) is equivalent to that of SOME serial schedule of the same transactions, even though operations may be interleaved
C) It always has conflicts
D) It cannot be interleaved at all
**Answer: B**

**Q240.** Conflict serializability is determined based on:
A) Any two random operations
B) Two operations (from different transactions) on the SAME data item where at least one is a write — these are "conflicting operations," and their relative order matters for equivalence
C) Only read operations
D) Operations on different data items
**Answer: B**

## Section 9: Serializability, Precedence Graph, Recoverability

**Q241.** A precedence graph (serialization graph) is used to test:
A) Deadlocks only
B) Conflict serializability — a node per transaction, with a directed edge Ti → Tj if Ti has an operation that conflicts with, and precedes, an operation of Tj
C) Index structures
D) Normalization
**Answer: B**

**Q242.** A schedule is conflict-serializable if and only if:
A) Its precedence graph has at least one cycle
B) Its precedence graph is ACYCLIC (contains no cycles); a topological sort of the acyclic graph gives an equivalent serial order
C) It contains no read operations
D) All transactions committed
**Answer: B**

**Q243.** View serializability differs from conflict serializability in that:
A) They are always identical for every schedule
B) View serializability is a broader (less strict) criterion based on matching initial reads, final writes, and read-from relationships between transactions, and it allows for some non-conflict-serializable schedules (like those with "blind writes") to still be considered serializable
C) View serializability is stricter than conflict serializability
D) View serializability doesn't consider writes at all
**Answer: B**

**Q244.** Every conflict-serializable schedule is:
A) Never view-serializable
B) Also view-serializable (conflict serializability implies view serializability, but not vice versa)
C) The same thing as view-serializable in all cases
D) Not a valid schedule
**Answer: B**

**Q245.** A "blind write" refers to:
A) A write operation preceded by a read of the same data item by the same transaction
B) A write operation on a data item WITHOUT a preceding read of that same item by the same transaction
C) A write that always fails
D) An operation on an index only
**Answer: B** — Blind writes are key to schedules that are view-serializable but not conflict-serializable.

**Q246.** A schedule is "recoverable" if:
A) A transaction can always be rolled back regardless of other transactions
B) Whenever a transaction Tj reads a data item written by Ti, Tj commits only AFTER Ti commits (ensuring no transaction commits based on data from an uncommitted/later-aborted transaction)
C) All transactions run serially
D) No transaction reads uncommitted data at all
**Answer: B**

**Q247.** A schedule "avoids cascading rollback" (is ACR / cascadeless) if:
A) Transactions never commit
B) Every transaction reads only data items that were written by transactions that have ALREADY COMMITTED (i.e., no dirty reads at all), preventing a chain of rollbacks if one transaction aborts
C) It allows dirty reads freely
D) All transactions are read-only
**Answer: B**

**Q248.** A "strict" schedule requires that:
A) Transactions can read or overwrite a data item written by another transaction at any time
B) A transaction can neither READ nor WRITE a data item until the transaction that last wrote it has COMMITTED or ABORTED — this simplifies recovery (easy UNDO using before-images)
C) Only reads are restricted, writes are unrestricted
D) It's the weakest recoverability level
**Answer: B**

**Q249.** The hierarchy of recoverability properties, from strongest (most restrictive) to weakest, is:
A) Recoverable ⊃ Cascadeless (ACR) ⊃ Strict ⊃ Serial
B) Serial ⊂ Strict ⊂ Cascadeless (ACR) ⊂ Recoverable (Serial is most restrictive/strongest; Recoverable is the weakest/broadest requirement)
C) They are all identical requirements
D) Recoverable is stricter than Strict
**Answer: B**

**Q250.** A "dirty read" occurs when:
A) A transaction reads data that has already been committed
B) A transaction reads a data item that was modified by another transaction that has NOT yet committed (and might later abort, making the read value invalid)
C) A transaction reads from an empty table
D) Data is read from a backup
**Answer: B**

## Section 10: Concurrency Control — Locking Protocols

**Q251.** The purpose of concurrency control protocols in a DBMS is to:
A) Speed up single-transaction execution only
B) Ensure that concurrently executing transactions produce serializable (correct, consistent) results, preventing anomalies like lost updates, dirty reads, and unrepeatable reads
C) Eliminate the need for transactions
D) Only manage disk space
**Answer: B**

**Q252.** A shared (S) lock on a data item allows:
A) The holding transaction to both read and write the item, and no other transaction may access it
B) The holding transaction (and other transactions that also acquire a shared lock) to READ the item, but no transaction may write to it while any shared lock is held
C) Only one specific transaction to read or write
D) No access to anyone
**Answer: B**

**Q253.** An exclusive (X) lock on a data item allows:
A) Multiple transactions to read and write simultaneously
B) Only the transaction holding the lock to both read AND write the item; no other transaction may hold any lock (shared or exclusive) on it concurrently
C) Only reading, no writing
D) Unlimited concurrent shared locks
**Answer: B**

**Q254.** Lock compatibility: Can two transactions simultaneously hold a Shared (S) lock on the same item?
A) No, never
B) Yes — multiple shared locks are compatible with each other (only S-X and X-X are incompatible)
C) Only if they are the same transaction
D) Only during commit
**Answer: B**

**Q255.** The Two-Phase Locking (2PL) protocol requires that within each transaction:
A) Locks can be acquired and released at any time in any order
B) All lock acquisitions (growing phase) must precede any lock release (shrinking phase) — once a transaction releases even one lock, it cannot acquire any new locks
C) All locks are released before any are acquired
D) Only exclusive locks are used
**Answer: B**

**Q256.** Basic 2PL guarantees:
A) Freedom from deadlock
B) Conflict serializability of schedules, but does NOT by itself guarantee freedom from deadlock or cascading rollback
C) Freedom from cascading rollback automatically
D) Nothing about correctness
**Answer: B**

**Q257.** Strict Two-Phase Locking (Strict 2PL) additionally requires that:
A) All locks are released immediately after use
B) All EXCLUSIVE (write) locks held by a transaction are released only AFTER the transaction commits or aborts (not before), which ensures strictness/recoverability and avoids cascading rollbacks
C) Growing phase never ends
D) No shared locks are allowed
**Answer: B**

**Q258.** Rigorous Two-Phase Locking requires that:
A) Only shared locks are held until commit
B) ALL locks (both shared and exclusive) are held until the transaction commits or aborts, providing the strongest guarantee (serializable, recoverable, cascadeless) and simplifying scheduling since transactions appear to execute in commit order
C) Locks are never released
D) It's identical to basic 2PL
**Answer: B**

**Q259.** Conservative (Static) Two-Phase Locking requires a transaction to:
A) Acquire locks one at a time as needed during execution
B) Predeclare and acquire ALL the locks it will ever need BEFORE it begins execution, preventing deadlock entirely (since a transaction never waits for a lock while holding others), but requiring advance knowledge of the data items to be accessed
C) Never acquire any locks
D) Only lock the last item accessed
**Answer: B**

**Q260.** Which of these is a known problem specifically prevented by locking protocols like 2PL, but which can occur in the absence of concurrency control (e.g., "lost update" problem)?
A) Two transactions read the same value, both compute updates based on that value, and the second transaction's write overwrites (silently loses) the first transaction's update
B) A transaction commits too quickly
C) Data becomes duplicated in storage
D) The schema becomes denormalized
**Answer: A**

**Q261.** Deadlock in the context of locking occurs when:
A) A transaction is waiting for a lock that will eventually be released
B) Two or more transactions are each waiting for a lock held by another transaction in the set, forming a cycle of dependencies where none can proceed
C) Only one transaction is active
D) All locks are shared locks
**Answer: B**

**Q262.** Deadlock detection is typically performed using a:
A) Precedence graph for serializability
B) Wait-for graph — a node per transaction with an edge Ti → Tj if Ti is waiting for a lock held by Tj; a cycle in this graph indicates deadlock
C) B+ tree
D) Hash table of locks only
**Answer: B**

**Q263.** Deadlock prevention protocols like Wait-Die and Wound-Wait use which information to decide whether a transaction should wait or be aborted?
A) The size of the transaction
B) Transaction timestamps (relative age/priority) — older vs younger transactions are treated differently to avoid cyclic waits
C) Random selection only
D) The number of locks held
**Answer: B**

**Q264.** In the Wait-Die scheme, if transaction Ti requests a lock held by Tj:
A) Ti always waits regardless of age
B) If Ti is OLDER than Tj, Ti is allowed to WAIT; if Ti is YOUNGER, Ti is aborted (DIES) and restarted later with the same original timestamp
C) Tj is always aborted
D) Both are aborted always
**Answer: B**

**Q265.** In the Wound-Wait scheme, if transaction Ti requests a lock held by Tj:
A) Ti always dies
B) If Ti is OLDER than Tj, Ti WOUNDS (forces abort of) Tj, which restarts; if Ti is YOUNGER, Ti WAITS
C) Both wait indefinitely
D) It is identical to Wait-Die in every respect
**Answer: B**

**Q266.** Both Wait-Die and Wound-Wait schemes guarantee freedom from deadlock because:
A) They allow cyclic waiting
B) The relative age-based rule ensures that a transaction only ever waits for OLDER (or only YOUNGER, depending on scheme) transactions consistently, preventing a cycle from forming, and restarted transactions retain their original timestamp (preventing starvation)
C) They use random priorities
D) Locks are never granted
**Answer: B**

**Q267.** Timeout-based deadlock handling works by:
A) Detecting cycles in a wait-for graph explicitly
B) Having a transaction that waits for a lock LONGER than a specified timeout period assumed to be deadlocked and rolled back — simple but can cause unnecessary rollbacks (false positives) or leave real deadlocks undetected if timeout is too generous
C) Predeclaring all locks
D) Never rolling back any transaction
**Answer: B**

**Q268.** Starvation in the context of locking occurs when:
A) A transaction is repeatedly aborted or made to wait indefinitely (e.g., always chosen as the deadlock victim, or continuously bypassed by higher-priority transactions) while other transactions proceed normally
B) All transactions commit simultaneously
C) A deadlock is detected immediately
D) Locks are never granted to anyone
**Answer: A**

**Q269.** Lock granularity refers to:
A) The type of lock (shared/exclusive) only
B) The SIZE of the data item being locked — from an entire database, to a table, to a page/block, down to an individual tuple/field; finer granularity increases concurrency but adds locking overhead
C) The number of transactions
D) The duration a lock is held
**Answer: B**

**Q270.** Multiple-granularity locking uses "intention locks" (IS, IX, SIX) to:
A) Replace shared and exclusive locks entirely
B) Allow a transaction to efficiently signal its intent to lock finer-granularity items lower in a hierarchy (e.g., database→table→page→row), so higher-level compatibility can be checked quickly without examining every lower-level item
C) Prevent all locking
D) Apply only to hash indexes
**Answer: B**

## Section 11: Timestamp Ordering, Optimistic CC, Multiversion CC & Recovery

**Q271.** In Timestamp Ordering (TO) protocol, each transaction is assigned:
A) A lock priority only
B) A unique timestamp (typically at start), used to determine the serialization order of conflicting operations — the schedule produced is equivalent to the serial order of the timestamps
C) A fixed set of pre-acquired locks
D) A hash value
**Answer: B**

**Q272.** In Basic Timestamp Ordering, for each data item Q, the system maintains:
A) Only a single timestamp
B) W-timestamp(Q) — the largest timestamp of any transaction that successfully wrote Q, and R-timestamp(Q) — the largest timestamp of any transaction that successfully read Q
C) A count of accesses only
D) No metadata is maintained
**Answer: B**

**Q273.** Under TO protocol, if transaction Ti with timestamp TS(Ti) attempts to WRITE data item Q, and TS(Ti) < R-timestamp(Q) or TS(Ti) < W-timestamp(Q):
A) The write proceeds normally
B) The write is rejected and Ti is rolled back (restarted with a new timestamp) because a younger transaction already read or wrote a more "current" value, violating serializability order
C) The write always succeeds silently
D) Nothing happens; it's ignored
**Answer: B**

**Q274.** Under TO protocol, if transaction Ti attempts to READ data item Q and TS(Ti) < W-timestamp(Q):
A) The read proceeds normally regardless
B) The read is rejected and Ti is rolled back, since Ti is trying to read a value that was overwritten by a "later" (in timestamp order) transaction — reading it would violate the required serialization order
C) The write is undone instead
D) It causes a deadlock
**Answer: B**

**Q275.** A major advantage of Timestamp Ordering protocols over lock-based protocols is:
A) They require more overhead
B) They are deadlock-free by construction — since transactions never wait for locks (a transaction is either allowed to proceed or is aborted/restarted immediately), there is no possibility of cyclic waiting
C) They cannot support serializability at all
D) They eliminate need for timestamps
**Answer: B**

**Q276.** The Thomas Write Rule is a modification to basic TO protocol that:
A) Rejects fewer writes than basic TO by allowing certain "obsolete" writes (where TS(Ti) < W-timestamp(Q) but TS(Ti) ≥ R-timestamp(Q)) to be safely IGNORED rather than causing a rollback, since the write would be immediately overwritten anyway
B) Increases the number of rollbacks
C) Applies only to read operations
D) Is identical to strict 2PL
**Answer: A**

**Q277.** Optimistic Concurrency Control (OCC) is based on the assumption that:
A) Conflicts between transactions are frequent, so locking should be used aggressively
B) Conflicts are RARE, so transactions execute without restrictive locking during a "read phase," and are only checked for conflicts at a "validation phase" before entering the "write phase"
C) All transactions must be serial
D) No validation is ever needed
**Answer: B**

**Q278.** The three phases of Optimistic Concurrency Control are:
A) Lock, Execute, Unlock
B) Read Phase (execute and buffer writes locally), Validation Phase (check for conflicts against other concurrently validated/committed transactions), and Write Phase (if validated, apply buffered writes to the database)
C) Commit, Abort, Retry
D) Start, Middle, End
**Answer: B**

**Q279.** OCC works best in environments where:
A) There are many conflicting writes on the same data (high contention)
B) There are relatively FEW conflicts among transactions (low contention), since aborting/restarting a transaction that fails validation can be costly if it happens often
C) Locking overhead is negligible
D) All transactions are long-running writes
**Answer: B**

**Q280.** Multiversion Concurrency Control (MVCC) works by:
A) Maintaining only a single version of each data item, as in standard locking
B) Maintaining MULTIPLE versions (a history) of each data item, so that read operations can be given an appropriate (often older/consistent) version without blocking or being blocked by concurrent write operations, improving concurrency for read-heavy workloads
C) Preventing all writes
D) Deleting old versions immediately
**Answer: B**

**Q281.** A key benefit of MVCC is that:
A) Reads and writes always conflict
B) READ operations generally never need to wait for or block WRITE operations (and vice versa), since reads can be served an appropriate earlier version, significantly boosting concurrency for mixed read/write workloads
C) It cannot support snapshot isolation
D) It requires only one version at all times
**Answer: B**

**Q282.** "Snapshot Isolation," commonly implemented via MVCC, ensures that a transaction:
A) Sees a live, constantly changing view of the database throughout its execution
B) Sees a consistent SNAPSHOT of the database as it existed at the start of the transaction, unaffected by concurrent transactions' subsequent updates (though it must handle write-write conflicts, e.g., "first committer wins")
C) Cannot perform any writes
D) Always sees uncommitted data from other transactions
**Answer: B**

**Q283.** Validation-based (optimistic) protocols determine serializability order typically based on:
A) Lock acquisition order
B) The order in which transactions enter their VALIDATION phase (their "validation timestamp"), checking that a transaction's read set doesn't overlap with the write sets of transactions validated after it started, etc.
C) Random ordering
D) Physical storage order
**Answer: B**

## Section 12: Recovery, Logging & Buffer Management

**Q284.** The Write-Ahead Logging (WAL) protocol requires that:
A) Data pages are written to disk before the corresponding log records
B) The log record for an update (including undo/redo information) MUST be written to stable storage BEFORE the corresponding data modification is written to the database on disk
C) Logging is optional
D) Only commit records need to be logged
**Answer: B**

**Q285.** In WAL-based recovery, an UNDO log record for a transaction contains information needed to:
A) Redo the operation on a fresh database
B) Reverse (roll back) the effect of an operation if the transaction does not commit — typically the "before image" (old value) of the data item
C) Compute a checksum
D) Nothing useful
**Answer: B**

**Q286.** A REDO log record contains information needed to:
A) Undo a committed transaction
B) Reapply (redo) an operation's effect if the change was not yet reflected on disk at the time of a crash but the transaction had committed — typically the "after image" (new value)
C) Delete the transaction record
D) Nothing useful
**Answer: B**

**Q287.** During recovery after a crash, the general strategy using a log is to:
A) UNDO all transactions regardless of commit status
B) REDO all transactions that committed (or reached commit in the log) to ensure their effects are reflected, and UNDO all transactions that did NOT commit (were active/incomplete at crash time) to remove any partial effects
C) Ignore the log entirely
D) Only redo, never undo
**Answer: B**

**Q288.** A "checkpoint" in the recovery/logging process is used to:
A) Slow down the system unnecessarily
B) Periodically record a consistent state marker in the log (and flush relevant buffers to disk), so that during recovery, the system need only consider transactions active since the last checkpoint, reducing the amount of log that must be processed
C) Delete all previous log records
D) Replace the need for logging entirely
**Answer: B**

**Q289.** The "Force" buffer management policy requires that:
A) Modified (dirty) pages can remain in the buffer indefinitely after commit
B) All pages modified by a transaction must be written (forced) to disk BEFORE the transaction commits, simplifying REDO (none needed) but adding commit-time I/O overhead
C) Pages are never written to disk
D) Only uncommitted transactions' pages are forced
**Answer: B**

**Q290.** The "No-Force" buffer management policy allows:
A) Nothing to ever be written to disk
B) A transaction to commit even if its modified pages have not yet been written to disk (they are written later, e.g., during normal buffer replacement), which REQUIRES a REDO capability during recovery in case of a crash before the pages are flushed
C) Only committed data on disk
D) Immediate undo of every transaction
**Answer: B**

**Q291.** The "Steal" buffer management policy allows:
A) A page modified by an uncommitted transaction to never be written to disk before commit
B) The buffer manager to write ("steal") a page modified by an uncommitted transaction to disk before that transaction commits (e.g., to free buffer space), which REQUIRES UNDO capability during recovery since the transaction might later abort
C) Only committed pages to be evicted
D) Deletion of the transaction log
**Answer: B**

**Q292.** The "No-Steal" policy means:
A) Pages modified by an active (uncommitted) transaction may be written to disk at any time
B) A page modified by an uncommitted transaction is never written to disk until the transaction commits, simplifying UNDO (none needed for that transaction if crash occurs) but potentially requiring large buffer space to hold all dirty pages of long transactions
C) Every page must be forced immediately
D) It's identical to the Steal policy
**Answer: B**

**Q293.** Which combination of buffer policies (Steal/No-Steal, Force/No-Force) is most commonly used in practice (e.g., ARIES-style recovery), because it offers the best runtime performance despite requiring both UNDO and REDO logic during recovery?
A) No-Steal, Force (requires neither undo nor redo, but poor performance)
B) Steal, No-Force (requires both undo and redo capability, but gives best runtime performance by allowing flexible buffer management)
C) No-Steal, No-Force
D) Steal, Force
**Answer: B**

**Q294.** The ARIES recovery algorithm performs recovery in which three phases (in order)?
A) Undo, Redo, Analysis
B) Analysis (determine dirty pages and active transactions at crash time), Redo (repeat history — reapply all logged updates to restore the state at crash), Undo (roll back all transactions that were not committed at crash time)
C) Redo, Analysis, Undo
D) Only a single Undo phase
**Answer: B**

**Q295.** In log-based recovery, a "commit log record" must be written and flushed to stable storage:
A) Before the transaction begins
B) After all of the transaction's other log records (and typically before/as part of confirming commit), so that after a crash the recovery process can tell whether a transaction had actually committed or not
C) It's never needed
D) Only for read-only transactions
**Answer: B**

**Q296.** Shadow paging is an alternative recovery technique that:
A) Uses a traditional log of before/after images
B) Maintains two page tables (a "current" table and a "shadow" table pointing to the pre-transaction state); updates are made to NEW copies of pages, and on commit, the current table replaces the shadow table (atomic switch) — no UNDO log is needed since old pages remain untouched until commit
C) Requires no page tables at all
D) Is identical to write-ahead logging
**Answer: B**

**Q297.** A "fuzzy checkpoint" differs from a simple checkpoint in that:
A) It requires stopping all transaction activity while the checkpoint is taken
B) It allows transactions to continue executing (and buffer flushing to proceed asynchronously/gradually) while the checkpoint is being recorded, improving system availability, at the cost of slightly more complex recovery logic
C) It never writes anything to the log
D) It is identical to a normal checkpoint
**Answer: B**

**Q298.** Idempotency of REDO operations during recovery (i.e., applying the same REDO log record multiple times has the same effect as applying it once) is important because:
A) It slows down recovery unnecessarily
B) It allows the recovery process to safely reapply REDO operations even if a crash occurs during recovery itself, since redoing an already-applied update again causes no incorrect additional effect
C) It's not actually a useful property
D) It only applies to UNDO
**Answer: B**

**Q299.** Which of the following best summarizes why concurrency control AND recovery mechanisms are both essential and closely related in a DBMS?
A) They address entirely unrelated problems
B) Concurrency control ensures correctness (isolation/serializability) among simultaneously executing transactions, while recovery ensures atomicity and durability in the presence of failures — together they uphold the full set of ACID properties, and design choices (e.g., strict schedules, steal/no-steal policies) directly interact between the two
C) Only one of them is needed in practice
D) Recovery handles concurrency and locking handles failures
**Answer: B**

**Q300.** In a distributed database, the Two-Phase Commit (2PC) protocol ensures atomicity of a transaction across multiple sites by having a coordinator:
A) Immediately commit at all sites with no confirmation
B) First send a "prepare" (voting) request to all participating sites and wait for all to vote YES (ready to commit) or any NO; only if ALL vote YES does the coordinator send a global "commit" message (otherwise it sends "abort" to all) — ensuring all sites either commit or abort together
C) Allow each site to decide independently with no coordination
D) Skip voting entirely
**Answer: B**

**Q301.** A key drawback of Two-Phase Commit (2PC) is that:
A) It has no drawbacks
B) It is a BLOCKING protocol — if the coordinator fails after participants have voted YES (entered the "prepared" state) but before sending the final decision, participants may be forced to wait (blocked), potentially holding locks, until the coordinator recovers
C) It never uses locks
D) It cannot guarantee atomicity across sites
**Answer: B**


---

## Summary of Coverage

| Section | Topic | Question Range |
|---|---|---|
| 1 | ER Model (entities, attributes, weak entities, cardinality, EER/specialization, aggregation) | Q1–Q40 |
| 2 | Relational Model & Relational Algebra (algebra ops, joins, division) | Q41–Q85 |
| 3 | Tuple & Domain Relational Calculus | Q86–Q100 |
| 4 | Integrity Constraints (keys, referential actions, NULLs) | Q101–Q120 |
| 5 | Functional Dependencies & Normal Forms (1NF–5NF, BCNF, anomalies, decomposition) | Q121–Q165 |
| 6 | File Organization (heap, sequential, hashing, clustering) | Q166–Q190 |
| 7 | Indexing (dense/sparse, multilevel, B-tree, B+ tree) | Q191–Q224 |
| 8 | Transactions — ACID, States, Schedules | Q225–Q240 |
| 9 | Serializability, Precedence Graph, Recoverability | Q241–Q250 |
| 10 | Locking Protocols (2PL variants, deadlock) | Q251–Q270 |
| 11 | Timestamp Ordering, OCC, MVCC | Q271–Q283 |
| 12 | Recovery, Logging, Buffer Management, 2PC | Q284–Q301 |

**Total: 300+ MCQs**

### Study tip
For a competitive exam, revisit these high-yield trap areas repeatedly:
- 3NF vs BCNF exception (prime attribute), and why BCNF may not preserve dependencies.
- Difference between conflict-serializable and view-serializable (blind writes).
- Strict vs Rigorous vs Cascadeless vs plain Recoverable schedules (the hierarchy).
- B-tree vs B+ tree node contents and why B+ trees dominate real DBMS implementations.
- Wait-Die vs Wound-Wait rules (who waits vs who dies/is wounded).
- Steal/No-Steal, Force/No-Force combinations and which require UNDO/REDO.
