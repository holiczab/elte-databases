# Cursors

Every SQL statement executed by the Oracle Server has an individual cursor associated with it:

- Implicit cursors: Declared for all DML and PL/SQL SELECT statements
- Explicit cursors: Declared and named by the programmer

## Controlling Explicit Cursors

- DECLARE: Create a named SQL area
- OPEN: Identify the active set
- FETCH: Load the current row into variables
- EMPTY?: Test for existing rows, return to FETCH if rows found
- CLOSE: Release the active set

### Example: Basic Cursor Usage

```sql
set serveroutput on
DECLARE 
  CURSOR curs1 IS SELECT deptno, ename FROM nikovits.emp WHERE deptno = 10;
  rec curs1%ROWTYPE;
BEGIN
  OPEN curs1;
  LOOP
    FETCH curs1 INTO rec;
    EXIT WHEN curs1%NOTFOUND;
    dbms_output.put_line(to_char(rec.deptno)||' - '||rec.ename);
  END LOOP;
  CLOSE curs1;
END;
/
```

<pre>
Results:
-----------
10 - MILLER
10 - CLARK
10 - KING
</pre>

## Cursor Lifecycle

### Opening the Cursor

- Open the cursor to execute the query and identify the active set.
- If the query returns no rows, no exception is raised.
- Use cursor attributes to test the outcome after a fetch.

### Fetching Data from the Cursor

- Retrieve the current row values into output variables.
- Include the same number of variables.
- Match each variable to correspond to the columns positionally.
- Test to see if the cursor contains rows.

### Declaring the Cursor

- Do not include the INTO clause in the cursor declaration.
- If processing rows in a specific sequence is required, use the ORDER BY clause in the query.

### Closing the Cursor

- Close the cursor after completing the processing of the rows.
- Reopen the cursor, if required.
- Do not attempt to fetch data from a cursor once it has been closed.

## Explicit Cursor Attributes

Obtain status information about a cursor:

- %ISOPEN (Boolean): Evaluates to TRUE if the cursor is open
- %NOTFOUND (Boolean): Evaluates to TRUE if the most recent fetch does not return a row
- %FOUND (Boolean): Evaluates to TRUE if the most recent fetch returns a row; complement of %NOTFOUND
- %ROWCOUNT (Number): Evaluates to the total number of rows returned so far

### Example: Implicit cursor (for a DELETE statement)

```sql
set serveroutput on
DECLARE
  v_rows_deleted VARCHAR2(30);
  v_job emp.job%TYPE := 'SALESMAN';
BEGIN
  DELETE FROM emp WHERE job = v_job;
  v_rows_deleted := (SQL%ROWCOUNT ||' row(s) deleted.');
  DBMS_OUTPUT.PUT_LINE (v_rows_deleted);
  ROLLBACK;  -- rollback the transaction, we don't really want to delete
END;
/
```

<pre>
Results:
-----------
5 row(s) deleted.
</pre>

## Cursors and Records

Process the rows of the active set conveniently by fetching values into a PL/SQL RECORD.

## Cursor FOR Loop

- The cursor FOR loop is a shortcut to process explicit cursors.
- Implicit open, fetch, and close occur.
- The record is implicitly declared.

## Cursor Parameters

- Pass parameter values to a cursor when the cursor is opened and the query is executed.
- Open an explicit cursor several times with a different active set each time.

### Example: Explicit cursors, 3 forms of cursor usage and cursors with parameter

```sql
set serveroutput on
DECLARE 
  CURSOR curs1(p_deptno NUMBER DEFAULT 10) IS SELECT ename, sal FROM emp WHERE deptno = p_deptno;
  CURSOR curs2(p_deptno NUMBER) IS SELECT ename, sal from emp where deptno = p_deptno;
  rec curs1%ROWTYPE;
BEGIN
  OPEN curs1();            -- default parameter
  LOOP
    FETCH curs1 INTO rec;
    EXIT WHEN curs1%NOTFOUND;
    dbms_output.put_line('curs1: '||rec.ename||' - '||to_char(rec.sal));
  END LOOP;
  CLOSE curs1;

  FOR rec IN curs2(20) LOOP   -- parameter
    dbms_output.put_line('curs2: '||rec.ename||' - '||to_char(rec.sal));
  END LOOP;

  FOR rec IN (SELECT ename, sal FROM emp WHERE deptno=30) LOOP
    dbms_output.put_line('curs3: '||rec.ename||' - '||to_char(rec.sal));
  END LOOP;
END;
/
```

<pre>
Results:
-----------
curs1: CLARK - 2450
curs1: KING - 5000
curs1: MILLER - 1300
curs2: SMITH - 800
curs2: JONES - 2975
curs2: SCOTT - 3000
curs2: ADAMS - 1100
curs2: FORD - 3000
curs3: ALLEN - 1600
curs3: WARD - 1250
curs3: MARTIN - 1250
curs3: BLAKE - 4250
curs3: TURNER - 1500
curs3: JAMES - 950
</pre>

### Using a CURSOR with a WITH clause
You can define a cursor based on a query that uses the `WITH` clause (Common Table Expression).

