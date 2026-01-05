# PL/SQL Basics

## Benefits of PL/SQL

Improved Performance

![alt text](../assets/P8/plsql.png){ width=700 }

## Parts of a PL/SQL block

A PL/SQL block consists of declaration, executable, and exception handling sections.

```sql
set serveroutput on  -- required if we want to see the output
DECLARE                 
  v NUMBER := 0;
BEGIN
  DBMS_OUTPUT.PUT_LINE('It''s ok ...');  -- notice the double quotes
  v := 1/v;
  DBMS_OUTPUT.PUT_LINE('It is not ...');
EXCEPTION
  WHEN ZERO_DIVIDE THEN
    DBMS_OUTPUT.PUT_LINE('Division by zero');
END;
/        -- !!! Always end PL/SQL blocks with a '/' character !!!
```

<pre>
Results:
---------
It's ok ...
Division by zero
</pre>

## Block Types

![alt text](../assets/P8/block.png){ width=700 }

## Lexical Elements

### Delimiters, identifiers, literals, comments

Examples for delimiters:

- '+'   Addition operator
- ':=' Assignment operator
- '<<'  Label delimiter (begin)
- '>>'  Label delimiter (end)
- '!='  Relational operator (not equal)

### Identifiers

Identifiers can denote the following PL/SQL objects:

- Constants, Cursors, Exceptions, Keywords, Labels, Packages, Reserved words, Subprograms, Variables, Types
- Predefined identifiers in STANDARD package, e.g. ZERO_DIVIDE exception

### Literals (numeric, character, string, logical, date)

- integer:       12
- real:          12.0
- char:          'a'
- string:      'abc', ''  (null string, actually NULL)
- logical:     TRUE, FALSE, NULL
- date:        DATE '2011-12-25' 

### Comments: single line and multiline

```sql
-- single line comment

/* multiline
   comment
*/
```

## Variables

### PL/SQL Variables:

- Scalar
- Composite
- Reference
- LOB (large objects)

Non-PL/SQL variables: Bind and host variables

```sql
-- we cannot put a space into delimiters ( := )
BEGIN
  count := count + 1;   -- correct
  count : = count + 1;  -- incorrect
END;
/
```

### DATETIME and INTERVAL literals

```sql
DECLARE
  d1 DATE      := DATE '1998-12-25';
  t1 TIMESTAMP := TIMESTAMP '1997-10-22 13:01:01';
  t2 TIMESTAMP WITH TIME ZONE :=   TIMESTAMP '1997-01-31 09:26:56.66 +02:00';
  
  -- Three years and two months
  -- For greater precision, use the day-to-second interval
  i1 INTERVAL YEAR TO MONTH := INTERVAL '3-2' YEAR TO MONTH;
 
  -- Five days, four hours, three minutes, two and 1/100 seconds
   i2 INTERVAL DAY TO SECOND := INTERVAL '5 04:03:02.01' DAY TO SECOND;
BEGIN
  NULL;
END;
/
```

## Variable Initialization and Keywords

Using:

- Assignment operator (:=)
- DEFAULT keyword
- NOT NULL constraint

## Base Scalar Datatypes

- VARCHAR2 (maximum_length)
- NUMBER [(precision, scale)]
- DATE
- CHAR [(maximum_length)]
- LONG
- LONG RAW
- BOOLEAN
- BINARY_INTEGER
- PLS_INTEGER

### Variable declarations

```sql
DECLARE
  part_number       NUMBER(6);     -- SQL data type
  part_name         VARCHAR2(20);  -- SQL data type
  in_stock          BOOLEAN;       -- PL/SQL-only data type
  part_price        NUMBER(6,2);   -- SQL data type
  part_description  VARCHAR2(50);  -- SQL data type
BEGIN
  NULL;
END;
/
```

### Constant declarations

```sql
DECLARE
  credit_limit     CONSTANT REAL    := 5000.00;  -- SQL data type
  max_days_in_year CONSTANT INTEGER := 366;      -- SQL data type
  urban_legend     CONSTANT BOOLEAN := FALSE;     -- PL/SQL-only data type;
BEGIN
  NULL;
END;
/
```

## Boolean Variables

- Only the values TRUE, FALSE, and NULL can be assigned to a Boolean variable.
- The variables are connected by the logical operators AND, OR, and NOT.
- The variables always yield TRUE, FALSE, or NULL.
- Arithmetic, character, and date expressions can be used to return a Boolean value.

## Composite Datatypes

- PL/SQL TABLES
- PL/SQL RECORDS

### Initialization

```sql
DECLARE
  hours_worked    INTEGER := 40;
  employee_count  INTEGER := 0;
  pi     CONSTANT REAL := 3.14159;
  radius          REAL := 1;
  area            REAL := (pi * radius**2);
BEGIN
  NULL;
END;
/
```

### Default initializations

