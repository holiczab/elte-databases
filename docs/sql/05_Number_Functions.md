# Number Functions

Number functions perform operations on numeric values.

## Key Number Functions (Oracle SQL)

- **ROUND**: Rounds value to specified decimal places
- **TRUNC**: Truncates value to specified decimal places (no rounding)
- **MOD**: Returns remainder of division

## Examples

| Function                  | Result   | Description                                                                 |
|---------------------------|----------|-----------------------------------------------------------------------------|
| ROUND(45.926, 2)          | 45.93    | Rounds to 2 decimal places (standard rounding: 0.006 → up)                  |
| TRUNC(45.926, 2)          | 45.92    | Truncates to 2 decimal places (cuts off excess without rounding)            |
| MOD(1600, 300)            | 100      | Returns remainder of 1600 ÷ 300 (1600 - 5×300 = 100)                         |

## DUAL Table

**DUAL** is a dummy table that you can use to view results from functions and calculations.

## Using the ROUND Function

![alt text](../assets/P3/round.png){ width=700 }

## Using the TRUNC Function

![alt text](../assets/P3/trunc.png){ width=700 }

## Using the MOD Function

For all nikovits.employees with job title of Sales Representative, calculate the remainder of the salary after it is divided by 5,000.

```sql
SELECT last_name, salary, MOD(salary, 5000)
FROM nikovits.employees
WHERE job_id = 'SA_REP';
```

| LAST_NAME  | SALARY | MOD(SALARY,5000) |
|------------|--------|------------------|
| Tucker     | 10000  | 0                |
| Bernstein  | 9500   | 4500             |
| Hall       | 9000   | 4000             |
| Olsen      | 8000   | 3000             |
| Cambrault  | 7500   | 2500             |
| Tuvault    | 7000   | 2000             |
| King       | 10000  | 0                |
