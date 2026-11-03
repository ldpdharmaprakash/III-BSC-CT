# Weeks 7–9 – Oracle Database Security and PL/SQL

**Course practical:** Online Library Management System  
**Database:** Oracle Database 21c XE / SQL*Plus  
**Purpose:** Teaching notes, practical SQL/PL/SQL statements, expected results, and exercises.

> **Important environment note:** Oracle 21c XE is a multitenant database. Perform classroom user, role, tablespace, and application-table activities in the `XEPDB1` pluggable database, not in `CDB$ROOT`. Connect as an authorized administrator and verify the container before making changes:
>
> ```sql
> SHOW USER;
> SHOW CON_NAME;
> ```
>
> If connected as SYSDBA in `CDB$ROOT`, switch using:
>
> ```sql
> ALTER SESSION SET CONTAINER = XEPDB1;
> ```
>
> Do not drop or modify Oracle-maintained users, tables, or tablespaces. Use a dedicated application schema for the library project.

---

# Week 7 – Managing Oracle Database Users and Security

## 1. Learning objectives

Students will learn to:

- Create and manage Oracle database users.
- Understand tablespaces and assign default and temporary tablespaces.
- Create and manage roles.
- Grant and revoke system and object privileges.
- Apply basic database security practices.

## 2. Database user

A database user is an account used to connect to Oracle and own database objects such as tables, views, sequences, and procedures. A user needs the `CREATE SESSION` system privilege to log in.

### 2.1 Create users

Run these commands as an administrator in `XEPDB1`:

```sql
CREATE USER LIB_ADMIN IDENTIFIED BY LibAdmin123 DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
CREATE USER LIB_STAFF IDENTIFIED BY LibStaff123 DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
CREATE USER LIB_VIEWER IDENTIFIED BY LibView123 DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
```

Expected result for each statement:

```text
User created.
```

If a user already exists, Oracle reports `ORA-01920`. Check first:

```sql
SELECT USERNAME FROM DBA_USERS WHERE USERNAME IN ('LIB_ADMIN','LIB_STAFF','LIB_VIEWER');
```

### 2.2 Grant login and system privileges

```sql
GRANT CREATE SESSION TO LIB_ADMIN;
GRANT CREATE SESSION TO LIB_STAFF;
GRANT CREATE SESSION TO LIB_VIEWER;
GRANT CREATE TABLE, CREATE VIEW, CREATE SEQUENCE, CREATE PROCEDURE, CREATE ROLE TO LIB_ADMIN;
```

`CREATE SESSION` permits login. The other privileges allow `LIB_ADMIN` to create the specified objects and roles in its own schema.

### 2.3 Connect as a user

Use SQL*Plus connection syntax. Replace the service name if your PDB service differs:

```sql
CONNECT LIB_ADMIN/LibAdmin123@localhost:1521/XEPDB1
SHOW USER;
SHOW CON_NAME;
```

Expected user and container:

```text
USER is "LIB_ADMIN"
XEPDB1
```

### 2.4 Alter a user

As an administrator:

```sql
ALTER USER LIB_STAFF IDENTIFIED BY StaffNew123;
ALTER USER LIB_VIEWER ACCOUNT LOCK;
ALTER USER LIB_VIEWER ACCOUNT UNLOCK;
```

Expected result: `User altered.`

### 2.5 Drop a user

Only drop a user when its account and objects are no longer required. `CASCADE` removes the user's schema objects as well.

```sql
DROP USER LIB_VIEWER;
```

If the user owns objects, Oracle may require:

```sql
DROP USER LIB_VIEWER CASCADE;
```

Do not execute this in a shared or production database without authorization and a backup.

## 3. Tablespaces

A tablespace is a logical storage unit in an Oracle database. It contains segments for database objects. A datafile provides the physical storage for a permanent tablespace.

### 3.1 View existing tablespaces

As a user with dictionary access:

```sql
SELECT TABLESPACE_NAME, STATUS, CONTENTS FROM DBA_TABLESPACES ORDER BY TABLESPACE_NAME;
```

### 3.2 Create a tablespace

The following example is for a typical Windows Oracle XE installation. Run as SYSDBA or an administrator with the required privilege, and confirm the Oracle datafile directory on your computer before using the path.

```sql
CREATE TABLESPACE LIBRARY_TS DATAFILE 'C:\app\oracle\oradata\XE\XEPDB1\library_ts01.dbf' SIZE 50M AUTOEXTEND ON NEXT 10M MAXSIZE 500M;
```

