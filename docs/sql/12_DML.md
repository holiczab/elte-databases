# DML (Data Manipulation Language)

## DML Statements in SQL

- A **DML statement** is executed when you:
    - Add new rows to a table
    - Modify existing rows in a table
    - Remove existing rows from a table

- A **transaction** consists of a collection of DML statements that form a **logical unit of work**.

## Adding a New Row to a Table

![alt text](../assets/P5/newrow.png){ width=700 }

### INSERT Statement Syntax

- Add new rows to a table by using the **INSERT** statement.

```sql
INSERT INTO table [(column [, column...])]
VALUES (value [, value...]);
```

With this syntax, only one row is inserted at a time.

### Inserting New Rows

- Insert a new row containing values for each column.
- List values in the **default order** of the columns in the table.
- Optionally, list the columns explicitly in the `INSERT` clause.
- Enclose character and date values in **single quotation marks**.

```sql
INSERT INTO departments (department_id, department_name, manager_id, location_id)
VALUES (70, 'Public Relations', 100, 1700);
```

### Inserting Rows with NULL Values

#### Implicit Method

- Omit the column from the column list.
- The omitted column(s) will receive **NULL** (or default value if defined).

```sql
INSERT INTO departments (department_id, department_name)
VALUES (30, 'Purchasing');
```

#### Explicit Method

- Specify the NULL keyword in the VALUES clause.

```sql
INSERT INTO departments
VALUES (100, 'Finance', NULL, NULL);
```

### Copying Rows from Another Table

- Write your **INSERT** statement with a **subquery**.
- **Do not use** the `VALUES` clause.
- Match the number of columns in the `INSERT` clause to those in the subquery.

```sql
INSERT INTO target_table [(column1, column2, ...)]
SELECT column1, column2, ...
FROM source_table
[WHERE condition];
```

## Changing Data in a Table

![alt text](../assets/P5/change.png){ width=700 }

### UPDATE Statement Syntax in Oracle SQL

- Modify existing rows with the **UPDATE** statement.

```sql
UPDATE table
SET column = value [, column = value, ...]
[WHERE condition];
```

- Update more than one row at a time (if required).

- Specific row or rows are modified if you specify the WHERE clause.

```sql
UPDATE employees
SET department_id = 70
WHERE employee_id = 113;
```

- All rows in the table are modified if you omit the WHERE clause.

```sql
UPDATE copy_emp
SET department_id = 110;
```

### Updating Two Columns with a Subquery

Update employee 114's **job** and **salary** to match that of employee 205.

```sql
UPDATE employees
SET job_id = (SELECT job_id
              FROM employees
              WHERE employee_id = 205),
    salary = (SELECT salary
              FROM employees
              WHERE employee_id = 205)
WHERE employee_id = 114;
```

## Removing a Row from a Table

![alt text](../assets/P5/remove.png){ width=700 }

### DELETE Statement in Oracle SQL

- You can remove existing rows from a table by using the **DELETE** statement.

```sql
DELETE [FROM] table
[WHERE condition];
```

- Specific rows are deleted if you specify the WHERE clause.

```sql
DELETE FROM departments
WHERE department_name = 'Finance';
```

- All rows in the table are deleted if you omit the WHERE clause.

```sql
DELETE FROM copy_emp;
```

### Deleting Rows Based on Another Table

- Use **subqueries** in `DELETE` statements to remove rows from a table based on values from another table.

```sql
DELETE FROM employees
WHERE department_id = 
    (SELECT department_id
     FROM departments
     WHERE department_name LIKE '%Public%');
```

## TRUNCATE Statement in Oracle SQL

- Removes **all rows** from a table, leaving the table **empty** and the **table structure** intact.
- Is a **data definition language (DDL)** statement rather than a DML statement.
- **Cannot easily be undone** (implicit commit — no ROLLBACK possible).

```sql
TRUNCATE TABLE table_name;
```

Example:

```sql
TRUNCATE TABLE copy_emp;
```

## Using a Subquery in an INSERT Statement

```sql
INSERT INTO 
    (SELECT employee_id, last_name, email, hire_date, job_id, salary, department_id
     FROM employees
     WHERE department_id = 50)
VALUES (99999, 'Taylor', 'DTAYLOR', 
       TO_DATE('07-Jun-99', 'DD-MON-RR'), 
       'ST_CLERK', 5000, 50);
```

Verify the results:

![alt text](../assets/P5/verify.png){ width=700 }
