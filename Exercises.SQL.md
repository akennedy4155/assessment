
# Query Exercises
## Alex Kennedy, Data Engineer Oct. 2020
The following are some exercises in writing different types of queries against a very simple and small dataset.

If you don't have convenient access to a database engine, 
[https://rextester.com/l/oracle_online_compiler]https://rextester.com/l/oracle_online_compiler) is a good paste-and-execute 
SQL evaluator. [http://sqlfiddle.com](http://sqlfiddle.com) is also a good option, but requires building up the schema to 
query against separately.

While the basis query was written with Oracle in mind, it can be easily refactored to run on any database engine that supports 
CTEs and window/analytical functions. For example, to run on Postgres, simply remove the `FROM dual` statements. The 
statements to convert strings to dates might also need to be adjusted based on the engine of your selection.

The intent of these exercises is to demonstrate a working knowledge of SQL in general rather than familiarity with one RDBMS 
engine over another, so please feel free to use the engine of your choice.

# Dataset
This dataset represents user pedometer (steps) data. A piece of data will be recorded with the number of steps taken on a 
particular date.
```sql
WITH
  cte_users AS (
    SELECT 1 AS id, 'Terra' AS name FROM dual UNION ALL
    SELECT 2,       'Locke'         FROM dual UNION ALL
    SELECT 3,       'Edgar'         FROM dual UNION ALL
    SELECT 4,       'Sabin'         FROM dual UNION ALL
    SELECT 5,       'Celes'         FROM dual
  ),
  cte_steps AS (
    SELECT
      id,
      user_id,
      steps,
      -- Oracle-specific syntax, other engines may require another approach
      TO_DATE(activity_date, 'yyyy-mm-dd') AS activity_date
    FROM (
      SELECT 1 AS id, 1 AS user_id, '2020-01-01' AS activity_date,  7001 AS steps FROM dual UNION ALL
      SELECT 2,       1,            '2020-01-03',                   5000          FROM dual UNION ALL
      SELECT 3,       1,            '2020-01-05',                   4000          FROM dual UNION ALL
      SELECT 4,       1,            '2020-01-05',                   1000          FROM dual UNION ALL
      SELECT 5,       1,            '2020-02-01',                   6000          FROM dual UNION ALL
      SELECT 6,       1,            '2020-04-01',                   10500         FROM dual UNION ALL
      SELECT 7,       2,            '2020-02-01',                   8000          FROM dual UNION ALL
      SELECT 8,       2,            '2020-02-04',                   11000         FROM dual UNION ALL
      SELECT 9,       2,            '2020-02-07',                   14001         FROM dual UNION ALL
      SELECT 10,      2,            '2020-04-15',                   6500          FROM dual UNION ALL
      SELECT 11,      4,            '2020-01-17',                   12000         FROM dual UNION ALL
      SELECT 12,      4,            '2020-01-21',                   14500         FROM dual UNION ALL
      SELECT 13,      4,            '2020-02-10',                   11000         FROM dual UNION ALL
      SELECT 14,      4,            '2020-05-10',                   16500         FROM dual UNION ALL
      SELECT 15,      5,            '2020-02-25',                   7500          FROM dual UNION ALL
      SELECT 16,      5,            '2020-02-25',                   2000          FROM dual
    )
  )
SELECT * FROM cte_users
```

# Exercises
Using the above basis query, replace `SELECT * FROM cte_users` with a query that operate on the CTEs to select the described 
results for each exercise.

While we feel comfortable with the way each exercise is presented, if you feel that there's room for interpretation on a given 
point, please feel free to make any/all reasonable assumptions.

In your submission back to HealthMine, please include the specific query (excluding the CTEs - those would just be redundant) 
and the output of the query for each of the exercises.

1. Write a query that returns all user step events, sorted by `name` then `activity_date`.  Only include users that have step 
events.
    * Columns: `name`, `date`, `steps`
```sql
SELECT cu.name, cs.activity_date, cs.steps
FROM cte_steps cs
INNER JOIN cte_users cu
    ON cu.id = cs.user_id
ORDER BY cu.name, cs.activity_date
```
__Results__
| row_num  | NAME  | ACTIVITY_DATE         | STEPS | 
|----------|-------|-----------------------|-------| 
| 1        | Celes | 2/25/2020 12:00:00 AM | 7500  | 
| 2        | Celes | 2/25/2020 12:00:00 AM | 2000  | 
| 3        | Locke | 2/1/2020 12:00:00 AM  | 8000  | 
| 4        | Locke | 2/4/2020 12:00:00 AM  | 11000 | 
| 5        | Locke | 2/7/2020 12:00:00 AM  | 14001 | 
| 6        | Locke | 4/15/2020 12:00:00 AM | 6500  | 
| 7        | Sabin | 1/17/2020 12:00:00 AM | 12000 | 
| 8        | Sabin | 1/21/2020 12:00:00 AM | 14500 | 
| 9        | Sabin | 2/10/2020 12:00:00 AM | 11000 | 
| 10       | Sabin | 5/10/2020 12:00:00 AM | 16500 | 
| 11       | Terra | 1/1/2020 12:00:00 AM  | 7001  | 
| 12       | Terra | 1/3/2020 12:00:00 AM  | 5000  | 
| 13       | Terra | 1/5/2020 12:00:00 AM  | 1000  | 
| 14       | Terra | 1/5/2020 12:00:00 AM  | 4000  | 
| 15       | Terra | 2/1/2020 12:00:00 AM  | 6000  | 
| 16       | Terra | 4/1/2020 12:00:00 AM  | 10500 | 

2. Write a query that returns all users that have multiple step events.
    * Columns: `name`
```sql
SELECT cu.name
FROM cte_users cu
INNER JOIN cte_steps cs
    ON cu.id = cs.user_id
GROUP BY cu.name
HAVING COUNT(*) > 1
```
__Results__
| row_num   | NAME  | 
|-----------|-------| 
| 1         | Celes | 
| 2         | Terra | 
| 3         | Locke | 
| 4         | Sabin | 


__Add a new CTE so that we can extract just the required columns from the query.  Given the prompt, we just need name and want 
to get rid of the count that associated with that.__

3. Write a query that returns all users that recorded steps on multiple days.
    * Columns: `name`
```sql
...),
step_event_count AS (
    SELECT cu.name, cs.activity_date, COUNT(*)
    FROM cte_users cu
    INNER JOIN cte_steps cs
        ON cu.id = cs.user_id
    GROUP BY cu.name, cs.activity_date
)
select name
from step_event_count
group by name
having count(*) > 1
```
__Results__
| row_num  | NAME  | 
|----------|-------| 
| 1        | Terra | 
| 2        | Locke | 
| 3        | Sabin | 

4. Write a query that returns the number of steps that each user has taken all-time.  If the user doesn't have any events, 
they should be included with 0 steps.
    * Columns: `name`, `step count`
```sql
select 
    cu.name, 
    nvl(sum(cs.steps), 0) as step_count
from cte_users cu
left join cte_steps cs
    on cu.id = cs.user_id
group by cu.name
```
__Results__
| row_num  | NAME  | STEP_COUNT | 
|----------|-------|------------| 
| 1        | Celes | 9500       | 
| 2        | Terra | 33501      | 
| 3        | Edgar | 0          | 
| 4        | Locke | 39501      | 
| 5        | Sabin | 54000      | 
5. Write a query that returns the number of steps taken in each month in 2020.  Only include months where step events 
occurred.
    * Columns: `month`, `step count`
```sql
select 
    sum(steps), 
    TO_CHAR(TO_DATE(activity_month, 'MM'), 'MONTH') AS monthname
from (
    select extract(month from activity_date) as activity_month, steps
    from cte_steps
    where extract(year from activity_date) = '2020'
) sub
group by activity_month
```
__Results__
| row_num   | STEP_COUNT | MONTHNAME | 
|-----------|------------|-----------| 
| 1         | 43501      | JANUARY   | 
| 2         | 59501      | FEBRUARY  | 
| 3         | 17000      | APRIL     | 
| 4         | 16500      | MAY       | 

6. Write a query that returns the number of steps taken in each month in 2020 through today.  If a month doesn't have any step 
events, it should be included with 0 steps.
    * Columns: `month`, `step count`
```sql
...),
all_months as (
	--- https://stackoverflow.com/questions/4582861/list-all-the-months-using-oracle-sql
    select to_char(date '2000-12-01' + numtoyminterval(level,'month'),'MONTH') as month
    from dual
    connect by level <= 12
),
step_count_to_month as (
    select 
        sum(steps) as step_count, 
        activity_month
    from (
        select extract(month from activity_date) as activity_month, steps
        from cte_steps
        where extract(year from activity_date) = '2020' and activity_date < current_date
    ) sub
    group by activity_month
)
select 
    month, 
    nvl(step_count, 0)
from step_count_to_month sctm
right join all_months am
    on TO_CHAR(TO_DATE(sctm.activity_month, 'MM'), 'MONTH') = am.month
```
__Results__
| row_num   | MONTH     | NVL(STEP_COUNT,0) | 
|-----------|-----------|-------------------| 
| 1         | JANUARY   | 43501             | 
| 2         | FEBRUARY  | 59501             | 
| 3         | APRIL     | 17000             | 
| 4         | MAY       | 16500             | 
| 5         | JULY      | 0                 | 
| 6         | AUGUST    | 0                 | 
| 7         | MARCH     | 0                 | 
| 8         | DECEMBER  | 0                 | 
| 9         | OCTOBER   | 0                 | 
| 10        | SEPTEMBER | 0                 | 
| 11        | NOVEMBER  | 0                 | 
| 12        | JUNE      | 0                 | 
7. Write a query that returns the average number of steps taken per user in each month all-time, rounded to 2 decimal places.  
Only include months where step events occurred.
    * Columns: `month`, `name`, `average steps`
```sql
select
    round(avg(steps), 2) as steps_in_month,
    name,
    TO_CHAR(TO_DATE(extract(month from activity_date), 'MM'), 'MONTH') as activity_month
from cte_steps cs
inner join cte_users cu
    on cu.id = cs.user_id
group by name, extract(month from activity_date)
```
__Results__
| row_num   | STEPS_IN_MONTH | NAME  | ACTIVITY_MONTH | 
|-----------|----------------|-------|----------------| 
| 1         | 16500          | Sabin | MAY            | 
| 2         | 4250.25        | Terra | JANUARY        | 
| 3         | 10500          | Terra | APRIL          | 
| 4         | 4750           | Celes | FEBRUARY       | 
| 5         | 11000          | Sabin | FEBRUARY       | 
| 6         | 6000           | Terra | FEBRUARY       | 
| 7         | 11000.33       | Locke | FEBRUARY       | 
| 8         | 6500           | Locke | APRIL          | 
| 9         | 13250          | Sabin | JANUARY        | 

8. ***Without*** using window/analytical functions, write a query that returns all step events that occurred on a user's first 
day of activity.
    * Columns: `name`, `date`, `steps`
```sql
...),
min_day_per_user as (
    select
        min(activity_date) as min_day,
        user_id
    from cte_steps
    group by user_id
)
select name, activity_date, steps
from cte_steps cs
inner join min_day_per_user min_day
    on cs.activity_date = min_day.min_day and cs.user_id = min_day.user_id
inner join cte_users cu
    on cs.user_id = cu.id
```
__Results__
| row_num   | NAME  | ACTIVITY_DATE         | STEPS | 
|-----------|-------|-----------------------|-------| 
| 1         | Terra | 1/1/2020 12:00:00 AM  | 7001  | 
| 2         | Locke | 2/1/2020 12:00:00 AM  | 8000  | 
| 3         | Sabin | 1/17/2020 12:00:00 AM | 12000 | 
| 4         | Celes | 2/25/2020 12:00:00 AM | 7500  | 
| 5         | Celes | 2/25/2020 12:00:00 AM | 2000  | 

9. ***With*** using window/analytical functions, write a query that returns the first step event that occurred per user.  If 
necessary, break ties using the higher step count.
    * Columns: `name`, `date`, `steps`
```sql
date_order as (
    select
        cs.*,
        row_number() over(
            partition by user_id
            order by 
                activity_date,
                steps desc
            ) as min_date
    from cte_steps cs
)
select name, activity_date, steps
from date_order date_order
join cte_users ce
    on date_order.user_id = ce.id
where min_date = 1
```
__Results__
| row_num   | NAME  | ACTIVITY_DATE         | STEPS | 
|-----------|-------|-----------------------|-------| 
| 1         | Terra | 1/1/2020 12:00:00 AM  | 7001  | 
| 2         | Locke | 2/1/2020 12:00:00 AM  | 8000  | 
| 3         | Sabin | 1/17/2020 12:00:00 AM | 12000 | 
| 4         | Celes | 2/25/2020 12:00:00 AM | 7500  | 

10. Using window/analytical functions, write a query that returns all step events, and the number of steps taken in the 
previous event by the user.  If necessary, break ties using `cte_steps.id`.
    * Columns: `name`, `date`, `steps`, `previous steps`
```sql
select
    name,
    activity_date,
    steps,
    lag(steps) over (partition by user_id order by activity_date, cs.id) as previous_steps
from cte_steps cs
join cte_users cu
    on cs.user_id = cu.id
```
__Results__
| row_num   | NAME  | ACTIVITY_DATE         | STEPS | PREVIOUS_STEPS | 
|-----------|-------|-----------------------|-------|----------------| 
| 1         | Terra | 1/1/2020 12:00:00 AM  | 7001  | NULL           | 
| 2         | Terra | 1/3/2020 12:00:00 AM  | 5000  | 7001           | 
| 3         | Terra | 1/5/2020 12:00:00 AM  | 4000  | 5000           | 
| 4         | Terra | 1/5/2020 12:00:00 AM  | 1000  | 4000           | 
| 5         | Terra | 2/1/2020 12:00:00 AM  | 6000  | 1000           | 
| 6         | Terra | 4/1/2020 12:00:00 AM  | 10500 | 6000           | 
| 7         | Locke | 2/1/2020 12:00:00 AM  | 8000  | NULL           | 
| 8         | Locke | 2/4/2020 12:00:00 AM  | 11000 | 8000           | 
| 9         | Locke | 2/7/2020 12:00:00 AM  | 14001 | 11000          | 
| 10        | Locke | 4/15/2020 12:00:00 AM | 6500  | 14001          | 
| 11        | Sabin | 1/17/2020 12:00:00 AM | 12000 | NULL           | 
| 12        | Sabin | 1/21/2020 12:00:00 AM | 14500 | 12000          | 
| 13        | Sabin | 2/10/2020 12:00:00 AM | 11000 | 14500          | 
| 14        | Sabin | 5/10/2020 12:00:00 AM | 16500 | 11000          | 
| 15        | Celes | 2/25/2020 12:00:00 AM | 7500  | NULL           | 
| 16        | Celes | 2/25/2020 12:00:00 AM | 2000  | 7500           | 

11. If the data in the CTEs above were tables with no indexes, which indexes would you recommend (if any) to help satisfy the 
previous queries?

Would recommend indexing the primary keys of each table, which would be the ID field of each table since we are frequently 
joining and querying based on that field.  Also, since we are sorting and running a few WHERE clauses on the `activity_date` 
field it would make sense to index this as well to speed up the joins and the 'where' filters.  From here, as the tables got 
more complicated my next steps to optimize would running explain on any problem queries and trying to find long-running clauses that could be sped up 
without over-indexing the table.