If that path does not exist on your computer, use the actual datafile directory from your Oracle installation. Do not copy a guessed path blindly.

### 3.3 Assign a tablespace to a user

```sql
ALTER USER LIB_ADMIN DEFAULT TABLESPACE LIBRARY_TS;
```

### 3.4 Verify tablespace details

```sql
SELECT TABLESPACE_NAME, FILE_NAME, BYTES/1024/1024 AS SIZE_MB FROM DBA_DATA_FILES ORDER BY TABLESPACE_NAME;
```

### 3.5 Drop a tablespace

Only use this for a tablespace created specifically for practice and after confirming that its contents are no longer needed:

```sql
DROP TABLESPACE LIBRARY_TS INCLUDING CONTENTS AND DATAFILES;
```

## 4. System privileges and object privileges

**System privileges** allow a user to perform database-level actions, such as creating a table or connecting to the database.

**Object privileges** allow a user to perform actions on a particular object, such as selecting from or inserting into a table.

### 4.1 System privilege examples

```sql
GRANT CREATE SESSION TO LIB_STAFF;
GRANT CREATE SESSION TO LIB_VIEWER;
GRANT CREATE TABLE TO LIB_ADMIN;
```

Revoke a system privilege:

```sql
REVOKE CREATE TABLE FROM LIB_ADMIN;
```

### 4.2 Prepare sample library objects as LIB_ADMIN

Connect as `LIB_ADMIN` and create these small teaching tables if they do not already exist. They are simplified security-practical tables and are not a replacement for the full Week 3 schema.

```sql
CREATE TABLE BOOK (BOOK_ID NUMBER PRIMARY KEY, TITLE VARCHAR2(100), AVAILABLE_COPIES NUMBER);
CREATE TABLE MEMBER (MEMBER_ID NUMBER PRIMARY KEY, MEMBER_NAME VARCHAR2(100), EMAIL VARCHAR2(100));
INSERT INTO BOOK VALUES (201,'Database Systems',5);
INSERT INTO BOOK VALUES (202,'Learning Oracle SQL',3);
INSERT INTO MEMBER VALUES (101,'Arun','arun@example.com');
INSERT INTO MEMBER VALUES (102,'Meena','meena@example.com');
COMMIT;
```

If these tables already exist, do not repeat the `CREATE TABLE` statements. Use `DESC BOOK;` and `SELECT * FROM BOOK;` to inspect them.

### 4.3 Grant object privileges

Run as `LIB_ADMIN`, the owner of the tables:

```sql
GRANT SELECT ON BOOK TO LIB_STAFF;
GRANT SELECT, INSERT, UPDATE ON BOOK TO LIB_STAFF;
GRANT SELECT ON MEMBER TO LIB_VIEWER;
```

### 4.4 Revoke object privileges

Run as the object owner or an appropriately authorized administrator:

```sql
REVOKE UPDATE ON BOOK FROM LIB_STAFF;
REVOKE INSERT ON BOOK FROM LIB_STAFF;
```

## 5. Roles

A role is a named collection of privileges. Roles simplify privilege management when several users need similar access.

### 5.1 Create roles

Run as `LIB_ADMIN` after it has received `CREATE ROLE`:

```sql
CREATE ROLE LIB_STAFF_ROLE;
CREATE ROLE LIB_VIEWER_ROLE;
```

### 5.2 Grant privileges to roles

Run as `LIB_ADMIN`, the owner of the tables:

```sql
GRANT SELECT, INSERT, UPDATE ON BOOK TO LIB_STAFF_ROLE;
GRANT SELECT ON MEMBER TO LIB_VIEWER_ROLE;
```

### 5.3 Grant roles to users

Run as an administrator with authority to grant these roles:

```sql
GRANT LIB_STAFF_ROLE TO LIB_STAFF;
GRANT LIB_VIEWER_ROLE TO LIB_VIEWER;
```

### 5.4 Revoke a role

```sql
REVOKE LIB_STAFF_ROLE FROM LIB_STAFF;
```

### 5.5 View roles and privileges

