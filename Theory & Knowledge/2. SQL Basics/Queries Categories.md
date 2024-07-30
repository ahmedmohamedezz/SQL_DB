- read about ANSI Sql (american national standard institute)

- SQL queries categories:

  - DDL (data definition language => metadata & structure)

    - create

    ```SQL
    create table Students (
      ID number primary key,
      First_Name char(50) not null,
      Last_Name char(50) not null,
      Address char(50),
      City char(50),
      Country char(25),
      Birth_Date date
    )
    ```

    - alter (add & drop columns)

    ```SQL
    -- add col
    alter table Students add Postal_Code number

    -- drop col
    alter table Students drop column Country
    ```

    - drop

    ```SQL
    drop table Students
    ```

    - truncate: clear table data
      - trunctate **_can't_** be rolled back (so it's, considered ddl)

    ```SQL
    -- equivalent to delete statment without where clause
    truncate table customer
    ```

---

- DCL (data control language => security & permission)

  - grant (add privilage to user)

  ```SQL
  -- ahmed can only view the employees data
  grant select on table Employees to ahmed_ezz;

  -- khaled & mary can do anything with the data (DML operations)
  grant all on table Department to khaled, mary;

  -- ahmed can share this privilage with other users
  -- ahmed can give this privilage to other users
  grant select on table T to ahmed with grant option
  ```

  - revoke (remove privilage from user)

  ```SQL
  -- can't update the table anymore
  revoke update on table Department from mary;

  -- no privilages
  revoke all on table X from mary, ahmed;
  ```

---

- DML (data manipulation language => data)

  - insert

  ```SQL
  -- insert with columns
  -- order maintaines, first param -> first col, 2nd -> 2nd, and so on
  insert into employees (Fname, Lname, ssn) values ('ahmed', 'ezz', 123)

  -- insert values (order maintaines)
  insert into employees values ('ahmed', 'hassan', 234)

  -- insert specific cols
  insert into employees (Fname, ssn) values ('alaa', 'wagdy', 456)

  -- insert multiple rows
  insert into employees values ('aaa', 'bbb', 111), ('ccc', 'ddd', 222)
  ```

  - update

  ```SQL
  -- without where clause => update all employees salaries
  update employees
  set salary = 1200, dno = 10
  where ssn = 112
  ```

  - delete

    - delete **_can_** be rolled back

  - truncate command commits changes automatically, so they can't be rolled back

  - delete command doesn't apply to physical schema directly, you must commit first, so they can be rolled back

  ```SQL
  -- without where clause => delete all employees data
  delete from employees
  where ssn = 112
  ```

  - select
  - for practice over more SQL query topics, refer for queries file

  ```SQL
  -- if column name contains spaces => put it between []
  select name, salary, [birth date]
  from employee
  where salary >= 1500 and salary <= 2500  -- between 1500 and 2500

  select *
  from employee
  where superssn = 123 or superssn = 456  -- in (123, 456)

  -- select without repition
  select distinct dno, superssn
  from employee
  ```

---

- DQL (data query language => display)

  - select + (agg. functions, grouping, joins, union, subqueries)

- TCL (transaction control language => execution)

  - begin transaction
  - commit
  - rollback

- database are just files on the hard disk

  - 2 files
    - .mdf => has the tables, ...
    - log file => has the transactions

- to add tables through UI
  - right click on tables in SSMS, and choose to add a new table
- to add foriegn key through diagrams

  - choose diagrams => select tables
  - connect from PK => FK

- datatypes:

  - varchar(mxlen)
  - int
  - date (mm-dd-yyyy)

- note that null value can be used in math. expression
  => where name != null
  - instead, use 'is null', 'is not null'

> Look at practice file to see commands examples
