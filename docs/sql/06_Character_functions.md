## Character Functions

![alt text](../assets/P3/charf.png){ width=700 }

## Case-Manipulation Functions in SQL

These functions convert the case for character strings.

## Common Case-Manipulation Functions

| Function                       | Result              | Description                                      |
|--------------------------------|---------------------|--------------------------------------------------|
| LOWER('SQL Course')            | sql course          | Converts all characters to lowercase             |
| UPPER('SQL Course')            | SQL COURSE          | Converts all characters to uppercase             |
| INITCAP('SQL Course')          | Sql Course          | Converts the first letter of each word to uppercase and the rest to lowercase |

## Using Case-Manipulation Functions

Display the employee number, name, and department number for employee Higgins:

```sql
SELECT employee_id, last_name, department_id
FROM nikovits.employees
WHERE last_name = 'higgins';
```

| EMPLOYEE_ID | LAST_NAME | DEPARTMENT_ID |
|-------------|-----------|---------------|

```sql
SELECT employee_id, last_name, department_id
FROM nikovits.employees
WHERE LOWER(last_name) = 'higgins';
```

| EMPLOYEE_ID | LAST_NAME | DEPARTMENT_ID |
|-------------|-----------|---------------|
| 205         | Higgins   | 110           |

## Character-Manipulation Functions in SQL

These functions manipulate character strings.

## Common Character-Manipulation Functions

| Function                                      | Result                  | Description                                                                 |
|-----------------------------------------------|-------------------------|-----------------------------------------------------------------------------|
| CONCAT('Hello', 'World')                      | HelloWorld              | Concatenates two strings (alternative: use `||` operator)                   |
| SUBSTR('HelloWorld', 1, 5)                    | Hello                   | Returns a substring starting at position 1 with length 5                    |
| LENGTH('HelloWorld')                          | 10                      | Returns the length of the string                                            |
| INSTR('HelloWorld', 'W')                      | 6                       | Returns the position of the first occurrence of 'W' (1-based index)         |
| LPAD(salary, 10, '*')                         | ****24000               | Left-pads the value to a total length of 10 using '*' (example: salary=24000)|
| RPAD(salary, 10, '*')                         | 24000*****              | Right-pads the value to a total length of 10 using '*'                       |
| REPLACE('JACK and JUE', 'J', 'BL')            | BLACK and BLUE          | Replaces all occurrences of 'J' with 'BL'                                   |
| TRIM('H' FROM 'HelloWorld')                   | elloWorld               | Removes all leading occurrences of 'H' from the string                       |

## Using the Character-Manipulation Functions

![alt text](../assets/P3/usingchar.png){ width=700 }