```sql
SELECT ROLE FROM DBA_ROLES ORDER BY ROLE;
SELECT GRANTEE, GRANTED_ROLE FROM DBA_ROLE_PRIVS WHERE GRANTEE LIKE 'LIB_%' ORDER BY GRANTEE;
SELECT GRANTEE, PRIVILEGE FROM DBA_SYS_PRIVS WHERE GRANTEE LIKE 'LIB_%' ORDER BY GRANTEE, PRIVILEGE;
SELECT GRANTEE, OWNER, TABLE_NAME, PRIVILEGE FROM DBA_TAB_PRIVS WHERE GRANTEE LIKE 'LIB_%' ORDER BY GRANTEE, TABLE_NAME;
```

## 6. Test the permissions

Connect as `LIB_STAFF`:

```sql
CONNECT LIB_STAFF/LibStaff123@localhost:1521/XEPDB1
SELECT * FROM LIB_ADMIN.BOOK;
```

If `SELECT` has been granted, the book rows should be displayed. If the role or object privilege has been revoked, Oracle may return `ORA-00942` or a privilege-related error depending on the statement and object visibility.

Connect as `LIB_VIEWER`:

```sql
CONNECT LIB_VIEWER/LibView123@localhost:1521/XEPDB1
SELECT * FROM LIB_ADMIN.MEMBER;
```

A viewer with only `SELECT` permission should be able to read the member table but should not be able to insert or update its rows.

## 7. Database security practices

- Give each person or application a separate database account.
- Grant only the privileges required for the user's work.
- Prefer roles for managing groups of privileges.
- Do not share administrator passwords.
- Use strong passwords and change default or classroom passwords before real deployment.
- Lock accounts that should not be used.
- Avoid granting broad privileges such as `DBA` to ordinary users.
- Back up important data before destructive operations.

## 8. Week 7 practical exercises

1. Create three users named `LIB_ADMIN`, `LIB_STAFF`, and `LIB_VIEWER`.
2. Grant `CREATE SESSION` to all three users.
3. Grant table and view creation privileges to `LIB_ADMIN`.
4. Create a tablespace named `LIBRARY_TS` using a valid local datafile path.
5. Create a role named `LIB_READ_ROLE` and grant `SELECT` on `LIB_ADMIN.BOOK` to it.
6. Grant the role to `LIB_VIEWER` and test the permission.
7. Revoke the role and test the permission again.
8. Lock and unlock `LIB_VIEWER`.

---

# Week 8 – Developing Basic PL/SQL Programs

## 1. Learning objectives

Students will develop PL/SQL programs using:

- PL/SQL block structure.
- Variables, constants, and data types.
- Assignment statements and arithmetic operators.
- Comments and embedded SQL statements.
- Conditional statements, loops, and nested blocks.

## 2. Introduction to PL/SQL

PL/SQL (Procedural Language/SQL) is Oracle's procedural extension to SQL. SQL retrieves and manipulates data; PL/SQL adds programming features such as variables, conditions, loops, and exception handling.

A PL/SQL block has three main sections:

```sql
DECLARE
    -- Optional declaration section
BEGIN
    -- Mandatory executable section
EXCEPTION
    -- Optional exception-handling section
END;
/
```

- `DECLARE`: declares variables, constants, cursors, and other local items.
- `BEGIN`: contains executable statements.
- `EXCEPTION`: handles runtime errors.
- `/`: runs the block in SQL*Plus; it is not part of the PL/SQL block itself.

Before displaying output, enable:

```sql
SET SERVEROUTPUT ON;
```

## 3. First PL/SQL program

```sql
SET SERVEROUTPUT ON;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Welcome to PL/SQL');
END;
/
```

Expected output:

```text
Welcome to PL/SQL
PL/SQL procedure successfully completed.
```

## 4. Variables and data types

A variable stores a value that can change while a PL/SQL block executes. Common types include `NUMBER`, `VARCHAR2`, `DATE`, and `BOOLEAN`.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_NAME VARCHAR2(30) := 'Arun';
    V_MARK NUMBER := 85;
    V_RESULT VARCHAR2(10);
BEGIN
    V_RESULT := 'PASS';
    DBMS_OUTPUT.PUT_LINE('Name: ' || V_NAME);
    DBMS_OUTPUT.PUT_LINE('Mark: ' || V_MARK);
    DBMS_OUTPUT.PUT_LINE('Result: ' || V_RESULT);
END;
/
```

Expected output:

```text
Name: Arun
Mark: 85
Result: PASS
```

## 5. Constants

A constant is declared using `CONSTANT`. Its value cannot be changed after initialization.

```sql
SET SERVEROUTPUT ON;
DECLARE
    C_FINE CONSTANT NUMBER := 5;
    V_DAYS NUMBER := 4;
    V_TOTAL NUMBER;
