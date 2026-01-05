# Cursor

Every SQL statement executed by the Oracle Server has an individual cursor associated with it:

- Implicit cursors: Declared for all DML and PL/SQL SELECT statements
- Explicit cursors: Declared and named by the programmer

Controlling Explicit Cursors:

   - DECLARE: Create a named SQL area
   - OPEN: Identify the active set
   - FETCH: Load the current row into variables
   - EMPTY?: Test for existing rows, return to FETCH if rows found
   - CLOSE: Release the active set

```sql
-- examples for cursors (implicit and explicit cursor)

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

Controlling Explicit Cursors:

   - Open the cursor.
   - Fetch a row from the cursor.
   - Continue until empty.
   - Close the cursor.

Opening the Cursor:

   - Open the cursor to execute the query and identify the active set.
   - If the query returns no rows, no exception is raised.
   - Use cursor attributes to test the outcome after a fetch.

Fetching Data from the Cursor:

   - Retrieve the current row values into output variables.
   - Include the same number of variables.
   - Match each variable to correspond to the columns positionally.
   - Test to see if the cursor contains rows.

Declaring the Cursor:

   - Do not include the INTO clause in the cursor declaration.
   - If processing rows in a specific sequence is required, use the ORDER BY clause in the query.

Closing the Cursor:

   - Close the cursor after completing the processing of the rows.
   - Reopen the cursor, if required.
   - Do not attempt to fetch data from a cursor once it has been closed.

Explicit Cursor Attributes:

   - Obtain status information about a cursor.
   - %ISOPEN (Boolean): Evaluates to TRUE if the cursor is open
   - %NOTFOUND (Boolean): Evaluates to TRUE if the most recent fetch does not return a row
   - %FOUND (Boolean): Evaluates to TRUE if the most recent fetch returns a row; complement of %NOTFOUND
   - %ROWCOUNT (Number): Evaluates to the total number of rows returned so far

```sql
-- Implicit cursor (for a DELETE statement)
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

Cursors and Records:

   - Process the rows of the active set conveniently by fetching values into a PL/SQL RECORD.

Cursor FOR Loop:

   - The cursor FOR loop is a shortcut to process explicit cursors.
   - Implicit open, fetch, and close occur.
   - The record is implicitly declared.

Cursor Parameters:

   - Pass parameter values to a cursor when the cursor is opened and the query is executed.
   - Open an explicit cursor several times with a different active set each time.

```sql
-- Explicit cursors, 3 forms of cursor usage and cursors with parameter
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

```sql
-- Using a CURSOR for a WITH statement
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

<pre>
Results:
-----------
ACCOUNTING | 2917 | 751
RESEARCH | 2175 | 9
SALES | 1800 | -366
MARKETING | 2133 | -33
</pre>

Explicit Locking:

   - Explicit locking lets you deny access for the duration of a transaction.
   - Lock the rows before the update or delete.

Cursor Updates and Deletes:

   - Use cursors to update or delete the current row.
   - Include the FOR UPDATE clause in the cursor query to lock the rows first.
   - Use the WHERE CURRENT OF clause to reference the current row from an explicit cursor.

```sql
-- Update with a cursor  -> WHERE CURRENT OF
-- FOR UPDATE locks rows in the table
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

```sql
SELECT ename, sal FROM emp WHERE deptno=10;  -- you can see the updated values
ROLLBACK;
SELECT ename, sal FROM emp WHERE deptno=10;  -- old values are restored
```

```sql
-- What happens when we update the table during cursor loop?
-- answer: cursor doesn't see the new values
-- result set will be fixed when opening the cursor
set serveroutput on
DECLARE
 v_date    date := SYSDATE + 1;
BEGIN
  FOR rec IN (SELECT * FROM dept for update) LOOP
    dbms_output.put(to_char(sysdate, 'hh24:mi:ss')||' --> ');
    dbms_output.put_line(rec.deptno);
    SELECT SYSDATE + 2/(24*60*60) INTO v_date FROM dual; -- 2 seconds
    WHILE sysdate < v_date LOOP  NULL;   END LOOP;   
    update dept set deptno = deptno+1;
  END LOOP;
