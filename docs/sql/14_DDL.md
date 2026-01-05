# DDL (Data Definition Language)

## Database Objects

| Object    | Description                                      |
|-----------|--------------------------------------------------|
| Table     | Basic unit of storage; composed of rows and columns |
| View      | Logically represents subsets of data from one or more tables |
| Sequence  | Generates numeric values (e.g., for auto-incrementing IDs) |
| Index     | Improves the performance of some queries          |
| Synonym   | Gives alternative names to objects                |

## Naming Rules for Database Objects

- **Must begin with a letter** (A–Z or a–z).
- **Must be 1–30 characters long** (Oracle 12.2 and later: up to 128 bytes for most objects).
- **Must contain only**:
    - Alphanumeric characters (A–Z, a–z, 0–9)
    - Underscore (`_`)
    - Dollar sign (`$`)
    - Pound sign (`#`)
- **Must not duplicate** the name of another object owned by the same user.
- **Must not be** an Oracle server **reserved word** (e.g., SELECT, TABLE, FROM).

## CREATE TABLE Statement

You must have:

- **CREATE TABLE** privilege
- A **storage area** (quota in a tablespace)

```sql
CREATE TABLE [schema.]table
    (column datatype [DEFAULT expr] [, ...]);
```

What You Specify
- Table name
- Column name, column data type, and column size

### Referencing Another User's Tables

- Tables belonging to other users are **not** in the user's schema.
- You should use the **owner's name** as a **prefix** to those tables.

This is called **schema-qualified** table reference.

```sql
SELECT * FROM schema_name.table_name;
```

### DEFAULT Option

- Specify a **default value** for a column during an **INSERT**.
- If no value is provided for the column in the INSERT, the default is used automatically.

```sql
column_name datatype DEFAULT expression
```

- Literal values, expressions, or SQL functions are legal values.
- Another column's name or a pseudocolumn are illegal values.
- The default data type must match the column data type (Oracle performs implicit conversion if possible).

```sql
CREATE TABLE hire_dates
    (id          NUMBER(8),
     hire_date   DATE DEFAULT SYSDATE);
```

### Creating Tables

Create the Table

```sql
CREATE TABLE dept
    (deptno      NUMBER(2),
     dname       VARCHAR2(14),
     loc         VARCHAR2(13),
     create_date DATE DEFAULT SYSDATE);
```

Confirm Table Creation

```sql
DESCRIBE dept; -- not SQL Statement
```

![alt text](../assets/P5/desc.png){ width=700 }

## Data Types

Oracle provides a variety of data types to store different kinds of information efficiently.

| Data Type                  | Description                                      |
|----------------------------|--------------------------------------------------|
| VARCHAR2(size)             | Variable-length character data (up to 4000 bytes; size in bytes or characters) |
| CHAR(size)                 | Fixed-length character data (padded with spaces; up to 2000 bytes) |
| NUMBER(p, s)               | Variable-length numeric data (p = precision, s = scale) |
| DATE                       | Date and time values (century to seconds)        |
| LONG                       | Variable-length character data (up to 2 GB) — deprecated |
| CLOB                       | Character large object (up to 4 GB)              |
| RAW and LONG RAW           | Raw binary data (deprecated)                     |
| BLOB                       | Binary data stored in an external file (up to 4 GB)                  |
| BFILE                      | Binary data stored in an external file (up to 4 GB) |
| ROWID                      | A base-64 number system representing the unique address of a row in its table |

### Date and Interval Types

| Data Type                  | Description                                      |
|----------------------------|--------------------------------------------------|
| TIMESTAMP                  | Date with fractional seconds                     |
| INTERVAL YEAR TO MONTH     | Stored as an interval of years and months        |
| INTERVAL DAY TO SECOND     | Stored as an interval of days, hours, minutes, and seconds |

### Datetime Data Types

- The **TIMESTAMP** data type is an extension of the **DATE** data type.
- It stores the year, month, and day of the **DATE** data type plus **hour, minute, and second** values as well as the **fractional second** value.
- You can optionally specify the **time zone**.

| Data Type                                      | Description                                                                 |
|------------------------------------------------|-----------------------------------------------------------------------------|
| TIMESTAMP [(fractional_seconds_precision)]     | Stores date, time, and fractional seconds (precision 0–9, default 6)         |
| TIMESTAMP [(fractional_seconds_precision)] WITH TIME ZONE | Includes time zone offset (e.g., +05:30) or time zone region name          |
| TIMESTAMP [(fractional_seconds_precision)] WITH LOCAL TIME ZONE | Stores in database time zone, automatically converts to user’s local time zone on retrieval |

### Interval Data Types

Oracle provides **interval** data types to store periods of time.

#### INTERVAL YEAR TO MONTH

- Stores a period of time using the **YEAR** and **MONTH** datetime fields.

```sql
INTERVAL YEAR [(year_precision)] TO MONTH
```

#### INTERVAL DAY TO SECOND

- Stores a period of time in terms of days, hours, minutes, and seconds.

