# Date Functions

## Working with Dates

- The Oracle database stores dates in an internal numeric format: century, year, month, day, hours, minutes, and seconds.
- The default date display format is DD-MON-YY. (DD-MON-RR)
    - Enables you to store 21st-century dates in the 20th century by specifying only the last two digits of the year
    - Enables you to store 20th-century dates in the 21st century in the same way

```sql
SELECT last_name, hire_date
FROM nikovits.employees
WHERE hire_date < '01-FEB-88';
```

| LAST_NAME | HIRE_DATE |
|-----------|-----------|
| King      | 17/06/87  |
| Whalen    | 17/09/87  |

## SYSDATE Function

SYSDATE is a function that returns:

- Date
- Time

## Arithmetic with Dates

- Add or subtract a number to or from a date for a resultant date value.
- Subtract two dates to find the number of days between those dates.
- Add hours to a date by dividing the number of hours by 24.

```sql
SELECT last_name, (SYSDATE-hire_date)/7 AS WEEKS
FROM nikovits.employees
WHERE department_id = 90;
```

| LAST_NAME | WEEKS                |
|-----------|----------------------|
| King      | 1995.593908730158730 |
| Kochhar   | 1877.451051587301587 |
| De Haan   | 1704.593908730158730 |

## Date Functions Available

| Function          | Description                              |
|-------------------|------------------------------------------|
| MONTHS_BETWEEN    | Number of months between two dates       |
| ADD_MONTHS        | Add calendar months to date              |
| NEXT_DAY          | Next day of the date specified           |
| LAST_DAY          | Last day of the month                    |
| ROUND             | Round date                               |
| TRUNC             | Truncate date                            |

## Using Date Functions

| Function                                          | Result             | Explanation                                                                 |
|---------------------------------------------------|--------------------|-----------------------------------------------------------------------------|
| MONTHS_BETWEEN('01-SEP-95', '11-JAN-94')          | 19.6774194         | Calculates the number of months between the two dates (later date first). Includes fractional months based on days. |
| ADD_MONTHS('11-JAN-94', 6)                        | '11-JUL-94'        | Adds 6 calendar months to the specified date.                               |
| NEXT_DAY('01-SEP-95', 'FRIDAY')                   | '08-SEP-95'        | Returns the date of the next Friday after 01-SEP-1995.                      |
| LAST_DAY('01-FEB-95')                             | '28-FEB-95'        | Returns the last day of the month for February 1995 (non-leap year).         |

Assume `SYSDATE = '25-JUL-2003'`.

| Function                          | Result          | Explanation                                                                 |
|-----------------------------------|-----------------|-----------------------------------------------------------------------------|
| ROUND(SYSDATE, 'MONTH')           | 01-AUG-2003     | Rounds to the nearest month. Since July 25 is past the 15th, rounds **up** to August 1. |
| ROUND(SYSDATE, 'YEAR')            | 01-JAN-2004     | Rounds to the nearest year. July 25 is past July 1, so rounds **up** to next year. |
| TRUNC(SYSDATE, 'MONTH')           | 01-JUL-2003     | Truncates to the first day of the current month (always sets day to 1).     |
| TRUNC(SYSDATE, 'YEAR')            | 01-JAN-2003     | Truncates to the first day of the current year (January 1).                 |

## TO_CHAR Function with Dates

### TO_CHAR(date, 'format_model')

The format model:

- Must be enclosed by single quotation marks
- Is case-sensitive
- Can include any valid date format element
- Has an fm element to remove padded blanks or suppress leading zeros
- Is separated from the date value by a comma TO_CHAR(date, 'format_model')

### Elements of the Date Format Model

| Element | Result                                      | Example (for December 24, 2025 - Wednesday) |
|---------|---------------------------------------------|---------------------------------------------|
| YYYY    | Full year in numbers                        | 2025                                        |
| YEAR    | Year spelled out (in English)               | TWENTY TWENTY-FIVE                          |
| MM      | Two-digit value for month                   | 12                                          |
| MONTH   | Full name of the month                      | DECEMBER                                    |
| MON     | Three-letter abbreviation of the month      | DEC                                         |
| DY      | Three-letter abbreviation of the day of the week | WED                                    |
| DAY     | Full name of the day of the week            | WEDNESDAY                                   |
| DD      | Numeric day of the month                    | 24                                          |

### Time Elements

Time elements format the time portion of the date.

| Format Element       | Example Result          | Description                                      |
|----------------------|-------------------------|--------------------------------------------------|
| HH24:MI:SS AM        | 15:45:32 PM             | 24-hour format with hours, minutes, seconds, and AM/PM indicator |

### Literal Text

Add character strings by enclosing them in **double quotation marks**.

| Format Example          | Result                  | Notes                                            |
|-------------------------|-------------------------|--------------------------------------------------|
| DD "of" MONTH           | 12 of OCTOBER           | "of" is output literally                         |

### Number Suffixes

Number suffixes spell out numbers (ordinal form).

| Format Element | Result       | Description                                      |
|----------------|--------------|--------------------------------------------------|
| ddspth         | fourteenth   | Spells the day with ordinal suffix (e.g., 1st, 2nd, 3rd, 14th) |

### Using the TO_CHAR Function with Dates

```sql
SELECT last_name,
    TO_CHAR(hire_date, 'fmDD Month YYYY')
    AS HIREDATE
FROM nikovits.employees;
```

| LAST_NAME   | HIREDATE          |
|-------------|-------------------|
| King        | 17 June 1987      |
| Kochhar     | 21 September 1989 |
| De Haan     | 13 January 1993   |
| Hunold      | 3 January 1990    |
| Ernst       | 21 May 1991       |
| Austin      | 25 June 1997      |
| Pataballa   | 5 February 1998   |
| Lorentz     | 7 February 1999   |
