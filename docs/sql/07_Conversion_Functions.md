# Conversion Functions

![alt text](../assets/P3/convert.png){ width=700 }

## Implicit Data Type Conversion

For assignments (e.g., in `INSERT`, `UPDATE`, or variable assignments), the Oracle server can **automatically** perform implicit data type conversions for certain compatible types.

| From                  | To        | Notes                                                                 |
|-----------------------|-----------|-----------------------------------------------------------------------|
| VARCHAR2 or CHAR      | NUMBER    | String must represent a valid number (e.g., '123.45' → 123.45)        |
| VARCHAR2 or CHAR      | DATE      | String must be in the default date format or a recognizable format (depends on NLS_DATE_FORMAT) |
| NUMBER                | VARCHAR2  | Number is converted to its string representation                       |
| DATE                  | VARCHAR2  | Date is converted to string using the current NLS_DATE_FORMAT         |

For **expression evaluation** (e.g., in `WHERE` clause conditions, `SELECT` list expressions, or function arguments), the Oracle Server can **automatically** perform implicit data type conversions for certain compatible types.

| From                  | To        | Notes                                                                 |
|-----------------------|-----------|-----------------------------------------------------------------------|
| VARCHAR2 or CHAR      | NUMBER    | String must represent a valid number; otherwise, runtime error (ORA-01722) |
| VARCHAR2 or CHAR      | DATE      | String must match the session's NLS_DATE_FORMAT or be recognizable   |

## Explicit Data Type Conversion

![alt text](../assets/P3/explicit.png){ width=700 }

## Using the TO_CHAR Function with Numbers

```sql
TO_CHAR(number, 'format_model')
```

These are some of the format elements that you can use with the TO_CHAR function to display a number value as a character:

| Element | Result/Description                              |
|---------|-------------------------------------------------|
| 9       | Represents a digit (leading zeros suppressed)   |
| 0       | Forces a zero to be displayed                   |
| $       | Places a floating dollar sign                   |
| L       | Uses the floating local currency symbol         |
| .       | Prints a decimal point                          |
| ,       | Prints a comma as thousands indicator           |


```sql
SELECT TO_CHAR(salary, '$99,999.00') SALARY
FROM nikovits.employees
WHERE last_name = 'Ernst';
```

| SALARY     |
|------------|
| $6,000.00  |

# Using TO_NUMBER and TO_DATE Functions

Convert a character string to a number format using the TO_NUMBER function:

```sql
TO_NUMBER(char[, 'format_model'])
```

Convert a character string to a date format using the TO_DATE function:

```sql
TO_DATE(char[, 'format_model'])
```

- These functions have an FX modifier.
- This modifier specifies the exact matching for the character argument and the format model of a TO_DATE function.

# Nesting Functions

- Single-row functions can be nested to any level.  
- Nested functions are evaluated from the **deepest level** to the **least deep level** (inside-out).

![alt text](../assets/P3/nest.png){ width=700 }

```sql
SELECT last_name,
    UPPER(CONCAT(SUBSTR (LAST_NAME, 1, 8), '_US'))
FROM nikovits.employees
WHERE department_id = 60;
```

| LAST_NAME  | UPPER(CONCAT(SUBSTR(LAST_NAME,1,8),'_US')) |
|------------|--------------------------------------------|
| Hunold     | HUNOLD_US                                  |
| Ernst      | ERNST_US                                   |
| Austin     | AUSTIN_US                                  |
| Pataballa  | PATABALL_US                                |
| Lorentz    | LORENTZ_US                                 |

# General Functions

The following functions work with any data type and pertain to using nulls:

    - NVL (expr1, expr2)
    - NVL2 (expr1, expr2, expr3)
    - NULLIF (expr1, expr2)
    - COALESCE (expr1, expr2, ..., exprn)

# NVL Function

Converts a null value to an actual value:

- Data types that can be used are date, character, and number.
- Data types must match:

  - NVL(commission_pct, 0)
  - NVL(hire_date, '01-JAN-97')
  - NVL(job_id, 'No Job Yet')

# Using the NVL Function

![alt text](../assets/P3/nvl.png){ width=700 }

# Using the NVL2 Function

![alt text](../assets/P3/nvl2.png){ width=700 }

# Using the NULLIF Function

![alt text](../assets/P3/nullif.png){ width=700 }

# Using the COALESCE Function

- The advantage of the **COALESCE** function over the NVL function is that the COALESCE function can take multiple alternate values.
- If the first expression is not null, the COALESCE function returns that expression; otherwise, it does a COALESCE of the remaining expressions.

```sql
SELECT last_name,
    COALESCE(manager_id,commission_pct, -1) comm
FROM nikovits.employees
ORDER BY commission_pct;
```

| LAST_NAME | COMM |
|-----------|------|
| Lee       | 147  |
| Johnson   | 149  |
| Marvins   | 147  |
| Banda     | 147  |
| Kumar     | 148  |
| Ande      | 147  |
| Greene    | 147  |
| Grant     | 149  |
| Tuvault   | 145  |
| Bates     | 148  |

# Conditional Expressions

- Provide the use of IF-THEN-ELSE logic within a SQL statement
- Use two methods:
    - CASE expression
    - DECODE function

# CASE Expression

Facilitates conditional inquiries by doing the work of an IF-THEN-ELSE statement:

```sql
CASE expr WHEN comparison_expr1 THEN return_expr1
     [WHEN comparison_expr2 THEN return_expr2
      WHEN comparison_exprn THEN return_exprn
      ELSE else_expr]
END
```

# IF THEN ELSE statement:

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

# DECODE Function in Oracle SQL

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

# Display the applicable tax rate for each employee in department 80:

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
