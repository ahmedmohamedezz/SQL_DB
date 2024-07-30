# Normalization

- normalization is a process that takes a table through a series of tests (normal forms)

  - to certify the goodness of the design & minimize redundancy
  - minimize anomalies (insert, update, delete)
    - insert anomaly: every record of employee has equivalent cols of mngrSsn & dept => redundancy
    - update anomaly: if we update the mngeSsn of 1 dept => update many records
    - delete anomaly: if the only employee in dept 'A' was fired => does this mean departemnt has closed ?
  - help avoid frequent null values

- we may also apply normalization rules to an old DB, of a file system to maintain a good design

### functional dependency:

- constraint between 2 or more attrs (columns)
- functional dependency between cols 'A' and 'B' means that `every value of A uniquely determines the value of B`
  - SSN -> name (name dependent on SSN)
  - pNumver -> {pName, pLocation}
  - {SSN, pNumber} -> hours

---

#### functional dependency types

- full: non-key attr depends on the value of the key

  - ename, address, dNumber **dependent on** SSN
  - hours -> {SSN, pNumber}

- partial: non-key attr depends on **part** of the value of the key

  - if PK = {SSN, pNumber}, so eName depends only on SSN

- transitive: non-key attr depends on non-key attr that depends on the value of the key

  - to know the name of dept, we must know dept number first
  - to know the dept number of employee, we must know the employee SSN first

---

#### First Normal Form

- if the relation must NOT have:

  - multi-valued attrs
  - repeating groups
  - composite attrs

- how to solve, break the table (relation), separate the cols with issues with PK in smaller table

#### Second Normal Form

- to say that a relation is in the 2nd NF:

  - it must satisfy the 1st NF
  - it should NOT contain any partial dependency

- to solve: take the non-key attr with the part of key it depends on into a new table

#### Third Normal Form

- to say that a relation is in the 2nd NF:

  - it must satisfy the 2nd NF
  - it should NOT contain transitive dependency

- to solve: separate the non-key columns that depend on each other in a separate table