```sql
INTERVAL DAY [(day_precision)] TO SECOND [(fractional_seconds_precision)]
```

## Including Constraints

- **Constraints** enforce rules at the **table level**.
- Constraints **prevent the deletion** of a table if there are dependencies.
- They ensure **data integrity** by restricting what data can be inserted, updated, or deleted.

### Valid Constraint Types

| Constraint Type   | Description                                                                 | Example Use Case                          |
|-------------------|-----------------------------------------------------------------------------|-------------------------------------------|
| NOT NULL          | Column cannot contain NULL values                                           | email, last_name                          |
| UNIQUE            | All values in the column (or combination) must be unique                    | employee_id (if not PK), email            |
| PRIMARY KEY       | Combines NOT NULL + UNIQUE; uniquely identifies each row                   | employee_id, dept_id                      |
| FOREIGN KEY       | Enforces referential integrity — value must exist in referenced table/key  | department_id references departments(dept_id) |
| CHECK             | Ensures column value satisfies a specific condition                        | salary > 0, job_id IN ('SA_REP','IT_PROG') |

### Constraint Guidelines

- You can **name a constraint**, or the Oracle server generates a name by using the **SYS_Cn** format.
- Create a constraint at either of the following times:
    - At the **same time** as the table is created
    - **After** the table has been created
- Define a constraint at the **column** or **table** level.
- View a constraint in the **data dictionary**.

### Defining Constraints

```sql
CREATE TABLE [schema.]table
(
    column datatype [DEFAULT expr]
        [column_constraint],
    ...
    [table_constraint] [, ...]
);
```

#### Column-Level Constraint

```sql
column [CONSTRAINT constraint_name] constraint_type
```

```sql
CREATE TABLE employees (
    employee_id NUMBER(6)
        CONSTRAINT emp_emp_id_pk PRIMARY KEY,
    first_name  VARCHAR2(20),
    ...
);
```

#### Table-Level Constraint

```sql
[CONSTRAINT constraint_name] constraint_type (column [, ...])
```

```sql
CREATE TABLE employees (
    employee_id NUMBER(6),
    first_name  VARCHAR2(20),
    ...
    job_id      VARCHAR2(10) NOT NULL,
    CONSTRAINT emp_emp_id_pk PRIMARY KEY (employee_id)
);
```

### NOT NULL Constraint

- Ensures that null values are not permitted for the column.

![alt text](../assets/P5/notnull.png){ width=700 }

### UNIQUE Constraint

![alt text](../assets/P5/unique.png){ width=700 }

### PRIMARY KEY Constraint

![alt text](../assets/P5/primary.png){ width=700 }

### FOREIGN KEY Constraint

![alt text](../assets/P5/foreign.png){ width=700 }

#### FOREIGN KEY Constraint: Keywords

- **FOREIGN KEY**: Defines the column in the child table at the table-constraint level
- **REFERENCES**: Identifies the table and column in the parent table
- **ON DELETE CASCADE**: Deletes the dependent rows in the child table when a row in the parent table is deleted
- **ON DELETE SET NULL**: Converts dependent foreign key values to null

### CHECK Constraint

- Defines a condition that each row must satisfy
- The following expressions are not allowed:
    - References to CURRVAL, NEXTVAL, LEVEL, and ROWNUM pseudocolumns
    - Calls to SYSDATE, UID, USER, and USERENV functions
    - Queries that refer to other values in other rows

```sql
..., salary NUMBER(2)
    CONSTRAINT emp_salary_min
    CHECK (salary > 0), ...
```

### Violating Constraints 1

```sql
UPDATE employees
SET    department_id = 55
WHERE  department_id = 110;
```

```sql
UPDATE employees
*
ERROR at line 1:
ORA-02291: integrity constraint (HR.EMP_DEPT_FK)
violated - parent key not found
```

Department 55 does not exist.

### Violating Constraints 2

- You cannot delete a row that contains a **primary key** that is **used as a foreign key** in

```sql
DELETE FROM departments
WHERE  department_id = 60;
```

```sql
DELETE FROM departments
*
ERROR at line 1:
ORA-02292: integrity constraint (HR.EMP_DEPT_FK)
violated - child record found
```

## Creating a Table by Using a Subquery

- Create a table and insert rows by combining the `CREATE TABLE` statement and the `AS subquery` option.

```sql
CREATE TABLE table
    [(column, column...)]
AS subquery;
```

- Match the number of specified columns to the number of subquery columns.
- Define columns with column names and default values.

```sql
CREATE TABLE dept80
AS
    SELECT  employee_id, last_name,
            salary*12 ANNSAL,
            hire_date
    FROM    employees
    WHERE   department_id = 80;
```

## ALTER TABLE Statement

- Use the ALTER TABLE statement to:
    - Add a new column
    - Modify an existing column
    - Define a default value for the new column
    - Drop a column

## Dropping a Table

- All data and structure in the table are deleted.
- Any pending transactions are committed.
- All indexes are dropped.
- All constraints are dropped.
- You cannot roll back the DROP TABLE statement.

```sql
DROP TABLE dept80;
```
