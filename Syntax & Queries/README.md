# DB Notes

> to review SQL syntax, refer to [W3S](https://www.w3schools.com/sql/default.asp)


> refer for [DateFunctions](https://www.w3schools.com/sql/func_sqlserver_current_timestamp.asp), for details about date manipulations in sql

---

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

- you can use joins with more than 2 tables

```sql
select Orders.OrderID, Customers.CustomerName, Shippers.ShipperName
from ((Ordersinner join Customers on Orders.CustomerID = Customers.CustomerID)
inner join Shippers on Orders.ShipperID = Shippers.ShipperID);
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

- you can use sub queries in select statment

```sql
select round(count(user_id) * 100 / (select count(user_id) from Users) ,2) as percentage
```

- see the next functions for date manipulation
  - datediff(date1, date2)
  - date_sub(date, interval 1 day)
  - date_add(date, interval 1 day)
  - datepart(year, date) 
  - day(date), month(date), year(date)

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

- you can merge multiple rows from different query like

```sql
select product_id, year as first_year, quantity, price
from Sales
where 
    (product_id, year)  -- matched pair
  in 
    (select product_id, min(year) as year from Sales group by product_id);
```