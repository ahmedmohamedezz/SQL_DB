# This file completes the Basic_SQL file, it's like step 2 in the way

### Hints and Functions

- some times date objects are not stored in date datatypes, instead `varchar` or `text` could be used

  - you can use `cast(col AS date)`, or `convert(date, col)` functions to extract their date value
  - then, if you need you can format date using `date_format(date, "%Y-%m")`, `year(date)`, or `to_char(date, "YYYY-MM")` functions

- note, that depending on the engine, and SQL type some functions may not work (SQL Server, MS_SQL, PostgreSQL)

- `date_format()` can work with strings directly, no need to convert to date data type

---

### Joins and Subqueries

- the following query joins 3 tables to get the count of paying & non-paying customers for each date, then we filter the result by enclosing it within a sub-query

```sql
select * from (
    select dnld.date,
        sum(if(acc.paying_customer='no', downloads, 0)) as non_paying,
        sum(if(acc.paying_customer='yes', downloads, 0)) as paying
    from
        ms_user_dimension
    join
        ms_acc_dimension acc using(acc_id)
    join
        ms_download_facts dnld using(user_id)
    group by dnld.date
    order by dnld.date
) as sub
where non_paying > paying;
```

---

### CTEs and Window Functions

- group by function makes an aggregation on the data, but it group rows into 1 result

- with cte, we can make an aggregation without affecting the number of rows (aggregation with respect to the table)

- refer to [window functions](https://www.sqlshack.com/use-window-functions-sql-server/) to see different aggregations that can be applied

- you will notice that after the aggregation we use special key words like

  - over: to determine partitions of table, and the ordering of the aggregation
  - partition by: used to determine partitions (each is treated as sub-table)

- look at the following example

```sql
-- this query return the same no. of rows as the table
-- it shows the sum of orders per city (partition by city)
-- all rows of the same city will have the same value in grand_total column

SELECT order_id, order_date, customer_name, city, order_amount
 ,SUM(order_amount) OVER(PARTITION BY city) as grand_total
FROM Orders
```

- here is an example of `cte` in `from` clause

```sql
select person_name
from (
    select *, sum(weight) over (order by turn) as total_weight from Queue
) as sub
where total_weight <= 1000
order by total_weight desc
limit 1
```

- the following query uses cte to rank salaries in each department
  - read about [dense_rank vs. rank](https://www.naukri.com/code360/library/difference-between-rank-and-denserank)

```sql
with CTE as (
    select *, dense_rank() over (partition by departmentId order by salary desc) as rnk
    from Employee
)
-- select * from cte
select d.name as Department, c.name as Employee, c.salary as Salary
from CTE c, Department d
where c.rnk <= 3 and c.departmentId = d.id
```

- suppose we want to get the ratio of (current month revenue / last month revenue)
  - using window function LAG(), we can get the value of the prev row in the result

```sql
WITH CTE AS (
    SELECT created_at, sum(value) / lag(sum(value)) over () AS ratio
    FROM SomeTable
    GROUP BY date_format(created_at, "%Y-%m")
    ORDER BY date_format(created_at, "%Y-%m")
)
SELECT * FROM CTE;

-- another similar idea
WITH T1 AS (
    SELECT date_format(created_at, "%Y-%m") AS ym, sum(value) AS cur
    FROM sf_transactions
    GROUP BY date_format(created_at, "%Y-%m")
    ORDER BY date_format(created_at, "%Y-%m")
), T2 AS (
    SELECT *, lag(cur) over () AS prev
    FROM T1
)
SELECT * FROM T2;
```

- see the next example, we can use multiple window functions like

```sql
(sum(value) - lag(sum(value)) over ()) / lag(sum(value)) over () as ratio
```

- suppose we want to calculate the month-over-month percentage ((cur_revenue - last_revenue) / last_revenue \* 100)

> approach: if we can get the revenue of each month then, using the LAG() window function & any sorting mechanism, we can solve the problem easily

- [window alias](https://mode.com/sql-tutorial/sql-window-functions) is like a variable that stores an expression, or some code
  - window alias is put after `where` & `group by` clauses, and before `order by` clause

```SQL
-- Sol. 1
SELECT date_format(created_at, "%Y-%m") AS ym,
    round((sum(value) - lag(sum(value), 1) over (order by date_format(created_at, "%Y-%m")))
        / lag(sum(value), 1) over (order by date_format(created_at, "%Y-%m"))
        * 100, 2) AS revenue_diff_pct
FROM sf_transactions
GROUP BY ym


-- The code maybe not readable enough, so we can use DRY (don't repeat yourself) responsible, and use a window alias
-- Sol. 2
SELECT date_format(created_at, "%Y-%m") AS ym,
    round((sum(value) - lag(sum(value), 1) over date_ordering)
        / lag(sum(value), 1) over date_ordering
        * 100, 2) AS revenue_diff_pct
FROM sf_transactions
GROUP BY ym
WINDOW date_ordering AS (order by date_format(created_at, "%Y-%m"))
```

- if in some query, if we want to distribute the date into buckets or percentiles (almost equal-sized groups), we can use the window function [NTILE(no. of buckets)](https://www.geeksforgeeks.org/ntile-function-in-sql-server/)

- the following query gets the top 5 percentile per state

```SQL
select state,fraud_score
from (
    select *,
        ntile(100) over (partition by state order by fraud_score desc) as rnk
    from fraud_score
) as sub
where rnk <= 5;
```