```sql
DECLARE 
  CURSOR curs1 IS 
  WITH
  tmp1 AS (
    SELECT deptno, round(AVG(sal)) dept_avg FROM emp
    GROUP BY deptno),
  tmp2 AS (
    SELECT round(AVG(sal)) gen_avg FROM emp)
  SELECT dname, dept_avg, gen_avg, dept_avg-gen_avg diff
  FROM tmp1, tmp2, dept WHERE tmp1.deptno = dept.deptno;
  rec curs1%ROWTYPE;
BEGIN
  OPEN curs1;
  LOOP
    FETCH curs1 INTO rec;
    EXIT WHEN curs1%NOTFOUND;
    dbms_output.put_line(rec.dname||' | '||rec.dept_avg||' | '||rec.diff);
  END LOOP;
  CLOSE curs1;
END;
/
```

## Explicit Locking with Cursors
Explicit locking lets you deny access to rows for the duration of a transaction. This is useful to prevent other sessions from modifying rows that you are about to update or delete.

- Use the `FOR UPDATE` clause in the cursor query to lock the rows.
- Use the `WHERE CURRENT OF` clause in an `UPDATE` or `DELETE` statement to reference the current row being processed by the cursor.

```sql
-- Update with a cursor using WHERE CURRENT OF
DECLARE 
  CURSOR curs1 IS SELECT ename, sal FROM emp WHERE deptno = 10 FOR UPDATE;
  rec curs1%ROWTYPE;
BEGIN
  OPEN curs1;
  LOOP
    FETCH curs1 INTO rec;
    EXIT WHEN curs1%NOTFOUND;
    UPDATE emp SET sal = sal + length(rec.ename) WHERE CURRENT OF curs1;
    dbms_output.put_line(rec.ename||' - '||to_char(rec.sal));
  END LOOP;
  CLOSE curs1;
END;
/
```

## Cursor Variables (REF CURSOR)
A cursor variable is a reference to a cursor. It allows you to pass cursors as parameters to subprograms and to open a cursor for a query that is determined at runtime.

- **Strongly typed**: `TYPE empcurtyp IS REF CURSOR RETURN emp%ROWTYPE;`
- **Weakly typed**: `TYPE genericcurtyp IS REF CURSOR;`

```sql
DECLARE 
  TYPE t_cur IS REF CURSOR;
  v_cur t_cur;
  
  PROCEDURE cursor_open(p_cur IN OUT t_cur) IS
  BEGIN
    OPEN p_cur FOR SELECT ename FROM emp WHERE sal > 3000;
  END;
  
  FUNCTION read_cursor(p_cur IN t_cur) RETURN varchar2 IS
    v emp.ename%TYPE;
  BEGIN
    FETCH p_cur INTO v;
    RETURN v;
  END;
BEGIN
  cursor_open(v_cur);
  dbms_output.put_line(read_cursor(v_cur));
  CLOSE v_cur;
END;
/
```
<pre>
Results:
-----------
BLAKE
</pre>

# Collections
A collection is an ordered group of elements of the same data type. PL/SQL offers three types of collections:

1.  **Associative Arrays (Index-by Tables)**: Unbounded sets of key-value pairs where the key can be an integer or a string. They can be sparse.
2.  **Nested Tables**: Unbounded, one-dimensional arrays that are initially dense but can become sparse. They can be stored in database columns.
3.  **Varrays (Variable-Size Arrays)**: Bounded, always-dense arrays. They can also be stored in database columns.

## Collection Methods
PL/SQL provides built-in functions and procedures to manage collections.

| Method  | Type      | Description                                          |
|---------|-----------|------------------------------------------------------|
| `DELETE`  | Procedure | Deletes one or more elements from a collection.      |
| `TRIM`    | Procedure | Deletes elements from the end of a varray or nested table. |
| `EXTEND`  | Procedure | Adds elements to the end of a varray or nested table. |
| `EXISTS`  | Function  | Returns `TRUE` if the specified element exists.      |
| `FIRST`   | Function  | Returns the first (smallest) index in a collection.  |
| `LAST`    | Function  | Returns the last (largest) index in a collection.    |
| `COUNT`   | Function  | Returns the number of elements in a collection.      |
| `LIMIT`   | Function  | Returns the maximum number of elements for a varray. |
| `PRIOR`   | Function  | Returns the index that precedes a specified index.   |
| `NEXT`    | Function  | Returns the index that succeeds a specified index.   |

### Example: Associative Array and Record

```sql
set serveroutput on
DECLARE
  TYPE rek_type IS RECORD(f1 INTEGER DEFAULT 10, f2 emp.ename%TYPE);
  rec rek_type;
  
  TYPE tab_type IS TABLE OF VARCHAR2(30) INDEX BY BINARY_INTEGER;
  tab tab_type;
  
  rec_dept dept%ROWTYPE;
  TYPE tab_type2 IS TABLE OF rec_dept%ROWTYPE INDEX BY BINARY_INTEGER;
  tab2 tab_type2;
BEGIN
  rec.f2 := 'KING';
  dbms_output.put_line(rec.f1||' -- '||rec.f2);
  
  tab(1) := 'Bubu'; tab(2) := 'Baba'; tab(3) := 'Bobo';
  FOR i IN tab.FIRST .. tab.LAST LOOP
    dbms_output.put_line(tab(i));
  END LOOP;
  
  SELECT * INTO rec_dept FROM dept WHERE deptno = 10;
  tab2(1) := rec_dept;
  dbms_output.put_line(tab2(1).deptno||' -- '||tab2(1).dname||' -- '||tab2(1).loc);
END;
/
```
<pre>
Results:
-----------
10 -- KING
Bubu
Baba
Bobo
10 -- ACCOUNTING -- NEW YORK
</pre>
