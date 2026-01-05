# Exception Handling

![alt text](../assets/P10/exception.png){ width=700 }

## Handling Exceptions

- **Trap the exception**: Exception is raised and trapped within DECLARE, BEGIN, EXCEPTION, END block.
- **Propagate the exception**: Exception is raised, not trapped, and propagates to calling environment.

## Exception Types

- Predefined Oracle Server: Implicitly raised
- Non-predefined Oracle Server: Implicitly raised
- User-defined: Explicitly raised

## Trapping Exceptions Guidelines

- WHEN OTHERS is the last clause.
- EXCEPTION keyword starts exception-handling section.
- Several exception handlers are allowed.
- Only one handler is processed before leaving the block.
- Reference the standard name in the exception-handling routine.

### Sample Predefined Exceptions

- NO_DATA_FOUND
- TOO_MANY_ROWS
- INVALID_CURSOR
- ZERO_DIVIDE
- DUP_VAL_ON_INDEX

### Example: Exception Handling

```sql
set serveroutput on
DECLARE
  except1 EXCEPTION;
  PRAGMA exception_init(except1, -20000);
BEGIN
  raise_application_error('-20001', 'except2');   -- comment this line
  RAISE except1;                                  -- then comment this line too 
  DECLARE
    V NUMBER := 1/0;                              -- finally change it to 1/1
  BEGIN
    V := 1/0;
  EXCEPTION 
    WHEN OTHERS THEN dbms_output.put_line('inner block');
  END;
EXCEPTION 
  WHEN OTHERS THEN dbms_output.put_line(SQLCODE||' ~~~ '||sqlerrm);
END;
/
```

<pre>
Results:
----------
first output:  -20001 ~~~ ORA-20001: except2
second output: -20000 ~~~ ORA-20000:
third output:  -1476 ~~~ ORA-01476: division by zero
fourth output: inner block
</pre>

## Trapping Non-Predefined Oracle Server Errors

- Declare: Name the exception (Declarative section)
- Associate: Code the PRAGMA EXCEPTION_INIT
- Reference: Handle the raised exception (Exception-handling section)

```sql
e_emps_remaining EXCEPTION;
PRAGMA EXCEPTION_INIT (e_emps_remaining, -2292)
```

## Trapping User-Defined Exceptions

- Declare: Name the exception (Declarative section)
- Raise: Explicitly raise the exception by using the RAISE statement (Executable section)
- Reference: Handle the raised exception (Exception-handling section)

```sql
e_invalid_product EXCEPTION;
RAISE e_invalid_product;
```

## Functions for Trapping Exceptions

- SQLCODE: Returns the numeric value for the error code
- SQLERRM: Returns the message associated with the error number

### Example: Raising an error from a subprogram

```sql
DECLARE
  v1 NUMBER :=0;
  FUNCTION f1 RETURN NUMBER IS
  except1 EXCEPTION;
  BEGIN
    raise except1;                                    -- comment this line
    raise_application_error('-20000', 'exception1');  -- finally comment this too
    RETURN 10;
  EXCEPTION 
    WHEN except1 THEN RETURN 20;
    WHEN OTHERS THEN RETURN 30;                       -- then comment this too
  END f1;
BEGIN
  v1 := f1; dbms_output.put_line(v1);
EXCEPTION 
  WHEN OTHERS THEN dbms_output.put_line(SQLCODE||' ~~~ '||sqlerrm);
END;
/
```

<pre>
Results:
------------
first output:   20
second output:  30
third output:   -20000 ~~~ ORA-20000: exception1
fourth output:  10
</pre>

## Calling Environments

- SQL*Plus: Displays error number and message to screen
- Sql Developer: Displays error number and message to screen
- Oracle Developer Forms: Accesses error number and message in a trigger by means of the ERROR_CODE and ERROR_TEXT packaged functions
- Precompiler application: Accesses exception number through the SQLCA data structure
- An enclosing PL/SQL block: Traps exception in exception-handling routine of enclosing block

