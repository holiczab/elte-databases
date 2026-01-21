## What Are Group Functions?

Group functions operate on sets of rows to give one result per group.

![alt text](../assets/P3/group.png){ width=700 }

## Types of Group Functions

- AVG
- COUNT
- MAX
- MIN
- STDDEV
- SUM
- VARIANCE

## Group Functions: Syntax

```sql
SELECT     [column,] group_function(column), ...
FROM       table
[WHERE     condition]
[GROUP BY  column]
[ORDER BY  column];
```

## Using the AVG and SUM Functions

```sql
SELECT AVG(salary), MAX(salary),
        MIN(salary), SUM(salary)
        FROM nikovits.employees
WHERE job_id LIKE '%REP%';
```

| AVG(SALARY)         | MAX(SALARY) | MIN(SALARY) | SUM(SALARY) |
|---------------------|-------------|-------------|-------------|
| 8272.727272727272727 | 11500       | 6000        | 273000      |

## Using the MIN and MAX Functions

```sql
SELECT MIN(hire_date), MAX(hire_date)
FROM nikovits.employees;
```

| MIN(HIRE_DATE) | MAX(HIRE_DATE) |
|---------------|----------------|
| 17/06/87      | 21/04/00       |

## Using the COUNT Function

## COUNT(*) returns the number of rows in a table:

```sql
SELECT COUNT(*)
FROM nikovits.employees
WHERE department_id = 50;
```

| COUNT(*) |
|----------|
| 45       |

## COUNT(expr) returns the number of rows with non-null values for expr:

```sql
SELECT COUNT(commission_pct)
FROM nikovits.employees
WHERE department_id = 80
```

| COUNT(COMMISSION_PCT) |
|-----------------------|
| 34                    |

## Using the DISTINCT Keyword

- COUNT(DISTINCT expr) returns the number of distinct non-null values of the expr.
- To display the number of distinct department values in the nikovits.employees table:

```sql
SELECT COUNT(DISTINCT department_id)
FROM nikovits.employees;
```

| COUNT(DISTINCT DEPARTMENT_ID) |
|-------------------------------|
| 11                            |

## Group Functions and Null Values

## Group functions ignore null values in the column:

```sql
SELECT AVG(commission_pct)
FROM nikovits.employees;
```

| AVG(COMMISSION_PCT)  |
|----------------------|
| .2228571428571428571 |

## The NVL function forces group functions to include null values:

```sql
SELECT AVG(NVL(commission_pct, 0))
FROM nikovits.employees;
```

| AVG(NVL(COMMISSION_PCT,0))          |
|-------------------------------------|
| .0728971962616822429906542          |