BEGIN
    V_TOTAL := C_FINE * V_DAYS;
    DBMS_OUTPUT.PUT_LINE('Fine Amount: ' || V_TOTAL);
END;
/
```

Expected output:

```text
Fine Amount: 20
```

## 6. Assignment statements and arithmetic operators

PL/SQL uses `:=` for assignment. Common arithmetic operators are `+`, `-`, `*`, `/`, and `**` (exponentiation).

```sql
SET SERVEROUTPUT ON;
DECLARE
    A NUMBER := 20;
    B NUMBER := 5;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Addition: ' || (A+B));
    DBMS_OUTPUT.PUT_LINE('Subtraction: ' || (A-B));
    DBMS_OUTPUT.PUT_LINE('Multiplication: ' || (A*B));
    DBMS_OUTPUT.PUT_LINE('Division: ' || (A/B));
    DBMS_OUTPUT.PUT_LINE('Power: ' || (A**2));
END;
/
```

Expected output:

```text
Addition: 25
Subtraction: 15
Multiplication: 100
Division: 4
Power: 400
```

## 7. Comments

Single-line comments begin with `--`. Multi-line comments are enclosed by `/*` and `*/`.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_COUNT NUMBER := 10; -- Stores the initial count
BEGIN
    /* Display the count value */
    DBMS_OUTPUT.PUT_LINE('Count: ' || V_COUNT);
END;
/
```

## 8. Embedded SQL statements

PL/SQL can execute SQL statements such as `SELECT`, `INSERT`, `UPDATE`, and `DELETE`. A `SELECT` statement that returns one row uses `INTO` to store the result.

### 8.1 SELECT INTO

Run this example in the schema that owns the Week 3 `BOOK` table:

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_TITLE BOOK.TITLE%TYPE;
    V_PRICE BOOK.PRICE%TYPE;
BEGIN
    SELECT TITLE, PRICE INTO V_TITLE, V_PRICE
    FROM BOOK
    WHERE BOOK_ID = 201;
    DBMS_OUTPUT.PUT_LINE('Book Title: ' || V_TITLE);
    DBMS_OUTPUT.PUT_LINE('Book Price: ' || V_PRICE);
END;
/
```

Expected output depends on the row stored with `BOOK_ID = 201`. If no such row exists, the block raises `NO_DATA_FOUND`; if more than one row matches, it raises `TOO_MANY_ROWS`.

### 8.2 INSERT using PL/SQL

Use this only if the specified key does not already exist and the columns match the actual `MEMBER` table:

```sql
BEGIN
    INSERT INTO MEMBER (MEMBER_ID, MEMBER_NAME, EMAIL, PHONE, JOIN_DATE, MEMBER_TYPE)
    VALUES (106, 'Kavin', 'kavin@example.com', '9876543210', SYSDATE, 'STUDENT');
    COMMIT;
END;
/
```

Expected output:

```text
PL/SQL procedure successfully completed.
```

### 8.3 UPDATE using PL/SQL

```sql
BEGIN
    UPDATE BOOK
    SET AVAILABLE_COPIES = AVAILABLE_COPIES + 1
    WHERE BOOK_ID = 201;
    COMMIT;
END;
/
```

### 8.4 DELETE using PL/SQL

```sql
BEGIN
    DELETE FROM MEMBER WHERE MEMBER_ID = 106;
    COMMIT;
END;
/
```

## 9. Conditional statements

### 9.1 IF statement

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_MARK NUMBER := 75;
BEGIN
    IF V_MARK >= 50 THEN
        DBMS_OUTPUT.PUT_LINE('PASS');
    ELSE
        DBMS_OUTPUT.PUT_LINE('FAIL');
    END IF;
END;
/
```

Expected output: `PASS`.

### 9.2 IF–ELSIF–ELSE statement

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_MARK NUMBER := 82;
BEGIN
    IF V_MARK >= 90 THEN
        DBMS_OUTPUT.PUT_LINE('Grade A+');
    ELSIF V_MARK >= 80 THEN
        DBMS_OUTPUT.PUT_LINE('Grade A');
    ELSIF V_MARK >= 70 THEN
        DBMS_OUTPUT.PUT_LINE('Grade B');
    ELSIF V_MARK >= 50 THEN
        DBMS_OUTPUT.PUT_LINE('Grade C');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Fail');
    END IF;