## RAISE_APPLICATION_ERROR Procedure

```sql
raise_application_error (error_number, message[, {TRUE | FALSE}]);
```

- A procedure that lets you issue user-defined error messages from stored subprograms
- Called only from an executing stored subprogram

### Example: Predefined exceptions

```sql
SELECT text FROM all_source WHERE type = 'PACKAGE'
AND name = 'STANDARD' AND lower(text) LIKE '%exception_init%';
```

Put comments before some statements (or delete comments) to test other exceptions.

```sql
SET SERVEROUTPUT ON
BEGIN                       -- 3 nested blocks
 DECLARE
  v1 emp.sal%TYPE;
  v2 emp.comm%TYPE;
  v3 INTEGER := 0;
 BEGIN
  v3 := 1/v3;                  -- comment this second
  BEGIN
   SELECT sal, comm INTO v1, v2 FROM emp WHERE ename LIKE 'S%';  -- then change it to  'X%'
  EXCEPTION
   WHEN too_many_rows THEN 
    BEGIN
     v1 := 1; v2 := 2;
    END;
  END;
  dbms_output.put_line(to_char(v1)||' -- '|| nvl(to_char(v2), 'null'));
 
 EXCEPTION
  WHEN zero_divide THEN dbms_output.put_line('zero divide');      -- comment this first       
  WHEN too_many_rows THEN dbms_output.put_line('too many rows');
 END;
 dbms_output.put_line('main program');
EXCEPTION
  WHEN OTHERS THEN dbms_output.put_line(SQLCODE || ' -- ' || sqlerrm);
END;
/
```

<pre>
Results:
-----------
first output:   zero divide
                main program
second output:  -1476 -- ORA-01476: division by zero
third output:   1 -- 2        too many rows handled
                main program
fourth output:  100 -- ORA-01403: No Data found
</pre>

### Example: Printing error codes

```sql
BEGIN
 dbms_output.put_line(sqlerrm(-6502));
END;
/
```

### Comprehensive Exception Handling Example

```sql
set serveroutput on
DECLARE 
  v_nev  VARCHAR2(20);
  v_szam NUMBER := 0;
  CURSOR emp_cur IS SELECT ename FROM emp;

  error1 EXCEPTION;
  pragma EXCEPTION_INIT(error1, -20001);
  error2 EXCEPTION;
  pragma EXCEPTION_INIT(error2, -20002);

  PROCEDURE err_proc(v NUMBER) IS
  BEGIN
    IF MOD(v, 2) = 0 THEN
      RAISE_APPLICATION_ERROR('-20001', 'error1');
    ELSE
      RAISE_APPLICATION_ERROR('-20002', 'error2');
    END IF;
  END;

BEGIN
  -- err_proc(1);                                     -- Raises error2
  -- err_proc(2);                                     -- Raises error1
  -- v_szam := 1/v_szam;                              -- Raises zero_divide
  -- SELECT ename INTO v_nev FROM emp WHERE empno < 0;   -- Raises no_data_found
  SELECT ename INTO v_nev FROM emp WHERE empno > 0;   -- Raises too_many_rows
EXCEPTION
  WHEN error1 THEN
    dbms_output.put_line('error1 occured');
  WHEN error2 THEN
    dbms_output.put_line('error2 occured');
  WHEN zero_divide THEN
    dbms_output.put_line('zero divide error');
  WHEN no_data_found THEN
    dbms_output.put_line('No Data Found error');
  WHEN too_many_rows THEN
    dbms_output.put_line('Too many rows error');
  WHEN OTHERS THEN
    dbms_output.put_line('something else ...');
END;
/
```

# Subprograms: Functions and Procedures

Subprograms are named PL/SQL blocks that can be called with parameters.

- **Procedures**: Perform an action.
- **Functions**: Compute and return a value.

## Procedural Parameter Modes

![alt text](../assets/P10/parameters.png){ width=700 }

