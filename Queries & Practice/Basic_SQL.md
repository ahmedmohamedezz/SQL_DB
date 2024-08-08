# DB Notes

### Sections

<!-- Comments -->

- [Select & Sort](#select-and-sort)
- [Joins & Aggregate Functions](#joins-and-aggregate-functions)
- [Subqueries](#subqueries)
- [Date Functions](#date-functions)
- [String Functions](#string-functions)
- [Readings](#readings)

> to review SQL syntax, refer to [W3S](https://www.w3schools.com/sql/default.asp)

---

### Select and Sort

- when you're dealing with numbers, nulls are not handled
- comparison operators deals only with numbers (skip NULLs)

```sql
# the following query will not return (referee_id = null)
select name from Customer where referee_id != 2;

# the following works as expected
select name from Customer where referee_id != 2 or referee_id is null;
```

- to group multiple operator in a sql statemnt, they maybe arranged as

```sql
select distinct(id) as Emp_id
from Employees
where age > 30 and city in ('florida', 'dubai')
order by age desc;
```

- to get the length of a varchar column value, use `length()` operator

```sql
select tweet_id from Tweets where length(content) > 15;
```

---

### Joins and Aggregate Functions

- you can use joins with more than 2 tables

```sql
-- Ex. 1
select Orders.OrderID, Customers.CustomerName, Shippers.ShipperName
from (
  (Ordersinner join Customers on Orders.CustomerID = Customers.CustomerID)
  inner join Shippers on Orders.ShipperID = Shippers.ShipperID);

-- Ex. 2
select L1.num as ConsecutiveNums
from (
    Logs L1 join Logs L2 on L1.num = L2.num and L1.id + 1 = L2.id
    ) join Logs L3 on L2.num = L3.num and L2.id + 1 = L3.id
group by L1.num;

-- Ex. 2 (reshape)
-- we can remove 'distinct', and add 'group by L1.num'
select distinct L1.num as ConsecutiveNums
from Logs L1, Logs L2, Logs L3
where L1.id + 1 = L2.id and L2.id + 1 = L3.id and L1.num = L2.num and L2.num = L3.num
```

- result of `full join` is huge & maybe filled with alot of `NULL's`

- you can join a table with itself

```sql
select A.CustomerName as CustomerName1, B.CustomerName as CustomerName2, A.City
from Customers A, Customers B
where A.CustomerID <> B.CustomerID
and A.City = B.City
order by A.City;
```

- the following snippets are exactly the same
- notice how can we remove `as` keyword

```sql
select EmployeeUNI.unique_id, Employees.name
from EmployeeUNI right join Employees
on EmployeeUNI.id = Employees.id;

select uni.unique_id, em.name
from EmployeeUNI as uni right join Employees as em
on uni.id = em.id;

select uni.unique_id, em.name
from EmployeeUNI uni right join Employees em
on uni.id = em.id;
```

- use `group by` to group results in categories

- the following query selects the customer visits with no transactions, and count how many times they did this

```sql
select customer_id, count(customer_id) as count_no_trans
from Visits v left join Transactions t -- get all visits & (transactions or null)
on v.visit_id = t.visit_id
where transaction_id is null -- filter all transactions to get nulls only
group by customer_id
```

- aggregate functions are used with `group by` clause to perform calculations to represent each group as a single value

- aggregate functions ignore null values 'except for COUNT()'

- aggregate functions can accept a boolean expression instead of column name

```sql
-- the following query counts all rows (count nulls also)
select count(*) from Products;

-- counts the number of products (DON'T count nulls)
-- count + col name => no nulls are considered
select count(ProductName) from Products;
```

- the sum() function can accept an expression

```sql
select sum(Price * Quantity)
from OrderDetails
```

- the following example demonstrates usin aggregate functions on portion of the data (get 2nd highest salary)

```sql
-- Sol. 1
select max(salary) as SecondHighestSalary
from Employee
where salary not in (
    -- top 1
    select max(salary) from Employee
)

-- Sol. 2
select max(salary) as SecondHighestSalary
from Employee
where salary < (select max(salary) from Employee)
```

- the following query calculates the avg selling price of items

```sql
-- note that we won't use predefined AVG() because we have a formula
-- round(val, digit) => round 'val' to 'digits' decimal
-- ifnull(val, res) => if val is null, return 'res' otherwise return 'val'
select p.product_id, ifnull(round(sum(price * units) / sum(units), 2), 0) as average_price
from Prices p left join UnitsSold u
on p.product_id = u.product_id and u.purchase_date between p.start_date and p.end_date
group by p.product_id
```

- the following code counts the number of active users in each day in the last month (if today is '2019-07-27')

```sql
select activity_date as day, count(distinct user_id) as active_users
from Activity
where activity_date between '2019-06-28' AND '2019-07-27'
group by activity_date
```

- consider the following order of clauses

```sql
select column_name(s)    -- 1
from table_name          -- 2
where condition          -- 3
group by column_name(s)  -- 4 before having
having condition         -- 5
order by column_name(s); -- 6
```

- the following 2 queries are not the same (trying to get class with >= 5 students)

- **USE** `group by` with `having`, or the query answer will not always be correct

- make sure that the field you use with `group by` is **unique** (key field)

```sql
-- correct
select class
from Courses
group by class
having count(student) >= 5

-- wrong (aggregate function are meant to be used with groups)
select class
from Courses
having count(student) >= 5
```

```sql
-- query: select days having temperature higher that the prev day (yesterday)

-- Sol. 1
select w1.id
from Weather w1 inner join Weather w2
on datediff(w1.recordDate, w2.recordDate) = 1   -- date1 - date2 = 1
where w1.temperature > w2.temperature


-- Sol. 2
select w1.id
from weather as w1
join weather as w2
on w1.recordDate = w2.recordDate + INTERVAL 1 DAY
where w2.temperature < w1.temperature

-- Sol. 3
select w1.id
from Weather w1 join Weather w2
on DATE_SUB(w1.recordDate, INTERVAL 1 DAY) = w2.recordDate
-- OR w1.recordDate = DATE_ADD(w2.recordDate, INTERVAL 1 DAY)
where w1.temperature > w2.temperature;

-- Sol. 4
select w1.id
from Weather w1, Weather w2
where
    (w1.recordDate = w2.recordDate + interval 1 day)
  and
    (w1.temperature > w2.temperature);
```

- suppose we want to get name of user who rated greatest number of movies
- and also the movie that has the most avg rating
- finally union the 2 results

- look at the following query and notice:
  - use of [using](https://stackoverflow.com/questions/11366006/mysql-join-on-vs-using) clause
  - [union all](https://www.w3schools.com/sql/sql_ref_union_all.asp) operator
  - unioned queries must be surrounded with ()
  - union => join unique values, union all => join + allow duplicates

```sql
-- part. 1
(select name as result
from Users join MovieRating using(user_id)
group by name -- if removed => error 'order by contains aggregate function'
order by count(*) -- no. of ratings
limit 1)

union all

-- part. 2
(select name as result
from Movies join MovieRating using(movie_id)
group by name
order by avg(rating)
limit 1)
```

- the following query counts the no. of categories (0, if not found)

```sql
-- Sol. 2
select "Low Salary" as category, count(income) as accounts_count
from Accounts
where income < 20000

union

select "Average Salary" as category, count(income) as accounts_count
from Accounts
where income between 20000 and 50000

union

select "High Salary" as category, count(income) as accounts_count
from Accounts
where income > 50000

-- IMPORTANT NOTE
-- the following query, won't work. because group by are meant to be used
-- with column values, not result of aggregate funcions
select "High Salary" as category, count(income) as accounts_count
from Accounts
where income > 50000
group by count(income)
```

- to calculate a ratio depending on values of field, you can use `avg()` + `if()`
  - each if() will return a result
  - avg() will sum results, then divide by thier number

```sql
round(avg(if(rating < 3, 1, 0)) * 100, 2) as query_percentage
```

- suppose we have a date in a certain format, and we need to select it with other format. we can do this in several ways

  - refer to [date_format](https://www.w3schools.com/sql/func_mysql_date_format.asp)

- our query wants to find some info related to each month
- the table stores the whole date 'yyyy-mm-dd' ,but we don't care about 'd'

- notice, how we `group by` month

```sql
-- Sol. 1
select date_format(trans_date, "%Y-%m") as month, sum(amount) as trans_total_amount
from Transactions
group by date_format(trans_date, "%Y-%m")


-- Sol. 2, since the new format 'yyyy-mm' is part of the table stored format 'yyyy-mm-dd', we can take part of it (sut string)
select left(trans_date, 7) as month, sum(amount) as trans_total_amount
from Transactions
group by left(trans_date, 7)

-- Sol. 3, using substring(string, startPos, len) method
select substring(trans_date, 1, 7) as month, sum(amount) as trans_total_amount
from Transactions
group by substring(trans_date, 1, 7)  -- consider the string 1-based (1st char index is 1)
```

- suppose, we want to make a query to get employee working in only 1 department, or the main department if many exists

- note that we need to filter based on 2 conditions

  - one using aggregate function `count()`
  - one to check a column value

- if we used `where` clause, aggegate function won't work, and if we used `having` the second check won't work

- so, we have 2 solutions
  1. treat each as a separate query and [union](https://www.w3schools.com/sql/sql_union.asp) the result
  2. use subquery in `where` clause

```sql
-- Sol. 1
select employee_id, department_id
from Employee
where primary_flag = 'Y'
union
select employee_id, department_id
from Employee
group by employee_id
having count(employee_id) = 1

-- Sol. 2
select employee_id, department_id
from Employee
where primary_flag = 'Y' or employee_id in (
    select employee_id from Employee group by employee_id having count(employee_id) = 1
    )
order by employee_id
```

- as we can union the results or queries, we can also get the intersection between result using `and` operator

```sql
-- this query gets sum based on 2 conditions
  -- 1. count(tiv) > 1
  -- 2. UNIQUE combinations of (lat & lon)
select round(sum(tiv_2016), 2) as tiv_2016
from Insurance
where tiv_2015 in (
    select tiv_2015
    from Insurance
    group by tiv_2015
    having count(tiv_2015) > 1
) and (lat, lon) in (
    select lat, lon
    from Insurance
    group by lat, lon
    having count(*) = 1
)
```

- in some queries, the [case](https://www.w3schools.com/sql/sql_case.asp) expression in sql may be useful
- suppose we want to manipulate the id value while fetching the data

```sql
select
    case
        when id = (select max(id) from Seat) and id % 2 = 1
            then id

        when id % 2 = 1
            then id + 1

        else
            id - 1
    end as id,
    student
from Seat
order by id;
```

> we can replace `else` part with `when id % 2 = 0 then id - 1`

- suppose, we want to get the min, or max value between several values
  - we can use functions greatest(), least()

```sql
-- check if valid traingle using sql
if(x+y+z > greatest(x, y, z) * 2, "Yes", "No")
```

---

# Subqueries

- to count occurences of some values in multiple columns, you can union the columns

```sql
-- count id value either as a requester, or accepter
with result as (
    (select requester_id as id from RequestAccepted)
    union all
    (select accepter_id as id from RequestAccepted)
)
select id, count(id) as num
from result
group by id
order by count(id) desc
limit 1
```

- you can use sub queries in select statment

```sql
select round(count(user_id) * 100 / (select count(user_id) from Users) ,2) as percentage
```

- to delete all duplicate emails leaving the one with min(id), we can do the following

```sql
-- Sol. 1
delete from Person
where id not in (
  select id from (
    select min(id) as id
    from Person
    group by email
  ) sub
)

-- Sol. 2
delete P1 -- specify which table to delete (P1, or P2 ?)
from Person P1, Person P2
where P1.email = P2.email and P1.id > P2.id
```

```sql
-- note that the following query will make an ERROR
-- you can't modify the same table which you use in the SELECT part

delete from Person
where id not in (
  -- missing select statement (select id from sub-query)
  select min(id)
  from Person
  group by email
)
```

- refer to [link](https://stackoverflow.com/questions/45494/mysql-error-1093-cant-specify-target-table-for-update-in-from-clause) for explanation

- you can merge multiple rows from different query like

```sql
select product_id, year as first_year, quantity, price
from Sales
where
    (product_id, year)  -- matched pair
  in
    (select product_id, min(year) as year from Sales group by product_id);
```

- we can use subqueries to calculate rolling window in some interval
- the following query get the avg, total in every 7-days window
- there is also solution using joins + subqueries

```sql
-- Sol. 1 (subqueries)
select visited_on,
    (
        select sum(amount)
        from Customer
        where visited_on between date_sub(C.visited_on, interval 6 day) and C.visited_on
    ) as amount,
    (
        select round(sum(amount) / 7, 2)
        from Customer
        where visited_on between date_sub(C.visited_on, interval 6 day) and C.visited_on
    ) as average_amount
from Customer C
-- consider only days with valid window size
where visited_on >= (select date_add(min(visited_on), interval 6 day) from Customer)
group by visited_on


-- Sol. 2 (joins + subqueris)
select C1.visited_on, sum(C2.amount) as amount, round(sum(C2.amount) / 7, 2) as average_amount
from (
    -- get all days with valid window size
    select visited_on
    from Customer
    where datediff(visited_on, (select min(visited_on) from Customer)) >= 6
    group by visited_on
) C1 left join Customer C2
-- match each row from C1 with 7 rows from C2
on datediff(C1.visited_on, C2.visited_on) between 0 and 6
group by C1.visited_on
order by C1.visited_on
```

- the following query calculate avg of confirmed mails (confimed / total)

```sql
select S.user_id,
  round(avg(if(C.action="confirmed", 1, 0)), 2) as confirmation_rate
from Signups S left join Confirmations C
on S.user_id = C.user_id
group by S.user_id
```

- the following queries return the same result, but the first is more efficient
  - the first solution will run the sub query only once, and then filter result in outer query
  - the second solution will run the sub query once for each row in the outer query which is less efficient

```sql
-- Sol. 1
SELECT ROUND(AVG(order_date = customer_pref_delivery_date) * 100, 2) AS immediate_percentage
FROM Delivery
WHERE (customer_id, order_date) IN (
    SELECT customer_id, MIN(order_date)
    FROM Delivery
    GROUP BY customer_id
);


-- Sol. 2
SELECT ROUND(AVG(order_date = customer_pref_delivery_date) * 100, 2) AS immediate_percentage
FROM Delivery
WHERE (customer_id, order_date) IN (
    SELECT customer_id, MIN(order_date)
    FROM Delivery
    GROUP BY customer_id
);
```

- sql **requires** a name for the sub-query of the from clause

```sql
select max(num) as num
from (    -- from + sub-query => must use alias for sub-query
    select num from MyNumbers group by num having count(num) = 1
) as singleNums
```

---

# Date Functions

> refer for [DateFunctions](https://www.w3schools.com/sql/func_sqlserver_current_timestamp.asp), for details about date manipulations in sql

- [date_format](https://www.w3schools.com/sql/func_mysql_date_format.asp)

- datediff(date1, date2)

  - [MySQL](https://www.w3schools.com/sql/func_mysql_datediff.asp) version

  - [SQL_Server](https://www.w3schools.com/sql/func_sqlserver_datediff.asp) version

- [date_sub(date, interval 1 day)](https://www.w3schools.com/sql/func_mysql_date_sub.asp)

- [date_add(date, interval 1 day)](https://www.w3schools.com/sql/func_mysql_date_add.asp)

- [datepart(part, date)](https://www.w3schools.com/sql/func_sqlserver_datepart.asp)

- day(date), month, [year](https://www.w3schools.com/sql/func_mysql_year.asp)

- [extract(part from date)](https://www.w3schools.com/sql/func_mysql_extract.asp)
  - Feb, 2020 `where extract(YEAR_MONTH from created_at) = 202002`

> notice that string functions can be used with date treating it as a string

- the following query selects the last price of the product before a certain date, along with 10 as the default price for the remaining product that it's price weren't changed

```sql
(
    select product_id, new_price as price
    from Products
    where (product_id, change_date) in (
        select product_id, max(change_date) as change_date
        from Products
        where change_date <= '2019-08-16'
        group by product_id
    )
)
union
(
    select product_id, 10 as price
    from Products
    group by product_id
    having min(change_date) > "2019-08-16"
)
```

---

# String Functions

- concat(str1, str2): concatenate strings

- group_concat(): group values of column in string

  - refer to [GFG](https://www.geeksforgeeks.org/mysql-group_concat-function/)

- left(str, n): return the left 'n' chars from 'str'

  - right(str, n): same but from the end

- length(str): string length

- upper(str): upperacse string

  - lower(str): same but lowercase

- substring(str, startInd, [no. of chars]): get a substring

  - 3rd parameter is optional => default [startInd, end of string]

- suppose, we want to know patients with certain type of diabetes

  - where col conditions contains 'DIAB1\*\*' as one of the patient conditions
  - ex. 'DIAB101 XX YY' or 'XX DIAB103 ZZ'

- we can use the [like](https://www.w3schools.com/sql/sql_like.asp) operator

  - either start with `DIAB1`, or has it (not included in other condition `XDIAB1` is not considered)

- we can also use sql [regexp](https://www.scaler.com/topics/regex-in-sql/)

```sql
-- Sol. 1
select *
from Patients
where conditions like '% DIAB1%' or conditions like 'DIAB1%'

-- Sol. 2
select *
from Patients
-- where DIAB1 is a boundary (start of word)
where conditions regexp '\\bDIAB1'; -- contains 'DIAB1'
```

- find users with valid mails
  - start with a letter
  - then chars, digits, ., \_, - are allowed
  - end with 'leetcode.com'

```sql
select *
from Users
WHERE mail REGEXP '^[A-Za-z][A-Za-z0-9_\\.\\-]*@leetcode\\.com$';
```

- suppose, we want to query for each day:

  - the no. of the products sold
  - the product name aggregated in string

- the `group_concat()` methods is suitable in such a situation

```sql
select sell_date, count(distinct product) as num_sold,
       group_concat(distinct product order by product separator ',') as products
from Activities
group by sell_date
order by sell_date
```

- to cast one data type into another, we can use the [cast](https://www.w3schools.com/sql/func_sqlserver_cast.asp) function

  - cast to int `cast(20.36 AS int)`
  - cast to varchar `cast(20.36 AS varchar)`
  - cast to date `cast(20.36 AS date)`

- also look at [convert(datatype, exp, [style])](https://www.w3schools.com/sql/func_sqlserver_convert.asp)

  - convert string to date `convert(datetime, '2020-02-19')`

- you can format date using [to_char(date)](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/TO_CHAR-datetime.html)function
  - to_char(date, "YYYY-MM") as year_month

---

# Common Table Expressions (CTE)

---

### Readings

- is it recommended to use select alias in group by ? [StackOverFlow](https://stackoverflow.com/questions/3841295/sql-using-alias-in-group-by)

- what is meant by `group by 1` or `order by 2` ? [StackOverFlow](https://stackoverflow.com/questions/7392730/what-does-sql-clause-group-by-1-mean)