END;
/
```

Expected output: `Grade A`.

## 10. Nested blocks

A nested block is a PL/SQL block placed inside another block. The inner block can have its own declaration and exception sections.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_OUTER NUMBER := 100;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Outer value: ' || V_OUTER);
    DECLARE
        V_INNER NUMBER := 50;
    BEGIN
        DBMS_OUTPUT.PUT_LINE('Inner value: ' || V_INNER);
    END;
END;
/
```

Expected output:

```text
Outer value: 100
Inner value: 50
```

## 11. Week 8 practical exercises

1. Write a PL/SQL block to display your name, department, and college.
2. Declare two numbers and display their sum, difference, product, and quotient.
3. Declare a constant fine rate and calculate the fine for a given number of overdue days.
4. Retrieve a book title and price using `SELECT INTO`.
5. Insert one member record using an embedded SQL statement.
6. Display PASS or FAIL using an `IF` statement.
7. Display a grade using `IF–ELSIF–ELSE`.
8. Write a nested block that displays separate outer and inner variables.

---

# Week 9 – Implementing PL/SQL Control Structures and Exception Handling

## 1. Learning objectives

Students will develop PL/SQL programs using:

- IF and CASE statements.
- Basic LOOP, WHILE LOOP, and FOR LOOP constructs.
- Exception handling techniques.
- Transaction control statements such as `COMMIT`, `ROLLBACK`, and `SAVEPOINT`.

## 2. CASE statement

A CASE statement selects one execution path based on a condition or expression.

### 2.1 Simple CASE

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_CHOICE NUMBER := 2;
BEGIN
    CASE V_CHOICE
        WHEN 1 THEN DBMS_OUTPUT.PUT_LINE('Add Book');
        WHEN 2 THEN DBMS_OUTPUT.PUT_LINE('Issue Book');
        WHEN 3 THEN DBMS_OUTPUT.PUT_LINE('Return Book');
        ELSE DBMS_OUTPUT.PUT_LINE('Invalid Choice');
    END CASE;
END;
/
```

Expected output: `Issue Book`.

### 2.2 Searched CASE

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_MARK NUMBER := 76;
BEGIN
    CASE
        WHEN V_MARK >= 90 THEN DBMS_OUTPUT.PUT_LINE('Grade A+');
        WHEN V_MARK >= 80 THEN DBMS_OUTPUT.PUT_LINE('Grade A');
        WHEN V_MARK >= 70 THEN DBMS_OUTPUT.PUT_LINE('Grade B');
        WHEN V_MARK >= 50 THEN DBMS_OUTPUT.PUT_LINE('Grade C');
        ELSE DBMS_OUTPUT.PUT_LINE('Fail');
    END CASE;
END;
/
```

Expected output: `Grade B`.

## 3. LOOP statement

A basic LOOP repeats statements until an `EXIT` condition is met.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_NUM NUMBER := 1;
BEGIN
    LOOP
        DBMS_OUTPUT.PUT_LINE('Number: ' || V_NUM);
        V_NUM := V_NUM + 1;
        EXIT WHEN V_NUM > 5;
    END LOOP;
END;
/
```

Expected output:

```text
Number: 1
Number: 2
Number: 3
Number: 4
Number: 5
```

## 4. WHILE LOOP

A WHILE LOOP repeats while its condition is true. The condition is checked before each iteration.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_NUM NUMBER := 1;
BEGIN
    WHILE V_NUM <= 5 LOOP
        DBMS_OUTPUT.PUT_LINE('Number: ' || V_NUM);
        V_NUM := V_NUM + 1;
    END LOOP;
END;
/
```

Expected output: numbers 1 through 5, each on a separate line.

## 5. FOR LOOP

A FOR LOOP repeats for a specified integer range. The loop index is declared automatically and is read-only inside the loop.

```sql
SET SERVEROUTPUT ON;
BEGIN
    FOR V_NUM IN 1..5 LOOP
        DBMS_OUTPUT.PUT_LINE('Number: ' || V_NUM);
    END LOOP;
END;
/
```

Expected output: numbers 1 through 5, each on a separate line.

### 5.1 Reverse FOR LOOP

```sql
SET SERVEROUTPUT ON;
BEGIN
    FOR V_NUM IN REVERSE 1..5 LOOP
        DBMS_OUTPUT.PUT_LINE('Number: ' || V_NUM);
    END LOOP;
END;
/
```

