# Query Exercises
The following are some exercises in writing different types of queries against a very simple and small dataset.

If you don't have convenient access to a database engine, [https://rextester.com/l/oracle_online_compiler]https://rextester.com/l/oracle_online_compiler) is a good paste-and-execute SQL evaluator. [http://sqlfiddle.com](http://sqlfiddle.com) is also a good option, but requires building up the schema to query against separately.

While the basis query was written with Oracle in mind, it can be easily refactored to run on any database engine that supports CTEs and window/analytical functions. For example, to run on Postgres, simply remove the `FROM dual` statements. The statements to convert strings to dates might also need to be adjusted based on the engine of your selection.

The intent of these exercises is to demonstrate a working knowledge of SQL in general rather than familiarity with one RDBMS engine over another, so please feel free to use the engine of your choice.

# Dataset
This dataset represents user pedometer (steps) data. A piece of data will be recorded with the number of steps taken on a particular date.
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
Using the above basis query, replace `SELECT * FROM cte_users` with a query that operate on the CTEs to select the described results for each exercise.

While we feel comfortable with the way each exercise is presented, if you feel that there's room for interpretation on a given point, please feel free to make any/all reasonable assumptions.

In your submission back to HealthMine, please include the specific query (excluding the CTEs - those would just be redundant) and the output of the query for each of the exercises.

1. Write a query that returns all user step events, sorted by `name` then `activity_date`.  Only include users that have step events.
    * Columns: `name`, `date`, `steps`
```sql
SELECT cu.name, cs.activity_date, cs.steps
FROM cte_users cu
INNER JOIN cte_steps cs
    ON cu.id = cs.user_id
ORDER BY cu.name, cs.activity_date
```
2. Write a query that returns all users that have multiple step events.
    * Columns: `name`
```sql
select name from (
    SELECT cu.name, COUNT(*)
    FROM cte_users cu
    INNER JOIN cte_steps cs
        ON cu.id = cs.user_id
    GROUP BY cu.name
    HAVING COUNT(*) > 1
)
```
select name from step_event_count
```
__Add a new CTE so that we can extract just the required columns from the query.  Given the prompt, we just need name and want to get rid of the count that associated with that.__
3. Write a query that returns all users that recorded steps on multiple days.
    * Columns: `name`
```sql
step_event_count AS (
    SELECT cu.name, cs.activity_date, COUNT(*)
    FROM cte_users cu
    INNER JOIN cte_steps cs
        ON cu.id = cs.user_id
    GROUP BY cu.name, cs.activity_date
)
select name from (
    select name, count(*)
    from step_event_count
    group by name
    having count(*) > 1
)
```
4. Write a query that returns the number of steps that each user has taken all-time.  If the user doesn't have any events, they should be included with 0 steps.
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
5. Write a query that returns the number of steps taken in each month in 2020.  Only include months where step events occurred.
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
6. Write a query that returns the number of steps taken in each month in 2020 through today.  If a month doesn't have any step events, it should be included with 0 steps.
    * Columns: `month`, `step count`
7. Write a query that returns the average number of steps taken per user in each month all-time, rounded to 2 decimal places.  Only include months where step events occurred.
    * Columns: `month`, `name`, `average steps`
8. ***Without*** using window/analytical functions, write a query that returns all step events that occurred on a user's first day of activity.
    * Columns: `name`, `date`, `steps`
9. ***With*** using window/analytical functions, write a query that returns the first step event that occurred per user.  If necessary, break ties using the higher step count.
    * Columns: `name`, `date`, `steps`
10. Using window/analytical functions, write a query that returns all step events, and the number of steps taken in the previous event by the user.  If necessary, break ties using `cte_steps.id`.
    * Columns: `name`, `date`, `steps`, `previous steps`
11. If the data in the CTEs above were tables with no indexes, which indexes would you recommend (if any) to help satisfy the previous queries?
