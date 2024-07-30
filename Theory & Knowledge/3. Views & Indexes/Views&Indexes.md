# Views and Indexes

### Views

- a view is a logical table based on other table(s) and/or view(s)

- views contain no data of it's own, it's just a window through which you can see tables data

- tables used to build the view are called **_base tables_**

- the view is stored as a select statement

```SQL
-- syntax to create views
-- create view vName [columns]
-- as (query)

create view vw
as
select fName, pName
from T1 join T2 on PNum = num
```

- you can use _check option_ to make sure any data inserted into view is valid

```SQL
create view suppliers
as
select * from Suppliers
where status > 15
-- with any DML operation
-- make sure (status > 15)
with check option;
```

- to modify view

```SQL
create or replace view suppliers
as
select * from Suppliers
where status > 15 and name like 'S%'
```

- to drop view

```SQL
drop view suppliers;
```

#### Why to use views?

- restrict data access

  - by providing specific views to specific users

- make complex queries easier

- provide data independence

  - we can take parts of tables independently without exposing all table data

#### Views types

- **_simple view_**
  - has 1 table
  - has no functions or groups of data
  - DML operations allowed
- **_complex view_**
  - has 1/more table
  - has functions and/or groups of data
  - DML operations not always allowed


---

### Indexes

- used to speed up records retrieval

- maybe defined on multiple columns

- same table can have more than 1 index

- created by user or DBMS
  - DBMS creates index for PK

- usually the index will contain values of some col(s) sorted with points to it's locations in memory

#### Disadvantages

- indices cause overhead on DML (insert, update, delete) operations since we need to modify 2 places

#### When to create an index

- when you use column, or table data heavily based on your search conditions

- if a column has a large number of nulls, index may save the time for looking at specific value

#### When not to create an index

- if the table is updated frequently

```SQL
-- create index
create index emp_idx on employee (salary);

-- drop
drop index emp_idx;
```