Expected output: numbers 5 through 1.

## 6. Nested loops

```sql
SET SERVEROUTPUT ON;
BEGIN
    FOR I IN 1..3 LOOP
        FOR J IN 1..2 LOOP
            DBMS_OUTPUT.PUT_LINE('I = ' || I || ', J = ' || J);
        END LOOP;
    END LOOP;
END;
/
```

Expected output:

```text
I = 1, J = 1
I = 1, J = 2
I = 2, J = 1
I = 2, J = 2
I = 3, J = 1
I = 3, J = 2
```

## 7. Exception handling

An exception is a runtime error or an exceptional condition that changes the normal flow of a PL/SQL program. The `EXCEPTION` section handles such conditions.

General structure:

```sql
BEGIN
    -- Executable statements
EXCEPTION
    WHEN exception_name THEN
        -- Handling statements
    WHEN OTHERS THEN
        -- General error handling
END;
/
```

### 7.1 ZERO_DIVIDE

```sql
SET SERVEROUTPUT ON;
DECLARE
    A NUMBER := 10;
    B NUMBER := 0;
    V_RESULT NUMBER;
BEGIN
    V_RESULT := A / B;
    DBMS_OUTPUT.PUT_LINE('Result: ' || V_RESULT);
EXCEPTION
    WHEN ZERO_DIVIDE THEN
        DBMS_OUTPUT.PUT_LINE('Error: Division by zero is not allowed.');
END;
/
```

Expected output:

```text
Error: Division by zero is not allowed.
```

### 7.2 NO_DATA_FOUND

This example assumes the `BOOK` table has a `BOOK_ID` column and no row with ID `99999`.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_TITLE BOOK.TITLE%TYPE;
BEGIN
    SELECT TITLE INTO V_TITLE
    FROM BOOK
    WHERE BOOK_ID = 99999;
    DBMS_OUTPUT.PUT_LINE('Book: ' || V_TITLE);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No book found for the given ID.');
END;
/
```

Expected output if no matching row exists: `No book found for the given ID.`

### 7.3 TOO_MANY_ROWS

This example deliberately selects all book titles into a scalar variable. It raises `TOO_MANY_ROWS` if more than one row exists.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_TITLE BOOK.TITLE%TYPE;
BEGIN
    SELECT TITLE INTO V_TITLE FROM BOOK;
    DBMS_OUTPUT.PUT_LINE('Book: ' || V_TITLE);
EXCEPTION
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('More than one book was found.');
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No books are available.');
END;
/
```

### 7.4 OTHERS exception

`WHEN OTHERS` handles exceptions not handled by earlier handlers. `SQLCODE` returns the error number and `SQLERRM` returns the error message.

```sql
SET SERVEROUTPUT ON;
BEGIN
    INSERT INTO BOOK (BOOK_ID, TITLE, CATEGORY_ID, AUTHOR_ID, PUBLISHER_ID, PRICE, TOTAL_COPIES, AVAILABLE_COPIES)
    VALUES (201, 'Duplicate Book', 1, 1, 1, 300, 5, 5);
EXCEPTION
    WHEN DUP_VAL_ON_INDEX THEN
        DBMS_OUTPUT.PUT_LINE('Error: Duplicate book ID or unique value.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error code: ' || SQLCODE);
        DBMS_OUTPUT.PUT_LINE('Error message: ' || SQLERRM);
END;
/
```

This example assumes the full Week 3 `BOOK` table and that `BOOK_ID = 201` already exists. If the key is not present, the insert may succeed; if other referenced IDs do not exist, a foreign-key exception may occur. Roll back after a successful test insert if it should not be retained.

## 8. User-defined exceptions

A user-defined exception is declared by the programmer and raised when a specific business rule is violated.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_COPIES NUMBER := 0;
    E_NO_COPIES EXCEPTION;
BEGIN
    IF V_COPIES <= 0 THEN
        RAISE E_NO_COPIES;
    END IF;
    DBMS_OUTPUT.PUT_LINE('Book can be issued.');
EXCEPTION
    WHEN E_NO_COPIES THEN
        DBMS_OUTPUT.PUT_LINE('Cannot issue the book: no copies available.');
