# Subqueries

## Introduction to Subqueries

- A **subquery** is a query that is nested inside another query.
- The outer query is called the **main query**.
- The subquery can return values to the main query to help it execute.

## Guidelines for Using Subqueries

- Enclose subqueries in parentheses.
- Place subqueries on the right side of the comparison condition.
- The ORDER BY clause in the subquery is not needed unless you are performing Top-N analysis.
- Use single-row operators with single-row subqueries, and use multiple-row operators with multiple-row subqueries.

## Types of Subqueries

![alt text](../assets/P4/typessub.png){ width=700 }

### Single-Row Subqueries

- Return **only one row**
- Use single-row comparison operators

| Operator | Meaning                  |
|----------|--------------------------|
| =        | Equal to                 |
| >        | Greater than             |
| >=       | Greater than or equal to |
| <        | Less than                |
| <=       | Less than or equal to    |
| <>       | Not equal to             |

#### Executing Single-Row Subqueries

```sql
SELECT last_name, job_id, salary
FROM nikovits.employees
WHERE job_id =
(SELECT job_id
FROM nikovits.employees
WHERE employee_id = 141)
AND salary >
(SELECT salary
FROM nikovits.employees
WHERE employee_id = 143);
```

| LAST_NAME            | JOB_ID               | SALARY               |
|----------------------|----------------------|----------------------|
| Nayer                | ST_CLERK             | 3200                 |
| Mikkilineni          | ST_CLERK             | 2700                 |
| Bissot               | ST_CLERK             | 3300                 |
| Atkinson             | ST_CLERK             | 2800                 |
| Mallin               | ST_CLERK             | 3300                 |
| Rogers               | ST_CLERK             | 2900                 |
| Ladwig               | ST_CLERK             | 3600                 |
| Stiles               | ST_CLERK             | 3200                 |
| Seo                  | ST_CLERK             | 2700                 |

### Using Group Functions in a Subquery

```sql
SELECT last_name, job_id, salary
FROM nikovits.employees
WHERE salary =
(SELECT MIN(salary)
FROM nikovits.employees);
```

| LAST_NAME            | JOB_ID               | SALARY               |
|----------------------|----------------------|----------------------|
| Olson                | ST_CLERK             | 2100                 |

### The HAVING Clause with Subqueries

- The Oracle server executes subqueries first.
- The Oracle server returns results into the HAVING clause of the main query.

```sql
SELECT   department_id, MIN(salary)
FROM     employees
GROUP BY department_id
HAVING   MIN(salary) > 
         (SELECT MIN(salary)
          FROM   employees
          WHERE  department_id = 50);
```

### What Is Wrong with This Statement?

```sql
SELECT employee_id, last_name
FROM   employees
WHERE  salary =
       (SELECT   MIN(salary)
        FROM     employees
        GROUP BY department_id);
``` 

![alt text](../assets/P4/error3.png){ width=700 }

Single-row operator with multiple-row subquery

### Will This Statement Return Rows?

```sql
SELECT last_name, job_id
FROM   employees
WHERE  job_id =
       (SELECT job_id
        FROM   employees
        WHERE  last_name = 'Haas');
```

![alt text](../assets/P4/error4.png){ width=700 }

Subquery returns no values.

### Multiple-Row Subqueries

- Return **more than one row**
- Use multiple-row comparison operators

| Operator | Meaning                                      |
|----------|----------------------------------------------|
| IN       | Equal to any member in the list              |
| ANY      | Compare value to each value returned by the subquery |
| ALL      | Compare value to every value returned by the subquery |

#### Using the ANY Operator in Multiple-Row Subqueries

```sql
SELECT employee_id, last_name, job_id, salary
FROM nikovits.employees
WHERE salary < ANY
(SELECT salary
FROM nikovits.employees
WHERE job_id = 'IT_PROG')
AND job_id <> 'IT_PROG';
```

| EMPLOYEE_ID          | LAST_NAME            | JOB_ID               | SALARY               |
|----------------------|----------------------|----------------------|----------------------|
| 132                  | Olson                | ST_CLERK             | 2100                 |
| 136                  | Philtanker           | ST_CLERK             | 2200                 |
| 128                  | Markle               | ST_CLERK             | 2200                 |
| 135                  | Gee                  | ST_CLERK             | 2400                 |
| 127                  | Landry               | ST_CLERK             | 2400                 |
| 191                  | Perkins              | SH_CLERK             | 2500                 |
| 182                  | Sullivan             | SH_CLERK             | 2500                 |
| 144                  | Vargas               | ST_CLERK             | 2500                 |
| 140                  | Patel                | ST_CLERK             | 2500                 |

#### Using the ALL Operator in Multiple-Row Subqueries

```sql
SELECT employee_id, last_name, job_id, salary
FROM nikovits.employees
WHERE salary < ALL
(SELECT salary
FROM nikovits.employees
WHERE job_id = 'IT_PROG')
AND job_id <> 'IT_PROG';
```

