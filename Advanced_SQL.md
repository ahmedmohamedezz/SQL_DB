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