END;
/
```

Expected output: `Cannot issue the book: no copies available.`

## 9. Transaction control statements

Transaction control statements manage changes made by DML statements.

- `COMMIT`: permanently saves the current transaction.
- `ROLLBACK`: cancels uncommitted changes in the current transaction.
- `SAVEPOINT`: marks a point within a transaction to which a partial rollback can return.

### 9.1 COMMIT

```sql
INSERT INTO MEMBER (MEMBER_ID, MEMBER_NAME, EMAIL, PHONE, JOIN_DATE, MEMBER_TYPE)
VALUES (107, 'Nila', 'nila@example.com', '9876500000', SYSDATE, 'STUDENT');
COMMIT;
```

### 9.2 ROLLBACK

```sql
UPDATE BOOK SET AVAILABLE_COPIES = AVAILABLE_COPIES - 1 WHERE BOOK_ID = 201;
ROLLBACK;
```

The uncommitted update is cancelled.

### 9.3 SAVEPOINT and partial ROLLBACK

```sql
UPDATE BOOK SET AVAILABLE_COPIES = AVAILABLE_COPIES - 1 WHERE BOOK_ID = 201;
SAVEPOINT AFTER_BOOK_UPDATE;
UPDATE BOOK SET AVAILABLE_COPIES = AVAILABLE_COPIES - 1 WHERE BOOK_ID = 202;
ROLLBACK TO AFTER_BOOK_UPDATE;
COMMIT;
```

The update to book 202 is undone; the earlier update to book 201 is committed. Check the rows before and after execution to observe the result. Use this example only with known existing book IDs and suitable available-copy values.

## 10. Week 9 practical exercises

1. Write a PL/SQL program using a simple CASE statement to display a library menu option.
2. Write a searched CASE program to display a grade based on marks.
3. Display numbers from 1 to 10 using a basic LOOP.
4. Display even numbers from 2 to 20 using a WHILE LOOP.
5. Display the multiplication table of a number using a FOR LOOP.
6. Use nested FOR loops to print a 3-by-3 number pattern.
7. Handle division by zero using `ZERO_DIVIDE`.
8. Handle a missing book using `NO_DATA_FOUND`.
9. Write a user-defined exception for an unavailable book.
10. Demonstrate `COMMIT`, `ROLLBACK`, and `SAVEPOINT` using a test record.

---

# Common SQL*Plus commands for Weeks 7–9

| Command | Purpose |
|---|---|
| `SHOW USER;` | Displays the connected user |
| `SHOW CON_NAME;` | Displays the current container |
| `DESC BOOK;` | Displays the structure of BOOK |
| `SELECT * FROM BOOK;` | Displays BOOK records |
| `SET SERVEROUTPUT ON;` | Displays output from `DBMS_OUTPUT` |
| `/` | Executes the PL/SQL block in the SQL*Plus buffer |
| `COMMIT;` | Saves the current transaction |
| `ROLLBACK;` | Cancels uncommitted changes |
| `HOST CLS` | Clears the Windows SQL*Plus screen |

# Important error messages

| Error | Common reason | What to check |
|---|---|---|
| `ORA-65096` | Attempt to create a local-style user in `CDB$ROOT` | Switch to `XEPDB1` before creating application users |
| `ORA-65066` | A common-user change requires all containers | Run the intended common-user change from the root with `CONTAINER=ALL` |
| `ORA-01920` | User or role name already exists | Check `DBA_USERS` and `DBA_ROLES` |
| `ORA-01917` | Referenced user or role does not exist | Verify that user/role creation succeeded |
| `ORA-01017` | Invalid username/password | Check credentials and connection service |
| `ORA-00942` | Table or view does not exist or is not accessible | Check schema, object name, and privileges |
| `ORA-01403` / `NO_DATA_FOUND` | `SELECT INTO` returned no rows | Check the search condition or handle the exception |
| `ORA-01422` / `TOO_MANY_ROWS` | `SELECT INTO` returned multiple rows | Make the condition unique or use a cursor/collection |

# Final practical checklist

- [ ] Verify the current user and container before database administration tasks.
- [ ] Create users only in the intended PDB.
- [ ] Assign appropriate tablespaces and privileges.
- [ ] Test roles and privileges by connecting as the intended users.
- [ ] Enable `SERVEROUTPUT` before running output-based PL/SQL examples.
- [ ] Execute each PL/SQL block using `/` on a new SQL*Plus line.
- [ ] Test exception handlers with controlled examples.
- [ ] Use `COMMIT` and `ROLLBACK` carefully and understand which changes are saved.