```sql
DECLARE
  counter INTEGER;  -- initial value is NULL by default
BEGIN
  counter := counter + 1;  -- NULL + 1 is still NULL
  IF counter IS NULL THEN
    DBMS_OUTPUT.PUT_LINE('counter is NULL.');
  END IF;
END;
/
```

<pre>
Results:
---------
counter is NULL.
</pre>

### All variables will be NULL
```sql
DECLARE
  null_string  VARCHAR2(80) := TO_CHAR('');
  address      VARCHAR2(80);
  zip_code     VARCHAR2(80) := SUBSTR(address, 25, 0);
  name         VARCHAR2(80);
  valid        BOOLEAN      := (name != '');
BEGIN
  NULL;
END;
/
```

### %TYPE Attribute
Use the `%TYPE` attribute to declare a variable based on a database column or another variable's type. This ensures type compatibility and adapts to changes in the underlying data structure.

- It inherits the data type but not constraints like `NOT NULL` from a column.
- When inherited from another variable, it also inherits the `NOT NULL` constraint if present.

```sql
-- Inherit from a database column
DECLARE
  v_name  emp.ename%TYPE;
BEGIN
  DBMS_OUTPUT.PUT_LINE('name=' || v_name);
END;
/
```

```sql
-- Inherit from another variable (including NOT NULL constraint)
DECLARE
  name     VARCHAR(25) NOT NULL := 'Smith';
  surname  name%TYPE := 'Jones';
BEGIN
  DBMS_OUTPUT.PUT_LINE('name=' || name);
  DBMS_OUTPUT.PUT_LINE('surname=' || surname);
END;
/
```

<pre>
Results:
---------
name=Smith
surname=Jones
</pre>

### %ROWTYPE Attribute
Use the `%ROWTYPE` attribute to declare a record variable that has the same structure as a table row, view, or cursor.

- It does not inherit column constraints (`NOT NULL`, `CHECK`) or initial values.

```sql
CREATE TABLE employees_temp (
  empid NUMBER(6) NOT NULL PRIMARY KEY,
  deptid NUMBER(6) CONSTRAINT c_employees_temp_deptid CHECK (deptid BETWEEN 100 AND 200),
  deptname VARCHAR2(30) DEFAULT 'Sales' );

DECLARE
  emprec  employees_temp%ROWTYPE;
BEGIN
  emprec.empid := NULL;         -- NOT Null constraint not inherited
  emprec.deptid := 50;          -- Check constraint not inherited
  DBMS_OUTPUT.PUT_LINE ('emprec.deptname: ' || emprec.deptname);  -- Initial value not inherited
END;
/
```

## Scope and Naming
- Identifiers within a PL/SQL unit must be unique.
- You can use labels (`<<label>>`) to qualify block names and resolve ambiguity between local and global variables.

```sql
-- Qualified names
SET SERVEROUTPUT ON
<<echo>>
DECLARE
  x  NUMBER := 5;
  
  PROCEDURE echo IS
    x  NUMBER := 0;
  BEGIN
    DBMS_OUTPUT.PUT_LINE('x = ' || x);             -- output: x = 0
    DBMS_OUTPUT.PUT_LINE('echo.x = ' || echo.x);   -- output: echo.x = 0 (variable of the procedure)
  END;
 
BEGIN
  echo;
END;
/
```

## Operators and Expressions
PL/SQL includes logical, arithmetic, relational, and concatenation operators.

### Logical Operators and NULL
- `NULL` values in logical expressions can lead to a three-valued logic result: `TRUE`, `FALSE`, or `NULL`.
- Two `NULL`s are not equal to each other.

```sql
set serveroutput on
DECLARE
  a NUMBER := NULL;
  b NUMBER := NULL;
BEGIN
  IF a = b THEN
    DBMS_OUTPUT.PUT_LINE('a = b');
  ELSIF a != b THEN
    DBMS_OUTPUT.PUT_LINE('a != b');
  ELSE
    DBMS_OUTPUT.PUT_LINE('Can''t tell if two NULLs are equal');
  END IF;
END;
/
```
<pre>
Results:
---------
Can't tell if two NULLs are equal
</pre>

### Concatenation
The concatenation operator (`||`) ignores `NULL` values.

```sql
set serveroutput on
BEGIN
  DBMS_OUTPUT.PUT_LINE ('apple' || NULL || NULL || 'sauce');  -- output: applesauce
END;
/
```

### Short-Circuit Evaluation
PL/SQL uses short-circuit evaluation. In a complex logical expression (`OR`, `AND`), subsequent parts may not be evaluated if the result is already determined.

```sql
DECLARE
  on_hand  INTEGER := 0;
  on_order INTEGER := 100;
BEGIN 
  IF (on_hand = 0) OR ((on_order / on_hand) < 5) THEN   -- Will not cause ZERO_DIVIDE exception
    DBMS_OUTPUT.PUT_LINE('On hand quantity is zero.');
  END IF;
END;
/
```