| EMPLOYEE_ID          | LAST_NAME            | JOB_ID               | SALARY               |
|----------------------|----------------------|----------------------|----------------------|
| 185                  | Bull                 | SH_CLERK             | 4100                 |
| 192                  | Bell                 | SH_CLERK             | 4000                 |
| 193                  | Everett              | SH_CLERK             | 3900                 |
| 188                  | Chung                | SH_CLERK             | 3800                 |
| 137                  | Ladwig               | ST_CLERK             | 3600                 |
| 189                  | Dilly                | SH_CLERK             | 3600                 |
| 141                  | Rajs                 | ST_CLERK             | 3500                 |
| 186                  | Dellinger            | SH_CLERK             | 3400                 |
| 133                  | Mallin               | ST_CLERK             | 3300                 |
| 129                  | Bissot               | ST_CLERK             | 3300                 |
| 180                  | Taylor               | SH_CLERK             | 3200                 |
| 138                  | Stiles               | ST_CLERK             | 3200                 |

## Using a Subquery to Solve a Problem

![alt text](../assets/P4/sub.png){ width=700 }

### Subquery Syntax

```sql
SELECT  select_list
FROM    table
WHERE   expr operator
        (SELECT select_list
         FROM   table);
```

- The subquery (inner query) executes once before the main query (outer query).
- The result of the subquery is used by the main query.

```sql
SELECT last_name
FROM nikovits.employees
WHERE salary >
(SELECT salary
FROM nikovits.employees
WHERE last_name = 'Abel');
```

| LAST_NAME            |
|----------------------|
| King                 |
| Kochhar              |
| De Haan              |
| Greenberg            |
| Russell              |
| Partners             |
| Errazuriz            |
| Ozer                 |
| Hartstein            |
| Higgins              |

## Correlated Subqueries

- **Correlated subqueries** are used for **row-by-row processing**.
- Each subquery is executed **once for every row** of the outer query.
- The inner query references columns from the outer query (making it "correlated").

<pre>
GET
candidate row from outer query
↓
EXECUTE
inner query using candidate row value
↓
USE
values from inner query to qualify or disqualify candidate row
</pre>

The subquery **references a column from a table in the parent (outer) query**.

```sql
SELECT column1, column2, ...
FROM table1 outer
WHERE column1 operator
    (SELECT column1, column2
     FROM table2
     WHERE expr1 = outer.expr2);
```

### Finding Employees with Above-Average Salary

Find all employees who earn more than the average salary in their department.

```sql
SELECT last_name, salary, department_id
FROM employees outer
WHERE salary > 
    (SELECT AVG(salary)
     FROM employees
     WHERE department_id = outer.department_id);
```

Each time a row from the outer query is processed, the inner query is evaluated.

### Correlated Subquery Example

Display details of those employees who have changed jobs at least twice.

```sql
SELECT e.employee_id, e.last_name, e.job_id
FROM employees e
WHERE 2 <= (
    SELECT COUNT(*)
    FROM job_history
    WHERE employee_id = e.employee_id
);
```

| EMPLOYEE_ID | LAST_NAME | JOB_ID  |
|-------------|-----------|---------|
| 101         | Kochar    | AD_VP   |
| 176         | Taylor    | SA_REP  |
| 200         | Whalen    | AD_ASST |

## Using the EXISTS Operator

- The **EXISTS** operator tests for the **existence** of rows in the result set of the subquery.
- It returns **TRUE** if the subquery returns at least one row, **FALSE** otherwise.
- **NOT EXISTS** does the opposite.

If a subquery row value is found:

- The search **does not continue** in the inner query (short-circuits after first match).
- The condition is flagged **TRUE**.

If a subquery row value is not found:

- The condition is flagged **FALSE**.
- The search **continues** in the inner query until all rows are checked or a match is found.

# Find Employees Who Have at Least One Person Reporting to Them

```sql
SELECT employee_id, last_name, job_id, department_id
FROM employees outer
WHERE EXISTS (
    SELECT 'X'
    FROM employees
    WHERE manager_id = outer.employee_id
);
```

| EMPLOYEE_ID | LAST_NAME   | JOB_ID   | DEPARTMENT_ID |
|-------------|-------------|----------|---------------|
| 100         | King        | AD_PRES  | 90            |
| 101         | Kochhar     | AD_VP    | 90            |
| 102         | De Haan     | AD_VP    | 90            |
| 103         | Hunold      | IT_PROG  | 60            |
| 108         | Greenberg   | FI_MGR   | 100           |
| 114         | Raphaely    | PU_MAN   | 30            |
| 120         | Weiss       | ST_MAN   | 50            |
| 121         | Fripp       | ST_MAN   | 50            |
| 122         | Kaufling    | ST_MAN   | 50            |

# Find All Departments That Do Not Have Any Employees

```sql
SELECT department_id, department_name
FROM departments d
WHERE NOT EXISTS (
    SELECT 'X'
    FROM employees
    WHERE department_id = d.department_id
);
```

| DEPARTMENT_ID | DEPARTMENT_NAME       |
|---------------|-----------------------|
| 120           | Treasury              |
| 130           | Corporate Tax         |
| 140           | Control And Credit    |
| 150           | Shareholder Services  |
| 160           | Benefits              |
| 170           | Manufacturing         |

# The WITH Clause

- Using the **WITH** clause, you can use the same query block in a `SELECT` statement when it occurs more than once within a complex query.
- The **WITH** clause retrieves the results of a query block and stores it in the user's **temporary tablespace**.
- The **WITH** clause **improves performance** by avoiding repeated execution of the same subquery.

# WITH Clause Example

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

## Null Values in a Subquery

```sql
SELECT  emp.last_name
FROM    employees emp
WHERE   emp.employee_id NOT IN
        (SELECT mgr.manager_id
         FROM   employees mgr);
```

no rows selected

x NOT IN (A, B, NULL) → Unknown