| Mode    | Description                                     |
|---------|-------------------------------------------------|
| `IN`      | (Default) Read-only value passed into the subprogram. |
| `OUT`     | Writable variable passed back to the calling environment. |
| `IN OUT`  | Readable and writable variable passed in and out. |

![alt text](../assets/P10/modes.png){ width=700 }

### `IN` Parameter Example
```sql
CREATE OR REPLACE PROCEDURE raise_salary (v_id IN emp.empno%TYPE) IS
BEGIN
  UPDATE emp SET sal = sal * 1.10 WHERE empno = v_id;
END raise_salary;
/
EXECUTE raise_salary(7369);
```

### `OUT` Parameter Example
```sql
CREATE OR REPLACE PROCEDURE query_emp (
    v_id IN emp.empno%TYPE,
    v_name OUT emp.ename%TYPE,
    v_salary OUT emp.sal%TYPE
) IS
BEGIN
  SELECT ename, sal INTO v_name, v_salary FROM emp WHERE empno = v_id;
END query_emp;
/
```

### `IN OUT` Parameter Example
```sql
CREATE OR REPLACE PROCEDURE format_phone (v_phone_no IN OUT VARCHAR2) IS
BEGIN
   v_phone_no := '(' || SUBSTR(v_phone_no,1,3) || ')'
                     || SUBSTR(v_phone_no,4,3) || '-'
                     || SUBSTR(v_phone_no,7);
END format_phone;
/
```

## Local vs. Stored Subprograms
- **Local** subprograms are declared within a PL/SQL block and exist only for that block's execution.
- **Stored** subprograms are created in the database schema and can be called from SQL or other PL/SQL blocks.

```sql
-- Stored function callable from SQL
CREATE OR REPLACE FUNCTION func_plus_2(num number) RETURN number IS
BEGIN
  RETURN(num + 2);
END;
/
SELECT func_plus_2(1000) FROM dual;

-- Stored procedure
CREATE OR REPLACE PROCEDURE proc_plus_2(num number) IS
BEGIN
  dbms_output.put_line(num + 2);
END;
/
EXECUTE proc_plus_2(2000);
```

## Advanced Subprogram Topics

### Overloading
You can create multiple subprograms with the same name, provided their parameter lists differ in number, order, or data type family.

```sql
DECLARE
  PROCEDURE proc1(p IN NUMBER) IS BEGIN DBMS_OUTPUT.PUT_LINE('number param'); END;
  PROCEDURE proc1(p IN VARCHAR2) IS BEGIN DBMS_OUTPUT.PUT_LINE('varchar2 param'); END;
BEGIN
  proc1(100);
  proc1('100');
END;
/
```

### Forward Declaration
A forward declaration allows you to declare a subprogram before it is fully defined, which is necessary for mutually recursive subprograms.

```sql
DECLARE
  PROCEDURE proc2(p IN NUMBER); -- Forward declaration

  PROCEDURE proc1(p IN NUMBER) IS
  BEGIN
    IF p < 10 THEN proc2(p+1); END IF;
  END;

  PROCEDURE proc2(p IN NUMBER) IS
  BEGIN
    IF p < 10 THEN proc1(p*2); END IF;
  END;
BEGIN
  proc1(1);
END;
/
```

### Restrictions on Functions Called from SQL
- Must be a stored function.
- Must not be a `GROUP` function.
- Can only take `IN` parameters.
- Parameter and return types must be standard SQL types (not PL/SQL-specific types like `BOOLEAN` or `RECORD`).
- Cannot contain DML statements (`INSERT`, `UPDATE`, `DELETE`).

## Procedure or Function?

![alt text](../assets/P10/porf.png){ width=700 }

| Procedure                     | Function                        |
|-------------------------------|---------------------------------|
| Executes as a PL/SQL statement | Invoked as part of an expression |
| Does not have a `RETURN` datatype | Must have a `RETURN` datatype  |
| Can return multiple values via `OUT` parameters | Must return a single value             |
