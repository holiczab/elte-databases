## The WITH Clause

- Using the **WITH** clause, you can use the same query block in a `SELECT` statement when it occurs more than once within a complex query.
- The **WITH** clause retrieves the results of a query block and stores it in the user's **temporary tablespace**.
- The **WITH** clause **improves performance** by avoiding repeated execution of the same subquery.

## WITH Clause Example

Using the `WITH` clause, write a query to display the department name and total salaries for those departments whose total salary is greater than the average salary across departments.

```sql
WITH 
dept_costs AS (
    SELECT d.department_name, SUM(e.salary) AS dept_total
    FROM employees e
    JOIN departments d ON e.department_id = d.department_id
    GROUP BY d.department_name
),
avg_cost AS (
    SELECT SUM(dept_total)/COUNT(*) AS dept_avg
    FROM dept_costs
)
SELECT *
FROM dept_costs
WHERE dept_total > 
    (SELECT dept_avg
     FROM avg_cost)
ORDER BY department_name;
```