## Subtypes
Subtypes allow you to create constrained or unconstrained aliases for existing data types, improving readability and enforcing domain integrity.

- **Unconstrained**: `SUBTYPE CHARACTER IS CHAR;`
- **Constrained**: `SUBTYPE INTEGER IS NUMBER(38,0);`

Oracle's `STANDARD` package predefines many useful subtypes like `NATURAL`, `INTEGER`, and `BINARY_INTEGER`.

```sql
-- User-defined subtypes
DECLARE
  SUBTYPE BirthDate IS DATE NOT NULL;
  SUBTYPE Counter IS NATURAL;
  SUBTYPE pinteger IS PLS_INTEGER RANGE -9..9;  
  TYPE NameList IS TABLE OF VARCHAR2(10);
  SUBTYPE DutyRoster IS NameList;
  TYPE TimeRec IS RECORD (minutes INTEGER, hours INTEGER);
  SUBTYPE FinishTime IS TimeRec;
  SUBTYPE ID_Num IS employees.employee_id%TYPE;
BEGIN
  NULL;
END;
/
```

## Data Type Conversion
PL/SQL can perform implicit conversions (e.g., `CHAR` to `NUMBER`), but it is best practice to use explicit conversion functions (`TO_CHAR`, `TO_DATE`, `TO_NUMBER`) to ensure clarity and prevent unexpected behavior.

## Conditional Control: CASE Expressions
The `CASE` expression evaluates a list of conditions and returns one of multiple possible result expressions. It is not a statement.

```sql
-- Simple CASE expression
set serveroutput on
DECLARE
  grade CHAR(1) := 'B';
  appraisal VARCHAR2(20);
BEGIN
  appraisal :=
    CASE grade
      WHEN 'A' THEN 'Excellent'
      WHEN 'B' THEN 'Very Good'
      ELSE 'No such grade'
    END;
    DBMS_OUTPUT.PUT_LINE ('Grade ' || grade || ' is ' || appraisal);
END;
/
```

```sql
-- Searched CASE expression (handles NULL)
set serveroutput on
DECLARE
  grade CHAR(1); -- NULL by default
  appraisal VARCHAR2(20);
BEGIN
  appraisal :=
    CASE
      WHEN grade IS NULL THEN 'No grade assigned'
      WHEN grade = 'A' THEN 'Excellent'
      ELSE 'No such grade'
    END;
    DBMS_OUTPUT.PUT_LINE ('Grade ' || grade || ' is: ' || appraisal);
END;
/
```

## SQL in PL/SQL
You can embed SQL DML statements directly into your PL/SQL blocks.

### Retrieving Data
Use `SELECT ... INTO` to fetch data from a single row into variables.

```sql
DECLARE
  v_orderdate ord.orderdate%TYPE;
  v_shipdate ord.shipdate%TYPE;
BEGIN
  SELECT orderdate, shipdate
  INTO v_orderdate, v_shipdate
  FROM ord
  WHERE id = 620;
END;
/
```

### Updating Data
Use the `UPDATE` statement as you would in standard SQL.

```sql
DECLARE
  v_sal_increase emp.sal%TYPE := 2000;
BEGIN
  UPDATE emp
  SET sal = sal + v_sal_increase
  WHERE job = 'ANALYST';
END;
/
```

## Control Statements

### Conditional (IF)
```sql
IF condition_1 THEN
  statements_1
ELSIF condition_2 THEN
  statements_2
ELSE
  else_statements
END IF;
```

### Conditional (CASE Statement)
```sql
CASE selector 
WHEN selector_value_1 THEN statements_1
WHEN selector_value_2 THEN statements_2
ELSE else_statements
END CASE;
```

### Basic LOOP
```sql
LOOP
  statements;
  EXIT WHEN condition;
END LOOP;
```

### WHILE LOOP
```sql
WHILE condition LOOP
  statements;
END LOOP;
```

### FOR LOOP
The loop index is implicitly declared and local to the loop.
```sql
FOR index IN [REVERSE] lower_bound..upper_bound LOOP
  statements;
END LOOP;
```

### GOTO and NULL Statements
- `GOTO label;` transfers control to a labeled block or statement.
- `NULL;` is a placeholder statement that does nothing, useful for improving readability or in places where a statement is syntactically required.

## Bind Variables
Bind variables are created in the host environment (like SQL*Plus) and can be referenced and modified by PL/SQL. In PL/SQL, they are prefixed with a colon (`:`).

```sql
-- In SQL*Plus
VARIABLE g_salary NUMBER

BEGIN
  :g_salary := 1000;
END;
/

PRINT g_salary
```

## Server Output
To see output from `DBMS_OUTPUT.PUT_LINE`, you must enable it in your client.

```sql
SET SERVEROUTPUT ON
BEGIN
  DBMS_OUTPUT.PUT_LINE('Hello World!');
END;
/
```
The `DBMS_OUTPUT` package writes to a server-side buffer, which the client tool then displays after the block finishes executing.
