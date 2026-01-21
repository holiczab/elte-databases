## Conditional Expressions

- Provide the use of IF-THEN-ELSE logic within a SQL statement
- Use two methods:
    - CASE expression
    - DECODE function

## CASE Expression

Facilitates conditional inquiries by doing the work of an IF-THEN-ELSE statement:

```sql
CASE expr WHEN comparison_expr1 THEN return_expr1
     [WHEN comparison_expr2 THEN return_expr2
      WHEN comparison_exprn THEN return_exprn
      ELSE else_expr]
END
```

## IF THEN ELSE statement:

```sql
SELECT last_name, job_id, salary,
    CASE job_id WHEN 'IT_PROG' THEN 1.10*salary
                WHEN 'ST_CLERK' THEN 1.15*salary
                WHEN 'SA_REP' THEN 1.20*salary
    ELSE salary END "REVISED_SALARY"
FROM nikovits.employees;
```

| LAST_NAME   | JOB_ID  | SALARY | REVISED_SALARY |
|-------------|---------|--------|----------------|
| King        | AD_PRES | 24000  | 24000          |
| Kochhar     | AD_VP   | 17000  | 17000          |
| De Haan     | AD_VP   | 17000  | 17000          |
| Hunold      | IT_PROG | 9000   | 9900           |
| Ernst       | IT_PROG | 6000   | 6600           |
| Austin      | IT_PROG | 4800   | 5280           |
| Pataballa   | IT_PROG | 4800   | 5280           |

## DECODE Function in Oracle SQL

Facilitates conditional inquiries by doing the work of a CASE expression or an IF-THEN-ELSE statement:

```sql
DECODE(col|expression, search1, result1
       [, search2, result2, ...]
       [, default])
```

```sql
SELECT last_name, job_id, salary,
    DECODE(job_id, 'IT_PROG', 1.10*salary,
            'ST_CLERK', 1.15*salary,
            'SA_REP', 1.20*salary,
            salary)
    REVISED_SALARY
FROM nikovits.employees;
```

| LAST_NAME   | JOB_ID  | SALARY | REVISED_SALARY |
|-------------|---------|--------|----------------|
| King        | AD_PRES | 24000  | 24000          |
| Kochhar     | AD_VP   | 17000  | 17000          |
| De Haan     | AD_VP   | 17000  | 17000          |
| Hunold      | IT_PROG | 9000   | 9900           |
| Ernst       | IT_PROG | 6000   | 6600           |
| Austin      | IT_PROG | 4800   | 5280           |
| Pataballa   | IT_PROG | 4800   | 5280           |

## Display the applicable tax rate for each employee in department 80:

```sql
SELECT last_name, salary,
        DECODE (TRUNC(salary/2000, 0),
                            0, 0.00,
                            1, 0.09,
                            2, 0.20,
                            3, 0.30,
                            4, 0.40,
                            5, 0.42,
                            6, 0.44,
                            0.45) TAX_RATE
FROM nikovits.employees
WHERE department_id = 80;
```

| LAST_NAME   | SALARY | TAX_RATE |
|-------------|--------|----------|
| Russell     | 14000  | .45      |
| Partners    | 13500  | .44      |
| Errazuriz   | 12000  | .44      |
| Cambrault   | 11000  | .42      |
| Zlotkey     | 10500  | .42      |
| Tucker      | 10000  | .42      |
| Bernstein   | 9500   | .4       |
| Hall        | 9000   | .4       |
| Olsen       | 8000   | .4       |