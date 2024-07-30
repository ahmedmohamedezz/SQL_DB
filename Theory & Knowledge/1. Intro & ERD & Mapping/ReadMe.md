# Intro To Database Fundamentals

- before having the current database systems, file systems were used to store data
- file systems had many disadvantages
  - duplication
  - inconsistency
  - data isolation & separation
  - no constraints validation
  - no backup or recovery mechanisms
- the data we insert into our db is stored on **_disk_**

- database systems overcomed the file systems drawbacks

  - now, we can query the data easily
  - add constraints on data (range of values, not null, ...)
  - backup & recovery
  - no redundancy

- **_database_**: is a collection of related data
- **_DBMS_** (database management system): software that is responsible of the creation & maintainence of the DB
- **_database system_**: data itself + DBMS

---

### Database Main Components

- the database mainly consists of 3 componenets:
  - application programs: UI users interact with
  - DBMS: 2 parts, one to process queries and the other to access the data
  - database: 2 parts, one consisting of the db metadata (transactions, tables info, authorizations) and the other has the data itself

---

### Database users

- system analyst: gather information from clients, and translate them to business
- designer: transform requirements into designs
- administrator (DBA):
  - handle db users & permissions, monitor db performance
  - create the DB & install DBMS
  - populate the initial data
- developers
- endusers

---

### DBMS Architecture

- DBMS consists of 3 schema architecture (external, conceptual, physical)

- **_external schema_**: the data each set of users can see, like interfaces to different types of users

  - finance department has a different schema than the HR department

- **_conceptual schema_**: schema that contains all the tables & relations of db

- **_physical schema_**: concerned with where the data is located, how much space allocated

- when an enduser makes a request, a the request is handleled in the 3 layers of the db schema, the process of transforming requests & results between levels is called **_mappings_**

---

### Entity Relationship Diagram (ERD)

- used to model the db objects & the relations between them

- entity types:

  - **_regular_**: doesn't depend on other entity
  - **_weak_**: depend (employee family info, if employee left => no need to store his family info)
    - represented by doulbe-lined rectangle

- each entity has its own properties => **_attributes_**

- attribute types:

  - **_key_**: has distinct values and can be used to identify instances
    - single key: id
    - composite: set of attrs
    - candidate: can be used as key
  - **_multi-valued_**: has multiple values for the same instance (phone numbers of 1 person)
    - represented by double-lined circle
  - **_composite_**: composed of set of attributes
  - **_simple_**: not divisible & have single value per instance
  - **_derived_**: calculated based on another attribute value

- relationships between entities are of 2 types: strong, identifying (weak) relationship
  - identifying is represented be double-lined diamond
- we need to define 3 properties considering any relation

  - **_degree_**: no. of enities in that relations (1 unary(self), 2 binary, ..., n-ary)
  - **_participation_**: is it mandatory for each entity to participate in relation ? (may = partial participation, must = full participation)
  - **_cardinality_**: max no. of instances per relation (1-1, 1-n, n-m)
    - in m-m n-ary (n >= 3) relations, check the cardinality between each pair participating in the relation, each side must have the same cardinality with all other tables

- relations can also have attrs (no. of hours a certain employee works on a certain project)

  - add it in employee ? can work on several projects
  - project ? has many employees
  - solution is to add it to the relation that links employees to projects

- when you read a relation between 2 entities in ERD
  - entity1 (participation) (relation name) (cardinality) entity2
  - ex. employee **must** **work** in **one** department
  - (employee)==M====[work]----1--(department)

---

### Mapping

#### Entities & Attrs Mapping

- when an entity has many candidates, choose 1 of them to be the PK
- composite attrs are divided into their components (name maybe mapped to 2 columns (firstName, lastName))
- the PK of weak entity is a combination of (PK of dependent table + attr in weak table)
  - this combination must be unique (ssn, child name) => no 2 children will have the same name and the same parent
- multi-valued attribute is moved to new table with (att + PK of original table)
  - PK of resulting table is both (att + PK of parent) => composite
- derived attributes are not put in the DB
  - it can be stored if we use it alot

#### Relations Mapping

#### 1-1 Binary & Unary Relations

- look at participations

- full participation on both sides

  - merge the 2 tables into 1 table with PK as any of the 2 tables
    - now any foreign key that should be put in either of the tables is put in that new merged table

- full & partial => put PK of partial as foreign key in full

  - if the relationship has attrs, they follow foreign keys (go to full side)

- partial on both sides (we have 2 solutions)
  - create new table with PK of both
  - take PK of one of the tables and put it as foreign key in the other table (easier to implement)

#### 1-n Binary & Unary Relations

- put PK of '1 side' in table at 'many side'

#### n-m Binary & Unary Relations

- create new table with PK of 2 entities as foreign keys + relationship attrs if any
- PK of the resulting table is the 2 PKs of the entities (composite PK)
- relationship attrs if any are put into the new table (follows the foreign keys)

#### Ternay Relationships

- don't consider cardinality, or participation

- new table with the PK of all the participating tables as foreign keys
  - PK of the new table is a combination of all FK of participating tables