END;
/
```

```sql
-- Cursor update for a join
-- LOC column will be updated in each step of iteration
-- Result set of the cursor:
-- TURNER  SALES
-- MARTIN  SALES
-- WARD    SALES
-- ALLEN   SALES

DECLARE
  CURSOR c1 IS  SELECT ename, dname  FROM emp, dept
    WHERE emp.deptno = dept.deptno AND job = 'SALESMAN' FOR UPDATE OF loc;
BEGIN
  FOR rec IN c1 LOOP
   -- UPDATE emp SET sal = sal + 1 WHERE CURRENT OF c1;
    UPDATE dept SET loc = loc|| '1' WHERE CURRENT OF c1;
  END LOOP;
END;
/
```

```sql
-- cursor variable
-- can be strongly typed (with return type) or weakly typed (without return type)
DECLARE 
  TYPE empcurtyp IS REF CURSOR RETURN emp%ROWTYPE;  -- strong
  TYPE genericcurtyp IS REF CURSOR;                 -- weak
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

# Collection

- A collection is an ordered group of elements having the same data type.
- Each element is identified by a unique subscript that represents its position in the collection.
- Associative array (or index-by table): Unbounded, String or Integer subscript, Either dense or sparse, Only in PL/SQL block, No Object Type Attribute
- Nested table: Unbounded, Integer subscript, Starts dense, can become sparse, Either in PL/SQL block or at schema level, Yes Object Type Attribute
- Variablesize array (Varray): Bounded, Integer subscript, Always dense, Either in PL/SQL block or at schema level, Yes Object Type Attribute

PL/SQL has three collection types: 

1. associative array (INDEX BY ...; we call it also PL/SQL table)
2. VARRAY (variable-size array), 
3. NESTED TABLE

# Collection methods

A collection method is a PL/SQL subprogram: either a function that returns information about a collection, or a procedure that operates on a collection.

| Method  | Type      | Description                                                                 |
|---------|-----------|-----------------------------------------------------------------------------|
| DELETE  | Procedure | Deletes elements from collection.                                           |
| TRIM    | Procedure | Deletes elements from end of varray or nested table.                        |
| EXTEND  | Procedure | Adds elements to end of varray or nested table.                             |
| EXISTS  | Function  | Returns TRUE if and only if specified element of varray or nested table exists. |
| FIRST   | Function  | Returns first index in a collection.                                         |
| LAST    | Function  | Returns last index in a collection.                                          |
| COUNT   | Function  | Returns number of elements in a collection.                                  |
| LIMIT   | Function  | Returns maximum number of elements that a collection can have.              |
| PRIOR   | Function  | Returns the index that precedes specified index.                             |
| NEXT    | Function  | Returns the index that succeeds specified index.                             |

```sql
/****************** record and associative array ****************/

set serveroutput on
DECLARE
  TYPE rek_type IS RECORD(f1 INTEGER DEFAULT 10, f2 emp.ename%TYPE);     -- type declaration
  rec rek_type;                                                          -- variable definition
  TYPE tab_type IS TABLE OF VARCHAR2(30) INDEX BY BINARY_INTEGER;        -- associative array type
  tab tab_type;                                                          -- variable of type assoc. array
  rec_dept dept%ROWTYPE;                                                 -- record variable
  TYPE tab_type2 IS TABLE OF rec_dept%ROWTYPE INDEX BY BINARY_INTEGER;   -- array of records
  tab2 tab_type2;
BEGIN
  rec.f2 := 'KING';
  dbms_output.put_line(rec.f1||' -- '||rec.f2);                          -- default value
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

Index-By Table:

   - An index-by table (also called an associative array) is a set of key-value pairs. Each key is unique and is used to locate the corresponding value.
   - The key can be either an integer or a string.
   - An index-by table is created using the following syntax. Here, we are creating an index-by table named salary_list, the keys of which will be of the subscript_type and associated values will be of the element_type.

Nested Tables:

   - A nested table is like a one-dimensional array with an arbitrary number of elements. However, a nested table differs from an array in the following aspects—
   - An array has a declared number of elements, but a nested table does not. The size of a nested table can increase dynamically.
   - An array is always dense, i.e., it always has consecutive subscripts. A nested array is dense initially, but it can become sparse when elements are deleted from it.